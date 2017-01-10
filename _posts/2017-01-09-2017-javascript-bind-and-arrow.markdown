---
layout: post
title:  "Bind and Arrow Functions in Javascript"
categories:
---
I've been brushing up on my Javascript skills lately and came across the `.bind()` and arrow functions.
Firstly, the bind method creates a new function from existing function when it gets called. It also creates a new scope for the newly created function by setting `this` to whatever is supplied to `.bind(this)`.

To see an example of how one might get the scope of `this` mixed up, let's consider this slightly modified example from Mozilla's Development Network. Let's open up a js console in chrome.
{% highlight Javascript %}
this.str = 'foo'; // this refers to global 'window'
str;
// "foo"

var module = {
  str: 'batz',
  getStr: function() { return this.str; }
};
module.getStr();
// "batz"

var  callStr = module.getStr;
callStr();
// "foo"; What?! Shouldn't it return "batz". 'this.str' in the function
// is actually getting called on the global scope! Bind to the rescue

var boundCallStr = callStr.bind(module); // bind this to the module object
boundCallStr();
// "batz" like we wanted!
{% endhighlight %}
Another advantage of using bind is the ability to leverage partially applied functions. By abstracting out our arguments supplied in our function and using bind, we can create a new function leveraging the functionality of another. Just for fun let's write a function that replaces all occurrences of a word in a string with 'foobar'.
{% highlight javascript %}
var phrase = "Vanilla ice ice baby. Vanilla ice ice baby."

var replaceWord = function (word, string) {
  const matcher = new RegExp(word, 'gi');
  return string.replace(matcher, 'foobar');
};
replaceWord('ice', phrase);
// "Vanilla foobar foobar baby. Vanilla foobar foobar baby."

var replaceBaby = replaceWord.bind(null, 'baby');
replaceBaby(phrase);
// "Vanilla ice ice foobar. Vanilla ice ice foobar."
{% endhighlight %}
Arrow functions introduced an easier way to write functions. The syntax is as follows
{% highlight javascript %}
var replaceWord = (word, string) => {
  const matcher = new RegExp(word, 'gi');
  return string.replace(matcher, 'foobar');
};
replaceWord('ice', phrase); // works the same way as the function defined earlier!
// "Vanilla foobar foobar baby. Vanilla foobar foobar baby."
{% endhighlight %}
Arrow functions are always anonymous and don't assign a new 'this'!
