---
layout: post
title:  "Linux Commands"
date:   2016-06-09
categories: Study-Notes
excerpt: git
comments: true
---

* content
{:toc}

### Linux Commends

1. find ip with host name
IP address using nslookup command in UNIX or Linux

stock_trader@system:~/test nslookup trading_system
Name:    trading_system.com
Address:  192.24.112.23

2. How will you find which operating system your system is running on in UNIX?
By using command "uname -a" in UNIX

3. How do you check how much space left in current drive ?
By using "df" command in UNIX. For example "df -h ." will list how full your current drive is. This is part of anyone day to day activity so I think this Unix Interview question will be to check anyone who claims to working in UNIX but not really working on it.

4. What is the difference between Swapping and Paging?

Swapping:
Whole process is moved from the swap device to the main memory for execution. Process size must be less than or equal to the available main memory. It is easier to implementation and overhead to the system. Swapping systems does not handle the memory more flexibly as compared to the paging systems.

Paging:
Only the required memory pages are moved to main memory from the swap device for execution. Process size does not matter. Gives the concept of the virtual memory. It provides greater flexibility in mapping the virtual address space into the physical memory of the machine. Allows more number of processes to fit in the main memory simultaneously. Allows the greater process size than the available physical memory. Demand paging systems handle the memory more flexibly.

5. What is difference between ps -ef and ps -auxwww?
where one culprit process was not visible by execute ps –ef command and we are wondering which process is holding the file.
ps -ef will omit process with very long command line while ps -auxwww will list those process as well.

6. How do you find how many cpu are in your system and there details?
cat /proc/cpuinfo

7. What is difference between HardLink and SoftLink in UNIX?

8. What is Zombie process in UNIX? How do you find Zombie process in UNIX?
When a program forks and the child finishes before the parent, the kernel still keeps some of its information about the child in case the parent might need it - for example, the parent may need to check the child's exit status. To be able to get this information, the parent calls 'wait()'; In the interval between the child terminating and the parent calling 'wait()', the child is said to be a 'zombie' (If you do 'ps', the child will have a 'Z' in its status field to indicate this.)
Zombie : The process is dead but have not been removed from the process table.

9. In a file word UNIX is appearing many times? How will you count number?
grep -c "Unix" filename
