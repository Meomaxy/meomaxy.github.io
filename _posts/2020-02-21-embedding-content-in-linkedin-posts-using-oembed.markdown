---
layout: post
title:  "Embedding Content in LinkedIn Posts Using oEmbed"
date:   2020-02-21 09:11:32 -0500
categories: oembed
---
<figure markdown="1">
![oEmbed is a format for allowing an embedded representation of a URL on third party sites.](/assets/images/2020-02-21-embedding-content-in-linkedin-posts-using-oembed/oembed.png)
<figcaption markdown="1">oEmbed is a format for allowing an embedded representation of a URL on third party sites.
</figcaption>
</figure>

One of the more expressive features of LinkedIn’s Publishing Platform is the ability to embed content from another site into an article.

For example, I could direct you to view this video:

[Embedding a Video on LinkedIn Publishing Platform](https://www.youtube.com/watch?v=WI_Ls-zD4VU&feature=youtu.be)

But even better, I can embed the video into the article, like this:

<iframe width="560" height="315" src="https://www.youtube.com/embed/WI_Ls-zD4VU" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# How do we do that?

YouTube provides a mechanism for [embedding video content in other web pages](http://www.htmlgoodies.com/tutorials/web_graphics/article.php/3480061/How-To-Add-a-YouTube-Video-to-Your-Web-Site.htm). If you view the video above on the YouTube site, you can see the code for embedding the video under the “Share” button.

<figure markdown="1">
![How to get embed html from a YouTube video](/assets/images/2020-02-21-embedding-content-in-linkedin-posts-using-oembed/embedding-an-image.jpg)
</figure>

The video can be embedded in another web page by including the snippet of HTML provided by YouTube:

{% highlight html %}
<iframe
  width="560"
  ﻿height="315"
  ﻿src="https://www.youtube.com/embed/WI_Ls-zD4VU"
  ﻿frameborder="0"
  ﻿allowfullscreen>
</iframe>
{% endhighlight %}

Unfortunately, you can’t use that embed HTML directly on the LinkedIn Publishing Platform. There is no feature for pasting a snippet of HTML into your article.

So, how is embedding done?

Well, how _might_ we do it? You may have noticed that the URL for the video:

```
https://www.youtube.com/watch?v=WI_Ls-zD4VU
```

Is similar to the URL inside the HTML for embedding the video:

```
https://www.youtube.com/embed/WI_Ls-zD4VU
```

So, it’s plausible to imagine that we recognize that this is a YouTube video, and that ID of this video is `WI_Ls-zD4VU`. Then, if we know the proper format for an embedded YouTube video URL, we can construct the embed HTML by inserting the ID in the right place.

We could have done it that way. The problem is that we’d like to support embedding for all sorts of content, and every service works slightly differently.

We would need to have custom code to implement embedding support for every provider. Not only that, but any provider might decide at any time to change how they implement embedding in a way that breaks our assumptions, causing the embeds to fail.

# Origins of oEmbed

![How to get embed html from a YouTube video](/assets/images/2020-02-21-embedding-content-in-linkedin-posts-using-oembed/oembed-all-service.png){:style="float: right;margin-right: 7px;margin-top: 7px;"}

Around 10 years ago, a company called [Pownce](https://en.wikipedia.org/wiki/Pownce) was facing exactly this problem. As the [story is told](https://blog.leahculver.com/2008/05/announcing-oembed-an-open-standard-for-embedded-content.html) by [Leah Culver](https://www.linkedin.com/in/leahculver/), she and her coworker at Pownce, [Mike Malone](https://www.linkedin.com/in/mmalone/), were having dinner with [Cal Henderson](https://www.linkedin.com/in/iamcal/) (Flickr) and [Richard Crowley](https://www.linkedin.com/in/richarddcrowley/) (OpenDNS). They chatted about this problem and conceived the idea of creating an open standard for sharing embed code. Cal drafted an initial spec overnight, and they named it [oEmbed](https://oembed.com/).

> It basically works by having your site say to YouTube, “Hey, I want to add this video to my post. Can you please send me the HTML? Thanks!” 
> <br> – [Skookum Monkey](https://skookummonkey.com/blog/awesome-simplicity-oembed-wordpress/?utm_source=ReviveOldPost&utm_medium=social&utm_campaign=ReviveOldPost)

Many content providers implement oEmbed, which LinkedIn uses to support embedding. Here is how it works in the case of YouTube.

Inside the HTML of the YouTube video above, there are two extra tags in the [<head>](https://www.w3schools.com/tags/tag_head.asp) section that tell us how to call YouTube’s oEmbed endpoint. Here’s one of them (the other one supports retrieving the same data in [XML](https://en.wikipedia.org/wiki/XML) format).

{% highlight html %}
<link rel="alternate"
  ﻿type="application/json+oembed"
  ﻿href="http://www.youtube.com/oembed?url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DWI_Ls-zD4VU&amp;format=json"
  ﻿title="Embedding a Video on LinkedIn Publishing Platform">
{% endhighlight %}

This tag tells us that to retrieve the oEmbed data for this video, we should get this URL:

```
https://www.youtube.com/oembed?url=https%3A%2F%2Fwww.youtube.com%2Fwatch%3Fv%3DWI_Ls-zD4VU&format=json
```

This URL returns a packet of data in [JSON](https://www.json.org/json-en.html) format that looks like this:

{% highlight json %}
{
  "thumbnail_height": 360,
  "provider_name": "YouTube",
  "html": "<iframe width=\"480\" height=\"270\" src=\"https://www.youtube.com/embed/WI_Ls-zD4VU?feature=oembed\" frameborder=\"0\" allowfullscreen></iframe>",
  "title": "Embedding a Video on LinkedIn Publishing Platform",
  "height": 270,
  "type": "video",
  "provider_url": "https://www.youtube.com/",
  "thumbnail_width": 480,
  "author_url": "https://www.youtube.com/channel/UCYvvghEq7ThacEF6PP0vgDA",
  "width": 480,
  "author_name": "David Max",
  "version": "1.0",
  "thumbnail_url": "https://i.ytimg.com/vi/WI_Ls-zD4VU/hqdefault.jpg"
}
{% endhighlight %}

This data provides us with lots of useful information. For example, included here is the title of the video, as well as the URL for the thumbnail image that goes with it. For the purposes of embedding, the most important field is the `"html"` field. It contains this same snippet of HTML we saw before that tells us how to embed the video into the article.

What’s so powerful about oEmbed is that we can use essentially the same technique to embed completely different kinds of content. For example, l can embed this [SoundCloud audio stream](https://soundcloud.com/greylock-partners/blitzscaling-19-jeff-weiner-on-establishing-a-plan-and-culture-for-scaling):

<iframe width="100%" height="300" scrolling="no" frameborder="no" allow="autoplay" src="https://w.soundcloud.com/player/?url=https%3A//api.soundcloud.com/tracks/249739519&color=%23ff5500&auto_play=false&hide_related=false&show_comments=true&show_user=true&show_reposts=false&show_teaser=true&visual=true"></iframe>

The same oEmbed mechanism allows us to obtain the HTML for embedding this content. In the HTML of the SoundCloud page is this link to the oEmbed endpoint:

{% highlight html %}
<link rel="alternate"
  ﻿type="text/json+oembed"
  ﻿href="https://soundcloud.com/oembed?url=https%3A%2F%2Fsoundcloud.com%2Fgreylock-partners%2Fblitzscaling-19-jeff-weiner-on-establishing-a-plan-and-culture-for-scaling&amp;format=json">
{% endhighlight %}

When we download the data from the oEmbed endpoint, we get:

{% highlight json %}
{
  "version": 1,
  "type": "rich",
  "provider_name": "SoundCloud",
  "provider_url": "http://soundcloud.com",
  "height": 400,
  "width": "100%",
  "title": "Blitzscaling 19 | Jeff Weiner on Establishing a Plan and Culture for Scaling by Greylock Partners",
  "description": "This is session 19 of Technology-enabled Blitzscaling, a Stanford University class taught by Reid Hoffman, John Lilly, Allen Blue, and Chris Yeh. This class features Reid Hoffman interviewing Jeff Weiner, the CEO of LinkedIn.",
  "thumbnail_url": "http://i1.sndcdn.com/artworks-000149288921-qvy4r3-t500x500.jpg",
  "html": "<iframe width=\"100%\" height=\"400\" scrolling=\"no\" frameborder=\"no\" src=\"https://w.soundcloud.com/player/?visual=true&url=https%3A%2F%2Fapi.soundcloud.com%2Ftracks%2F249739519&show_artwork=true\"></iframe>",
  "author_name": "Greylock Partners",
  "author_url": "https://soundcloud.com/greylock-partners"
}
{% endhighlight %}

In there is the HTML for embedding the SoundCloud player:

{% highlight html %}
<iframe
  width="100%"
 ﻿ height="400"
 ﻿ scrolling="no"
 ﻿ frameborder="no"
 ﻿ src="https://w.soundcloud.com/player/?visual=true&url=https%3A%2F%2Fapi.soundcloud.com%2Ftracks%2F249739519&show_artwork=true">
</iframe>
{% endhighlight %}

Notice that even though it’s different from YouTube, we can still use the same oEmbed mechanism to retrieve it.

Some other examples. The oEmbed links and embeddable HTML don’t follow any particular convention from one provider to another.

## [FOWA Miami - Leah Culver via Flickr](https://www.flickr.com/photos/hyku/2300680760/in/photolist-4viAeE-aaRLu1-5eHsAD-4ZcfCR-661WTp-3gNpSP-5rcg2w-4WZqzy-4viAny-6b1vLa-6qFngz-4vHDk2-3gMJYJ-4QHZAh-bhyC1x-4v5ctD-3Pdeoi-bwoaKt-475JtD-bhyyE6-taLus-4CJGGz-3gRNhb-4CJPWk-apzRjf-4CNXCL-5jUArC-a8N2Y2-4oBtBo-4vjGp5-4vewF2-4CJPSp-5WFqGy-7XuA8f-4vjwRA-5sWC81-4CP6cN-8xvYMJ-4vj8Tg-4vBnDe-4haZNJ-47no8m-4CJGrM-8xw2Dq-4viyTW-4oeY9P-4Ej66p-8NHBT6-4CP6g1-475Hnv):

<a data-flickr-embed="true" href="https://www.flickr.com/photos/hyku/2300680760/" title="FOWA Miami - Leah Culver by hyku, on Flickr"><img src="https://live.staticflickr.com/2169/2300680760_600aa44314_b.jpg" width="1024" height="685" alt="FOWA Miami - Leah Culver"></a><script async src="https://embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>

## Embedded HTML:

{% highlight html %}
<a data-flickr-embed="true" href="https://www.flickr.com/photos/hyku/2300680760/" title="FOWA Miami - Leah Culver by hyku, on Flickr"><img src="https://farm3.staticflickr.com/2169/2300680760_600aa44314_b.jpg" width="1024" height="685" alt="FOWA Miami - Leah Culver"></a><script async src="https://embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
{% endhighlight %}

## SlideShare [OAuth and OEmbed](https://www.slideshare.net/leahculver/o-auth-oembednyc09):

<iframe src="https://www.slideshare.net/slideshow/embed_code/key/eg8zV35wx3ux87" width="427" height="356" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="https://www.slideshare.net/leahculver/o-auth-oembednyc09" title="OAuth and OEmbed" target="_blank">OAuth and OEmbed</a> </strong> from <strong><a target="_blank" href="https://www.slideshare.net/leahculver">leahculver</a></strong> </div>

## Embedded HTML:

{% highlight html %}
<iframe src="https://www.slideshare.net/slideshow/embed_code/key/eg8zV35wx3ux87" width="427" height="356" frameborder="0" marginwidth="0" marginheight="0" scrolling="no" style="border:1px solid #CCC; border-width:1px; margin-bottom:5px; max-width: 100%;" allowfullscreen> </iframe> <div style="margin-bottom:5px"> <strong> <a href="https://www.slideshare.net/leahculver/o-auth-oembednyc09" title="OAuth and OEmbed" target="_blank">OAuth and OEmbed</a> </strong> from <strong><a target="_blank" href="https://www.slideshare.net/leahculver">leahculver</a></strong> </div>
{% endhighlight %}

## [Ben Ward on Twitter](https://twitter.com/benward/status/144870918613766144):

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Additional hugs and love to <a href="https://twitter.com/leahculver">@leahculver</a>, <a href="https://twitter.com/mjmalone">@mjmalone</a>, <a href="https://twitter.com/rcrowley">@rcrowley</a> and <a href="https://twitter.com/iamcal">@iamcal</a> for  inventing oEmbed.</p>
  — Ben Ward (@benward) <a href="https://twitter.com/benward/status/144870918613766144">December 8, 2011</a>
</blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

## Embedded HTML:

{% highlight html %}
<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Additional hugs and love to <a href="https://twitter.com/leahculver">@leahculver</a>, <a href="https://twitter.com/mjmalone">@mjmalone</a>, <a href="https://twitter.com/rcrowley">@rcrowley</a> and <a href="https://twitter.com/iamcal">@iamcal</a> for  inventing oEmbed.</p>
  — Ben Ward (@benward) <a href="https://twitter.com/benward/status/144870918613766144">December 8, 2011</a>
</blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>
{% endhighlight %}

# Support is Not Universal and Not Complete

Not every sort of content that supports embedding implements oEmbed. Some providers support embedding, but they expect that their content will be embedded in a page where there user can directly insert the snippet of HTML that implements the embedding.

And, not every content provider that supports oEmbed is enabled to embed content in on the LinkedIn Publishing platform. The LinkedIn platform does not simply pass through every embed HTML returned through the oEmbed protocol.

> When you’re creating a site that’s going to display HTML (as with video embeds), there’s always the potential for [cross-site scripting](https://en.wikipedia.org/wiki/Cross-site_scripting) (XSS) attacks from the site providing the code. 
> <br>– [Getting Started With OEmbed](https://www.wired.com/2010/02/get_started_with_oembed/)

One reason is security. In some cases, the embed HTML contains scripting calls or other features that are disallowed by default. There is a review process before a provider is enabled for embedding content. WordPress follows a [similar procedure](https://wordpress.org/support/article/embeds/).

# Content Providers, Show Us Your oEmbed!

If you are a content provider who supports embedding, please implement oEmbed. That is a prerequisite for allowing LinkedIn members to share your embedded content as LinkedIn updates or on the LinkedIn Publishing platform.

For those of you who are interested in having your provider's content supported for embedding within LinkedIn articles, please [submit a request](https://www.linkedin.com/help/linkedin/solve/contact), making sure to mention in the subject that your question pertains to embedding content in articles, thanks!

****

# Please join the conversation...

_What do you think? Comment on [LinkedIn](https://www.linkedin.com/pulse/embedding-content-linkedin-posts-using-oembed-david-max)._

_Thanks for reading. You can find my other LinkedIn articles [here](https://www.linkedin.com/today/author/davidpmax)._
