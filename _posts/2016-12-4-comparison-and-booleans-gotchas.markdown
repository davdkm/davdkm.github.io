---
layout: post
title:  "Comparison and Boolean Gotchas"
categories:
---
I've been reviewing my coding skills in Ruby to keep myself sharp and thought I'd write a post about some gotchas that I came across during my studies.

#### == === .eql? .equal?
Even newbies to Ruby know what `==` does. It's an operator that compares the value of two operands.
{% highlight ruby %}
a = 1
b = 1
a == b
=> true
b = 2
a == b
=> false
{% endhighlight %}
But what if we want to be more discriminating about our comparison? The `==` only compares values and doesn't check the Class of the objects. How do we also for the same Class? If you come from a Javascript background, you might be inclined to say `===`. BUT WAIT
{% highlight ruby %}
a = 1
a.class
=> Fixnum
b = 1.0
=> Float
a == b
=> true
a === b
=> true
{% endhighlight %}
It turns out `===` in Ruby and Javascript behave differently. If you'd like to compare value AND class in ruby, you have to invoke the method `.eql?`, while `===` is used to compare the class of the object, usually in `when` clauses of `case` statements.
{% highlight ruby %}
a, b = 1, 1
a.eql? b
=> true
b = 1.0
a.eql? b
=> false
Fixnum === a
=> true
Float === b
=> true
Fixnum === b
=> false
{% endhighlight %}
Finally when we want to compare if an object is the very same object, as in they not just have the same value and class, but also the same `object_id`, we use the `.equal?` method.
{% highlight ruby %}
a = 'Hello World!'
b = 'Hello World!'
a == b
=> true
a.eql? b
=> true
a.equal? b
=> false # !!! but why? Check the object_id
a.object_id
=> 70270346064520
b.object_id
=> 70270346044960 # Not the same object!
b = a
a.equal? b
=> true # if we check the object_id of both a and b they will return the same object
{% endhighlight %}

#### Order of Operations and Precedence
This question came up on a sample interview question. Consider the following
{% highlight ruby %}
x = true && false
=> false
y = true and false
=> false
# what is the value of x and y
{% endhighlight %}
Although both statements return `false`, the *value* of `y` is actually `true`. In ruby `&&` has a higher precedence than `=` in the order of operations and `and` has a lower precedence than `=`. So what ends up happening is more like following
{% highlight ruby %}
x = ( true && false) # x = false
( y = true ) and false # y = true, then false is run as a separate statement on its own
{% endhighlight %}
So keep in mind you should use `&&` and `||` for comparisons and `and` and `or` for flow control.
