--- 
layout: post
title: Exportar passwords de revelation a texto plano con ruby
tags: 
- passwords
- plain text
- "programaci\xC3\xB3n"
- revelation
- ruby
- trabajo
- xml
- xml-simple
status: publish
type: post
published: true
meta: 
  _edit_last: "2"
  delicious: s:78:"a:3:{s:5:"count";s:1:"0";s:9:"post_tags";s:0:"";s:4:"time";s:10:"1251897459";}";
---

<a href="http://oss.codepoet.no/revelation/">Revelation</a> tiene la opción de exportar el fichero de contraseñas a texto plano. El problema es que la organización de las entradas no es demasiado buena, porque las pone en una sola columna sin indentación, por lo que resulta dificil ver el anidación de las contraseñas en caso de que se usen carpetas.

Necesitaba tener las contraseñas en texto plano de forma que pudiese ver a simple vista esta información y fuese más facil de mantener. Esto se puede hacer fácilmente con <a href="http://www.ruby-lang.org/es/">ruby</a> y <a href="http://xml-simple.rubyforge.org/">xml-simple</a>:

{% highlight ruby%}
require 'rubygems'
require 'xmlsimple'

PASSWORD_FILE="./passwords.xml"

config = XmlSimple.xml_in(PASSWORD_FILE)

ce = config

indent = ""
INDENT_SPACES = "  "


def stepin(ce, indent)
  if !ce.nil?
    indent = indent + INDENT_SPACES
  end

  ce['entry'].each { |item|
    puts indent + item['name'].to_s
    if !item['entry'].nil?
      stepin(item, indent)
    elsif item['entry'].nil? &amp;&amp; item['type'] != 'folder'
      if item['description'] &amp;&amp; !item['description'].to_s.empty?
        puts indent + INDENT_SPACES + "description: " + item['description'].to_s
      end
      item['field'].each { |field_type|
        if !field_type['content'].nil?
          puts indent +
               INDENT_SPACES +
               field_type.values[0].gsub("generic-", "") +
               ": " +
               field_type.values[1]
        end
      }
    end
  }
end

stepin(ce, indent)
{% endhighlight %}

Para que esto funcione basta tener instalado <a href="http://rubygems.org/">rubygems</a> y xml-simple.
