---
layout: post
title:  "Procs and Lambdas in Ruby Pt 2"
categories:
---
Last week I wrote [a post](https://davdkm.github.io/2016/11/14/procs-and-lambdas.html) about the differences between special blocks of code called `Proc` and `lambda`. This post will cover how these closures can get around scope gates and pass around variables defined that were defined outside of the current scope. To remind ourselves how scopes work in ruby, here's an example

{% highlight ruby %}
outer_scope = 'this is defined outside of scope'

def some_method
  inside_scope = 'this is from inside the scope'
  puts inside_scope
  puts outer_scope
end

some_method
# this is from inside the scope
# => NameError: undefined local variable or method 'outer_scope' for main:Object
{% endhighlight %}
Now check out how we can pass the variable `outer_scope` with the `yield` keyword
{% highlight ruby %}
outer_scope = 'this is defined outside of scope'

def some_method
  inside_scope = 'this is from inside the scope'
  yield inside_scope
  puts 'back to the method scope'
end

some_method { |foobar| puts foobar; puts outer_scope }
# this is from inside the scope
# this is defined outside of scope
# back to the method scope
# => nil

{% endhighlight %}
As we can see, the method #some_method is passed a block kind of like an argument `{ |foobar| puts foobar; puts outer_scope }` which also passes in `outer_scope` from beyond the scope of the method.

Now let's try using a proc
{% highlight ruby %}
outer_scope = 'this is defined outside of scope'

def some_method(some_var)
  inside_scope = 'this is from inside the scope'
  proc { some_var + ' and ' + inside_scope } if some_var
end

new_proc = some_method(outer_scope) # We can assign our block to a variable!
new_proc.class # gets the class of the new_proc object
outer_scope = 'something else' # We've changed the value of outer_scope after we've defined new_proc
# => Proc
new_proc.call # the proc defined by new_proc is executed by call
# => "this is defined outside of scope and this is from inside the scope"
{% endhighlight %}
Here we've used a proc to combine the strings from `inner_scope` and `outer_scope` which was passed in when we defined `new_proc` and available to the method scope as `some_var`.

Proc and lambda are pretty similar (except for the differences discussed in my previous post), and we can change our method to use a lambda instead by
{% highlight ruby %}
def some_method(some_var)
  inside_scope = 'this is from inside the scope'
  lambda { some_var + ' and ' + inside_scope } if some_var
end

lamb = some_method(outer_scope)
lamb.class # => Proc
lamb # => #<Proc:0x00...@(irb):4 (lambda)>
{% endhighlight %}
Thanks for reading, and stay tuned in for more posts!
