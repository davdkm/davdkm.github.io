---
layout: post
title:  "Deploying A Simple Rails App to Heroku"
categories:
---

#The Problem
When I'm developing a rails website, my database is usually using SQLite3. It is pretty easy to get up and running and usually is enough for developing small scale apps. However, SQLite3 is not really suited for production environments, i.e. if you want this website out in the world wild web, you going to want a production centric database like Postgres or MySQL. For a small fun website on Heroku, you'll want to convert your app to use Postgres.

I had already connected my app to Heroku, but couldn't get it to deploy because it wasn't cool with using SQLite3.  Here's a overview of my journey to get it working.

#Installing Postgres
If you are developing on a Mac, you already have SQLite3 installed. If you don't already have Postgres installed, you'll want to download and install from the [Postgres site](http://postgresapp.com/) if you want to develop with it. The Postgres.app makes installation pretty easy.

Next you'll want to configure your app to use Postgres instead of SQLite. Find the line in your Gemfile in your site's root directory and remove it.
{% highlight Ruby %}
gem 'sqlite3'   #Remove this line
{% endhighlight %}
Replace it with
{% highlight Ruby %}
gem 'pg'    #Add this line
{% endhighlight %}
Then run `bundle install`. Now we can start configuring the database to use Postgres.

#Configuring the Database
You will need to convert your config/database.yml. Mine looked like this:
{% highlight Ruby %}
default: &default
    adapter: sqlite3
    pool: 5
    timeout: 5000

  development:
    <<: *default
    database: db/development.sqlite3

  # Warning: The database defined as "test" will be erased and
  # re-generated from your development database when you run "rake".
  # Do not set this db to the same as development or production.
  test:
    <<: *default
    database: db/test.sqlite3

  production:
    <<: *default
    database: db/production.sqlite3
{% endhighlight* %}

Changed it to
{% highlight Ruby %}
default: &default
  adapter: postgresql
  pool: 5
  timeout: 5000

development:
  <<: *default
  database: db/development

# Warning: The database defined as "test" will be erased and
# re-generated from your development database when you run "rake".
# Do not set this db to the same as development or production.
test:
  <<: *default
  database: db/test

production:
  <<: *default
  database: db/production
{% endhighlight* %}

Not having deployed a live site on anything, I was having trouble figuring why Heroku was deploying without errors, but everything was failing to load. I forgot to migrate! Since the site is using Heroku as a platform, I needed to prepend the heroku client commands to the usual rake commands
{% highlight shell %}
heroku run rake db:create
heroku run rake db:migrate
{% endhighlight %}

Visit your site on Heroku and happy developing!
