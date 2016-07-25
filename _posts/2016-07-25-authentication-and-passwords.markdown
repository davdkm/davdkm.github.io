---
layout: post
title:  "The Importance of Strong Passwords"
categories:
---

Information security is more important than ever. It seems like there's a new major security breach every few months where thousands if not millions of users have their personal identifiable information stolen. And that's just the ones the public know about.

As we already know, one of the best ways to keep passwords secure for our web applications is to never have them in the first place! OmniAuth was born from this need, but if we want to take a stab at handling our own authentication, we want to prevent users from creating accounts with passwords that are easy to crack with brute force attacks.

If you've worked through the lessons on [Learn.co](http://learn.co) from the FLATIRON SCHOOL, you probably know that storing passwords in plain text is a huge mistake, and that passwords should be hashed with a salt (`password_digest`) using the bcrypt gem and adding `has_secure_password` to your User model.

Well that's all gravy but what if some sweet naive little user decides that they want their password to be just 'password' or '1234'. What a terrible idea, if only there was a way to tell them what trouble they are inviting into their digital life. Thankfully, there are validations in Rails!

Validations help keep data in our models formatted properly to help prevent shenanigans occurring in our app. We can impose a restriction on our passwords to have a minimum length, by adding the following line into our User model.
{% highlight Ruby %}
#app/models/user.rb
validates :password, :presence => true,
                       :length => {:within => 8..100},
                       :on => :create
 {% endhighlight %}
 Now our user won't be created and saved to the database unless they provide a password that is at least 8 to 40 characters long. But our user of summer can still create a password of 'password'. We don't want that to happen either, so let's impose even more restrictions! Credit for the password format goes to the contributors from this [stackoveflow post](https://stackoverflow.com/questions/5123972/ruby-on-rails-password-validation)
 {% highlight ruby %}
 PASSWORD_FORMAT = /\A
  (?=.{8,32})          # Must contain 8 or more characters
  (?=.*\d)           # Must contain a digit
  (?=.*[a-z])        # Must contain a lower case character
  (?=.*[A-Z])        # Must contain an upper case character
  (?=.*[[:^alnum:]]) # Must contain a symbol
  /x                 

  validates :password,
              format: {
              with: PASSWORD_FORMAT,
              message: "please provide a strong password"
              }
 {% endhighlight %}
Let's dig into the magic here. `format: {with: PASSWORD_FORMAT, message: 'something'}` is running a check on the password attribute of our user. If the password doesn't conform with the regex definge in `PASSWORD_FORMAT` constant defined above, the validation will fail, `"please provide a strong password"` will be added to the error messages, and this record won't be saved to the database.

We can take a test drive for this code by starting up `rails c`. I've already got a User model created with Devise and added the above password validation. I'll create a new user in my console
{% highlight sql %}
> user.password = 'password'
 => "password"
> user.valid?
  User Exists (0.9ms)  SELECT  1 AS one FROM "users" WHERE ("users"."email" = 'test1@example.com' AND "users"."id" != 1) LIMIT 1
 => false
> user.errors.full_messages
 => ["Password please provide a strong password"]
> user.save
   (3.0ms)  BEGIN
  User Exists (0.5ms)  SELECT  1 AS one FROM "users" WHERE ("users"."email" = 'test1@example.com' AND "users"."id" != 1) LIMIT 1
   (0.3ms)  ROLLBACK
 => false

{% endhighlight %}
You can see from my rails console output, that I wasn't able to save the user because of I didn't provide a strong password. Let's try it again, this time with a stronger password.
{% highlight sql %}
user.password = 'Pa$$w0rd2016'
 => "Pa$$w0rd2016"
> user.valid?
  User Exists (0.7ms)  SELECT  1 AS one FROM "users" WHERE ("users"."email" = 'test1@example.com' AND "users"."id" != 1) LIMIT 1
 => true
> user.errors.full_messages
 => []
> user.save
   (0.3ms)  BEGIN
  User Exists (0.8ms)  SELECT  1 AS one FROM "users" WHERE ("users"."email" = 'test1@example.com' AND "users"."id" != 1) LIMIT 1
  SQL (0.4ms)  UPDATE "users" SET "encrypted_password" = $1, "updated_at" = $2 WHERE "users"."id" = $3  [["encrypted_password", "
  etc....
   (0.6ms)  COMMIT
 => true

{% endhighlight %}
Now our user is properly validated and is able so save to the database! Awesome! Your users information is now in slightly better hands!
