--- 
layout: post
title: "Gráficas de uso de memoria de passenger con Munin"
tags: 
- apache
- munin
- passenger
- plugin
- ruby
- Sistemas
status: publish
type: post
published: true
meta: 
  _edit_last: "2"
  delicious: s:78:"a:3:{s:5:"count";s:1:"0";s:9:"post_tags";s:0:"";s:4:"time";s:10:"1251897457";}";
---

Estaba instalando algunos plugins adicionales en el munin para monitorizar el estado de las peticiones que entran por Apache.
He encontrado <a href="http://muninexchange.projects.linpro.no/?view&amp;phid=595">uno</a> para monitorizar el uso de memoria pero se ha quedado anticuado con respecto a ciertos cambios que ha sufrido la salida del comando passenger-memory-stats. Por lo tanto ya no sirve.
Basándome en dicho plugin y en <a href="http://gist.github.com/20319">este</a> otro, he hecho algunos cambios y el código resultante es el siguiente:

{% highlight ruby %}
{% include passenger_munin.html %}
{% endhighlight %}

El plugin parsea los totales de memoria tanto de apache, como de nginx, como de apache passenger. En mi caso solo muestro los valores de apache y de passenger, pero si se quisiera mostrar los de nginx, bastaría con descomentar la linea:

{% highlight ruby %}
puts "nginx_memory.value #{memory['nginx']}"
{% endhighlight %}
