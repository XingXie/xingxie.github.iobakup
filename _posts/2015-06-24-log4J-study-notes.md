---
layout: post
title:  "Log4j Study Notes"
date:   2015-06-24
categories: Study-Notes
comments: true
---

## Different log level use
~~~ xml
 <loggers>    
  <logger name="logexample.Bar" level="TRACE"/>

 <logger name="logexample.MyApp" level="debug"/>
        <root level="error">    
       <AppenderRef ref="Console" />   
    </root>  

  </loggers>
~~~
<!--{: .language-xml}-->

## customized appender

Make sure the packages in configuration includes the self defined appenders. and MessageConsole tab is the name attribute of your MessageConsole

~~~ xml
<?xml version="1.0" encoding="UTF-8"?>    
<configuration status="error" packages="logexample">    
  <appenders>    
    <Console name="Console" target="SYSTEM_OUT">    
      <PatternLayout pattern="%d{HH:mm:ss.SSS} %-5level %class{36} %L %M - %msg%xEx%n"/>    
    </Console> 
        
  <MessageConsole name="MessageConsole">
      <PatternLayout pattern="%d{ISO8601} %-5p [%t] %c: %m %n" />
    </MessageConsole>
  </appenders>    
  <loggers>    
        <root level="error">    
       <AppenderRef ref="MessageConsole" />   
       <AppenderRef ref="Console" />   
    </root>  
  </loggers>    
</configuration>
~~~

## Basics

Properties can be defined by a properties file or by an XML file. Log4j looks for a file named log4j.xml and then for a file named log4j.properties. Both must be placed in the src folder.

The property file is less verbose than an XML file. The XML requires the log4j.dtd to be placed in the source folder as well.  The properties file does not support some advanced configuration options like Filters, custom ErrorHandlers and a special type of appenders, i.e. AsyncAppender. ErrorHandlers defines how errors in log4j itself are handled, for example badly configured appenders. Filters are more interesting. From the available filters, I think that the level range filter is really missing for property files.

This filter allows to define that a appender should receive log messages from Level INFO to WARN. This allows to split log messages across different logfiles. One for DEBUGGING messages, another for warnings, ...
The property appender only supports a minimum level. If you set it do INFO, you will receive WARN, ERROR and FATAL messages as well.

Here are two logfiles examples for a simple configuration:
￼￼￼￼￼￼￼￼￼￼￼￼￼￼￼
### direct log messages to stdout ###
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.SimpleLayout
log4j.rootLogger=debug, stdout

~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd" >
<log4j:configuration>
<appender name="stdout" class="org.apache.log4j.ConsoleAppender">
  <layout class="org.apache.log4j.SimpleLayout"></layout>
</appender>
<root>
  <priority value="debug"></priority>
  <appender-ref ref="stdout"/>
</root>
</log4j:configuration>
~~~

~~~ java
// You can load other configurations as well.
import org.apache.log4j.PropertyConfigurator;
import org.apache.log4j.helpers.Loader;
import org.apache.log4j.xml.DOMConfigurator;
............. snip ...........
// use the loader helper from log4j
URL url = Loader.getResource("my.properties");
PropertyConfigurator.configure(url);
// use the same class loader as your class
URL url = LogClass.class.getResource("/my.properties");
PropertyConfigurator.configure(url);
// load custom XML configuration
URL url = Loader.getResource("my.xml");
DOMConfigurator.configure(url);

// We could change the log level during runtime:
Logger root = Logger.getRootLogger();
root.setLevel(Level.WARN);

// or we can reload the configuration:
// PropertyConfigurator.configure(url);
DOMConfigurator.configure(url);
~~~

Seperate the file output: I want to have debugging messages to one file and other messages to another file. This can only be done with XML because we need a LevelRange filter.
in debug fuile, watch the filter attribute

~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd" >
<log4j:configuration>
<appender name="file"
  class="org.apache.log4j.RollingFileAppender">
  <param name="maxFileSize" value="100KB" />
<param name="maxBackupIndex" value="5" />
  <param name="File" value="test.log" />
  <param name="threshold" value="info" />
  <layout class="org.apache.log4j.PatternLayout">
    <param name="ConversionPattern"
      value="%d{ABSOLUTE} %5p %c{1}:%L - %m%n" />
  </layout>
</appender>
<appender name="debugfile"
  class="org.apache.log4j.RollingFileAppender">
  <param name="maxFileSize" value="100KB" />
  <param name="maxBackupIndex" value="5" />
  <param name="File" value="debug.log" />
  <layout class="org.apache.log4j.PatternLayout">
    <param name="ConversionPattern"
      value="%d{ABSOLUTE} %5p %c{1}:%L - %m%n" />
  </layout>
  <filter class="org.apache.log4j.varia.LevelRangeFilter">
    <param name="LevelMin" value="debug" />
    <param name="LevelMax" value="debug" />
  </filter>
</appender>
<root>
  <priority value="debug"></priority>
  <appender-ref ref="debugfile" />
  <appender-ref ref="file" />
</root>
</log4j:configuration>
~~~

## Work with Tomcat

Hint: If you do not place a configuration file in one of your applications, it will use the Tomcat log files for logging.

Hint: If you do not want Tomcat to use log4j to log but only your application, you can place log4j in the WEB-INF-lib directory of your application as well.

## Best practices for exception logging

* Do not use e.printStackTrace
e.printStackTrace prints to the console. You will only see this messages, if you have defined a console appender. If you use Tomcat or other application server with a service wrapper and define a console appender, you will blow up your wrapper.log. You can use log.error(e,e). The second parameter passed an exception and will print the stack trace into the logfile.

~~~ java
try {
    ......... snip .......
} catch (SomeException e) {
    log.error("Exception Message", e);
    // display error message to customer
}
~~~

* Don't log and throw again
Do not catch an exception, log the stacktrace and then continue to throw it. If higher levels log a message as well, you will end up with a stacktrace printed 2 or more times into the log files.

* Don't kill the stacktrace
This code will erase the stacktrace from the SQLException. This is not recommended, because you will loose important information about the exception. Better do the following.
That's all for this tutorial.

~~~ java
 try{
... some code
    }catch(SQLException e){
        throw new RuntimeException("My Exception name", e);
}￼￼
~~~
