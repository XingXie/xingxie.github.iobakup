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

## Linux Commends
Source: http://javarevisited.blogspot.com/2011/04/symbolic-link-or-symlink-in-unix-linux.html#ixzz4B6L66N8x

1. find ip with host name
IP address using nslookup command in UNIX or Linux
2. How will you find which operating system your system is running on in UNIX?
By using command "uname -a" in UNIX
3. How do you check how much space left in current drive ?
By using "df" command in UNIX. For example "df -h ." will list how full your current drive is. This is part of anyone day to day activity so I think this Unix Interview question will be to check anyone who claims to working in UNIX but not really working on it.
4. What is the difference between Swapping and Paging?
..* Swapping:
Whole process is moved from the swap device to the main memory for execution. Process size must be less than or equal to the available main memory. It is easier to implementation and overhead to the system. Swapping systems does not handle the memory more flexibly as compared to the paging systems.
..* Paging:
Only the required memory pages are moved to main memory from the swap device for execution. Process size does not matter. Gives the concept of the virtual memory. It provides greater flexibility in mapping the virtual address space into the physical memory of the machine. Allows more number of processes to fit in the main memory simultaneously. Allows the greater process size than the available physical memory. Demand paging systems handle the memory more flexibly.
5. What is difference between ps -ef and ps -auxwww?
where one culprit process was not visible by execute ps –ef command and we are wondering which process is holding the file.
ps -ef will omit process with very long command line while ps -auxwww will list those process as well.

6. How do you find how many cpu are in your system and there details?
cat /proc/cpuinfo

7. What is difference between HardLink and SoftLink in UNIX?
...Hard links cannot link directories.
Cannot cross file system boundaries.
Soft or symbolic links are just like hard links. It allows to associate multiple filenames with a single file. 

...However, symbolic links allows:
To create links between directories.
Can cross file system boundaries.
These links behave differently when the source of the link is moved or removed.
Symbolic links are not updated.
Hard links always refer to the source, even if moved or removed.

..* Using symbolic link: ln -s realfile symfilename

..* Update symbolic link: ln -nsf 1.3 latest

8. What is Zombie process in UNIX? How do you find Zombie process in UNIX?
When a program forks and the child finishes before the parent, the kernel still keeps some of its information about the child in case the parent might need it - for example, the parent may need to check the child's exit status. To be able to get this information, the parent calls 'wait()'; In the interval between the child terminating and the parent calling 'wait()', the child is said to be a 'zombie' (If you do 'ps', the child will have a 'Z' in its status field to indicate this.)
Zombie : The process is dead but have not been removed from the process table.

9. In a file word UNIX is appearing many times? How will you count number?
grep -c "Unix" filename

10. How do you find which processes are using a particular file?
By using lsof command in UNIX. It wills list down PID of all the process which is using a particular file.

11. How do you find which remote hosts are connecting to your host on a particular port say 10123?
By using netstat command execute netstat -a | grep "port" and it will list the entire host which is connected to this host on port 10123.

12. What is nohup in UNIX?
nohup is a special command which is used to run process in background, but it is slightly different than & which is normally used for putting a process in background. An UNIX process started with nohup will not stop even if the user who has stared log off from system. While background process started with & will stop as soon as user logoff.

13. What is ephemeral port in UNIX?
Ephemeral ports are port used by Operating system for client sockets. There is a specific range on which OS can open any port specified by ephemeral port range.

14. There is a file Unix_Test.txt which contains words Unix, how will you replace all Unix to UNIX?
You can answer this Unix Command Interview question by using SED command in UNIX for example you can execute following command to replace all Unix word to UNIX
sed s/Unix/UNIX/g fileName

15. Your application home directory is full? How will you find which directory is taking how much space?
By using disk usage (DU) command in Unix for example du –sh . | grep G  will list down all the directory which has GIGS in Size.

16. How do you find for how many days your Server is up?
By using uptime command in UNIX

17. You have an IP address in your network how will you find hostname and vice versa?
nsloopup 10.150.xxx.xx
