--- 
layout: post
title: "LDAP: Introduction"
tags: 
- introduction
- ldap
- openLDAP
- Sistemas
status: publish
type: post
published: true
meta: 
  _edit_last: "2"
---

This is the first post of a <a href="http://www.ietf.org/rfc/rfc2251.txt">LDAP</a> posts collection where I'll try to explain all what I learned about this technology for the last years.

###What is LDAP?
LDAP stands for <em>Lightweight Directory Access Protocol</em>.
LDAP <strong>is not</strong> a database
LDAP <strong>is</strong> A protocol to handle information from a Database
One of the most common applications of LDAP is as an authentication backend for an email server. Next, we can see an example:

<img src="{{ "/images/flujo_ldap_correo.png" | absolute_url }}" alt="flujo_ldap_correo" title="flujo_ldap_correo" width="650" height="168" class="aligncenter size-medium wp-image-338" style="border:none;" />

In this picture a happy user tries to check his email from his personal computer. So he sends his login information to the email server. The email server fetches this data and starts talking with the LDAP server. The LDAP server contains all the users information, so if the login information that it is receiving from the mail server matches the information of any user in its database, it is going to return a positive answer to the email server. At this point, the email server knows that the user sent a valid login information, so it will send him his new emails.

In the picture we can see that the email server doesn't know the physical database where the users information is stored. It only speaks with the LDAP server, and later the LDAP server try to compare the information with the database backend.

LDAP is extremely fast reading and searching information in the database. This is because the elements in a LDAP directory are arranged in a hierarchical tree, so searches are made always downwards. Because of that, LDAP is widely used as an authentication backend for all kind of services, specially the ones with a big amount of users. Write operations are not so fast, but in this kind of applications, users are used to read their personal information from the database, but not to change it.

#### Data types
In spite of being a hierarchical tree, all elements in the tree are equal. An element has one or more objeClasses that define the element purpose. Every objectclass has one or more required attributes and one or more optional attributes. So, in the end, an element belongs to one or more objectClasses and has one or more attributes depending on the objectClasses it belongs to.
For example, en element representing a person has the <em>person</em> objectClass. So, this element is required to define the attributes <em>sn</em> (surname) and <em>cn</em> (commonName). But at the same time it could have, if it wants it to, the <em>telephoneNumber</em> attribute.

There are a lot of schemas that let the chance to load any kind of data in the tree. You can also create your own schema if there's not any schema that satisfies your needs.

#### Structure
Like I said before, a LDAP directory is built with different kinds of elements arranged in a hierarchical tree structure. All the elements have a DN (Distinguished Name) that is an their identity inside the directory. An element DN is built with their own RDN (Relative Distinguished Name) and the RDN of the parent elements to the top element of the tree. This seems a bit confused, so let's explain it better. Let's see a small picture:

<img src="{{ "/images/ldap_diagram2.png" | absolute_url }}" alt="ldap_diagram2" title="ldap_diagram2" width="159" height="184" class="alignleft size-full wp-image-323" style="border:none; margin-right: 15px;" />

This is our directory tree. Let's suppose that I want this directory to authenticate all of the example.com domain users.
Usually when the top element of a tree is a domain, all the parts of the domain name are stored in separated elements. So, <em>com</em> and <em>example</em> are stored in different elements but following the same order.

The <em>com</em> element has the RDN "<em>dc=com</em>". His RDN is made up with an element attribute. This elements belongs to the <em>dcObject</em> objectclass. This objectClass defines that all element belonging to it has to set the <em>dc</em> (domainComponent) attribute. This element has more attributes but, the main one is <em>dc</em> so it is part of the RDN.

That is the same case with the element <em>example</em>. It belongs to <em>dcObject</em> objectclass. So its <em>dc</em> attribute is part of its RDN. There is another element above, so this elemend DN would be the sum of its RDN and the parent RDN, "<em>dc=example,dc=com</em>".

Next we have a simple container. We want to store people information, but we could also want to store, for example, groups information. A group and a person are two different kinds of elements. So in that case we would create two different containers in the tree. One for storing people, and another to store groups.
In this case, we only have the People container. This element belongs to <em>organizationalUnit</em> objectClass. This objectClass force the element to set the <em>ou</em> (organizationalUnitName) attribute. So this element RDN is "ou=People". And its DN is "ou=People,dc=example,dc=com".

Finally we have the "cn=Luis" element. This is a person and it belongs to the <em>person</em> objectClass. Like I said before, the <em>person</em> objectClass force the element to set the <strong>cn</strong> and <strong>sn</strong> attributes. The <em>cn</em> value is "Luis" and the <em>sn</em> value is "Bosque". We can choose any element attribute to make the RDN. We choosed the <em>cn</em> attribute, but could be also the <em>sn</em> attribute.
So, this element RDN is "cn=Luis" and its DN is "cn=Luis,ou=People,dc=example,dc=com".

<strong>Important: </strong>We can't have two elements in the tree with the same DN. So, in this specific case, we couldn't have two people with the name "Luis". It would be possible to have two people with the same attributes but their DN must be different. For example, we can call one of them <em>Luis</em> and the other <em>Luisico</em>.

We can see a bigger example of a LDAP directory tree. This is onle an example. I mean that you can use a LDAP directory to store any kind of information, not only people information. With the right schemas, you can use a directory to create any kind of data structure.

<img src="{{ "/images/ldap_diagram.png" | absolute_url }}" alt="ldap_diagram" title="ldap_diagram" width="726" height="530" class="aligncenter size-full wp-image-330" style="border:none;"/>

#### LDAP Implementations
There are a lot of servers implementing the LDAP protocol. <a href="http://www.openldap.org/">OpenLDAP</a>, <a href="http://en.wikipedia.org/wiki/Active_Directory">Active Directory</a>, <a href="http://www.sun.com/software/products/directory_srvr_ee/dir_srvr/index.xml">Sun ONE Directory Server</a>, <a href="http://www.novell.com/products/edirectory/">Novell Directory Server</a>, <a href="http://www.redhat.com/directory_server/">Red Hat Directory Server</a>, etc...

I already worked with some of them but I'm used to work with OpenLDAP, so, in the next posts I will focus on OpenLDAP.
When you install OpenLDAP, in a Linux machine for example, it creates a simple BDB database with a few example data. S

#### Database Backends
There are also several available database backends to use with OpenLDAP like <a href="http://en.wikipedia.org/wiki/Berkeley_DB">BDB</a>, HDB, <a href="http://en.wikipedia.org/wiki/LDAP_Data_Interchange_Format">LDIF</a>, etc..
When installing OpenLDAP, the server uses a BDB Database. It happens transparently to the user. So, the people that have just met LDAP don't need to care about the difference between protocol and database. 

Just install openldap and start working on it.
