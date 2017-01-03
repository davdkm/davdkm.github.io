---
layout: post
title:  "Bootstrap Quickstart"
categories:
---
Ever feel like coding a quick web site without wire framing the front end? Bootstrap templates offer a quick way of getting a page to some basic styling to Rails apps. I should have used this approach earlier in my development career, but for the reader that is just starting out building a basic rails app with no styling on the front end, read on!

A couple weeks ago, I wrote a post about a quick fix for building a responsive web site with the html meta tag `<meta name="viewport" content="width=device-width, initial-scale=1">`. Well, my brother took another look at the site I was working on and suggested just using a start template. Duh! There are a variety of templates for bootstrap and a couple of basic ones can be [found here](https://getbootstrap.com/getting-started/#examples). Just click a template, view the page source in your browser, and copy and paste the relevant code in the head and script tags into your `application.html.erb` file in your views/layouts folder. Just be sure you copy these bits
{% highlight html %}
<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1">
<!-- The above 3 meta tags *must* come first in the head; any other head content must come *after* these tags -->

<!-- Bootstrap core CSS -->
<link href="https://getbootstrap.com/dist/css/bootstrap.min.css" rel="stylesheet">

<!-- IE10 viewport hack for Surface/desktop Windows 8 bug -->
<link href="https://getbootstrap.com/assets/css/ie10-viewport-bug-workaround.css" rel="stylesheet">

<!-- Just for debugging purposes. Don't actually copy these 2 lines! -->
<!--[if lt IE 9]><script src="../../assets/js/ie8-responsive-file-warning.js"></script><![endif]-->
<script src="https://getbootstrap.com/assets/js/ie-emulation-modes-warning.js"></script>

<!-- HTML5 shim and Respond.js for IE8 support of HTML5 elements and media queries -->
<!--[if lt IE 9]>
  <script src="https://oss.maxcdn.com/html5shiv/3.7.3/html5shiv.min.js"></script>
  <script src="https://oss.maxcdn.com/respond/1.4.2/respond.min.js"></script>
<![endif]-->
{% endhighlight %}
And also in the script tag in the body to get the jquery for some bootstrap components to work correctly.
{% highlight html %}
<!-- Bootstrap core JavaScript
================================================== -->
<!-- Placed at the end of the document so the pages load faster -->
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
<script>window.jQuery || document.write('<script src="../../assets/js/vendor/jquery.min.js"><\/script>')</script>
<script src="https://getbootstrap.com/dist/js/bootstrap.min.js"></script>
<!-- IE10 viewport hack for Surface/desktop Windows 8 bug -->
<script src="https://getbootstrap.com/assets/js/ie10-viewport-bug-workaround.js"></script>
{% endhighlight %}
Now you have bootstrap loaded up in your app!
