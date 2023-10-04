---
layout: post
title: "Curbing Connection Churn in Zuul"
meta-description: Netflix’s Zuul Gateway eliminated tens of millions of connections and reduced almost all connection churn to backends
---

One of the longest-lived problems with [Zuul](https://github.com/Netflix/zuul) is how many connections it opened to back-end servers. We recently invested in modifying our connection pooling logic to make use of subsetting and fix these problems. It's worth reading the blog post if for nothing other than the phenomenal improvement graphs. Here's a sneak peek of the total connection counts per region falling off a cliff:

<img src="https://miro.medium.com/v2/1*5PdlBr4bFkC0H_QprQRnEg.png" alt="Zuul Total Connections" width="800"/>

Another highlight is the algorithm we used to build the distribution ring for the servers.


> The algorithm relies on the idea of low-discrepancy numeric sequences to create a naturally balanced distribution ring that is more consistent than one built on a randomness-based consistent hash. The particular sequence used is a binary variant of the Van der Corput sequence. As long as the sequence of added servers is monotonically incrementing, for each additional server, the distribution will be evenly balanced between 0–1. Below is an example of what the binary Van der Corput sequence looks like.

<div style="text-align:center"><img src="https://miro.medium.com/v2/0*n5Q8XAqo8V4qypwe" alt="VdC"/></div>

> Another big benefit of this distribution is that it provides a consistent expansion of the ring as servers are removed and added over time, evenly spreading new nodes among the subsets. This results in the stability of subsets and no cascading churn based on origin changes over time. Each node added or removed will only affect one subset, and new nodes will be added to a different subset every time.

For the full deep dive, check out the blog post [here](https://netflixtechblog.com/curbing-connection-churn-in-zuul-2feb273a3598).
