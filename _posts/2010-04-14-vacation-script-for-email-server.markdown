--- 
layout: post
title: Vacation script for email server
tags: 
- ldap
- postfix
- "programaci\xC3\xB3n"
- python
- Sistemas
- vacation
status: publish
type: post
published: true
meta: 
  _edit_last: "2"
  _wp_old_slug: vacation-script-to-email-server
---
Sometimes I have the need to install some kind of auto-respond feature on an email server. I already used a few times gnarwl.
Honestly, I don't like it. I spent a lot of time debugging and some parts of it made me crazy. Of course, it's only my opinion and admit that there are a lot of work put on that project.
Nevertheless I decided to make my own vacation script. The idea was to make something very easy to use and install, and also something easy to hack. I assume certain behaviours so I don't relay on the configuration file for a lot of things. This way, it results in a very simple configuration file.

I still want to add some features, but only the necessary to have a quality script without turning it in a very complex script.

The code is hosted in github. There you can rea the README file to more information about how it works and how to install it.
<a href="http://github.com/luisbosque/simple_vacation">http://github.com/luisbosque/simple_vacation</a>

You can also clone directly the repository and see the README with your favourite editor:

{% highlight bash %}
git clone git://github.com/luisbosque/simple_vacation.git
{% endhighlight %}

It would be great to have any kind of feedback that helps me to improve the code. Of course you can contact me for any kind of doubt or comment.
