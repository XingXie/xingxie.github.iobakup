---
layout: post
title:  "How to Schedule Oozie Spark Job in Cloudera CDH-5.4.2"
date:   2015-06-25
categories: Solutions
excerpt: Cloudera
comments: true
---

### Where is the debug log?
in / tmp/ logs/ hue/ logs/ application_XXXX_00xx,  you can download it and see why it doe not work
 
### Error: Failing Oozie Launcher, Main class [org.apache.oozie.action.hadoop.SparkMain], exit code [101]
 
 it is because your self-built jar file is not sumitted in the right place. It should be uploaded to this directory when editing the job:
 Jars/py files: click the Workspace, then it will direct you to the right hue-oozie-1xxxxx/xx, choose lib, then upload your jar file.
 
 You can refer to the [LINK](http://blog.cloudera.com/blog/2014/05/how-to-use-the-sharelib-in-apache-oozie-cdh-5/) for setting up your sharing lib.

### How to avoid sbt avoid assembly fail?

add repository and add % "provided"

~~~ sbt

name := "SparkStatistics"

version := "2.0"

scalaVersion := "2.10.5"

autoScalaLibrary := false

// If using CDH, also add Cloudera repo
resolvers ++= Seq(
 "Cloudera Repository" at "https://repository.cloudera.com/artifactory/cloudera-repos/",
 "eigenbase:eigenbase-properties" at "http://repository.pentaho.org/artifactory/repo/",
 "jhyde" at "http://conjars.org/repo"
)

libraryDependencies ++= Seq(
"org.apache.spark" % "spark-assembly_2.10" % "1.3.0-cdh5.4.2" % "provided",
"org.apache.hadoop" % "hadoop-client" % "2.6.0-cdh5.4.2" % "provided",
 "org.xerial.snappy" % "snappy-java" % "1.1.2-RC3",
 "com.github.nscala-time" %% "nscala-time" % "1.8.0",
"com.typesafe" % "config" % "1.3.0" % "provided"
//"org.apache.spark" %% "spark-streaming" % "1.3.0" % "provided",
//"org.apache.spark" %% "spark-core" % "1.3.0" % "provided",
//"net.liftweb" % "lift-json_2.11" % "3.0-M5-1",
//"org.scalaz" %% "scalaz-core_2.10" % "7.1.1"
//"com.datastax.spark" %% "spark-cassandra-connector" % "1.2.0-rc3"
//"com.datastax.spark" %% "spark-cassandra-connector-java" % "1.2.0-rc3"
)
~~~
