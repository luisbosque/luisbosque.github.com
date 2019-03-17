--- 
layout: post
title: Testing Clickhouse as logs analysis storage
status: publish
type: post
published: true
date: 2019-03-17 11:20
tags: 
- work
---

ELK is one of the most popular combined solutions for logs processing. However, after running Elasticsearch in production for several years, these are the pains I've felt:

* It's a complex piece of technology. Also, documentation about internals is not always clear. In general, it requires experts to make the right steps and to try to understand problems and limitations
* Although it provides all the tools needed to scale, sharding, etc.., doing it is far from straightforward, which makes debugging and troubleshooting hard when in front of a capacity problem or any other blocking issue
* Unless much efforts are spent on configuring and maintaining the indexes, mappings, etc.. its performance isn't good when compared with the resources usage. This makes it a costly solution

In my opinion, the reasons why it's a popular option for logs-processing are:

* The default installation and configuration are straightforward. You can have it running very easily
* It's part of a well-crafted log processing ecosystem, together with Logstash and Kibana
* Because of its document-oriented database nature it's very flexible if your logs schema change frequently (changes in "fields")

It's an excellent technology for this use case if you are starting and you need to be fast. Also, if your platform is not very demanding, it's probably ok for you not to spend much time on configuration. 

However, if logs-processing is not your business core, you can't probably invest many resources on acquiring expertise on the technology which makes it a trap. Additionally, if the way you process the logs and the information you store don't change frequently, you won't be taking all the advantages of Elasticsearch.

If you have worked with modern SQL databases you probably know that is few things you cannot do in terms of data queries, aggregations, mathematical functions, etc.. Clickhouse is a SQL database which is great especially if you want to process, store and query temporal events. It's Unix friendly and easy to manage and scale. I wanted to do this PoC to understand if it would make sense to try to replace Elasticsearch with Clickhouse for the use case of log-processing and what would be the trade-offs.

### The experiment

To keep it simple, I've used a pretty common processing stack which starts with Filebeat reading logs and sending them to a Logstash server which finally sends the output to Clickhouse.

<pre>   
    .------------.     .------------.     .------------.
    |            |     |            |     |            |
    |  Filebeat  |---->|  Logstash  |---->| Clickhouse |
    |            |     |            |     |            |
    '------------'     '------------'     '------------'
                                                 ^
                                                 |
                                                 v
                                          .------------.
                                          |            |
                                          |  Grafana   |
                                          |            |
                                          '------------'
</pre>

Inserting data in Clickhouse from Logstash is reasonably simple. Clickhouse provides an HTTP interface which is actually encouraged for production environments. There's also a [output plugin for Clickhouse](https://github.com/mikechris/logstash-output-clickhouse) which is indirectly forked from the Elasticsearch builtin HTTP plugin.

