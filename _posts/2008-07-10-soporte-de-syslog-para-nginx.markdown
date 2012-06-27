--- 
layout: post
title: Soporte de Syslog para Nginx
tags: 
- "\"logs centralizados\""
- "\"Marlon de Boer\""
- compilar
- nginx
- parche
- Sistemas
- syslog
- Trabalho
status: publish
type: post
published: true
meta: 
  _edit_last: "2"
  delicious: s:78:"a:3:{s:5:"count";s:1:"0";s:9:"post_tags";s:0:"";s:4:"time";s:10:"1247076398";}";
---

Estos días tenía la necesidad de hacer que <a href="http://nginx.net/">nginx</a> mandase sus logs al syslog local.
Nginx no tiene soporte nativo para usar syslog. Puede escribir los logs o en fichero o mandarlos a un pipe.
Estuve bsucando y encontré un <a href="http://bugs.gentoo.org/show_bug.cgi?id=222373">parche</a> de la versión 0.6.31 que resuelve el problema. <a href="http://www.ruby-forum.com/topic/153141">Aquí</a> habla también un poco del parche.

A pesar de que en ese último link explica por encima como hacerlo, lo voy a contar yo también.
<ul>
	<li>Bajamos la versión 0.6.31 de nginx en <a href="http://sysoev.ru/nginx/nginx-0.6.31.tar.gz">http://sysoev.ru/nginx/nginx-0.6.31.tar.gz</a>. Según el comentario de Marlon (el creador del parche) debería funcionar también con las versiones 0.6.30 y 0.6.29, y mirando los changelogs pienso que tambíen debería funcionar en la última que es la 0.6.32. Si alguien lo prueba que me lo diga.</li>
	<li>Descomprimimos:
</li>
{% highlight console %}
# cd /usr/src/
# tar xvzf nginx-0.6.31.tar.gz
{% endhighlight %}
	<li>Bajamos el parche:
</li>
{% highlight console %}
# wget http://bugs.gentoo.org/attachment.cgi?id=153345 -O nginx_syslog.patch
{% endhighlight %}
	<li>Parcheamos:
</li>
{% highlight console %}
# patch -p0 &lt; nginx_syslog.patch
{% endhighlight %}
	<li>Compilamos e instalamos nginx:
</li>
{% highlight console %}
# cd nginx-0.6.31
# ./configure --with-syslog
# make
# make install
{% endhighlight %}
</ul>

En mi caso antes de compilar he tenido que hacer un cambio en una linea de los fuentes del nginx. He tenido que substituir en el fichero <em>auto/cc/gcc</em> la siguiente linea:
{% highlight console %}
CFLAGS="$CFLAGS -Werror"
{% endhighlight %}
por:
{% highlight console %}
CFLAGS="$CFLAGS"
{% endhighlight %}

Esto únicamente hace que no se rompa la compilación al encontrar algun warning. En mi caso los warning que lanza la compilación se pueden ignorar tranquilamente, por lo que resulta seguro continuar con ellos.

Si todo ha ido bien deberíamos de tener funcionando nginx. Faltaría unicamente configurarlo y ponerlo en marcha. En el fichero de configuración no hace falta indicar nada para que mande correctamente los logs al syslog. Por defecto los manda a la facility daemon.

Yo lo he probado ya en servidores en producción y por el momento funciona estupendamente.

Desde aquí agradezco el esfuerzo a <a href="http://mjdeboer.hyves.nl/">Marlon de Boer</a>, que no he conseguido encontrar un blog suyo donde hacerlo.
