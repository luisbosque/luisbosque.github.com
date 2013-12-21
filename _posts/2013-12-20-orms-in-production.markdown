--- 
layout: post
title: Standard ORMs in production
status: publish
type: post
published: true
date: 2013-12-20 09:00
tags: 
- orm
- opinion
- reliability
---

Some days ago I complained about standard ORMs in twitter. My complains were supposedly a little bit out of context. So i'm goint to try to explain my opinion about it.

First of all I need to admit that I use standard ORM's at work (at home I try to do write my own object mappers). However, using them doesn't mean that I'm blind and not able to see important problems about them.

I can start with what, in my opinion, standard ORMs are good for: (notice that I'm always using the word 'standard')  

* They are good for prototyping
* They are good for simple applications
* They are good for simple infrastructure environments
* They are good for environments where the level of performance is not critical or the amount of backend hits it's not extremely high
* They are good for applications where you don't really care about of none of the previous points

A brief explanation about ORMs to help with the missing context.
ORM = Object-Relational Mapping. As I understand, this is a technique to map application objects with relational databases types.
So, for example, in your application, instead of writing a SQL query in order to get all the users of your platform, you just run in your app something like

{% highlight ruby %}
User.all
{% endhighlight %}

and automagically this will run in your database something like:

{% highlight sql %}
SELECT * FROM users
{% endhighlight %}

This is good. Fast development. Easy to remember. No need to know about what is happening with your database or how to interact with it.
And that is precisely the problem. When you put an application in production and you want it to have a predictable behaviour and you want it to have a good performance you can't afford being a ignorant about what is going on with your database under the hood. You need to control any call to your database and be able to do a performance tunning.

If you spend some time debugging calls to your database you will probably realize that your ORM makes a important number of calls that probably are not necessary for your case. Some of them are even bad for your application performance or behaviour. For example SQL queries that are badly optimized for your models or your application logic. These calls are probably fine to guarantee that everything works in the same way in different scenarios, which makes the ORM good for standard application. The thing is that when you put a complex application in production you cannot consider it a standard application anymore. You really need control what is happening under the hood.

So, in my opinion, you can't say that you have a reliable and high performance application in production if you are using a standard ORM. Otherwise, either you don't know what you are talking about or you spend too much time studying and tunning your ORM. Probably more time that you would spend doing your own object mapper.
