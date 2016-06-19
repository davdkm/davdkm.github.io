---
layout: post
title:  "Instance Methods in Ruby"
categories:
---

To talk about instance methods in Ruby, you have to first talk about classes and objects. A `Class` in ruby is a factory for making objects. What is an object? In, Ruby, it's everything. `2 + 2` will return a Fixnum object, `'any strings'` will return a string object, `true` and `false` are boolean type objects. When you want to program anything with a bit of complexity, it becomes easier/necessary to use a `Class`.

When you churn out an object created by a class, you are creating an instance of that class. It inherits all the methods defined by that class and a few more you may never have to use. Let's try making a class and instantiating a new object. For this post, let's make a `Dog` and `Person` class, and create a few instance methods that can be called on newly created objects by that class.

We'll be coding our solution to the Instance Method Lab. Create a ruby file in your terminal. I'm on a macbook using osx so I'll create a new ruby file with

{% highlight Bash %}
touch lib/dog.rb
touch lib/person.rb
{% endhighlight %}

We want to create our `Dog` class. Let's start by defining it in the `dog.rb` file

{% highlight Ruby %}
class Dog  # define our class

end
{% endhighlight %}

Nice. We have a `Dog` class, but new instances of `Dog` wouldn't have much to do. Let's change that make our dogs be able to bark with instance methods. We define an instance method like this

{% highlight Ruby %}
class Dog
  def bark  # define our instance method
    puts 'Woof!'
  end
end
{% endhighlight %}

Now if we want to make a dog bark, we'd instantiate a new dog with

{% highlight Ruby %}
# lib/dog.rb

fido = Dog.new  # create/instantiate a new dog and assign it to the variable 'fido'
fido.bark  # calls the #bark instance method on the newly created instance of Dog
{% endhighlight %}

`fido.bark` would return `"Woof!"`. How would you get `fido.sit` to return `'The Dog is sitting'` ?
With our `Person` class, we want our new person to respond to a #talk instance method.

{% highlight Ruby %}
class Person  # starts defining our Person class
  def talk  # start defining our #talk method
    puts "Hello World!"  # prints out 'Hello World!' to the output
  end  # closes out our definition of the #talk method. Super important.
end  # closes out our definition of the Person class. Also super important.
{% endhighlight %}

Now, if we were to create a new instance of the Person class, `dave = Person.new`, and then call the #talk instance method on the newly instantiated person `dave.talk`, we'll see the message `'Hello World!'`. How would you get our new person to respond to a #walk method that prints `'The Person is walking'` to the console?  

Run your tests with `rspec` in the terminal, after you coded out the rest of the instance methods!
