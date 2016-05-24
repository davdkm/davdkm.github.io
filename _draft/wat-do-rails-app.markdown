---
layout: post
title:  "Wat Do?! A Simple Rails App"
categories:
---

Intro:

This post is a journal for the process of creating a website in Rails. Our objective is to create a website that utilizes authentication and authorization (and also allow authentication from an outside service). It will also utilize nested resources and routes. For this project, we're going to build a site that can manage events. We want our users to be able to create events that other users can sign up to attend, and tag their own events with tag categories. Let's get started!

Getting Started:
First, we create our models and migrations. I use regular old rails generators to create the files with:
{% highlight Bash %}rails generate model Event{% endhighlight %}
and fill in the changes needed afterwards.
