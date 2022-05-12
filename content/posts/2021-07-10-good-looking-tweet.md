---
title: Good looking tweet
date: 2021-07-10
description: "Make your blog posts look better on Twitter."
image: images/posts/twitter.jpeg
images:
  - images/posts/twitter.jpeg
tags:
  - Hugo
  - Twitter
---

When I was posting these blog post URLs in my Twitter feed, they seemed a bit wishy-washy.

{{< tweet 1415067244900532227 >}}

You see other good-looking tweets, and yours looks like something done without giving a shit. 💩

So let's make them look better.

After checking [https://developer.twitter.com](https://developer.twitter.com/en/docs/twitter-for-websites/cards/overview/summary-card-with-large-image), for it to display a card with an image, you have to add meta tags to your blog Html header.

You should add the following meta tags:

```html
<meta name="twitter:card" content="summary_large_image" />
<meta name="twitter:site" content="@nytimes" />
<meta name="twitter:creator" content="@SarahMaslinNir" />
<meta name="twitter:title" content="Parade of Fans for Houston’s Funeral" />
<meta
  name="twitter:description"
  content="NEWARK - The guest list and parade of limousines with celebrities emerging from them seemed more suited to a red carpet event in Hollywood or New York than than a gritty stretch of Sussex Avenue near the former site of the James M. Baxter Terrace public housing project here."
/>
<meta
  name="twitter:image"
  content="http://graphics8.nytimes.com/images/2012/02/19/us/19whitney-span/19whitney-span-articleLarge.jpg"
/>
```

So let's add those to my [Hugo](https://gohugo.io/) blog.

But wait, after digging a bit into Hugo's [documentation](https://gohugo.io/templates/internal#twitter-cards), you need to set the correct variables in your post front matter.

```yml
---
title: Good looking tweet
date: 2021-07-10
description: "Make your blog posts look better on Twitter."
images:
  - images/twitter.jpeg
tags:
  - Hugo
  - Twitter
---
```

Well, that was easy. 😅

You can check the look and feel of your tweet using this [card validador](https://cards-dev.twitter.com/validator).

Once again, a reminder to my future self, from Confucius:

> If a craftsman wants to do good work, he must first sharpen his tools.

{{< tweet 1415218957213216768 >}}

Looking a lot better now 🤩
