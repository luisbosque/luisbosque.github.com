--- 
layout: post
title: Una historia de capistrano, crontabs y pipes
tags: 
- capistrano
- crontab
- deploy
- pipe
- "programaci\xC3\xB3n"
- ruby
- Sistemas
- sudo
- trabajo
- tuberia
status: publish
type: post
published: true
meta: 
  delicious: s:78:"a:3:{s:5:"count";s:1:"0";s:9:"post_tags";s:0:"";s:4:"time";s:10:"1248771321";}";
  _edit_last: "2"
---

Al montar el deploy de una nueva máquina con <a href="http://www.capify.org/">capistrano</a> he querido afinar un poco la carga de crontabs.
No me gusta poner las tareas de crontab en el /etc/crontab. Creo que es una muy mala práctica. En vez de eso prefiero que cada usuario tenga su propia tabla de crontabs y para ello hago uso del comando <a href="http://linux.die.net/man/1/crontab">crontab</a>.
En muchas ocasiones hace falta definir crontabs para más de un usuario. Para no andar añadiendo una linea en el script de deploy por cada fichero de crontab que se tenga doy por hecho que los nombres de todos los archivos con las tablas de crontab siguen el mismo formato, que es "crontab ..
El código pues para capistrano es:

{% highlight ruby %}
task :app_deploy, :roles =&gt; [:app] do
  Dir["./appserver/etc/crontab.*"].each { |crontab|
    sudo "sh -c 'cat #{release_path}/#{crontab} | \
    crontab -u  #{File.extname(File.basename(crontab)).delete('.')} -"
  }
end
{% endhighlight %}

En este código hay varias cosas que explicar.
Primero se presupone pues que si quiero, por ejemplo, añadir el crontab para el usuario www-data simplemente lo crearé y lo guardaré en appserver/etc/crontab.www-data
En cuanto al código, por un lado el bloque lo que hace es cargar en un array los ficheros con un nombre que coincida con el patrón comentado anteriormente. Hay que tener en cuenta que el código en ruby se ejecuta en la máquina desde la que se lanza el deploy y lo que se le pasa al comando <em>run</em> o <em>sudo</em> es un comando unix que se va a ejecutar en la máquina en la que se quiera hacer el deploy.

Por otro lado está el hecho de que cuando se ejecutan con sudo dos comandos unidos por una tubería, el sudo se va a aplicar únicamente al primero.
Si se hace:

{% highlight console %}
$ sudo echo '* * * * *  date &gt; /tmp/date' | crontab -u root -
{% endhighlight %}

el sudo se va a aplicar únicamente al comando <em>echo</em> y no al comando <em>crontab</em> por lo que eso no funcionará ya que no tenemos privilegios suficientes.

Por lo tanto para no tener que repetir el comando sudo a los dos lados de la tubería y también para no complicar el comando en capistrano, lo que se puede hacer es englobar toda la sentencia en una subshell de la siguiente forma:

{% highlight console %}
$ sudo sh -c 'echo "* * * * *  date &gt; /tmp/date" | crontab -u root -'
{% endhighlight %}
