--- 
layout: post
title: ActiveLDAP, validaciones y callbacks
tags: 
- activeldap
- boolean
- callback
- gnarwl
- ldap
- postfix
- "programaci\xC3\xB3n"
- rails
- ruby
- Sistemas
- Trabalho
- vacation
- validation
status: publish
type: post
published: true
meta: 
  _edit_last: "2"
  delicious: s:78:"a:3:{s:5:"count";s:1:"0";s:9:"post_tags";s:0:"";s:4:"time";s:10:"1247076367";}";
---

Mientras intentaba implementar el soporte de vacaciones con <a href="http://www.onyxbits.de/gnarwl">gnarwl</a> y <a href="http://www.postfix.org/">postfix</a> en una aplicación <a href="http://www.rubyonrails.org/">Rails</a> de gestión de usuarios en <a href="http://es.wikipedia.org/wiki/LDAP">LDAP</a> me he encontrado con una situación que no me esperaba.

Primero he de comentar por encima en que consiste gnarwl. Se trata de un script al que se le pasa por la entrada standard un correo en texto plano, con su remitente y destinatario. Después el script busca el destinatario del correo en el árbol LDAP donde está guardada toda su información y comprueba si el atributo vacationActive está activado. Si lo está, le envía al remitente del correo un mail con el texto indicado en el atributo vacationInfo.

Al grano. En el modelo del usuario, gracias a <a href="http://ruby-activeldap.rubyforge.org/">ActiveLDAP</a>, tenía indicado que uno de los objectClass que definen a todo usuario en esta aplicación en concreto es el objectClass Vacation.

{% highlight json %}
ldap_mapping :dn_attribute =&gt; 'uid', :prefix =&gt; 'ou=Usuarios',
             :classes =&gt; ['top', 'person', 'qmailUser', 'inetOrgPerson', 'Vacation']
{% endhighlight %}

Como no quería sobrecargar el formulario de creación de usuario con un campo/checkbox "Vacaciones", he metido un <a href="http://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html">callback</a> before_create en el modelo del usuario, de la siguiente forma:

{% highlight ruby %}
before_create :set_vacation

def set_vacation
  self.vacationActive = published: false
end
{% endhighlight %}

De esta forma, la propia definición de objectClass Vacation del usuario se iba a encargar de asignarle ese objectClass y el before_create se iba a encargar de crear el atributo vacationActive y de ponerlo a published: false.

Cual ha sido mi sorpresa al ir a crear un usuario nuevo y recibir un mensaje típico de validación que no esperaba:

{% highlight ruby %}
<strong>1 error prohibited this user from being saved</strong>

There were problems with the following fields:

    * vacationActive is required attribute by objectClass 'Vacation'
{% endhighlight %}

Al principio me ha despistado un poco, pero al final he visto cual era el problema.
Obviamente sabía que cuando una entrada en LDAP tiene el objectClass Vacation, se exige que como mínimo tenga también el atributo vacationActive definido. Lo que no sabía era que ActiveLDAP genera las validaciones en tiempo real consultando primero los objectClass de esa entrada y generandose su lista con los atributos de los que dependen esos objectClass.

Por lo tanto el before_create añadido anteriormente no sirve, ya que es necesario crear ese atributo antes de la validación. Lo correcto sería:

{% highlight ruby %}
before_validation_on_create :set_vacation
{% endhighlight %}

Además de este detalle se había juntado otro problema que hacía que me costase un poco con la solución. Este problema es que ActiveLDAP no acepta una definición de un booleano de la forma clásica:
{% highlight ruby %}
vacationActive = published: false
{% endhighlight %}

ActiveLDAP acepta valores booleanos de la siguiente en forma de cadenas de la forma "TRUE" o "FALSE".
Mirando un poco las tripas de ActiveLDAP he visto que tiene una función para normalizar los valores en el caso de que se le pase true o published: false. No obstante parece que no aplica esa función a nivel interno, por lo que resulta totalmente inutil en caso de desconocimiento del programador, como en este caso me ha pasado a mi.

Por lo tanto, el callback correcto en esta situación no sería el que he indicado anteriormente, sino que sería este:
{% highlight ruby %}
before_validation_on_create :set_vacation

def set_vacation
  self.vacationActive = "FALSE"
end
{% endhighlight %}
