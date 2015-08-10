--- 
layout: post
title: The art of blaming everyone but yourself
status: publish
type: post
published: true
date: 2015-08-10 09:00
tags: 
- personal
- tech
- work
---

As a systems engineer I've always been very close to complains and blames. It's not rare seeing things break. Code bugs, platform bugs, unexpected behaviours in the platform, etc.. There are several things that may lead to a outage or any other type of service degradation. And in these cases blames and complains start appearing everywhere.

Today AWS (Amazon Web Services) has experienced a partial outage of their S3 platform. I'm lucky to be mostly surrounded by people that, at least in part, understand the nature of a internet outage. However in these situatios (included today one) I see people blaming the external service outage because it makes their own platform to stop working.

So, this is the thing. You cannot blame anyone else but you if your platform stops working because a external service is down. If your platform stopped working is because one of these two reasons:

* You don't have a backup plan to execute when that service is down. Guess what. That's your fault. No matter how redundant or stable a service is. Shit happens, and if you fully trust on the uptime of that service you are not being very smart.
* Your platform heavily depends on the other service. It's totally built on it and without it your platform cannot exist. Assume it. If your business is about selling cupcakes in a school entrance and tomorrow the school is closed because a snow storm you cannot blame the school. That happens. You just accept it.

Of course you can and must consider SLA or any other warranties any service is going to give you. But that's pure statistic based on known scenarios. Not everyone is able to create backup plans for everything for different reasons. That's ok. But if you don't have it you cannot blame other for your fault. Either you assume that it may happen or you just shut up.

I'm not even going to mention people that make fun of other services going down.
