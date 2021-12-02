---
layout: post
title: "Securing Netflix Studios At Scale"
meta-description: Using Zuul to secure Netflix studio in the cloud
---

Over the years, Netflix has evolved from a streaming company to a
full-fledged studio. Being a tech company, we wanted to leverage that expertise to build a studio of the future. In this future world, there is an application for every part of the process -- from creative, to production, to marketing, and eventually all the way to playback. We call it the "Studio in the Cloud."

As you can imagine, one of the biggest concerns for these applications is security because of all the sensitive information related to pre-release content. So how do we get these applications on the internet as fast as possible and as securely as possible? 

Well, we already have a [secure gateway](https://github.com/Netflix/zuul) for streaming customers, so why don't we also leverage it for Studio applications? That's exactly what we did, and it has streamlined deployment while also increasing security. It has been an unmitigated success story. Check out the blog post [here](https://netflixtechblog.com/the-show-must-go-on-securing-netflix-studios-at-scale-19b801c86479). 
