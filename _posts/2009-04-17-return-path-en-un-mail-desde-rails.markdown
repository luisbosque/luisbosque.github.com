--- 
layout: post
title: Return-Path en un mail desde Rails
tags: 
- actionmailer
- bounced
- General
- mail headers
- postfix
- rails
- sendmail
status: publish
type: post
published: true
meta: 
  _edit_last: "2"
  delicious: s:78:"a:3:{s:5:"count";s:1:"0";s:9:"post_tags";s:0:"";s:4:"time";s:10:"1251897472";}";
---

Pongamos la clásica funcionalidad web en la que un usuario lee una noticia y tiene el típico enlace "Mandar la noticia a un amigo" de forma que dicho amigo va a recibir un mail enviado por un servicio de noticias, pero va a saber que quien se lo ha querido enviar es su colega. Lo que necesito entonces es poder mandar emails desde rails que cumplan las siguientes condiciones:

* La dirección de origen que aparezca en el email, es decir la cabecera <em>From</em> ha de ser una genérica asociada al servicio, como por ejemplo <em>noreply@example.com</em>.
* El <em>Reply-To</em> ha de ser otra dirección, pongamos la del usuario que ha querido enviar el mail a su amigo.
* Quiero estar seguro de que cada email que se envía desde este servicio se envía correctamente y si no lo hace, al menos conocer la razón de porque no ha sido así. Por lo tanto necesito que si falla el envío, el servidor SMTP donde ha fallado sea capaz de devolverme una respuesta para que mi sistema pueda ir parseando emails de respuesta de envios fallidos. Eso sí lo que no quiero es que estas respuestas vayan a parar al buzón del sistema del usuario con el que está arrancada la aplicación web, ya que se me pueden mezclar en ese buzón un montón de cosas y no solo las respuestas de esos envíos concretos de correo. Por lo tanto necesito poder indicarle a rails esa dirección de respuesta de errores. La cabecera que se encarga de proporcionarle está información a los servidores de correo es la de <em>Return-Path</em>. Pongamos que está dirección es <em>fail@example.com</em>

Así que el código de ejemplo de un envío de este tipo sería el siguiente:
En el config/environments/production.rb:

{% highlight ruby %}
config.action_mailer.delivery_method = :sendmail
{% endhighlight %}

y en la clase del ActionMailer::Base:

{% highlight ruby %}
class MessageMailer &lt; ActionMailer::Base
  def message_email(email_data)
    recipients  email_data[:recipient]
    from        "noreply@example.com"
    headers     "Return-Path" =&gt; "fail@example.com"
    reply_to    email_data[:sender_email]
    subject     email_data[:subject]
    body        :body =&gt; email_data[:body]
  end
end
{% endhighlight %}

Si se tienen en la aplicación muchos métodos de envío de correos diferentes, para no tener que indicar siempre el header de Reply-To, se puede poner a mano en el environment como parametro de sendmail:

{% highlight ruby %}
  config.action_mailer.delivery_method = :sendmail
  config.action_mailer.sendmail_settings = {
    :location =&gt; '/usr/sbin/sendmail',
    :arguments =&gt; '-i -t -f fail@example.com'
  }
{% endhighlight %}
