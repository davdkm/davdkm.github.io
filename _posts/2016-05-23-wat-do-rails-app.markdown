---
layout: post
title:  "Wat Do?! A Simple Rails App"
categories:
---

##Intro:

This post is a journal for the process of creating a website in Rails. Some degree of familiarity with Ruby and Rails by the reader is assumed. Our objective is to create a website that utilizes authentication and authorization (and also allow authentication from an outside service). It will also utilize nested resources and routes. For this project, we're going to build a site that can manage events. We want our users to be able to create events that other users can sign up to attend, and tag their own events with tag categories. Some code will be intentionally left out but my full code can be found on my repo, linked at the bottom of this post. Let's get started!

##Getting Started:

First, we create our models and migrations. I use regular old rails generators to create the files with

{% highlight shell %}
  rails generate model Event
{% endhighlight %}

and fill in the changes needed afterwards.

{% highlight ruby %}
  class CreateEvents < ActiveRecord::Migration
    def change
      create_table :events do |t|
        t.string :name
        t.string :location
        t.text :description
        t.datetime :start_time
        t.datetime :end_time
        t.string :time_zone
        t.references :user, index: true

        t.timestamps null: false
      end
    end
  end
{% endhighlight %}

Continue to generate and write out migrations for the `User`, `Tag` and `Comment` models. Our comment model will also serve as a join table that can have an attribute `:content` to hold the text for a comment in addition to tying a user_id to an event. We'll also want a second join table for `Event`s and `Tag`s and a model for `EventTag`, since `Event`s can have many `Tag`s, and `Tag`s can have many `Event`s.

{% highlight ruby %}
  class CreateComments < ActiveRecord::Migration
    def change
      create_table :comments do |t|
        t.text :content
        t.references :user, index: true
        t.references :event, index: true

        t.timestamps null: false
      end
    end
  end

  # add another migration file
  class CreateJoinTableEventTags < ActiveRecord::Migration
    def change
      create_table :event_tags do |t|
        t.integer :event_id
        t.integer :tag_id
      end
    end
  end

{% endhighlight %}

And also another join table for the users and events tables so we can build out the feature to allow users to 'attend' events. We'll call this join table `Schedule`s.

{% highlight ruby %}
  class CreateSchedules < ActiveRecord::Migration
    def change
      create_table :schedules do |t|
        t.integer :event_id  # works the same as t.references :event, index: true
        t.integer :user_id

        t.timestamps null: false
      end
    end
  end
{% endhighlight %}

Now to add Devise to handle our authentication needs. Include the devise gem in our Gemfile. We'll also include Pundit and Omniauth while we're at it to handle roles and outside authentication service. I've decided to use GitHub for this site.  

{% highlight ruby %}
  gem 'devise'
  gem 'pundit'
  gem 'omniauth'
  gem 'omniauth-github'
  gem 'dot-env'
{% endhighlight %}

Run bundle to install the gems

{% highlight shell %}
  bundle install
{% endhighlight %}

GitHub omniauth requires that we get `GITHUB_KEY` and `GITHUB_SECRET` keys from github for authentication.  You can obtain these directly from github by logging in or creating an account and creating a new application. Get the keys from GitHub and create a new a file the site's root directory called .env (note: this is a file with no name that has an extension of .env). The dot-env gem included above will allow rails to grab the values from the .env file.

{% highlight shell %}
  touch .env
{% endhighlight %}

Now add the keys to the .env file, replacing `<YOUR_KEY>` AND `<YOUR_SECRET>` with your values from GitHub (or whatever provider you chose)

{% highlight ruby %}
  GITHUB_KEY=<YOUR_KEY>
  GITHUB_SECRET=<YOUR_SECRET>
{% endhighlight %}

Then add Devise to Users so that it can do it's magic

{% highlight shell %}
  rails generate devise:install Users
{% endhighlight %}

Then to add Omniauth capability to our Users

{% highlight shell %}
  rails generate migration AddOmniauthToUsers provider:string uid:string
{% endhighlight %}

Add roles to our User model. Normal users will be create by default. Admins can modify everything but will not be able to be created from sign up.

{% highlight ruby %}
  class AddRolesToUsers < ActiveRecord::Migration
    def change
      add_column :users, :role, :integer, default: 0
    end
  end
{% endhighlight %}

We'll also need to make changes to our config/devise.rb file to include a config line that will allow Omniauth to work with Devise. We add this line to the devise.rb file instead of omniauth config since we are using Devise for authentication.

