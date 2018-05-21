---
layout: post
title: "Open Sourcing Zuul 2"
meta-description: Zuul 2 open source announcement
---

Today we officially announced the release of [Zuul 2](https://medium.com/netflix-techblog/open-sourcing-zuul-2-82ea476cb2b3). It's an
exciting day for the team and the blog post linked hightlights some of the other work we've been doing.

Here's a summary of the major features included in the open source
release:

>Today we are releasing many core features. Here are the ones we’re most excited about:
>
>**Server Protocols**
* *HTTP/2* — full server support for inbound HTTP/2 connections
* *Mutual TLS* — allow for running Zuul in more secure scenarios
>
>**Resiliency Features**
* *Adaptive Retries* — the core retry logic that we use at Netflix to increase our resiliency and availability
* *Origin Concurrency Protection* — configurable concurrency limits to protect your origins from getting overloaded and protect other origins behind Zuul from each other
>
>**Operational Features**
* *Request Passport* — track all the lifecycle events for each request, which is invaluable for debugging async requests
* *Status Categories* — an enumeration of possible success and failure states for requests that are more granular than HTTP status codes
* *Request Attempts* — track proxy attempts and status of each, particularly useful for debugging retries and routing

You can find on instructions on getting started on the [Github wiki](https://github.com/Netflix/zuul/wiki/Getting-Started-2.0). We have a slew of features 
that we're working on and will release shortly, so stay tuned.
