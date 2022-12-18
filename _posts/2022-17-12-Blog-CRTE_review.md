---
layout: post
author: dalemazza
title: CRTE Review
date: 2022-17-12
category: Blog
tags: Blog
excerpt_separator: <!--more-->
---


<img src="/assets/CRTE1.jpg" class="align-center" alt="" width="auto" height="auto">

Recently I passed the Certified Red Team Expert certification by Pentester Academy. This was a really good course that pushes you to the limit of attacking Active Directory within a realistic fully patched and AV on environment. This course throws you in the deep end with no lab manual and requires you to conduct research on your findings. Here are my thoughts on the course and exam along with some tips. 
<!--more-->

### Quick overview   
  
The course covers many techniques some being:
* Domain Enumeration
* Local Privilege Escalation
* Domain Privilege Escalation
* Double / Triple Hop within PSremote
* Advanced MSSQL attacks
* ACL abuse
* Firewall attacks

### Course Content

The course doesn't give you much material. They give you a few videos and 80 pages in a PDF. This is based on the premise that you already have basic AD knowledge or have completed the CRTP course. This course is very much a challenge lab. It involves a lot of research finding attack vectors that might not stick out at first. Overall the course taught me a lot of cool techniques to add to my arsenal. During the course I really honed my methodology for enumeration, exploitation and persistence.

One cool bit on the course that stood out was attempting a triple hop within Powershell. This was really fun thinking around ways to get as far as I can inside a network far past the traditional 1 hop allowed.

### The exam
For the exam you get 48 hours to get RCE on 5 machines, once this has ended you get a further 48 hours to complete a report detailing everything you had done to achieve it. 48 hours to complete the RCE portion is plenty in my opinion. I had got 5/5 machines with 24 hours still left and I also had a good nights sleep. The exam is a lot easier then the lab was.

You start with a machine attached to the network and have to elevate privileges. One thing here that did take a while was the exam is slightly more up-to date then the lab which meant the AMSI bypass did not work. After spending 30 mins or so looking for one I finally found one that worked. The PE did not take long to find and then the real exam started.

Nothing particularly was hard after this point, nothing that normal enumeration with powerview, powerupsql or mimikatz couldn't find. The only thing that was awkward was the routes you had to take to get around the firewall....until you get access to it anyway ;)

### The report
This is one of the most important parts of the exam and also pen testing in general. You need to make sure you can provide a detailed, accurate report that explains exactly how to replicate what you had achieved within the engagement. For this exam in particular once you have explained the path you took to exploit a machine, you then have to explain the mitigations that can be in place to stop it. Overall my report was 26 pages long.


### Exam Tips
* Take plenty of screenshots and save them with a meaningful name to refer to later
* Take notes of the path you have taken to help with report writing
* Try having regular breaks as this really helps clearing your thoughts
* The exam is quite linear and doesn't involve going back to previously compromised machines
* Do not rely on bloodhound as the attack path may not show
* KISS
* 48 hours is plenty of time do not panic and take your time to enumerate thoroughly


## Conclusion
This has been another fantastic course from Pentester Academy and I recommend them to anyone that really wants to jump into AD and become a good standard.