{% highlight ruby %}
  # add this line inside the Devise.setup block to allow omniauth with github
  # scope: 'user' will allow our site to use the info in the user hash sent back from github
  Devise.setup do |config|
    config.omniauth :github, ENV['GITHUB_KEY'], ENV['GITHUB_SECRET'], scope: 'user'
  end
{% endhighlight %}

Then we want to include our migrations to create our tables

{% highlight shell %}
  rake db:migrate
  # seed our database with fake data if you have some seed data
  rake db:seed
{% endhighlight %}

Now to make sure that our routes are setup properly. Remember we want to make use of nested routes for our comments, since a comment belongs to a user. In addition to setting up a Static controller to handle our home page, we'll also want to create a new controller to handle omniauth callbacks for omniauth sign ins. This controller will handle all the data that is sent back from github (callbacks)

{% highlight ruby %}
  Rails.application.routes.draw do
    resources :tags
    devise_for :users, :controllers => { :omniauth_callbacks => "users/omniauth_callbacks" }

    resources :users do
      resources :comments
    end
    resources :schedules
    resources :events

    root 'static#home'
  end
{% endhighlight %}

Create the omniauth callbacks controller. I've decided to create a app/controllers/users folder and stick a newly created omniauth_callbacks_controller.rb which inherits from the Devise's omniauth controller.

{% highlight ruby %}
  # app/controllers/users/omniauth_callbacks_controller.rb
  class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController

    def github
      # You need to implement the method below in your model (e.g. app/models/user.rb)
      @user = User.from_omniauth(request.env["omniauth.auth"])
      if @user.persisted?
        sign_in_and_redirect @user
        set_flash_message(:notice, :success, :kind => "Github") if is_navigational_format?
      else
        session["devise.github_data"] = request.env["omniauth.auth"]
        redirect_to new_user_session_path, flash: {error: 'There was an error.'}
      end
    end

    # handle uh oh behavior so your site doesn't freak out when authentication fails
    def failure
      redirect_to root_path
    end

end
{% endhighlight %}

Modify the User.rb model file to look for or create users from GitHub. You might want also want to add a couple of simple validations along with defining pundit roles with enum and devise options to allow omniauth. The below also has some ActiveRecord association macros needed for interacting with other models.

{% highlight ruby %}
  class User < ActiveRecord::Base

    devise :database_authenticatable, :registerable,
           :recoverable, :rememberable, :trackable, :validatable, :omniauthable, :omniauth_providers => [:github]

    has_many :schedules
    has_many :events, through: :schedules#, source: :event
    has_many :locations, through: :events

    has_many :comments

    validates :name, presence: true
    validates :email, uniqueness: true


    enum role: [:normal, :moderator, :admin]

    def self.from_omniauth(auth)
      where(provider: auth.provider, uid: auth.uid).first_or_create do |user|
        user.email = auth.info.email
        user.password = Devise.friendly_token[0,20]
        user.name = auth.info.name || user.auth.info.username
      end
    end

  end

{% endhighlight %}

##Building MVC:

Now to build out the rest of the models, views, and controllers. We start by building out the rest of models with the proper association macros

{% highlight ruby %}
  class Schedule < ActiveRecord::Base
    belongs_to :user
    belongs_to :event
  end

  class Tag < ActiveRecord::Base
    has_many :event_tags
    has_many :events, through: :event_tags
  end

  class EventTag < ActiveRecord::Base
    belongs_to :event
    belongs_to :tag
  end

  class Comment < ActiveRecord::Base
    belongs_to :user
    belongs_to :event
  end
{% endhighlight %}

The event class will have a bit more code to help validate attributes and also make the time objects for the event more readable in our views

{% highlight ruby %}
  class Event < ActiveRecord::Base
    belongs_to :organizer, :foreign_key => "user_id", :class_name => 'User'
    has_many :schedules
    has_many :users, through: :schedules#, source: :user
    has_many :comments
    has_many :event_tags
    has_many :tags, through: :event_tags
    accepts_nested_attributes_for :tags

    validates_presence_of :description, :name, :location, :start_time, :end_time
    validate :event_cannot_start_in_the_past, :event_cannot_end_before_start_time

    def start_time
      super.in_time_zone(time_zone) if time_zone
    end

    def end_time
      super.in_time_zone(time_zone) if time_zone
    end

    def self.sort_by_start_time
      self.order('start_time')
    end

    def readable_start_time
      self.start_time.strftime("%A, %d %b %Y %l:%M %p")
    end

    def readable_end_time
      self.end_time.strftime("%A, %d %b %Y %l:%M %p")
    end

    def tags_attributes=(tag_attributes)
      tag_attributes.values.each do |tag_attribute|
        tag = Tag.find_or_create_by(tag_attribute) if tag_attribute[:name].present?
        self.tags << tag if tag
      end
    end

    def event_cannot_start_in_the_past
      if self.start_time.present? && self.start_time < DateTime.now
        errors.add(:start_time, "Event cannot start in the past")
      end
    end

    def event_cannot_end_before_start_time
      if self.end_time.present? && self.start_time.present? && self.end_time < self.start_time
        errors.add(:start_time, "Event cannot end before it starts.")
      end
    end
  end
{% endhighlight %}

