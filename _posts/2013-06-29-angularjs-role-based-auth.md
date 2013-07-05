--- 
layout: post
title: AngularJS -- Quick Route-Based Authentication
meta-description: A quick guide for setting up route-based authentication in AngularJS
#published: false
---

We've recently started on a new product at [work](http://www.learndot.com "Learndot")
using the [AngularJS framework](http://www.angularjs.org "AngularJS").
One of the most useful resources when starting out was the _#angularjs_
room on [freenode](http://freenode.net/ "Freenode"). There seems to be a really large, vibrant
community around AngularJS and that's always helpful when starting from
scratch on a new framework. One of the best resources in the room was
[David Mosher](https://twitter.com/dmosher "David Mosher"), who helped us get set up with
[Lineman](https://linemanjs.com "Lineman") -- an awesome build framework based on
[Grunt](http://gruntjs.com/ "Grunt") -- and pointed us to some of his [video tutorials](http://www.youtube.com/user/vidjadavemo). 
Honestly, these were a great resource for getting started and highly recommend watching them. The following tutorial is heavily inspired by 
the examples David provided. 

<!--more-->

#### Basic Routing

Before starting with authentication you'll need to have some routes.
Let's start with a logged out state, and a logged in state:

{% highlight javascript %}
angular.module("yourApp").config(function ($routeProvider) {

  $routeProvider.when('/login', {
    templateUrl: 'login.html',
    controller: 'LoginController'
    resolve: {
      //...
    }
  });

  $routeProvider.when('/app', {
    templateUrl: 'app.html',
    controller: 'AppController'
    resolve: {
      //...
    }
  });

});
{% endhighlight %}

In this simple example we have a state for logging in and another state
for seeing the app once logged in. In order to limit access to authenticated users we are going to have 
  to check their status before allowing them to view the _app_ state.

#### Adding Authentication

This is the real meat of the solution. They key is adding this new
function that watches the _$routeChangeStart_ event and reacts on it.
Once this event fires, we check if the route is clean (using some fancy
[underscore.js](http://underscorejs.org/ "UnderscoreJS")). It's easier to enumerate all the
routes that don't require rather than all the ones that do. Presumably,
there are significantly more that do.

{% highlight javascript %}
angular.module('yourApp').run(function ($rootScope, $location, AuthenticationService) {

  'use strict';

  // enumerate routes that don't need authentication
  var routesThatDontRequireAuth = ['/login'];

  // check if current location matches route  
  var routeClean = function (route) {
    return _.find(routesThatDontRequireAuth,
      function (noAuthRoute) {
        return _.str.startsWith(route, noAuthRoute);
      });
  };

  $rootScope.$on('$routeChangeStart', function (event, next, current) {
    // if route requires auth and user is not logged in
    if (!routeClean($location.url()) && !AuthenticationService.isLoggedIn()) {
      // redirect back to login
      $location.path('/login');
    }
  });
});
{% endhighlight %}

As you can see it's fairly intuitive. If the route needs to be
authenticated and the user is not logged in, then redirect them back to
the login page. This doesn't necessarily have to be a redirect, this can
be an API request to see if their session cookie is still active. You
can build on this and make it as sophisticated as you need -- but the
skeleton will probably remain the same.

#### Handling Failed Authentication from the API

There is another possibility, of course, in which the user is not
authenticated and initiates an action that results in API request. The
result will be a _401_ response and possibly some sort of authentication
exception. Handling this _401_ should be done as follows (all credit
goes to [David Mosher](https://twitter.com/dmosher "David Mosher") on this one):

{% highlight javascript %}
angular.module('yourApp').config(function ($httpProvider) {

  'use strict';

  var logsOutUserOn401 = ['$q', '$location', function ($q, $location) {
    var success = function (response) {
      return response;
    };

    var error = function (response) {
      if (response.status === 401) {
        //redirect them back to login page
        $location.path('/login');

        return $q.reject(response);
      } 
      else {
        return $q.reject(response);
      }
    };

    return function (promise) {
      return promise.then(success, error);
    };
  }];

  $httpProvider.responseInterceptors.push(logsOutUserOn401);
});
{% endhighlight %}

As you can see, we add an interceptor to capture the promises from the _$httpProvider_ and
validate them to make sure they are not _401_ errors. If they happen to
be _401_ then we send the user back to the login page.

#### Mixing in UI-Router

asdf
