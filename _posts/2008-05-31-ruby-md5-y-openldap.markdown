--- 
layout: post
title: Ruby, MD5 y OpenLDAP
tags: 
- activeldap
- Base64
- digest
- Madrid
- md5
- openLDAP
- rails
- ruby
- SASL
- security
- seguridad
- SSL
- TLS
- Trabalho
- web
status: publish
type: post
published: true
meta: 
  _edit_last: "794513"
  delicious: s:78:"a:3:{s:5:"count";s:1:"1";s:9:"post_tags";s:0:"";s:4:"time";s:10:"1247076448";}";
---

Si lo que quieres es almacenar las contraseñas de usuarios de OpenLDAP encriptadas con el algoritmo MD5, hay que tener en cuenta tres cosas.
La primera es que muchos clientes utilizan un recurso para saber de forma automática en que algoritmo está encriptada la contraseña contra la que van a intentar autenticarse. Este recurso, es añadir el nombre del algoritmo delante de la contraseña, por ejemplo:
<code>{MD5}OFj2IjCsPJFfMAxmQxLGPw==</code>

Si no se le fuerza al cliente a que use un algoritmo u otro para autenticarse, mirará (si el cliente está bien implementado) si la contraseña de LDAP tiene entre llaves el nombre del algoritmo y usará en consecuencia ese algoritmo para enviar la contraseña.

Otro tema  a tener en cuenta, es el tipo de encriptación MD5. Normalmente la mayoría de librerías de MD5 permiten la encriptación de una cadena en formato hexadecimal y en binario. Para OpenLDAP, no nos sirve ninguno de los dos formatos a pelo. Lo que debemos hacer para que la contraseña esté en el formato correcto en el árbol LDAP, es encriptarla en MD5 binario y codificarla después en Base64 para que sea legible. La implementación de esto en ruby, y más concretamente en un modelo de Rails sería así:

{% highlight ruby %}
require 'digest/md5'
require 'base64'
class User &lt; ActiveLdap::Base
  before_save :encrypt_password

  def encrypt_password
    self.userPassword = "{MD5}" + Base64.encode64(Digest::MD5.digest(self.userPassword))
  end
end
{% endhighlight %}

De ActiveLDAP hablaré un poco otro día.

Lo último a tener en cuenta es que el hecho de guardar contraseñas en MD5 en el árbol LDAP no quiere decir que nuestro sistema sea seguro. Sin contar con todos esos temas de desencriptación por fuerza bruta y todo eso, está el tema del envío de la contraseña por un medio no seguro. Si nuestro cliente (podría ser en este caso una web), no está en la misma máquina que el árbol LDAP, al autenticarse con el método SIMPLE, la contraseña viajará en claro por la red hasta el LDAP.
Para evitar esto tenemos dos opciones. O hacemos que las autenticaciones sean con el método SASL (con DIGEST-MD5 por ejemplo) o simplemente le damos soporte SSL/TLS al slapd, de forma que todo tipo de flujo de paquetes entre el servidor y el cliente vayan encriptados.
