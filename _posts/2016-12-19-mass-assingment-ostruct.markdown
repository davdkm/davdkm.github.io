---
layout: post
title:  "OpenStruct and Struct"
categories:
---
I was recently introduced to OpenStruct in Ruby and how it can create objects via metaprogramming. It does this by creating a method defined by hash key's and sets the value's to the value's. This is similar to mass assignment with keywords in Ruby. Let's take an example

{% highlight ruby %}
require 'ostruct'

student = OpenStruct.new
student.name = 'Jane'
student.age = '20'
student.major = 'Sociology'
# Alternativetly we can create a student with a Hash
student = OpenStruct.new(:name => 'Jane', :age => '20', :major => 'Sociology')
{% endhighlight %}
This creates a new object OpenStruct object `student` that now has the methods #name, #age, and #major methods available to it instead of throwing undefined method errors.
{% highlight ruby %}
student.name  # => 'Jane'
student.age # => '20'
student.major # => 'Sociology'
{% endhighlight %}
OpenStruct can add whatever object we want by just defining it like so
{% highlight ruby %}
student.favorite_food # => nil
student.favorite_food = 'Mac & Cheese'
student.favorite_food # => 'Mac & Cheese'
{% endhighlight %}
As the ruby-docs mentions, OpenStruct can get a bit costly in performance. A cheaper way and more restricted way to create objects can be achieved with Struct.
#### New Objects with Struct
Struct objects need to have their methods defined before hand, unlike OpenStruct.
{% highlight ruby %}
Professor = Struct.new(:name, :age) # => Professor
prof = Professor.new('Richard', '60') # => <struct Professor name="Richard" age="60">
prof.name # => 'Richard'
prof.age # => '60'
{% endhighlight %}
If you look at the output for when we defined `Professor = Struct.new(:name, :age)` we are actually defining a new rubly class object! You can confirm this by running `Professor.class` and it will return `Class`. However, Struct are not able to define new methods (attrs) on the fly
{% highlight ruby %}
prof.department = 'Computer Science' # No method exception!!!
{% endhighlight %}
So here we have a couple of different ways to create ruby objects with attributes. However, I'm still not sure when we'd need to actually use such techniques. Perhaps when making really simple classes but I think I'd rather define my classes with mass assignment with keywords, like in [this](https://learn.co/tracks/full-stack-web-development/object-oriented-ruby/metaprogramming/mass-assignment-and-metaprogramming) lesson from the Flatiron School.
