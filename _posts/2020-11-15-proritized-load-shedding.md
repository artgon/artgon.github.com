---
layout: post
title: "Keeping Netflix Reliable Using Prioritized Load Shedding"
meta-description: Prioritized load shedding to increase availability
---

Running [Zuul](https://github.com/Netflix/zuul) at Netflix, we often see traffic spikes. Typically the spikes are caused by retry storms because of bugs on device builds or service deploys, and more rarely due to DDoS attacks or network failures. Zuul needs to maintain availability for both itself and its backends when faced with unsustainable load. As a result, we need to reduce the load, meaning customer requests will be throttled. Of course, the interesting question is, which requests do we drop?

We've invested in building a system to prioritize traffic and shed
the lower priority non-playback traffic first, progressively increasing to higher
priorities.  

You can read the full blog post [here](https://netflixtechblog.com/keeping-netflix-reliable-using-prioritized-load-shedding-6cc827b02f94), it has a lot of detail but here's the real money shot:

![Load shedding](https://miro.medium.com/max/1400/0*zhw-qRWIQAfnSiBU)

There are some exciting aspects to what we built, particularly the cubic progression of priority throttling. 

> ![Cubic](https://miro.medium.com/max/1294/0*zmyyiRWI49KCFoEP)

> The graph above is an example of how the cubic function is applied. As the overload percentage increases (i.e. the range between the throttling threshold and the max capacity), the priority threshold trails it very slowly: at 35%, it’s still in the mid-90s. If the system continues to degrade, we hit priority 50 at 80% exceeded and then eventually 10 at 95%, and so on.

This feature has been effective in practice. If there is a blip or a minor issue (even a medium issue), there is almost no impact on customers trying to watch a movie. They won’t get affected until we get into the steep side of the cubic curve, which means things are in a really bad state.

There are plenty more tidbits in the [post](https://netflixtechblog.com/keeping-netflix-reliable-using-prioritized-load-shedding-6cc827b02f94) (check out the backpressure
mechanism). It's worth reading if you're interested in building resilient systems at scale.
