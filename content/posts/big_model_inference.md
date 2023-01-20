---
title: "A Brief How-To on Local Inference for extremely Large Language Models"
date: 2023-01-22T12:39:31+01:00
draft: true
---

If you are here for the TL;DR: Simply add `device_map="auto"` for any model loaded with `transformers >= 4.20.0`, which will distribute model weights across all GPUs and main memory.
```python
model = AutoModel.from_pretrained("model-name", device_map="auto")
```

Recently, I have become increasingly interested in experimenting with zero- and few-shot prompting on extremely large language models. Without any clear literature to back me up on this, I would classify a model as "extremely large" once it becomes infeasible to load the full architecture into a single 16GB VRAM GPU (without any overhead for the actual sequences to work on). As a rule of thumb, this puts the maximum count of full precision (fp32) parameters at around 4 billion (see the chapter [Estimating the Model Memory Footprint] for a more extensive explanation).
In this post, we'll primarily deal with the question on how to deploy larger models on one's own compute infrastructure and not on potentially costly cloud deployments.


## Why this Post?

By now, there exists a multitude of xLLMs [citations](), with about one new model popping up every 2-3 weeks (subjective impression). About 25-50% of these additionally release public checkpoints (often at reduced sizes), which theoretically allows the community to experiment with them. We also see several providers that have set out to fill an emergent niche of "MaaS" (models as a service) platforms that capitalize on the difficulty of getting these models to run and maintain.
Note that we are not even talking about "efficiently" deploying such models, but with ever-increasing parameter counts, the literal deployment itself becomes very challenging.
Despite some [great tutorials]() on the topic of inference deployments (primarily for "the Cloud" (tm)), it is concerning that one finds very little information on how to actually run such models on existing local compute.
This is especially critical for academics, where oftentimes some compute infrastructure is available (albeit on limited hardware budgets), but running costly [Cloud inference modules]() or [racking up bills through API calls]() is simply not feasible. Instead, this article deals with bare-metal deployments of "medium-sized" extremely large models (up to 30B parameters).

Also, since starting this experimentation, [Tim Dettmers]() has stirred some attention with [his work on **8** bit model weights](), at no loss of quality compared to 16 bit weights (footnote: It should be mentioned that this compares to `bfloat16`, more on this later). With reduced-size model weights becoming more available, it will become essential for reasarchers and practitioners alike to figure out ways of incorporating this into their workflows.

What this is NOT: I am merely exploring deployment options for *inferencing* on such models; actively training (fine-tuning) such architectures is not something I am (currently) interested in, especially since the whole point of prompting lies in the ability to perform arbitrary tasks without much (or any) existing training data. If you are interested in such scenarios, I can still recommend you to read on, and explore some more of the linked documents or videos that I will reference throughout this post.

## Requirements and Hardware

I myself have performed all experiments on our group's local compute server that is specced with two Nvidia TITAN RTX (24 GB each), an Intel Xeon Silver 4210, as well as 64GB of main memory. All of the experiments were run with CUDA version 11.2. This is in my experience a realistic compute budget for any mid-sized university group related to Computer Science. Especially ML-focused groups and university-wide compute clusters frequently have way more compute power available, so your mileage might vary ([by a lot!]()). But I also consider this a suitable reference post for estimating what deployments are realistically possible on one's own compute budget.

Further, all of these experiments are tailored to *PyTorch* deployments with Huggingface's `transformers` library (version 1.10 and 4.21.0, respectively). I do not have any experience with JAX-based models, and my deployment experience in Tensorflow is also rather limited. From what I can tell, general memory estimates and model sizes should be relatively similar across platforms, unless there is some unknown overhead introduced by one platform or another (AFAIK, for inference this is usually not the case). However, I do not know whether distribution features are available through these other libraries to begin with.

Finally, I want to stress again that you should absolutely make sure that you have `transformers>=4.20.0`, as this will make a lot of the actual model loading almost trivial, as I've painfully found out during my experimentation.
Also, in [my experience](linstackoverflowtransformers) debugging diverse projects, versioning conflicts are the primary cause for errors when working with `transformers`.

