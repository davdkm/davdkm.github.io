---
layout: post
title:  "Procs and Lambdas in Ruby Pt 1"
categories:
---
If you've been learning how to develop software in Ruby, you are probably familiar with blocks (closures). Blocks are statements that are fed into method calls as arguments which then executes those statements. Here's an example of a block given to a popular ruby method.
{% highlight ruby %}
[1, 2, 3].each {|x| puts x}
[1, 2, 3].each do |x|
  puts x
end  
{% endhighlight %}
In this post, we're going to introduce Proc and lambdas. A Proc is similar to a block in that it also is a statement or statements, but unlike a block which are like anonymous functions, `Proc` is a ruby class and new procs are instances of the class.
{% highlight ruby %}
new_proc = Proc.new {|x| puts x} # => #<Proc:0x007fd9a080e450@(irb):1>
new_proc.call('Hello World') # Hello World
new_proc.class # => Proc
{% endhighlight %}
As you can see, we create a new Proc object that defines a block of code. We call the proc with `call` and supply it with an argument `'Hello World'` which is then passed on the block defined by the proc.

Similar to a proc, ruby also has `lambda`.  A lambda is a special kind of `Proc`. One of the differences between a lambda and a regular proc is that a lambda have argument number restrictions. A proc can be supplied with more arguments than required, but lambda's will throw errors that the number of arguments is off.
{% highlight ruby %}
new_lamdba = lambda {|x| puts x} # => #<Proc:0x007fd99f8a65f8@(irb):4 (lambda)>
new_lamdba.class # => Proc
new_lamdba.call('Hello World') # => Hello World

new_proc('hello world', 'foobar') # => hello world
new_lamdba('hello world', 'foobar') # ArgumentError: wrong number of arguments (2 for 1)
{% endhighlight %}
Another key difference with proc and lambda is that returns work a little differently in each.  Lambda returns from it self while procs returns from the method. Lets check this behavior with some examples
{% highlight ruby %}
def lambda_returns
  l = lambda { return 'foobar'}
  l.call # returns foobar from the call and continues to the next line
  puts 'Hello World'
end

lambda_returns # Hello World => nil
{% endhighlight %}
And the same as a proc
{% highlight ruby %}
def proc_returns
  p = Proc.new { return 'foobar'}
  p.call
  puts 'Hello World' # shouldn't get hit because Proc already returned foobar
end

proc_returns # => nil
{% endhighlight %}
In my next post, I'll go into how Procs and lambdas can be used to pass data between scopes.
