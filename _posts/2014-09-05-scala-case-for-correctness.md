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

iLorem Ipsum is simply dummy text of the printing and typesetting
industry. Lorem Ipsum has been the industry's standard dummy text ever
since the 1500s, when an unknown printer took a galley of type and
scrambled it to make a type specimen book. It has survived not only five
centuries, but also the leap into electronic typesetting, remaining
essentially unchanged. It was popularised in the 1960s with the release
of Letraset sheets containing Lorem Ipsum passages, and more recently
with desktop publishing software like Aldus PageMaker including versions
of Lorem Ipsum.

#### Easy Concurrency

Lorem Ipsum is simply dummy text of the printing and typesetting
industry. Lorem Ipsum has been the industry's standard dummy text ever
since the 1500s, when an unknown printer took a galley of type and
scrambled it to make a type specimen book. It has survived not only five
centuries, but also the leap into electronic typesetting, remaining
essentially unchanged. It was popularised in the 1960s with the release
of Letraset sheets containing Lorem Ipsum passages, and more recently
with desktop publishing software like Aldus PageMaker including versions
of Lorem Ipsum.

#### No Null Values

Lorem Ipsum is simply dummy text of the printing and typesetting
industry. Lorem Ipsum has been the industry's standard dummy text ever
since the 1500s, when an unknown printer took a galley of type and
scrambled it to make a type specimen book. It has survived not only five
centuries, but also the leap into electronic typesetting, remaining
essentially unchanged. It was popularised in the 1960s with the release
of Letraset sheets containing Lorem Ipsum passages, and more recently
with desktop publishing software like Aldus PageMaker including versions
of Lorem Ipsum.

[^1]: When calling Java code you may get null values but you cannot set a value to null in Scala code. 
