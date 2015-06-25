---
layout: post
title:  "Coding Tricks, Skills, Shortcuts and Notes"
date:   2015-06-24
categories: handy-notes
excerpt: handy-notes
comments: true
---

* content
{:toc}

### Eclipse Shortcut

* command+option+R: refactor
* command+d delete a line
* option+up/down: move this line upper one line/lower one line

### java doc 
~~~ java 
/**
* describe the function
*<p>
* @parameter: some thing like this
*/
~~~

### symbolic link
move to the directory you want the symbolic link, then:

~~~ shell
ln -s path-to-actual-folder name-of-link
~~~

to confirm, do:

~~~ shell
ls -ld name-of-link
~~~

In the log folder one upper level, I type in 

~~~ shell
ln -s /mnt/Documents/ .
~~~

Then you will see Documents => /mnt/Documents

### Maven build Assembly

1. create a maven project wings-analytics, create a subfolder in modules, under modules, create a new project --> others --> maven module, put the name wings-spark.

2. put the code in wings-spark, change the pom accordingly. (add parents, etc, relative path)

3. copy an assmbly file to the modules, change the pom accordingly. Add the two modules in parent's pom.

4. import two existing maven projects to eclipse. build parent then eclipse:eclipse.
