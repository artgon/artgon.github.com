--- 
layout: post
title: Introducing Objectify for Scala
meta-description: Introducing the Objectify framework for structuring Scala web applications
published: false
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

- Policies
- Services
- Resolvers

#### Advantages

- Single Responsibility/Encapsulation
- DI w/o Singletons
- Easier to reason about application
  - Smaller, easier to understand classes
- Simpler, shorter tests
- No magic

I introduced the framework at a Scala meetup a little ago, and you can
find the slides [here](http://www.slideshare.net/artgon/scala-meetup-objectify-15072182). 
You can also fork it on Github [here](https://github.com/learndot/Objectify.scala).
