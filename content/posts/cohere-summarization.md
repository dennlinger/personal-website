---
title: "Cohere Summarization"
date: 2023-03-05T20:51:56+01:00
hero: "https://dennis-aumiller.de/posts/hero.png"
tags: ["Summarization"]
categories: ["Summarization"]
draft: false
---

Last week, [Cohere.ai](https://cohere.ai) announced their [new dedicated summarization endpoint](https://txt.cohere.ai/summarize-beta/)!
For someone currently doing their PhD on text summarization, this is both [worrying](https://i.kym-cdn.com/entries/icons/original/000/025/817/Screen_Shot_2018-03-30_at_11.34.27_AM.png), but obviously also a rather intriguing development: while recent advancements have been focusing on rather broadly applicable models (think, [chatGPT](https://openai.com/blog/chatgpt/)), providing more task-specific alternatives seems to be the [niche that Cohere is carving out for themselves](https://txt.cohere.ai/multilingual/).  
Adding to the surprise of seeing a dedicated summarization endpoint is the fact that text summarization is really hard; in the last 50 years, a lot of progress has been made, but our current state-of-the-art models still suffer from annoying problems such as [correctly retaining factuality of the input text](https://aclanthology.org/2020.emnlp-main.750/).
Another problem is the actual definition of "summaries" in different domains. Methods for generating a "good" summary of a news article are generally useless when it comes to generating the summary of a court ruling, or generating radiology reports from doctor notes.
Due to the combination of these (and other) factors, there are comparatively few productive settings in which summarization is actively used. To my knowledge, the two main applications using some form of summarization right now are some news aggregators, summarizing information from multiple news articles (which primarily uses extractive methods, meaning directly copying existing sentences from the input documents), as well as the recently introduced ["Document TL;DR" generator in Google Docs](https://ai.googleblog.com/2022/03/auto-generated-summaries-in-google-docs.html) (the latter using a variant of their own PEGASUS neural model). 

This lack of tooling and remaining issues also means that companies (read: "paying customers") are relatively unlikely to employ summarization systems or plan task settings where they would be directly used. While the imagined benefits of summarization systems are high, the current risks associated with their usage and the practical issues arising from their usage still outweigh these benefits.
However, the fact that Cohere has a [free tier available](https://txt.cohere.ai/free-developer-tier-announcement/), as well as their claim of having a broadly applicable endpoint, makes it drastically more available to not only customers, but also the associated research community (and other folks keen on exploring AI-based tooling).
To get a first feel of the new offering, I took the API for a spin, and want to put this new endpoint into the context of currently ongoing research.

## What we know about their model

The [blog post announcing the feature](https://txt.cohere.ai/summarize-beta/) gives about as much detail as one can hope from a company: Opening with an accurate and concise system-generated TL;DR, the article then goes on to talk about some of the basic properties available.
To me, the most interesting fact is the unusually long context available for this model (50,000 *characters* max).
To put this length into context, we can compare it to the usually employed unit of "subword tokens". These are vaguely similar to syllables, meaning a word can be broken down into individual sub-parts, to cut down on the required vocabulary for any collection of texts (note that this is a very generic simplification of the topic and should not be taken at face value; in fact, most subwords specifically do *not* correspond to actual syllables).
Assuming a mean subword token length of about 6 characters (based on some basic back-of-the-envelope math on top of `bert-base-uncased` model's [vocabulary file](https://huggingface.co/bert-base-uncased/blob/main/vocab.txt)), the popular 512 subword limit of most transformer models comes to about ~3,100 characters in length for comparison.
Extrapolating to around 50,000 characters puts this in the ballpark range of 8,192 subword units of context length, which is on the extremely high end of contexts considered in recent research.

However, a key detail about the model is hidden further down in the article:

"*[...]We extract key sentences from the document up to the context length, and then generate a synthetic document composed of the combination of those key sentences.*"

### Hybrid Summarization to the rescue!
This is a strong indicator that Cohere makes use of a joint architecture which is known as a *hybrid* summarization system, and is a popular trick to circumvent the general length limitations of transformer models. As described in the quote, the first part of a hybrid summarization pipeline consists of a "selector" method, which will highlight and extract a number of segments (usually sentences) in the original document. In research, this is also called an "extractive" summarization system, and will ensure that any text part is guaranteed to appear in the input document, which makes generation both cheaper and faster. Ultimately, this first stage is therefore used to drastically cut down the context length, without sacrificing too much loss in relevant content for the final summary at this stage.  
Only during the subsequent stage, a larger (and more expensive) language model is then run to actually generate what we call an "abstractive" system, i.e., generated text does *not* necessarily appear in the original input, and can constitute a re-write of segments (or, more worryingly, can be made up completely).
Importantly, this use of preliminary selectors, which are often comparatively cheap approaches, drastically improve the computational efficiency of the entire pipeline compared to a "direct" approach, such as feeding an input text directly into a GPT-like model. 


### A likely extractor
There are of course different ways in which the pre-selection can be done, and nothing more is said in the article itself. Without further information, we can speculate on the specific details based on popular extractive models (which constitute the first part of the pipeline).  
Given the input length limitations, it is unlikely that Cohere is employing an extractive model which is performing extractive summarization with Transformers, e.g., [PreSumm](https://github.com/nlpyang/PreSumm) or similar libraries. This is based primarily on the fact that the length limitations are also applicable in extractive settings. Furthermore, utilizing a supervised extractive model requires additional training data on "segment relevance" (i.e., judging which sentences are to be extracted); this again is highly dependent on the text's domain and potentially limiting the generalizing qualities of such a system.  
In my opinion more likely is a pipeline employing an unsupervised approach. Here, no specific training data is required, and oftentimes these methods have a decent performance across different domains.
One very likely approach is the combination of [LexRank](https://arxiv.org/abs/1109.2128), a tried-and-tested method from 2004, augmented with modern sentence embeddings. This use case is [even suggested by Nils Reimers](https://github.com/UKPLab/sentence-transformers/tree/master/examples/applications/text-summarization), himself Director of Machine Learning at Cohere.
Especially with Cohere providing dedicated (multilingual) embedding endpoints, this is also a neat way to integrate several of their endpoints in one go.  
Finally, it can be argued that unsupervised approaches also provide a more reasonable cost for inference, depending on their use of neural architectures internally. However, using embeddings generated with neural models kind of defeats any argument on performance, so this is no certainty.

### Details on the abstractive part
Based on further parameters available, there seems to be good reason to assume that the abstractive model is largely based on Cohere's existing `.generate()` architecture, with some additional fine-tuning towards specific summarization tasks.
We can assume that some labeled training data was obtained to learn the distinction of different output styles (paragraph vs bullets) and extractiveness (note that this parameter is not available in the Beta playground endpoint, but rather [only mentioned in the docs](https://docs.cohere.ai/reference/summarize-2#3-define-model-settings)).
I have tried to extract portions of the used prompt, however, it seems that Cohere has sufficient query blockers in place to prevent a "quick investigation" from my side, at least.
Based on the "additional command" parameter, it seems that a model following the instruction-fine-tuning paradigm was already used for this purpose, which would make this more similar to Cohere's "command-xlarge-nightly" model than the older (more generic) "xlarge" variant.


## Testing the waters

With the speculations about implementation details out of the way, we can now actually take a look at the systems and some preliminary outputs tested by yours truly.
Note that this is still a rather anecdotal evaluation and a more thorough investigation is planned for the future.

In particular, I have a few general points that I look out for when interacting with summarization systems and that I will also focus on in this preliminary investigation:

1. Are they capable of retaining factuality between the input and the summary?
2. What part of the text is relevant information extracted from? Are relevant sentences clustered around the beginning, or actually appearing later in the text?
3. How does the model perform for texts from different domains (medical, legal, financial texts)?
4. How well does the system perform for texts that are *not in English*?
5. Are there any other non-grammatical outputs, such as repetitions or otherwise incoherent segments?




## What's next?

This poses a question to which we certainly have a less clear picture right now.  
In my opinion, there are several ways which are logical and immediate follow-ups on the existing model endpoint.
I expect that they will **expand the examples and documentation** website. Currently, linked pages  in [their docs about summarization](https://cohere.ai/use-case-summarization) still refer to the general-purpose generation endpoint (see, "Build a summarizaiton app" in the bottom left), which is precisely **not** the one to use for this task.  
Secondly, **explicit training for multilingual summary** experiences could be a possible extension. The model already exhibits a (albeit weak) multilingual tendency, which allows users to explore foreign-to-English summarization settings. If Cohere manages to extend their functionality to some of the other languages with a potential customer base, this could be a realistic extension.  
Something that will also likely be expanded is the **degree of customization possible**. While the (beta) endpoint already includes generic parameters such as "length", "extractiveness" and a mild form of styling ("bullets" or "paragraphs" being the available choices), recent trends in summarization research have shifted to a much broader degree of "controllable generation". Similar directions can be observed in the "more general LLM research", where instruction-tuned tooling is strongly preferred by users over more generic LLMs.  
While there is also an option to give user-specified free-form instructions is already in place, it **currently does not seem to affect the produced output** too much. If Cohere manages to expand on the customization options available this way, a much more generic set of "summarization-like" tasks could be approached. This includes, for example, writing [entity-centric summaries](https://aclanthology.org/2022.acl-long.237.pdf), where the summary provides a much more targeted text compared to a (broader) input document, or [data-to-text](https://ai.googleblog.com/2021/01/totto-controlled-table-to-text.html) settings, which is highly relevant in financial/business settings.  
However, more important will be the degree of **adoption by clients**; I suspect that we will see the emergence of more tailored endpoints in the near future, which specifically cater to the needs of different specialized domains, again, possibly for specific legal/financial/medical documents. Here, a much stricter focus on factuality is generally required, and if *useful* application scenarios can be identified, it requires the adoption of slightly more consistent summary generation than currently possible. I am not convinced at this point that the "extractiveness" parameter delivers the required level of factual accuracy, yet.

In summary (no pun intended), it seems that Cohere has not released a major groundbreaking leap forward at the scale of chatGPT -- but they might not have to. Providing a highly usable experience to a smaller set of dedicated users can pay off for them in the long run; a concise feature set with *real application value* to users will ultimately beat the comparatively flashy but less straightforward chatGPT.
As for myself, I am currently evaluating the model on a broader dataset for performance on academically used evaluation metrics; so stay tuned for a part two!

