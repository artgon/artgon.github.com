--- 
layout: post
title: "Fun Facts: Double Brace Initialization"
published: true
meta: 
  _edit_last: "2"
tags: 
- double brace initialization
- Fun facts
- Java
- Java 7
type: post
status: publish
---
Java has many detractors and one of the most common gripes is that you can't create a List, Map, Collection, etc. with an initial set of values. As with most other hate-speech directed at Java, this is not true! There is a handy feature colloquially called <em>double brace initialization</em>. It is used as follows:


{% highlight java %}
Map<String, Long> planetDistancesFromSunMap = new HashMap<String, Long>()
{ {
	put( "Mercury", 57900000L );
	put( "Venus", 108200000L );
	put( "Earth", 149600000L );
	put( "Mars", 227900000L );
	put( "Jupiter", 778300000L );
	put( "Saturn", 1427000000L );
	put( "Uranus", 2871000000L );
	put( "Neptune", 4497100000L );
} };
{% endhighlight %}

As you can see it's clean and particularly useful for the Map interface. The way it works is:

- The first pair of braces create an *anonymous class*
- The second pair of braces create an *instance initialization block*, which allows any supported Map functions to be run against it.


If you're thinking, "Geez wiz! This is too good to be true!!" Well you're not the only one, and as you may expect, there is a significant drawback to this method. Every time you use this method, the JVM creates a new class. This means that if you run this type of method in a loop of 1000 iterations, 1000 <strong>NEW </strong>.class files<strong> </strong>will be created. A poster on <a href="http://stackoverflow.com/questions/924285/efficiency-of-java-double-brace-initialization">StackOverflow</a> ran an experiment with this method vs. the standard filling the Collection after it is initialized method. The result, creating 3 ArrayLists, came to roughly 190ms for double brace initialization and 0ms for the standard way. This is not good.

Do not fear, there is yet hope for the future, in Java 7. One of the proposed changes is to add the ability to initialize Collections as they are declared:

{% highlight java %}
final List<Integer> piDigits = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5, 9];

final Set<Integer> primes = { 2, 7, 31, 127, 8191, 131071, 524287 };

final Map<Integer, String> platonicSolids = { 4 : "tetrahedron", 6 : "cube", 8 : "octahedron", 12 : "dodecahedron", 20 : "icosahedron" };
{% endhighlight %}

There's a saying around the office that goes something like, "Thanks Josh Bloch!" These new alterations to the Collections package definitely fall into the that category. Looking forward to Java 7!
