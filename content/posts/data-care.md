---
title: "Why I care about your datasets -- And you should, too"
date: 2022-09-01T20:37:14+02:00
draft: true
---

With over [9,200](https://huggingface.co/datasets) publicly (and easily accessible) datasets hosted on Huggingface alone, we are living in what can be considered as "the golden era of open data".
Over is the time when we had to anxiously await responses from authors of related work, hoping to be blessed with a download link to some obscure server, or [download sites that are missing any form of CSS styling](https://www.cis.lmu.de/~schmid/tools/TreeTagger/) where one never is quite sure how much longer the site might stay up?
It comes as no surprise that we are seeing an influx of resource papers at major NLP conferences, and fortunately also an increase in [resources beyond English](https://lrec2022.lrec-conf.org/en/about/hot-topics/).
In this post, I want to highlight some of the major limitations I see in the current "data-centric approach" to Machine Learning. A large share can be attributed to the automated construction of resources, with not enough fail-safe practices in place.
As a community, we need to learn how to effectively incorporate more exploratory investigation and post-processing analysis into the construction processes of such resources.

## Why **I** care about data

As a PhD student, I immensely benefit from the "groundwork" that others are doing for my own research, especially if it is publicly available and ready for usage in my own projects. I can disseminate or even re-purpose existing data, which frees up time that I would otherwise have to dedicate to data acquisition on my own time.
This time save alone should be enough reason for me to care about data, but with the emergence of pre-trained model checkpoints ("base models") becoming more widely spread, the original data has now an even more profound impact on my own applications, where I might use the model checkpoints as a starting point for later work.

Now, for the particular article, I was originally inspired by a recent project in my own research.
When I started investigating models for a feasibility study of German summarization systems, I was quite surprised (and a little bit disappointed) by the general care people take when (re-)using datasets.
One of the initial problems is the fact that resource papers, despite their gain in popularity, seem to have somewhat of a bad reputation as bringing very little value to the community without further experimentation or model contributions aside from the data.
To some degree, this might seem justified: Who would need a fifth dataset for Natural Language Inference, especially if the existing ones have high standards in their annotation procedure and a history of being used in aggregated benchmarks?
But this also paints a very one-sided picture, by ignoring the nuanced contributions of novel datasets. Perhaps existing dataset do not cater towards a niche domain? Or does the dataset exhibit another major distribution shift from what we see in the existing ones?
Of course, all of this has to be properly outlined and verified in order to be properly considered.
Yet, I firmly believe that, **given a careful data processing, a new resource is (almost always) a good idea and helpful contribution**:

1. At this point in time, our Machine Learning systems are [literally bound by training data](Google paper). While this article talks about self-supervised training setups, high quality task-specific data is an additional resource to learn from.
2. We rarely have a perfect understanding of how previous datasets were created. One of my favorite quotes, "The devil's in the details, but the details aren't in the paper" touches on an extremely valid point when it comes to collection processes: Even when annotator guidelines are provided, or appendices filled with hyperparameters, an exact specification is almost never possible without original source code, and even then it is difficult to reproduce the set. Creating a new resource helps us learn what issues can arise, and *potentially discover new aspects to an age-old topic*.

The crux is that many people seem to have this romanticized view of data collection: If it's on the web, it can be scraped/accessed without much problem. But anyone who has tried to replicate raw data from the web will quickly realize that it is a lot more involved:

* Web scrapers require careful setup to operate within even a single domain. Hurdles can be something simple as [PHP-based calendar views](example on Heidelberg website), leading to millions of "unique" URLS, or simply `robots.txt` files that have settings allowing a single request every minute.
* This does not even mention other server-side issues or connectivity problems. Scraping websites that might not be available anymore need to be caught, or otherwise retried without crashing or delaying a larger collection process.
* Even with plenty of "ready-to-deploy" tools out there, manual adjustments to the configuration or changes during the crawl might be required.

## Garbage In, Garbage Out

[Insert XKCD comic here](https://xkcd.com/1838/)  
The [age-old wisdom](https://en.wikipedia.org/wiki/Garbage_in,_garbage_out) is quite representative why **you** should also care about data; without any properly processed data, it is near impossible to expect sensible results in our later experimentation.
More tragically, though, bad data can also distort our understanding of model's performance.