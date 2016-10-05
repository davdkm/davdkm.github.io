---
layout: post
title:  "Heroku Minification Woes"
categories:
---
Recently when deploying one of [my projects](https://whats-good.herokuapp.com/) to heroku, I came across a problem. Half of my site wasn't working. Specifically, all the links to my site that connected to the backend end were not being routed properly. It took me an obscenely long amount of time and a lot of trial and error to finally arrive at the actual problem.

####Minification
I built my project to be a single page app with an AngularJS front end with a Rails API back end. Angular controllers under go something called minification when the website is deployed in a production environment. This helps to reduce file sizes so resources load faster in live environments by renaming long variable names. So the following
{% highlight js %}
function add(numberOne, numberTwo) {
    return numberOne + numberTwo;
}
{% endhighlight %}
would become
{% highlight js %}
function add(a,b){return a+b;}
{% endhighlight %}
That's awesome! But what's not awesome is when you deploy your site and forget about annotating dependency injection. The process of shortening variable names isn't so awesome when your controllers are dependent on other controllers, services, directives, and components. All of those names get minified too, but you probably don't have any angular providers called `a, b, c` but more like `PostsController, UserService, $http`. After minification, angular will look for `a, b, c` and your controllers start to break. In my case, I had some `resolve` data that needed to be injected in on state changes from ui.router. So my `app.js` file looks something like
{% highlight js %}
angular
  .module('app', [
    'ui.router',
    'templates'
  ])
  .config([
    '$httpProvider',
    '$stateProvider',
    '$urlRouterProvider',
    function ($httpProvider, $stateProvider, $urlRouterProvider) {
      $stateProvider
        .state('home', {
          url: '/home',
          templateUrl: 'home/home.html',
          controller: 'HomeController as home'
        })
        .state('home.posts', {
          url: '/posts',
          templateUrl: 'home/posts/index.html',
          controller: 'PostsController as posts',
          resolve: {
            posts: function (PostsService) {
              return PostsService.getPosts(); # injecting PostsService but will get minified to not PostsService
            }
            categories: function (CategoriesService) {
              return CategoriesService.getCategories();
            }
          }
        })

      $urlRouterProvider.otherwise('home');
    }
  ])
{% endhighlight %}
and my PostsController could look something like
{% highlight js %}
function PostsController(posts, categories, $filter) { # posts would get minified
  var ctrl = this;

  ctrl.posts = posts.data;
  ctrl.categories = categories.data;

}

angular
  .module('app')
  .controller('PostsController', PostsController)
{% endhighlight %}
####The Solution: Safe Dependency Injection
The solution to get around this is to properly use the `$inject` function to tell angular that we are expecting our dependencies in a specific order. The order here is all that matters, we can name it whatever we'd like i.e.
{% highlight js %}
function PostsController(random, variables, foobar) {
  var ctrl = this;

  ctrl.posts = posts.data;
  ctrl.categories = categories.data;

}

PostsController.$inject = ['posts', 'categories', '$filter'];
angular
  .module('app')
  .controller('PostsController', PostsController)
{% endhighlight %}
Notice the line `PostsController.$inject = ['posts', 'categories', '$filter'];`. The array that gets injected and matches the order of dependencies in the line `function PostsController(random, variables, foobar)`. It doesn't matter what we name elements here, angular just knows that that the first element injected `'posts'` will correspond to the first argument `'random'` and `'categories'` will correspond to `'variables'` and so on.

On the `app.js` side, annotation would look something like
{% highlight js %}
resolve: {
            posts: ['PostsService', function (PostsService) { # PostsService getting injected
              return PostsService.getPosts();
            }],
            categories: ['CategoriesService', function (CategoriesService) { # CategoriesService getting injected
              return CategoriesService.getCategories();
            }]
          }
{% endhighlight %}
Once I injected all my dependencies across my various controllers/components/directives/services, site started routing properly like it did in development! The lesson here? If you plan to deploy a production version of your angular site, make sure you properly annotate your dependencies!
Or just use [ng-annotate](https://github.com/olov/ng-annotate).
