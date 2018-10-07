--- 
layout: post
title: Dynamic traffic tagging with Consul
status: publish
type: post
published: true
date: 2018-03-11 18:00
tags: 
- tech
---

Last week I attended the [WeCodeFest](https://wecodefest.com/) a highly technical event in Valladolid, Spain focused on sharing and learning. I have the chance to prepare a talk about service discovery with Consul. Check the [slides](https://docs.google.com/presentation/d/1IalJnfz3RPU6Qe8YI0ckdEBCLu9flK7HfIR0RBGLQtQ/edit#slide=id.g34e278bc84_0_0).

I wanted to showcase Consul and its powerful possibilities. I started giving some context about why, at CARTO, we started to use Consul for different purposes in order to have a more agile platform. The most important one was service discovery.

Talking about agile, at the end of the presentation I had to run a little bit more than I wanted because I was running out of time. So, I've decided to write this blog post to talk specifically about the most juiciest part of the demo.

### The goal

The goal is to have a simple HTTP proxy server that receives requests with different domains and proxies them to specific backend servers that are dynamically configured as upstreams. 
It seems simple. But there's an extra ball. We want to be able to switch traffic for some domains requests to a specific backend server. And we want to do it dynamically. A possible use case of this is, for example when you have deployed a new version to one backend and you want only your user to test that version in production.

This exercise is a way to show built-in service discovery, tagging and k/v capabilities in Consul. It's not production ready configuration, so don't deploy it to your servers without further development.

We are going to use just one consul server for this demo. The hostname will be cns01. We will also provision n backend servers that we will call web0x.

### Consul basic setup

A consul agent must run in every node. Every consul cluster requires, at least, one server in order to have consensus in the cluster. The rest of agents run in client mode. However, the binary (agent) running is the same in both roles and the only thing that changes from one role to another is the configuration.

Installing consul is pretty easy. Take a look at this [guide](https://www.consul.io/intro/getting-started/install.html).

This is a very basic example configuration for a server node:

{% highlight json %}
{
    "bootstrap_expect": 1,
    "data_dir": "/opt/consul/data",
    "datacenter": "localdatacenter",
    "log_level": "INFO",
    "server": true,
    "raft_protocol":1
}
{% endhighlight %}

The configuration for a client role is pretty similar. Just some small changes:

{% highlight json %}
{
    "data_dir": "/opt/consul/data",
    "datacenter": "localdatacenter",
    "log_level": "INFO",
    "server": false,
    "raft_protocol":1
}
{% endhighlight %}

### Install the proxy

This would be the host cns01. Start by installing and configuring consul in server mode.

We don't need a lot of servers for this demo. We can also install our proxy in this same node. No need to separate consul server from other services. Obviously, this is not a good practice in production, but, again, this is a demo.

I'm going to use nginx for this example, but you can use any other proxy. I will try to explain the example config so you can adapt it to whatever proxy you want.

Also, open your /etc/hosts and add a couple of example domains. I used vagrant for this demo with mapped ports in localhost so, in my case, the proxy server was in localhost. For example:

{% highlight bash %}
127.0.0.1       luisico.consul-wecodefest.lan
127.0.0.1       noluisico.consul-wecodefest.lan
{% endhighlight %}

These are the domains you will use to send requests to your proxy.

Your consul server is supposed to be running now. If you take a look to the members of the consul cluster you should only see this node:

{% highlight bash %}
$ consul members
Node           Address          Status  Type    Build  Protocol  DC               Segment
cns01.dc1.lan  10.0.3.123:8301  alive   server  1.0.1  2         localdatacenter  <all>
{% endhighlight %}

### Install the backend services

You can start with a couple of backend servers. Install and configure consul in client mode.

I'm also using nginx as a backend server example with a basic config that points any traffic to `/var/www/html/index.html`. 

Assuming your consul agent is already running too in these two servers, you can get the list of the members the same way we did with the previous one.

Only themselves appear in their own members list. We need to [join](https://www.consul.io/docs/commands/join.html) these nodes to the cluster. In this case, let's join each of them to the consul server:

{% highlight bash %}
$ hostname
web01.dc1.lan
$ consul join 10.0.3.123
Successfully joined cluster by contacting 1 nodes.
$ consul members
Node           Address          Status  Type    Build  Protocol  DC               Segment
cns01.dc1.lan  10.0.3.123:8301  alive   server  1.0.1  2         localdatacenter  <all>
web01.dc1.lan  10.0.3.231:8301  alive   client  1.0.1  2         localdatacenter  <default>
{% endhighlight %}

{% highlight bash %}
$ hostname
web02.dc1.lan
$ consul join 10.0.3.123
Successfully joined cluster by contacting 1 nodes.
$ consul members
Node           Address          Status  Type    Build  Protocol  DC               Segment
cns01.dc1.lan  10.0.3.123:8301  alive   server  1.0.1  2         localdatacenter  <all>
web01.dc1.lan  10.0.3.231:8301  alive   client  1.0.1  2         localdatacenter  <default>
web02.dc1.lan  10.0.3.174:8301  alive   client  1.0.1  2         localdatacenter  <default>
{% endhighlight %}

### Setup a new consul service

If everything went as expected we should already have these things:
* The host cns01 with a consul agent running in server mode and a nginx (or another proxy) running
* Two hosts web0x each of them with a consul agent in client mode and a nginx (or any other http service) running. Let's say they are listening in the port 8080
* All the consul agents joined together in the same cluster

At this point, although the nginx and the consul servers are running together every node, they don't know each other. The goal is to have consul know that services running in the port 8080 so we can discover this information from any node in the consul cluster.

{% highlight bash %}
$ curl -s http://127.0.0.1:8500/v1/catalog/services |python -m json.tool
{
    "consul": []
}
{% endhighlight %}

Ok, let's add a new service in consul. This is as simple as adding a new json config to consul. Put this json file in the same directory than you have your agent.json:

{% highlight json %}
{
    "service": {
        "address": "10.0.3.174",
        "checks": [
            {
                "http": "http://10.0.3.174:8080/",
                "interval": "5s",
                "timeout": "2s"
            }
        ],
        "name": "backend-web",
        "port": 8080,
        "tags": [
            "default"
        ]
    }
}
{% endhighlight %}

Add this config in both web0x nodes. Then reload consul:

{% highlight bash %}
$ consul reload
Configuration reload triggered
{% endhighlight %}

Let's check know the list of services:

{% highlight bash %}
$ curl -s http://127.0.0.1:8500/v1/catalog/services |python -m json.tool
{
    "backend-web": [
        "default"
    ],
    "consul": []
}
{% endhighlight %}

Oh yeah!
Now, let's see what nodes in the cluster have this service running:

{% highlight bash %}
$ consul catalog nodes -service=backend-web
Node           ID        Address     DC
web01.dc1.lan  464efb7d  10.0.3.231  localdatacenter
web02.dc1.lan  266a02b6  10.0.3.174  localdatacenter
{% endhighlight %}

<div class="note">
  <div class="admonition-title">
    Note
  </div>
		Note that I keep using different ways to gather information from consul. There are several ways to resquest info to consul. You can use HTTP API, DNS and consul command also supports some operations. Take a look to the <a href="https://www.consul.io/api/index.html">API</a> doc and play with it
</div>


### The service tagging 

At this point, we could already generate upstreams configuration in nginx using the services information from consul. We could have an upstream config more or less like this:

{% highlight nginx %}
upstream default_servers {
  server 10.0.3.231:8080;
  server 10.0.3.174:8080;
}
{% endhighlight %}

We are able to just discover backends that have running (and healthy) "backend-web" services. We want to go further, though and have different upstreams with different groups of upstreams based on different service tags.

So, what are service tags? Consul allows associating tags (array of strings) to any service, through the service config or through the HTTP API. If you take a look to the config we used to create the "backend-web" service you will find this:

{% highlight json %}
"tags": [
    "default"
]
{% endhighlight %}

That is the list of tags associated with the service. It's not required to add tags to a service. Actually, the tag "default" name is intentional. We are going to assume that each "backend-web" service has just one tag. The default tag is "default" but we can change that name to "tagged". We will see this later. At this moment our two services have both the "default" tag, so we should have the same result when looking for the nodes with just the "backend-web" service than when looking for the "backend-web" service **and** the tag "default". Let's see it:

{% highlight bash %}
$ dig backend-web.service.consul @127.0.0.1 -p 8600 +short
10.0.3.231
10.0.3.174
{% endhighlight %}

{% highlight bash %}
$ dig default.backend-web.service.consul @127.0.0.1 -p 8600 +short
10.0.3.231
10.0.3.174
{% endhighlight %}

Play a little bit with this. Take one of the web servers and do some changes and, with DNS queries check how the results are different:
* Stop the nginx service and run the DNS queries again. Only the other web server IP should appear in both results
* Replace the default tag with another name, reload consul config and run both DNS queries again. Both web servers IPs should appear in the first query result. Only one of them will appear in the second query result

### The traffic tagging

Our services are already being monitored with consul and we are able to tag each of them. Now, we need a way to tell nginx that when the service receives requests to one domain, they must be proxied to a group of upstreams and when the requests are sent to another domain they need to be proxied to a different upstreams group. We are going to use a key/value. Yes, Consul has a distributed [k/v storage](https://www.consul.io/api/kv.html).

We will use the key `nginx/tagging/tagged_users` to store n domains. In the example, we will look at the value of this key and if there's any domain it will be mapped to the "tagged" upstreams group in nginx.

Let's see the configurations we need for this, starting with the main virtual host nginx configuration:

{% highlight nginx %}
server {
  listen 80;

  root /var/www/html;

  server_name *.consul-wecodefest.lan;

  location / {
    proxy_pass http://$upstreams_group;
  }
}
{% endhighlight %}

This config basically says that any request to a subdomain of `consul-wecodefest.lan` will be proxied to a variable upstream group. This upstream variable will be defined with this upstreams config file:

{% highlight nginx %}
upstream default_servers {
  server 10.0.3.231:8080;
  server 10.0.3.174:8080;
}

map $host $upstreams_group {
  default default_servers;
}
{% endhighlight %}

That is the piece of config that will map every request to one upstream or another. This is the config we would obtain when we have both web servers running healthy, both with the default tag and the consul key `nginx/tagging/tagged_users` contains an empty string.

Now, the exciting part. We want traffic tagging to be highly dynamic. This means that, under normal circumstances, we should be able to keep changing tags in services, the tagging key value, etc.. without having to manually re-configure nginx. The dynamic piece of nginx config, in this case, is the upstreams file. So, we are going to keep a [consul-template](https://github.com/hashicorp/consul-template) daemon watching for services, tags, and k/v changes so it can dynamically change the upstreams config and reload it without us knowing and without interrupting the service.

This is the consul template we are going to use to generate that piece of upstreams configuration:

{% highlight jinja %}
{% raw %}
{{ range $tag, $services := service "backend-web" | byTag }}
upstream {{ $tag }}_servers {
{{ range $services }}
  server {{ .Address }}:{{ .Port }};
{{ end }}
}

{{ end }}

map $host $upstreams_group {
  default default_servers;

  {{ range $domain := (key "nginx/tagging/tagged_users" | split ",") }}
  "{{ $domain }}" tagged_servers;
  {{ end }}
}
{% endraw %}
{% endhighlight %}

Basically this template will look for services with the name "backend-web" and group them by tag. It will render an upstream block for every tag found within that service name.

After that, it will generate a host-to-upstream-group list with all the domains found in the `nginx/tagging/tagged_users` key so their traffic goes to the "tagged" upstream group. Note that, as I commented at the beginning this is a demo. As you can see, several things could be improved. Don't use it in production.

We can render this template with something like this:
{% highlight bash %}
$ consul-template -template "/etc/nginx/sites-available/upstreams.ctmpl:/etc/nginx/sites-enabled/upstreams:systemctl reload nginx"
{% endhighlight %}

Note that consul-template will keep running indefinitely watching for changes in all the elements described in the template, triggering the template rendering and reloading nginx over and over again.

Now, you should be able to send requests to your proxy using any of the domains you have configured in your hosts file. By default, you should see that all requests are balanced between the two web servers.

Now, play!!
* Change tags in your web services definitions. Replace some "default" ones with "tagged" ones
* Keep changing the tagged domain in the k/v. For example: `consul kv put nginx/tagging/tagged_users "noluisico.consul-wecodefest.lan"`. Put a blank value again in the k/v
* Put some nginx services down
* Provision new web servers

..and keep sending requests to your proxy. Now you have a full dynamic traffic balancing and tagging with Consul!

Cheers
