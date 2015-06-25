---
layout: post
title:  "How to install Android Debug Bridge ADB"
date:   2015-06-24
categories: Android-Debug-Bridge
excerpt: 
comments: true
---

* content
{:toc}

[Instructions to install it on Mac](http://forums.macrumors.com/showthread.php?t=1605300)
[Instructions on how to use ADB](http://developer.android.com/tools/help/adb.html)

To capture a log, connect the tablet to the computer with the USB cable.  To check for the connection, type the command on the terminal:

~~~ shell
adb devices
~~~

Sample output:

~~~ shell
$ adb devices
~~~

List of devices attached
DT08A12345680140328    device

If you do not see the device listed, make sure the Developer option is set with USB debugging enabled in the tablet's Setting.

Next, type the command:

~~~ shell
adb logcat
~~~

The tablet log should start scrolling on the terminal window.
