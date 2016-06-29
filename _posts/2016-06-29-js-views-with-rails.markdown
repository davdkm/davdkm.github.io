---
layout: post
title:  "Rendering Views In JavaScript With Rails"
categories:
---

This post details the process of changing my event tracking app to use javascript. The backend will still be handled by rails, but we want to change it so that the index and show views use some ajax to render additional details for an event. In addition, we'd like our views to render without having to use redirects for the create and update actions.

#Active Model Serializer
We're going to use [active model serializer](https://github.com/rails-api/active_model_serializers) to help us convert our data into json objects. Let's add the gem to our Gemfile to make it available to us.
{% highlight Ruby %}
gem 'jquery-turbolinks' # works around js loading problems with turbolinks, add it to app manifest with require jquery.turbolinks
gem 'active_model_serializers'
{% endhighlight %}
then run `bundle install` so we can start to use it on the app.

We can now create a model serializer by running the generator in our terminal with `rails generate serializer Event`. This will create a new folder and ruby class file in app/serializers/event_serializer.rb.
{% highlight ruby %}
class EventSerializer < ActiveModel::Serializer

end
{% endhighlight %}
We can add `attributes` to the event serializer which rails will use to return whatever object attributes you'd like. For my event model, the attributes I'm interested follow:
{% highlight Ruby %}
class EventSerializer < ActiveModel::Serializer
  attributes :id, :name, :location, :readable_start_time, :readable_end_time, :description # an instance of my Event class has these attributes, defined the migrations
  has_one :organizer # has an organizer through active record associations
  has_many :users # has many users attending the event, also through AR associations
  has_many :comments # more associations
end
{% endhighlight %}
In order to properly serialize the object, I also need to create serializer classes for the `User` and `Comment` models with the rails generate command. Generating the user serializer is pretty straight forward, but the comments will need something more. The event has users attending it, but I also want event comments to also return the users who created the comment. I got it to return the correct values with this
{% highlight Ruby %}
class CommentSerializer < ActiveModel::Serializer
  attributes :id, :content
  attribute :user

  def user # I had to add this so that comments also returns the comment's author
    UserSerializer.new(object.user).attributes
  end
end
{% endhighlight %}

#Getting the Controller to Return JSON
Now to actually get our rails app to return a string formatted in json we need to change our events controller to spit back json from our events serializer.
{% highlight Ruby %}
#EventsController
def index
  @events = Event.all
  respond_to do |format|
    format.html
    format.json { render json: @events } # uses the event serializer to convert the object into json
  end
end
{% endhighlight %}
I added similar logic to the `show` action. Now if I start up my rails server and navigate to `localhost:3000/events.json`, i'll get something like this
{% highlight json %}
{
id: 8,
name: "Function-based methodical hub",
location: "Ondricka Port",
readable_start_time: "Friday, 27 May 2016 10:41 AM",
readable_end_time: "Friday, 27 May 2016 11:41 AM",
description: "Mlkshk hoodie franzen. Skateboard cliche pitchfork pop-up marfa plaid twee migas.",
organizer: {
  id: 8,
  name: "gerda"
},
users: [
  {
    id: 8,
    name: "gerda"
  }
],
comments: [
  {
    id: 8,
    content: "Direct trade pour-over goth chartreuse single-origin coffee.",
    user: {
      id: 8,
      name: "gerda"
    }
  }
]
},
// etc,
{% endhighlight %}

