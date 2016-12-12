---
layout: post
title:  "Viewport and Mobile Browsing"
categories:
---
Recently I asked someone to give me some design feedback for my project sites. Among some really helpful feedback that I had overlooked or simply had not thought of, she mentioned that it was really hard to view on a mobile device. It turns out, my site layout was only responsive on the desktop that I'd been developing on. Oops!

Fortunately, it was a pretty simple but an important fix so I thought I'd write a quick blog post about it. For more complicated sites that rely on bootstrap, it's common to write some custom style in LESS with media queries. My sinatra project currently doesn't require any fanciness. I'm still working on dressing it up, so for now I'll rely on bootstraps' `.container-fluid` and the html meta tag for setting the viewport to the device width.

For my divs I change the class from regular `container` to `container-fluid` to automagically adjust the width. Then with the following html tag added to the head of my `layout.erb`, the site should now be a bit more mobile friendly.
{% highlight html %}
<meta name="viewport" content="width=device-width, initial-scale=1.0">
{% endhighlight %}
I'll end up adding some custom styles to use when the screen sizes change, but this was a really quick and dirty solution for my needs at the time.