#Controllers:
We want anyone who comes to the site to be able to view our site, but only users who are signed in to be able to create and go to events. We can `authorize` actions by defining policies. We can use Pundit to craft these policies in a `app/policies/resource_policy.rb` file. Here's one for our `Event`.

{% highlight ruby %}
  class EventPolicy < ApplicationPolicy

    def create?
      user.admin? || user.moderator? || user.normal?
    end

    def update?
      user.admin? || user.moderator? || record.try(:organizer) == user
    end

    def destroy?
      user.admin? || user.moderator? || record.try(:organizer) == user
    end

  end

{% endhighlight %}

Now, only users who created the `record`, in this case our event, can make changes or create events, while moderators and admins who are signed in can do the same to any event. Now our controller actions will need a way to call on these policies. We can do that with `authorize`

{% highlight ruby %}
  class EventsController < ApplicationController

    def create
      Time.zone = params[:event][:time_zone]
      @event = current_user.events.build(event_params)
      @event.organizer = current_user
      if authorize @event
        if @event.save
          redirect_to event_path(@event)
        else
          render :new, flash: {error: 'Uh oh'}
        end
      end
    end

    def edit
      @event = Event.find(params[:id])
      @users = @event.users
      @user = current_user
    end

    def update
      @event = Event.find(params[:id])
      Time.zone = @event.time_zone
      if authorize @event
        if @event.update(event_params)
          redirect_to event_path(@event)
        else
          render :edit
        end
      end
    end

    def destroy
      @event = Event.find(params[:id])
      authorize @event
      if @event.delete
        redirect_to events_path, flash: {message: 'Event was succussfelly removed.'}
      else
        render :show
      end
    end

  end
{% endhighlight %}

#Views:
To finish up this post, I'll include the code for the event views, which utilizes '<%= render @event %>'
{% highlight ERB %}
<%= render @event %>

<h4><%= 'Attender'.pluralize(@users.count) %> of this Event:</h4>

<p>
  <% @users.map do |attender| %>
    <%= link_to (attender.name || attender.email), user_path(attender), class: "button" %>
  <% end %>
</p>

<p>
  <% if user_signed_in? %>
    <%= form_tag schedules_path, method: 'POST' do %>
      <%= hidden_field_tag :user_id, current_user.id %>
      <%= hidden_field_tag :event_id, @event.id %>
      <%= submit_tag "Go to this event", class: 'button confirm' %>
    <% end %>
    <%=  button_to "Not going anymore", {:controller => :schedules, :action => 'destroy', :id => @event.id}, :method => :delete, class: 'button cancel' unless @users.nil? %>
  <% end %>
</p>
</div>

<%= render 'comments/comments' %>
<%= render 'tags/tags' %>

{% endhighlight %}

When `<%= render @events %>` is called, Rails will automagically look in the `app/views/events` folder for any `_event.hmtl.erb` file. It will pass each element in the `@events` array and perform the code in the block in the file, whether it's a single event in a `#show` action or many events in `#index` action.
`app/views/events/_event.html.erb`
{% highlight ERB %}
<div class="right-info">
  <h4>
    <strong><%= link_to event.name, event_path(event) %> by <%= link_to event.organizer.name, user_path(event.organizer) %></strong>
  </h4>
  <p>
    <strong>Location:</strong> <%= event.location %>
  </p>
  <p>
    <strong>Begins:</strong> <%= event.readable_start_time %>
  </p>
  <p>
    <strong>Ends:</strong> <%= event.readable_end_time %>
  </p>
  <p>
    <strong>Description:</strong> <%= event.description %>
  </p>
  <p>
    <%= link_to 'Edit this event', edit_event_path(event), class: 'button' %>
  </p>
</div>
{% endhighlight %}

##Conclusion:
This has been a quick guide to creating a Rails app with authentication and authorization. This post is by no means extensive or complete, but you can find my full repo on [github](https://github.com/davdkm/wat-do).
