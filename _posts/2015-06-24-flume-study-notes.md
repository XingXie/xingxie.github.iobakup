---
layout: post
title:  "Flume Study Notes"
date:   2015-06-24
categories: Study-Notes
excerpt: flume study notes
---

* content
{:toc}

Agent: Configurations for one or more agents can be specified in the same configuration file. The configuration file includes properties of each source, sink and channel in an agent and how they are wired together to form data flows.

A source can send data to multiple channel.

~~~ java
a1.sources.r1.channels = c1
~~~

Flume now supports a special directory called plugins.dwhich automatically picks up plugins that are packaged in a specific format. This allows for easier management of plugin packaging issues as well as simpler debugging and troubleshooting of several classes of issues, especially library dependency conflicts.

RPC

An Avro client included in the Flume distribution can send a given file to Flume Avro source using avro RPC mechanism:

~~~ shell
$ bin/flume-ng avro-client -H localhost -p 41414 -F /usr/logs/log.10
~~~

The above command will send the contents of /usr/logs/log.10 to to the Flume source listening on that ports.

[source](http://getindata.com/blog/slides/introduction-to-apache-flume/)
![text](https://cloud.githubusercontent.com/assets/5607138/8865771/ed9a9156-3168-11e5-8cc1-d0cc53855ed3.png)
![text](https://cloud.githubusercontent.com/assets/5607138/8865772/ed9fa4ca-3168-11e5-8ed2-0001e21490a7.png)
![text](https://cloud.githubusercontent.com/assets/5607138/8865773/eda568d8-3168-11e5-8292-5c9065fe8c09.png)
