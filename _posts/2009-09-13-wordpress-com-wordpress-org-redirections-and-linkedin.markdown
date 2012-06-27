--- 
layout: post
title: wordpress.com, wordpress.org, redirections and LinkedIn
tags: 
- "301"
- "302"
- linkedin
- Ocio
- Personal
- plugin
- redirections
- Sistemas
- wordpress
status: publish
type: post
published: true
meta: 
  _edit_last: "2"
---
I wanted to change my <a href="http://wordpress.com/">wordpress.com</a> blog to a self-hosted wordpress blog.
This is the scenario:

* My old blog is lbosque.wordpress.com.
* My new blog is blog.luisbosque.com.
* I would like to keep the indexed information in google, so people that open a old indexed post could actually access the information and not receive a 404 status.
* Wordpress.com doesn't offer the posibility of add a 301 redirection in their system to put your blog outside of it, which is totally understandable.

<strong>UPDATE:</strong> In 2010/05/31 wordpress.com started to do 301 permanent redirections to custom domains.

So, what I did, is this:

* I added an extra domain (blog.luisbosque.com) in my wordpress.com blog. This costs about 10$/year, which is not much money. What I can do with this, is to set the new domain as a primary domain, so when I try to open http://lbosque.wordpress.com, wordpress redirects the request to http://blog.luisbosque.com. Unfortunatelly this is a 302 Redirection which is not a good thing to Google indexing, but there is no choice.
* Next, I installed the last wordpress.org version in my server and imported all the current blog information. Configured the webserver to use the blog.luisbosque.com domain.
* The next step was to change the blog.luisbosque.com DNS zone to point to my new server instead of doing a CNAME to lbosque.wordpress.com. So now when someone try to open http://lbosque.wordpress.com, wordpress.org redirects it to http://blog.luisbosque.com that is actually hosted on my server. My old blog is still hosted in wordpress.com, but because the redirections is no more visible.

So, now, I have to try to change all of my old blog references and wait to Google have indexed the most part of the new blog. Later, I'll remove the lbosque.wordpress.com blog.

I found a problem when I changed the URL on the LinkedIn Wordpress plugin. Within this plugin, you have to enter your wordpress blog URL, and the plugin updates the LinkedIn application with the latest blog posts. This is great, but it has a problem when the domain you enter is in a external hosting, but it is also registered as a wordpress.com domain. The plugin tries to search this blog in the wordpress.com database, so if it finds it (in my case, it does it) it stops looking for it in their real server.
The solution I figured out is to use an auxiliar domain, this way:

* I configure in my web server an auxiliar domain, for example blog.luisbosque.es. I configure it so when someone try to open http://blog.luisbosque.es the webserver do a 301 redirection to http://blog.luisbosque.com
* I change the LinkedIn wordpress plugin blog URL to blog.luisbosque.es. This domain is not registered in the wordpress.com database so the plugin tries to fetch the feeds within my server, which actually redirects the request to the final domain blog.luisbosque.com

The result is that the posts title that appears in the wordpress application within LinkedIn belong to the blog.luisbosque.com domain and not to the blog.luisbosque.es auxiliar domain.
This is not the cleanest solution. It's a bit tricky, but it works, which it's enough for me.

Thanks to the fast and useful <a href="http://support.wordpress.com/contact/">wordpress support</a> answer, that helped me to figure out how to override this behaviour.
