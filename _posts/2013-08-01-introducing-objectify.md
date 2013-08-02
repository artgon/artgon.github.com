--- 
layout: post
title: Introducing Objectify for Scala
meta-description: Introducing the Objectify framework for structuring Scala web applications
---

It's a framework we developed almost a year ago, that has been running
in production for over 10 months. It's so simple and effective that we
have barely had to make any commits to it since. The framework I'm
referring to is [Objectify.scala](https://github.com/learndot/Objectify.scala).

Objectify was inspired by [James Golick's](http://jamesgolick.com)
framework, [Objectify.rb](https://github.com/bitlove/objectify). He
[introduced](http://jamesgolick.com/2012/5/22/objectify-a-better-way-to-build-rails-applications.html) 
it by describing why Rails needs to move away from fat models, and create a service layer. 
In a sense this a move towards the Java/J2EE way of creating
applications and it's something both [Joe](http://joegaudet.com) and I
agreed with. 

When looking at our own codebase at the time, which was built on
[Scalatra](http://www.scalatra.org/) (a Scala port of [Sinatra](http://www.sinatrarb.com/)), there was definitely a lack
of organization and structure. Since the application had become fairly
complex, the way the Scalatra routes were separated into files and
policies enforced, was mainly using inheritance and traits. We didn't
feel that this was a scalable solution for our architecure going
forward.

We decided to create a light-weight framework, based on Objectify.rb, to
help us structure our Scalatra application. In particular, the key
features that we developed to do this were dependency injection and
policy management.

<!--more-->

#### Components

There are essentially four types of components for implemented the
front-end of your application with Objectify: policies, services,
responders and resolvers.

__Policies__ are simple -- they implemented a method that looks at the
request and returns true or false. If the user is allowed to proceed or
not. If the user is not allowed to proceed, then they are forwarded to a
PolicyResponder (e.g. return a 404), otherwise the service is executed.

__Services__ are the meat of your project. They do the heavy lifting,
whether it be returning a data set or creating/updating objects in the
database. When a service is done executing it returns some sort of
result to be passed on to its ServiceResponder.

__Responders__ are responsible for taking a result and serializing into
whatever format the service outputs. For example it can take a
Scala list of users and convert it to a JSON array of user objects.
There are two types of responders, one for failed policies and one for
services.

__Resolvers__ are the glue that wires everything together. They allow
objects to be injected into all three of the above types. The most
common way to use them, is for extracting values from the request and
populating them into objects. For example, we use resolvers to grab the
ID in the path for a PUT or DELETE. We also use them to parse the JSON
in a POST body into a Scala DTO.

#### Advantages

- Single Responsibility/Encapsulation
- DI w/o Singletons
- Easier to reason about application
  - Smaller, easier to understand classes
- Simpler, shorter tests
- No magic

#### Details

I introduced the framework at a Scala meetup a little ago, and you can
find the slides [here](http://www.slideshare.net/artgon/scala-meetup-objectify-15072182). 
You can also fork it on Github [here](https://github.com/learndot/Objectify.scala).
