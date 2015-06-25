---
layout: post
title:  "How to Fix Hue Warning"
date:   2015-06-25
categories: Solution
excerpt: Cloudera
comments: true
---
CDH 5.4.2 and HUE 3.7.

### Spark notebook warning

you need to start livy server

1.switch to the root sudo su root
2.in /opt/cloudera/parcels/CDH/lib/hue, run 

~~~ shell
./build/env/bin/hue livy_server
~~~

### Oozie share lib is not default

1.With CM, click on the Oozie service, then on the 'Actions' button on the
top right and then 'Install Oozie share lib',

in our case, the lib is installing in /vol/oozie not /user/oozie. 
therefore, we need to copy it between hdfs locations. 

2.http://www.cloudera.com/content/cloudera/en/documentation/core/v5-2-x/topics/cdh_ig_oozie_configure.html
login into server, sudo su root

~~~ shell
$ sudo -u hdfs hadoop fs -mkdir /user/oozie
$ sudo -u hdfs hadoop fs -chown hue:hue /user/oozie
~~~

3.in hue UI, browse /vol/hue/, copy the share directory to /user/oozie

4.back to command line,

~~~ shell
sudo -u hdfs hadoop fs -chown -R oozie:oozie /user/oozie
~~~
