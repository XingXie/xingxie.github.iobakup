---
layout: post
title:  "Flume Study Notes"
date:   2015-06-24
categories: Flume
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
