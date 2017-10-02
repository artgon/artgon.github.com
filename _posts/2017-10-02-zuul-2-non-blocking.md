---
layout: post
title: "Zuul 2.0: The Journey to Non-Blocking"
meta-description: How we moved Zuul - Netflix's API gateway - to Netty
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/2oXqbLhMS_A" frameborder="0" allowfullscreen></iframe>

This is a talk I gave at [Strange Loop](https://www.thestrangeloop.com/) about how we rebuilt [Zuul](https://github.com/netflix/zuul/) on
Netty and RxJava. The main themes of the talk are: what it took to
rebuild it, what it's been like operating it in production and how to make the
right decision about going blocking vs non-blocking. Here is the
abstract:

>Zuul 2.0 is the latest iteration of the gateway application fronting Netflix's API and underlying microservices. It was borne of a need to handle an ever-growing amount of traffic and a similarly ever-growing number of microservices to front. We largely rebuilt Zuul from the ground up, leveraging the Netty framework for its high-performance, non-blocking, event-loop architecture and combined it with RxJava interfaces for a simpler programming model in our filters. Despite our lofty performance expectations for this project we ended up with some mixed results and can definitively say that going asynchronous, non-blocking is not a panacea.
>
>This talk will be a deep dive into how we progressively refactored and rebuilt Zuul from a blocking Tomcat application to a non-blocking Netty application and the results we have seen from running it in production over the last year. Specifically, I will review the journey of combining the RxJava and Netty frameworks in rebuilding Zuul, discuss how to make the decision on whether your systems need to be non-blocking, and provide the good and bad with some real-world scenarios.

