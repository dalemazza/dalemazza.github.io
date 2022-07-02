---
layout: post
author: dalemazza
title: CRTP Review
date: 2022-07-02
category: Blog
tags: Blog
excerpt_separator: <!--more-->
---


<img src="/assets/crtp.jpg" class="align-center" alt="" width="auto" height="auto">

A couple of weeks back I started the Pentester academy Certified Red Team Professional course. The course ended up being the best course I have taken to date, teaching all manor of active directory attacks. The premise of the course is that it is taught on a fully patched network with AV turned on along with real time protection. The course focuses on AD misconfigs and how to abuse them.
<!--more-->

### Quick overview   
  
The course covers many techniques some being:
* Domain Enumeration
* Local Privilege Escalation
* Domain Privilege Escalation
* Persistence
* Trust Attacks
* Powershell
* Bypasses

### Course Content

The course consists of the a 339 page PDF, 14 hours worth of videos, a lab walkthrough manual and an extensive lab consisting of several domains and forests. It also has a CTF style challenges to collect from the lab as you learn.

The PDF is set out in a way that it teaches a particular attack or technique, you are then given a task to complete that on the lab environment. This helps cement the knowledge from the PDF.

The course also has a CTF style challenge. These are additional flags that can be found on your journey through the lab,these are things like user's hash, groups, name of services running and much more.


### The exam

The exam consists of 6 machines including 1 being your initial foothold. To succeed in the exam you have to obtain command execution on 5 machines other than your foothold machine. When ready you simply click start exam, after 15 minutes the exam environment is good to go. You have 25 hours access to the lab then 48 hours after the lab ends to submit your lab report.

After starting the exam I enumerated the domain quite fast as it is not filled with rabbit holes or objects that are not of use. I quickly found myself on the second machine. This is where I was stuck for around 16 hours!! Eventually after adding a flag to my command I found what I was after. A couple of hours later I had access to all 5 servers.

I wrote my exam report which ended up being 20 pages long. It has to include the path you took, tools used and why they were used, screen shots and mitigations on how to protect against abuses/exploits.


### Exam Tips
* Learn mimikatz inside and out!
* Take plenty of screenshots and save them with a meaningful name to refer to later
* Take notes of the path you have taken to help with report writing
* Try having regular breaks as this really helps clearing your thoughts
* The exam is quite linear and doesnt involve going back to previously compromised machines
* Do not rely on bloodhound as the attack path may not show


## Conclusion

One last thing to mention is that everything I used to pass the exam was inside the learning material provided. Overall this was a fantastic experience and really has really helped me drill down on my AD techniques and understand why and how to abuse them.
