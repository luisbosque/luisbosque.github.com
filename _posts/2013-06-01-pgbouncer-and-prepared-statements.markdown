--- 
layout: post
title: PgBouncer and prepared statements
status: publish
type: post
published: true
date: 2013-06-01 10:00
---

Just a small post than can save some time to someone with the same problem.

First of all, some basic concepts:

* PgBouncer is a lightweight connection pooler for PostgreSQL (description taken from [http://wiki.postgresql.org/wiki/PgBouncer](http://wiki.postgresql.org/wiki/PgBouncer)
* PgBouncer can work with different pooling strategies; session, transaction, statement
* In PostgreSQL you can prepare a statement in order to use it later. This way you gain performance since the prepared statement has been already parsed, planned, etc.. It just needs to be executed. This statement only lasts for the duration of the session ([http://www.postgresql.org/docs/9.1/static/sql-prepare.html](http://www.postgresql.org/docs/9.1/static/sql-prepare.html))

At this point it's easy to see where the problem comes. Using prepared statements with transaction or statement pooling strategies in pgbouncer is incompatible.
If your code or any of your application use prepared statements you have three options:

* Stop using prepared statements
* Stop using pgbouncer with your prepared statements ready code
* Think about moving to session pooling strategy in pgbouncer
