---
layout: post
title:  "How to Find Solutions or Debug?"
date:   2015-06-26
categories: Solutions
excerpt: debug
comments: true
---

Many times, in my research and work, I've encountered some problems not easy to figure out. Here is some of my abstract 
steps to take.


### Identify Your Issue ###

When you run a comment line job in a remote machine, and it does not work. It might have several probable issues:

* **Check your log first!** Log can tell exceptions, missing parameters, connection fail, etc. Be sure you know where your log is located, how many logs are exsited, and your log level is open to what (DEBUG, INFO, OR WARN?)

* Analysis the log. Several workflow steps you can follow: 
1. Jar dependencies (some HOME dierectory, output/input file existance)

2. Connection to db/other server with the correct url, path, permission, user/pwd

3. Exception in detailed class, source file

4. resource usage during computation, memory leak, heap size, etc

5. Clean up program, be careful of concurrency

### Find Solution ###

1. Stackoverflow is an extreme resource to start. Configurations, unsolved bugs, and weird issues can be found discussed 
in some mailing archieve. For example, I am using Cloudera CDH 5.4.2 Oozie but cannot run sqoop1 job with upsert support.
No luck on google/stackoverflow, I found the answer in [maillist](https://mail-archives.apache.org/mod_mbox/sqoop-user/201505.mbox/%3C115033115.3593807.1431111624295.JavaMail.yahoo@mail.yahoo.com%3E)

2. If no luck on easy solution, try to find similar setup/programs other people did. Compare their setting/code or 
borrow them in your enviorment, start from the simple helloworld/word-count, and add other codes incrementally. Do not be lazy to refer to the offical document the software provided. It might be discussed in older version, solved or no solved.

3. Minimize your issues, comment out all unneccessary code/jar/task, strike to the target.

4. Last resort: shout loud! Send your questions to authors/companies/communities to seek help. 

### Which Solution? ###

1. You might think most thumbs up in stackoverflow is best. For me, I will try to test alternitives with newer versions.
Solutions with newer publish date might use new tech and better ways.

2. Watch out the trade-off and side-effect. And does it harm my future extensions if applying it?

3. Do you like the solution? If not, spend some more time to find a better one. 
