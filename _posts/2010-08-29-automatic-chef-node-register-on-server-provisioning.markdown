--- 
layout: post
title: Automatic chef node register on server provisioning
tags: 
- automation
- bootstrap
- chef
- ruby
- Sistemas
- validation
status: publish
type: post
published: true
meta: 
  _edit_last: "2"
  _wp_old_slug: ""
---

When I started playing with chef, I rapidly came to a question. How can I do to automate the new nodes registration on chef server when creating new virtual instances?
I have a few server images. All my images have chef-client and the necessary files to do a bootstrap client setup installed.
Besides these files I have a small "init" script and a config file for this script.
The script basically run the bootstrap chef-client set up with chef-solo and restart the chef-client. If the process goes well, it removes the server validation.pem stored in /etc/chef/ and makes sure that the script is not going to run anymore by setting a config variable.

Here is the config file:

{% highlight bash %}
BOOTSTRAP=0
BOOTSTRAP_URL="http://s3.amazonaws.com/chef-solo/bootstrap-latest.tar.gz"
CHEF_SOLO="/etc/chef/solo.rb"
CHEF_JSON="/root/chef.json"
VALIDATION_PEM="/etc/chef/validation.pem"
{% endhighlight %}

and here is the "init" script:

{% highlight bash %}
#!/bin/bash

CHEF_CONF="/etc/default/chef-client"

if [[ -f $CHEF_CONF ]]
then
  . $CHEF_CONF
fi

if [[ $BOOTSTRAP != 1 ]]
then
  exit 0
fi

rm /etc/chef/client.pem

chef-solo -c $CHEF_SOLO -j $CHEF_JSON -r $BOOTSTRAP_URL
chef-client

if [[ $? == 0 ]]
then
  sed -r -i "s/BOOTSTRAP=.*/BOOTSTRAP=0/" $CHEF_CONF
  rm $VALIDATION_PEM
fi

/etc/init.d/chef-client restart
{% endhighlight %}

I believe that there are a lot of approaches, and maybe better than mine. For example, mine is not valid if you are going to use it with public AMI in amazon. Even if these AMIs have chef-client installed you must provide your server validation.key. The best solution in this case would be to make a SSH connection to the new server right after it starts and copy the validation.pem and the necessary chef configurations.