## Choosing a Model
Primarily, our expected downstream task performance depends on what size of model we end up running with. A [recent collaboration](https://arxiv.org/pdf/2206.07682.pdf) between Google Research, Standford, UNC Chapel Hill and Deepmind links emergent behavior of successful few-shot performance to the increasing model size, which is a rather depressing insight from a competitive academic standpoint.

However, there are still many options left that we can experiment with, that give a reasonable approximation of what working with even larger models will be like (minus some of the emergent properties).
Some of the particular choices for my experimentation are listed here (names reflect the Huggingface hub reference):

- **[t5-3b](https://huggingface.co/t5-3b)**: To illustrate the size comparison with one already rather big model, we can use the T5 3B variant.
- **[EleutherAI/gpt-j-6B](https://huggingface.co/EleutherAI/gpt-j-6B)**: This is the first "large-scale" model that I had originally planned on loading, which caused issues when naively loading it onto a single GPU in my hardware stack.
- **[EleutherAI/gpt-neox-20b](https://huggingface.co/EleutherAI/gpt-neox-20b)**: 20B checkpoint of EleutherAI's GPT-NeoX project. This seriously requires a multi-GPU setup to inference on GPU-only (with some tricks, it *might* run on a single A100 or similar GPU with 40GB VRAM; however, this hardware is really top-of-the-line and might not be available to as many users).
- **[facebook/opt-30b](https://huggingface.co/facebook/opt-30b)[facebook/opt-66b](https://huggingface.co/facebook/opt-66b)**: Two variants of the OPT model with more extreme sizes.
- **[bigscience/bloom](https://huggingface.co/bigscience/bloom)**: The 176B parameter variant and BigScience's flagship model. This has a similar parameter count to GPT-3 (if not the same, I'm not perfectly familiar with architectural choices). Note that the download alone will block around 300GB on your hard disk.


## Estimating the Model Memory Footprint

Realistically, however, we are frequently more limited by the available compute infrastructure than available model weights. No matter how hard we try, once a majority of layers does no longer fit onto a single GPU, naive model loading will fail, generally indicating a Cuda memory allocation error.
To avoid problems like this, it makes sense to estimate the consumed memory by a particular model. For this, we only need to know two things:

1. The number of model parameters, and
2. The data type in which weights are stored.

Both of them, however, are not necessarily trivial to figure out. For model parameters, many give an indicator of the number of parameters through their name, especially for larger model checkpoints, e.g. "T5-3B" comes with roughly three billion parameters. Similarly, for the data type used for storing parameters, we can check the `config.json` file in the [Huggingface Model Hub](https://hf.co/models). Per default, it will use `float32`, however, we can sometimes find different specifications for the attribute `torch_dtype`, as can be seen in the config for [OPT-30B](https://huggingface.co/facebook/opt-30b/blob/main/config.json).

If we want to be more precise, we can always check this on a loaded model (even if it is just residing in main memory, which can be challenging at this stage of planning.
For the model parameters, the most straightforward way to determine the number of parameters is in my experience the following snippet(credit goes to: [this Pytorch forum thread](https://discuss.pytorch.org/t/how-do-i-check-the-number-of-parameters-of-a-model/4325/9)):

```python
from transformers import AutoModel
model = AutModel.from_pretrained("t5-3b")
num_params = sum(p.numel() for p in model.parameters() if p.requires_grad)
print(num_params)
```

For the above model "T5-3B", this actually gives us about 5% fewer parameters, meaning "only" around 2.85 billion parameters.
To gather the data type used for weights, we can also simply call
```python3
model.dtype()
```
which in the above case of "T5-3B" will return `float32`.
The final expected in-memory estimate is then simply
```
memory_estimate = num_params * (dtype_size / 8) / (1024**3)
```
This estimate already represents the final estimate in GB of memory, which comes out to slightly less than 11 GB of main memory.

## Straightforward Solution: `device_map`

While I originally spent quite some time digging through the [`accelerate` docs](), it turns out that with version `4.20.0` and upwards, distributed inference mode is natively supported through the `transformers` library (shout out to [this early forum post](https://discuss.huggingface.co/t/facebook-opt-30b-model-inferencing/18837) for clarifying!).
It makes the actual loading (almost) trivial, assuming you are currently loading your models in a similar fashion to the following:

```python
import torch
from transformers import AutoModelForCausalLM

device = "cuda" if torch.cuda.is_available() else "cpu"
model = model.from_pretrained("model-name", device = device)
```

You can simply specify an automated `device_map`, which will schedule a mostly reasonable distribution of a model across all available devices.
This starts with filling up available GPU memory (in my experience, this also works if some of the GPUs are already partially occupied with other workloads) and extends to CPU main memory once the GPUs can take no more layers.

Therefore, the only adaptation needed is changing the model loading to:
```python
model = model.from_pretrained("model-name", device_map="auto")
```


## Caveats and Limitations
One of the things that I am unclear about, and which can potentially have a detrimental effect on your prediction performance, is the actual software architecture behind the distributed deployments. I have put it on my TODO list to learn more about the actual inner workings of `accelerate`, but I have not yet had the time to investigate much more.
From my crude understanding, some of the more nuanced aspects might have been left out of this particular article.

Secondly, inference time! Once models spill over to disk, inference time becomes a distinct limitation. For BLOOM, I had to wait over 10 minutes to obtain a single prediction (with <20 tokens length), before it eventually crashed (I haven't really tried again since then). Even though the spilled content has to be read-only at inference time, the speed is still several orders of magnitude lower than a model completely loaded into main memory.
But even without any spillage, inference will take some time, and much longer than with smaller models that fit on a single (or multiple) GPU.

I eventually was planning to perform some further benchmarks in terms of inference speed, but the more recent developments (8 bit inference, free tier for co:here models) make it extremely difficult to obtain comparable results.

