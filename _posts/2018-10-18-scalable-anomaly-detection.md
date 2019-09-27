---
layout: post
title: "Scalable Anomaly Detection (with Zero Machine Learning)"
meta-description: Netflix real-time anomaly detection system
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/6UwcqiNsZ8U" frameborder="0" allowfullscreen></iframe>

I gave another [Strange Loop](https://www.thestrangeloop.com/) talk this year. It was about our real-time anomaly detection system called Raju. Here is the abstract:

>In a large scale distributed system, detecting and pinpointing failures gets exponentially harder as an architecture gets more complex. Netflix's cloud architecture is composed of thousands of services and hundreds of thousands of VMs and containers. Failures can happen at any level and can often cascade quickly, some can cause massive outages on several systems, while others only only break one or two. This creates a needle in a haystack problem that requires automated and precise detection. Zuul, as the front door for all of Netflix's cloud traffic, sees all requests and responses and is ideally positioned to identify and isolate only the broken paths in the maze of microservices.
>
>We leveraged Zuul to stream real-time events for each request-response and built an anomaly detector to automatically identify and alert services in trouble. We scaled this detector to thousands of nodes, handling millions of requests, without a single line of machine learning. Sometimes you need machine learning and sometimes you don't. Although it's en vogue to apply machine learning to every problem, it can be more practical and approachable to solve certain problems with old-fashioned math!
>
>In this talk, we'll discuss how we built this system with stream processing, anomaly detection algorithms, and a rules engine. We will also deep-dive into the anomaly detection algorithm and show how sometimes a simple, elegant algorithm can be just as good as any sophisticated machine learning.
