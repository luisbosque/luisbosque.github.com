--- 
layout: post
title: newlocation.info
status: publish
type: post
published: true
date: 2012-07-26 10:00
---

My last personal project is alive. Its name: [newlocation.info](http://newlocation.info).

I had a problem. I had my blog and other stuff in a virtual server but I decided to close it. I didn't need a entire server for a buch of things.
One of the things I didn't want to lose was my blog. I re-built it with jekyll and uploaded as a github page. Also, I changed its domain from blog.luisbosque.com to blog.luisico.net. Everything was fine until here.

The problem? SEO. My old content was indexed in search engines, like google's one. I didn't want to lose my content but I didn't longer have a handy nginx server to make my own redirections, so I created this service.

The idea is pretty simple. I create a redirection through the form. In my case a redirection from blog.luisbosque.com to blog.luisico.net. It's a 301 redirection. Also I want to keep the paths. This means that if I try to access to a old content in http://blog.luisbosque.com/2010/03/04/a-random-post/ I'm going to be redirected to http://blog.luisico.net/2010/03/04/a-random-post/ not just http://blog.luisico.net/

So, at this point, the redirection service is already able to handle that. The only thing I need to do now is change my luisbosque.com domain and point blog.luisbosque.com to the redirection server name; in this case r.newlocation.info

That's it.  My readers are happy because the can keep accessing the old URL's and also google is not going to lose any of my indexed content and it's going to learn that all my indexed content is now in another domain.

The service is still beta. I'm planning to add some cool stuff like user signup in order to have more control over our own redirections, and maybe possibility to add regex redirections.

BTW, it's a free service.
