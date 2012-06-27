--- 
layout: post
title: Default server in nginx
tags: 
- default server
- nginx
- Sistemas
status: publish
type: post
published: true
meta: 
  _edit_last: "2"
---

By default nginx accepts requests from any domain that points to the nginx server. If you want that only requests from domains that are configured in nginx be processed you cand add a default rule in nginx:


{% highlight nginx %}
server {
  listen       80 default;
  server_name  _;
  deny         all;
}
{% endhighlight %}

So, if you try to do a request from a domain not configured in nginx, it will return a 403 status (forbidden)
