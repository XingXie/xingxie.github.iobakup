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

### Eclipse Shortcut:
command+option+R: refactor
command+d delete a line
option+up/down: move this line upper one line/lower one line

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

ln -s path-to-actual-folder name-of-link

to confirm, do:

ls -ld name-of-link

In the log folder one upper level, I type in ln -s /mnt/Documents/ .
Then you will see Documents => /mnt/Documents
