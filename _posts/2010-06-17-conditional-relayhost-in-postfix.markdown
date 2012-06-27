--- 
layout: post
title: Conditional relayhost in Postfix
tags: 
- nexthop
- postfix
- relayhost
- Sistemas
- transport
status: publish
type: post
published: true
meta: 
  _edit_last: "2"
---

Let's suposse that your server sends all the mail through a relayhost. But you need emails to a specific domain to be sent directly without passing by the relayhost. To solve this problem you need to configure your transport table with different nexthops depending on the destination domain.

For example, if I want to send directly emails to gmail.com accounts. And I want the rest of emails to be sent through a relayhost in my network (10.0.0.5), I need to write this transport table config file:

{% highlight yaml %}
gmail.com    :
*            smtp:10.0.0.5
{% endhighlight %}

Don't forget to postmap the transportsf file and include the transport_maps config parameter in your main.cf:

{% highlight yaml %}
transport_maps = hash:/etc/postfix/transport
{% endhighlight %}
