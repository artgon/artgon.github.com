---
layout: post
title: "Scala: the Case for Correctness"
meta-description: Choosing Scala should not just be for brevity or speed -- it's for correctness
published: false
---

Recently being thrown back into the world of Java development has made
me evaluate my original conviction in Scala. Having worked exclusively
with Scala for the past two years, I'm acutely aware of all it's various 
pain points -- and it has many. I've also enjoyed some of Java's advantages
over Scala, like faster compilation times and mature tooling.

That being said, I still think Scala is the better option and the key reason
why is correctness. 

<!--more-->

The main selling points for Scala have generally been the 
syntactic sugar and the functional programming paradigms, but correctness is 
so much more important. The key features being: _immutability by default_, 
_easy concurrency_ and _no null values_.[^1]


#### Immutability by Default


_Case 1: Labelling all variables as final_

Example:

// TODO

_Case 2: Utility classes require boiler plate_

Example:

// TODO

_Case 3: Try/catch flows with awkward return points_

Example:

// TODO

#### Easy Concurrency

Easy concurrency sounds like the famous last words of a cocky 
brogrammer. What I mean by the word "easy" is that the it's easy
to reason about. The concurrency method encouraged is the actor 
model. The biggest advantage of the actor model is not one of 
performance or clustering -- it's correctness. Having a guarantee
that the code inside an actor will only ever be run by one thread
makes the code incredibly easy to reason about. 

Example:

// TODO

#### No Null Values

The most common exception in Java application is the NullPointerException. There
is a weird separation of objects and primitives. Primitives cannot be set to
null, while objects get set to null all the time. I would argue that the majority
of Java programmers still use null types as a return knowingly, as just another
strategy for control flow.

JDK 8 has alleviated some of this pain with an _Optional<T>_ class that allows you
to wrap null types and unwrap them safely. Unfortunately, the _Optional_ object can 
be set to null!

Example:

// TODO


[^1]: When calling Java code you may get null values but you cannot set a value to null in Scala code. 
