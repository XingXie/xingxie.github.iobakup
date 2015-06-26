---
layout: post
title:  "Use SBT with Eclipse"
date:   2015-06-25
categories: Installation
excerpt: SBT Eclipse
comments: true
---

## SBT Project Structure

/src/
  main/
    resources/
       <files to include in main jar here>
    scala/
       <main Scala sources>
    java/
       <main Java sources>
  test/
    resources
       <files to include in test jar here>
    scala/
       <test Scala sources>
    java/
       <test Java sources>
build.sbt

## Steps to Build

1.Suppose you have installed sbt. you need to install some plugins. New version might be available. 

1.1.sbt-assembly tool (https://github.com/sbt/sbt-assembly).

For sbt 0.13.6+, in your project, add sbt-assembly as a dependency in project/assembly.sbt:

~~~ sbt
addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.13.0")
~~~

1.2.sbt eclipse 
Open $ ~/.sbt/plugins/build.sbt (Mac/Linux)
Add the following lines (the empty line in between is important)
  
~~~ shell  
addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "2.2.0")
~~~

see https://github.com/typesafehub/sbteclipse 

**sbt run**: to download dependencies

if you want to download sources *addwith-source=true* 

To open the project in Eclipse: File -> Import -> Existing Projects into Workspace

2.Entering the project directory, there is several basic commands:

**sbt update**: when you add new libraries in build.sbt, update the dependencies

**sbt eclipse**: make your dependency changes effective on eclipse IDE

**sbt assmbly** or **sbt -Dscala-2.11=true assembly**: create a fat jar in target/scala-2.11/xxxx.jar

3.After assembly, make sure your static files in src/main/resources/xxx.txt are included in target/scala2.11/class
the jar file should be scp to remote servers

(More Details]( http://www.scala-sbt.org/0.13/tutorial/Directories.html.)

