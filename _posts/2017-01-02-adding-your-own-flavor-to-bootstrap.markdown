---
layout: post
title:  "Adding Your Own Flavor to Bootstrap"
categories:
---
Bootstrap is a great way to jumpstart a new project. But at some point, you're going to want to add your own styling to set it apart. There's two ways of doing this: styling inline and in css stylesheets. Inline styles are useful for adding css for a single element. Here's an example of adding inline style with the `style` attribute.

{% highlight html %}
<div class="panel-group">
  <div class="panel panel-default">
    <div class="panel-heading" style="color: red;">
      A Bootstrap Heading in Red by inline style attribute.
    </div>
    <div class="panel-body">
      The body of the panel.
    </div>
  </div>
</div>
{% endhighlight %}

However, what if we had multiple elements that we wanted to add style to? Well we can add our styling in a css stylesheet. In rails, this will usually mean adding it to our 'application.scss' file (or in the automatically created stylesheets for rails resources). This file is located in the asset/stylesheets folder. That also means that it will be automatically loaded by our rails app. We can add the same line of code in our stylesheet

{% highlight css %}
/* application.scss */
.custom {
  color: red;
}
{% endhighlight %}

And then in our html all we need to do is add our newly created css class to the element

{% highlight html %}
<div class="panel-group">
  <div class="panel panel-default">
    <div class="panel-heading" style="color: red;">
      A Bootstrap Heading in Red by inline style attribute.
    </div>
    <div class="panel-body custom">
      The body of the panel with the styling from 'custom' added.
    </div>    
  </div>
</div>
{% endhighlight %}

Happy coding!