I've used a batch of raw nginx logs, and, I'm sending the most relevant information to Clickhouse. Find [here](https://gist.githubusercontent.com/luisbosque/3f73e53b08de3ec7313ede94fa77a5a9/raw/6559b17ab6f14cf4c1b585d4a5469848f7ee720d/clickhouse-logstash.conf) an example of the Logstash configuration. This [blog post](https://www.altinity.com/blog/2017/12/18/logstash-with-clickhouse) does a good job explaining better this configuration.

The Clickhouse configuration I'm using is the default one.

I've created a MergeTree table in Clickhouse to store the logs. The table is partitioned with the datetimes and the vhost, but this is obviously one of the parts of the experiment that should be adapted depending on each particular case.

{% highlight sql %}
CREATE TABLE grafana.events_vhost ( logdate Date,  logdatetime DateTime,  clientip String,  url String,  verb String,  response Int32,  bytes Int32,  vhost String,  ident String,  response_time Float32,  referrer String,  agent String) ENGINE = MergeTree(logdate, (logdate, vhost), 8192)
{% endhighlight %}

The Logstash plugin inserts the output with the format [JSONEachRow](https://clickhouse.yandex/docs/en/interfaces/formats/#jsoneachrow).

During this test, I've inserted in Clickhouse **80M logs**, one event per row.                                          

Once the logs are stored in the database, you can start sending queries. You don't need anything else because Clickhouse provides all the functions you need to analyze the data. Take a look to [aggregation functions](https://clickhouse-docs.readthedocs.io/en/latest/agg_functions/).

Regarding the graphical interface. For many years I've worked with Kibana as the Elasticsearch frontend. Kibana works only with Elasticsearch data. This means we need to look for an alternative that can be used on top of Clickhouse. Honestly, it's pretty hard to find an equivalent to Kibana that offers similar experience and possibilities. Even less if we talk about an interface for logs exploration and SQL based. The closest alternative, in my opinion, is Grafana.

Fortunately, there's already a [Clickhouse plugin](https://github.com/Vertamedia/clickhouse-grafana) for Grafana. It's pretty straightforward to use and moderately easy to modify if you need to extend it. 

For the sake of simplicity, I've created a basic dashboard with the typical value/time-based graph and a couple of widgets with aggregated values like the number of requests per host and a list of requests ordered by time.

This is the final dashboard:

![grafana-dashboard](https://gist.githubusercontent.com/luisbosque/3f73e53b08de3ec7313ede94fa77a5a9/raw/55c2eb3eeadcfa1b32a9793dddeadc91644e59f6/grafana-clickhouse-dashboard.png)

All widgets are filtered by the time selector and additionally by vhost. If you look at the create table SQL query above you will see that I've used vhost as the index. That's why I've created a filter by vhost. Obviously choosing the right indexes must be done carefully based on your everyday use case and will have a significant impact on the performance of your filters and the experience you have using your dashboard to explore the logs.

To have all the widgets well synchronized you must make sure that every widget contains the same filters. This is the example of the SQL query of the main graph:

{% highlight sql %}
SELECT
    $timeSeries as t,
    count()
FROM $table
WHERE $timeFilter
AND vhost LIKE '$vhost'
GROUP BY t
ORDER BY t
{% endhighlight %}

### Some numbers

Some numbers about the experiments:

* Hosted on a laptop with i7-7500U and 8GB of RAM and SSD disk
* 80M logs (one event per row)
* Queries filtered by data and the custom index takes milliseconds to be processed. Queries not filtered by the custom index will take hundreds of milliseconds or even one second to be processed

### The conclusions

My conclusions of this approach, when compared with the typical ELK are:

* First, as commented, I've used Filebeat and Logstash just because I didn't want to spend time on that part. You can use whatever you want, but if you like these, you've seen that it's perfectly possible.
* Clickhouse (and probably most of the databases out there) is not a big fan of many small data insertions. If you can afford buffering data, so there are less frequent and bigger insertions, Clickhouse will much appreciate it. Actually, the Logstash output used in this PoC allows choosing the number of rows to insert every time.
* MergeTree tables work with [partitions](https://clickhouse.yandex/docs/en/operations/table_engines/custom_partitioning_key/). Clickhouse creates partitions, clearly identifiable looking directly at the filesystem. It's pretty easy to archive old data (after aggregating it, for example, in a different place) without interrupting operations at all.
* It's pretty easy to scale a Clickhouse MergeTree table. There is little magic regarding data sharding and replicas. The performance gain is practically linear which makes it pretty easy to understand how many resources to add to your storage. You just need to understand a bit how a Linux system works and get some metrics.
* Because of the nature of a relational database adding or removing attributes is not as flexible as it can be in Elasticsearch. If tomorrow you need to do a schema change of the data you are storing, obviously you need to perform an operation in Clickhouse. This operation, depending on your approach, can be an ALTER table or just creating a new table. As I commented, operating with Clickhouse is pretty straightforward. You can perfectly implement a blue/green strategy with some extra effort.
* The biggest drawback is mostly related with the flexibility that you have with Kibana vs. Grafana. Kibana is pretty interactive. Building a Grafana dashboard that allows you the same flexibility may take some time and probably some hacks. It's important that you understand well and define exactly what you expect from a logs exploration tool and create the dashboard you need. Obviously, this will require that you choose carefully the better indexing strategy with the MergeTree tables.

