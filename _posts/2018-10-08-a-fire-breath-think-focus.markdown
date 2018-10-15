--- 
layout: post
title: A fire! Breath, think and focus
status: publish
type: post
published: true
date: 2018-10-14 09:00
tags: 
- work
---

Downtimes are something common among any web application. No matter how much effort you put you cannot totally avoid them. What you can do is to make sure you have a plan ready when it's time.

Obviously, depending on how much impact a downtime may have on your business, you will decide to make, for example, redundancy your business core. But that's a different story. Let's focus now on what happens when the fire starts.

First of all, fires require a specific mindset. No matter how thorough are the decisions you make while you are on it, your time to solve it is limited. Don't waste it on anything that does not directly help you to get closer to your goal.

### Understand the problem

Like any other issue, you need all the info possible about the context before you start working.

- How do we know about this problem? Who reported it? Was it reported internally? Or was it a user? 
- Who is it affecting to? Do we know if it affects to just one user/person?
- When did it start? When did people start reporting the problem?
- How is it detected? What's the best description we have about it?

All those points are part of the problem itself, probably from the user perspective. But that gives you only half part of the context. For the other half you need to answer a simple question: Why is this a problem? or why is this important?

It may seem obvious. Any user problem is a problem. Well, yes, but understanding why it is important for you and your user will make a huge difference about how you approach the situation. The reasons may vary. Maybe you are losing money, or breaching a QoS commitment, or losing reputation or maybe leaving the situation like this may incur in further degradation of the service. 

### Reproduce the issue

It's not that you don't trust your sources, but it's important that you are able to reproduce the issue, or at least see it in action. You will need to test it several times during this process and you must make sure all the info you have is the correct. Test different scenarios; maybe different users, subdomains, urls. You can also check caches, etc... All the info you are going to gather while you try to reproduce it is going to be very useful to help you focus your effort once you start debugging.

Now you have the full context and you have been able to reproduce it, you can start making decisions:

- How urgent it is that I solve the situation?
- Who must keep updated about the process?
- Do I need somewhere else to help solving the issue?
- How long can I go, in terms of budget, to solve this situation?

<img src="{{ "/images/this-is-fine.png" }}" alt="This is fine - KC Green" title="KC Green" width="600" height="300" class="aligncenter" style="border:none;"/>

### Trace the cause

Sometimes you may feel the urge to say "the problem is too much traffic". I'm pretty sure it's not. The root problem is not "environmental", it's technical. If the trigger is a peak of traffic, look for the specific spot within your that is breakking under "too much traffic". Don't just try to solve the problem provisioning more servers because most times, you will find yourself wasting your precious time on something that won't probably make any difference.

Now you have full context of the problem you can start debugging. This is the moment where it's easier to lose focus. Start with the first thing that will more likely give you clues about an anomaly. Metrics and logs. No matter if you have an automated metrics and logs processing platform or you need to script your way into your services health, don't wander around. Start checking the things that will confirm you there is a problem: HTTP response status from logs, error logs from any of your services, unusual systems values (load, cpu, io, ...), etc.. When you look for events like requests responses I suggest to start looking for volume. Check how many errors are you having in the last minute, hours, etc.. That will take you less time and will help you move among all the information you need to check before going into detail of any of them.

You may keep finding "the root cause" and constantly understand that this cause is just a consequence of another cause. Many times you will find a cascade effect in your platform. You need to keep pulling the string. You may be even lucky and find that more than one thing has broken. So much fun.

You may spend some time doing some treasure hunt but remember, you are just getting the clues that allow you to get deeper and solve the problem. Time is critical here and if it looks like a duck, swims like a duck, and quacks like a duck, then it probably is a duck. If you have a solid clue, go ahead.

### Understand the root cause

With complex systems the root cause of your problems will probably be a cascade of several ones when each of them is a consequence of the previous one. Other times you can even find that two things are broken at the same time. Let's assume there's one just broken thing. You are in front of the beast. You have already isolated the problem. Now it's the time you think "fuck!". 

Of course every action must be done within certain company parameters, like costs and the bug impact. The rest of parameters are not relevant. Too much work, if it's the only option to solve a problem within your company needs, is not a parameter.

There's just one leader in the resolution of the problem. No more. Always announce what you are going to do and ask if it's helpful or anyone is already working on it

diverge/converge

share frequent updates with the rest of the team, specially support

metrics
logs
grep

Two things will likely happen during this session. You may don't know how to reproduce the issue. You may not be able to find the root cause. You may not know how to solve the problem once you have found it. Dealing with uncertainty and the unknown is what differences a experienced engineer from one that is not. 

Record all your steps and what you are doing and testing and what you have discarded

Make sure your fix is not making things worse. Provisions instances that don't work or with different code. Take care with solving the situation just putting more servers

Divide and conquer! never have anyone stopped not doing anything

Use your intuition and experience and try to gain some time initiating plans before you are fully sure about the root cause of the problem
