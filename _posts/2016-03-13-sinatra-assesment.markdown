---
layout: post
title:  "Sinatra Assesment - Aks Me!"
categories:
---

So, for my Sinatra assessment, I was tasked to create a website that showcased our ability to utilize CRUD (Create, Read, Update, Delete/Destroy).  After mulling it over a bit, I wanted to keep it simple but still be something that I might enjoy building. My idea was to build Aks Me!, a question and answer site.

I decided to tackle this project a little differently than my previous CLI project, in that I actually wanted to map out what I wanted to accomplish first!  That meant thinking up my MVC model and ORM before I wrote a single line of code.  I wanted Users to be able to post many Questions and Comments. Users belong to a Group. A Question and a Comment belong to a User. Question has many Comments. A Group has many Users, and has many Questions and Comments through Users. I recently learned about ActiveRecord Aliasing, so I also want to throw some aliased associations in there as well.

I also wanted to utilize TDD (Test Driven Development) to help me along as well, which meant writing tests in Rspec to make sure that the individual parts of site are working as intended.

The site started with a file structure scaffold that I copied from previous lesson work.  I created the app dir that would hold the application controller, models and views needed for routes and object relational mapping. The config/environment file would define and load all the files required for my project. The db folder would hold my ActiveRecord migrations and database. The spec folder would hold all my tests to be written and checked. The config.ru file would load my controllers to actually start a server for my site. The Gemfile would list all the ruby gems I would need during development and production. Finally, the Rakefile gives me the tools to create my database and a console to play in.

Thinking about my domain more, I came to the conclusion that adding another has many to many relation with the groups was getting a little too complicated, so I ditched the group model to keep things simple.  Questions are created by Users, which I aliased as authors, and Comments are created by Users, aliased as commenters.  The associations were added to the model classes which inherit from ActiveRecord::Base.

Time to write a couple of tests in rspec! I wanted to keep the model tests pretty simple and just wanted to test that each object could be associated with other class objects using the associations made in ActiveRecord. Here's a snippet of my associations

{% highlight ruby %}
class Comment < ActiveRecord::Base
  belongs_to :commenter,
             :class_name => "User",
             :foreign_key => "user_id"

  belongs_to :question
end

class Question < ActiveRecord::Base
  belongs_to :author,
             :class_name => "User"

  has_many :comments
  has_many :commeters,
           :through => :comments,
           :source => :commenter
end

class User < ActiveRecord::Base
  has_many :comments
  has_many :commented_posts,
           :through => :comments,
           :source => :question

  has_many :authored_questions,
           :class_name => "Question",
           :foreign_key => "author_id"

  has_secure_password
end

{% endhighlight %}

Now that I have my associations built that tests passing, it's time to make the much easier task of building routes and views.  I went with almost now fluff on the views, I just wanted it to be functional. With the routes, I wanted users to only be able to use the site while logged in, and defined a method #redirect_if_not_logged_in to check params[:id] on requests.

For my CRUD routes, I used patch and delete with Rack::Override in the config.ru for updating and deleting questions and comments. Post requests creates new questions, users, and comments, and get renders everything. For a look at my source code, here's a link to my repo on github

https://github.com/davdkm/aks-me
