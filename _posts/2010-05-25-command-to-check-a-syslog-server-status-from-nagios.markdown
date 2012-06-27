--- 
layout: post
title: Command to check a Syslog server status from Nagios
tags: 
- nagios
- perl script
- "programaci\xC3\xB3n"
- Sistemas
- syslog
- trabajo
status: publish
type: post
published: true
meta: 
  _edit_last: "2"
---

I've been using Nagios for a long time and now I have to check a Syslog server. I was looking for a command to check directly a remote Syslog server. 
I didn't find anything that fit my needs so I did one by myself.

To use this command, the Syslog server has to bee listening on the UDP 514 port. It's only made to check Syslog servers that are available on a network.
The process is simple:

1. The script sends a message to the syslog server
2. The script start a small UDP server on localhost
3. The remote syslog server receives the message and sends it back to the server where the script was executed from
4. The script UDP server receives the message and check if it is correct

The Syslog server needs to know how to match the specific message from the check script and send it back to the source script, so it needs a small previous configuration.

The code is hosted in github. There you cand find detailed information about how to run this script and how to configure a Syslog server in the README file.
<a href="http://github.com/lbosque/check_syslog">http://github.com/lbosque/check_syslog</a>

You can also clone directly the repository and see the README with your favourite editor:
{% highlight bash %}
git://github.com/lbosque/check_syslog.git
{% endhighlight %}
