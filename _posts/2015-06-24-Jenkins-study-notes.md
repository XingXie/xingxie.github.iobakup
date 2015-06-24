---
layout: post
title:  "Jenkins Study Notes"
date:   2015-06-24
categories: Jenkins
excerpt: Jenkins study notes
comments: true
---

* content
{:toc}

## Jenkins in MAC

When using launchctl the port will be 8080.

To have launchd start jenkins at login:

~~~ shell
    ln -sfv /usr/local/opt/jenkins/*.plist ~/Library/LaunchAgents
~~~

Then to load jenkins now:

~~~ shell
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist
~~~

Or, if you don't want/need launchctl, you can just run:

~~~ shell
    java -jar /usr/local/opt/jenkins/libexec/jenkins.war
~~~

You can launch it in web browser. http://localhost:8080



## Basic Continuous Integration

* Check the code repo for changes every few minutes
   the build must be automate-able Build or compile the code
* Run your tests: unit, regression, etc
   of course you have to have tests to run!
* Alert if problems
* Gather metrics if appropriate

## ￼Distributed Builds

* Master + slave nodes

Start Master-only, add slaves later

Slaves launched by ssh, WMI+DCOM, custom script, Java Web Start (JNLP)

Slaves can be different platforms Allows build/testing on different OSes

* Jenkins watches the code repo for commits

Check out code

Builds it with your build ‘scripts’

Runs your tests: unit, performance, UI, ...

Reports problems by email, IM, etc

Tracks and graphs metrics and trends Can build artifacts, deploy, etc





