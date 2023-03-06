---
title: "Lunch Notes: 'The Internet Never Forgets', or does it?"
date: 2023-01-20T12:39:31+01:00
hero: images/other-hero.jpg
draft: false
---

I recently started clearing out some of my old browser bookmarks, something I highly encourage you to do, too!
Given that I have been in the academic world for close to ten years now, I still mix a lot of my personal browsing with the professionally required one, which created more and more bloat over time in my browser.  
Aside from the Mari Kondo-esque liberation it gives, I initially went through my bookmarks to finally start separating them into various categories, putting a more distinct divide between my "personal" and "professional" online footprint.
In some sense, it also felt like clearing out a childhood bedroom, given that I have consistently used the same browser (Firefox) for close to fifteen years now, and certainly not all bookmarks hold the same relevance today as back then.

Growing up with the Internet, the title phrase "the Internet never forgets" has been deeply ingrained into my memory, and I always more or less believed this to be true.
But actually going through some of my "web memories" from longer ago meant re-visiting sites that I initially assumed to give me a glimpse into the past and, to my surprise, many of them were no longer accessible.  
I know that the original meaning of the phrase is generally supposed to indicate the free spreading of information, and unwanted sharing of sensitive information; however, while I could certainly track some of the saved pages via various [web archives](https://archive.org/) or even Googling them, it seems that the Internet is in fact more forgetful about things than we might have originally anticipated.

This is especially exacerbated by the rise of more dynamic web content, culminating in the recent trend of [SPA](https://en.wikipedia.org/wiki/Single-page_application) sites. Not only do they make it more difficult [to perform effective web search over them](https://seranking.com/blog/single-page-application-seo/), but they also diminish the chances of page content being included in web archives.  
To broadly categorize the types of "dead links" I found, these are the most prominent instances in my personal experience:

  1. Memes and other static content from "managed sites", e.g., Reddit, news sites, etc., was among the most prominent dead links. I can attribute this to two major reasons: Either users deleting their accounts (and with them some of the posts) or simply cleansing of old content after a certain time period. These are also by far the most likely to still appear in archives, as they come with their individual URL and have a high likelihood of being crawled due to popularity of the domain.
  2. Websites where the entire domain was taken down. These include primarily personal websites or smaller sites, which are nowadays frequently replaced with managed/hosted sites or sub-communities on other platforms (again, e.g., Reddit). This mainly removes the burden of protecting one's own servers against attackers, which is incredibly tedious (I vaguely remember a paper talking about how the first attacks on a new server nowadays start within an hour of it first going online. If you know of the title of this work, please reach out, I could not find it!). Also, hosting costs have decreased significantly over the past ten years, but due to different platforms, older content was not moved to newer pages.
  3. Funnily enough, a lot of academic pages also went "MIA". I had a particular number of math scripts that were no longer accessible, as the respective authors seem to have moved academic institutions. Based on personal experience, I can also say that this is (in part) to blame on the institutions' weak admin support. Case in point, my academic mail would be deleted soon after I finish graduation, and I have to petition to extend the accessibility beyond my student days.


What makes all of this particularly challenging is that bookmarks in themselves do not hold a lot of information on the actual content ("why did I even bookmark this in the first place?"). This makes the already tedious follow-up of searching for this particular content unnecessarily hard and something I will likely not do for many of the out-of-date bookmarks.

In a neat tie to my current research in text processing, this also has interesting effects on reproducibility of large-scale web-based datasets. Given that we often only provide the links to websites instead of the static content (partially for storage space reasons, partially because of legal rights), we cannot ascertain a fully reproducible result, and should never naively assume that content is available in the same state in the future (even on pages like Wikipedia, content will evidently change over time). And simultaneously, having a single point of failure for centralized access of data is similarly worrying, if just for the unlikely case of that particular hoster going belly-up.

With that said, I did have a great time reveling in the memories of pages that I had not accessed in a decade (and which were still working); and I again encourage you to do the same! If you do find any similar types of "missing content", feel free to let me know!


PS: I have just enabled custom "hero images" (as they are called in Hugo), which you can see at the top. They are chosen entirely by aesthetics and not necessarily by relation to the topic itself. The image in this post is from my trip to Stavanger for ECIR 2022 (a conference report I co-authored being available [in the BCS Informer](https://irsg.bcs.org/informer/2022/05/ecir-2022-experience-report/)); the swords are about 10 metres tall and called ["Sverd I Fjell"](https://en.wikipedia.org/wiki/Sverd_i_fjell).