##Getting Views to Use JSON
We want our javascript to be run after the document loads and also between page loads without needing hard refreshes. I elected to go with a combination of document ready and page load functions to achieve this.
{% highlight javascript %}
// app/assets/javascript/event.js
$(document).ready(function() {

});
$(document).on('page:change', function() {
  attachListeners();
});
{% endhighlight %}
which calls my `attachListeners` function
{% highlight javascript %}
var attachListeners = function() {
  eventDetail();
  loadUsers();
  newEventListener();
  editEventListener();
}
var eventDetail = function() {
  $(".js-more").on("click", function() {
    var id = $(this).data("id");
    $.get("/events/" + id + ".json", function(event) {
      var dom = '';
      dom += "<p><strong>Begins:</strong> " + event.readable_start_time + "</p>";
      dom += "<p><strong>Ends: </strong> " + event.readable_end_time + "</p>";
      dom += "<p><strong>Description:</strong> " + event.description + "</p>";
      $("#event-" + id + "-detail").html(dom);
    });
    $(this).remove();
  });
}
// loadUsers uses similar logic
{% endhighlight %}
The above code hijacks the submit button on the rails form with `class="js-more"`. We can pass the value of the attribute `data-id=` number of the specific button that was pressed using the `this` keyword. Ajax gets called with `$.get()` with the arguments being a restful url that we build with the exact event object we want. When ajax receives a response (json from our serializer), we make some html elements by using js object notation and replace the html with the id `#event-id-detail`. Finally, it removes the button that was used to call the function.

#Creating/Updating Events
Well that was okay, but we really want to use js objects and maybe use some prototype methods instead of manually writing out the new html elements in our javascript file. I'll use Handlebars to help create html elements a little easier. Download [handlebars.js](http://handlebarsjs.com/) and drop it into your asset pipeline. Let's define our Event constructor
{% highlight javascript %}
function Event(attributes) {
  this.id = attributes.id;
  this.name = attributes.name;
  this.description = attributes.description;
  this.location = attributes.location;
  this.organizer = attributes.organizer;
  this.users = attributes.users;
  this.comments = attributes.comments;
  this.time_zone = attributes.time_zone;
  this.readable_start_time = attributes.readable_start_time;
  this.readable_end_time = attributes.readable_end_time;
  this.tag_ids = attributes.tag_ids;
}

Event.prototype.renderDiv = function () {
  return Event.template(this)
};
{% endhighlight %}
Now to build out an ajax function to convert to format that rails understands, then posts it the restful url, and define a function to do when it succeeds.
{% highlight javascript %}
var newEventListener = function () {
  $("form#new_event").on("submit", function(e) {
    e.preventDefault();   //hijacks our submit button so we can use ajax instead
    var $form = $(this);
    var action = $form.attr("action");
    var params = $form.serialize();    //takes our form and formats it
    $.ajax({
      url: action,
      method: 'POST',
      dataType: 'json',
      data: params
    })  // our ajax post method
    .success(function(json) {
      eventSuccess(json);
    })
    .error(function(response) {
      console.log('yu broke it?', response);
    });
  })
}

var eventSuccess = function(json) {
  var event = new Event(json);   // creates a new js object using our constructor
  var eventDiv = event.renderDiv();  // uses a event prototype method to build the html which uses handlebars
  $(".right-info").removeClass('form');
  $(".right-info").html(eventDiv);  //replaces the div with the html built using template
}
{% endhighlight %}
Now we can create a new object with `var newObj = new Event(attributes)`, and then call a prototype method renderDiv with (which we'll build later) with `newObj.renderDiv`. To do that, we'll need the power of handlebars' compiler. We'll need to build out a template that we can use when a new event is returned by our controller. In our new event views, we stub out what our html will look like in script tag directly in our view.

Super important, the `[[]]` really should be curly braces.
{% highlight html %}
<script id="event-template" type="text/x-handlebars-template">
  <h4>
    <strong><a href="/events/[[id]]">[[name]]</a> by <a href="/users/[[organizer.id]]">[[organizer.name]]</a></strong>
  </h4>
  <p><strong>Location:</strong> [[location]]</p>
  <p><strong>Begins:</strong> [[readable_start_time]]</p>
  <p><strong>Ends: </strong> [[readable_end_time]]</p>
  <p><strong>Description:</strong> [[description]]</p>
  <p><a class="button" href="/events/[[id]]/edit">Edit this event</a></p>
</script>
{% endhighlight %}
The attributes in between the {{}} handlebars will get interpolated as the event attributes. To use the compiler, in our event.js file, we add in the document ready function
{% highlight javascript %}
$(document).ready(function() {
  $(function(){
    Event.templateSource = $("#event-template").html();  //points to the script tag that has our template
    Event.template = Handlebars.compile(Event.templateSource);  //compile!
  })
});
{% endhighlight %}

Now our app should be using ajax to create and update an event and render views without reloading!
