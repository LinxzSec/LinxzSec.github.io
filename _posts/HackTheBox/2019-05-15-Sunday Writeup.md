---
layout: post
title:  "HackTheBox - Sunday Writeup"
categories: [HackTheBox]
tags: [pentesting]
draft: true
---

# Introduction

## Enumeration

### NMAP

We'll start off with our usual full port nmap scan to see what kinda' stuff is running on the box, I did also run a UDP scan too like usual however again in this case nothing was running on UDP. Below you can see the output from our TCP scan and we have a rather interesting service running which I don't think we've seen before.

```
nmap -sV -sC -p- -oA fullports 10.10.10.76 --max-retries 0
Nmap scan report for 10.10.10.76
Host is up (0.038s latency).
Not shown: 64133 filtered ports, 1398 closed ports
PORT      STATE SERVICE VERSION
79/tcp    open  finger  Sun Solaris fingerd
|_finger: No one logged on\x0D
111/tcp   open  rpcbind
22022/tcp open  ssh     SunSSH 1.3 (protocol 2.0)
| ssh-hostkey: 
|   1024 d2:e5:cb:bd:33:c7:01:31:0b:3c:63:d9:82:d9:f1:4e (DSA)
|_  1024 e4:2c:80:62:cf:15:17:79:ff:72:9d:df:8b:a6:c9:ac (RSA)
58473/tcp open  rpcbind
Service Info: OS: Solaris; CPE: cpe:/o:sun:sunos
```

As you can see have something running called "finger" and we can also see the box is Sun Solaris box which is based off the Linux Kernel, I believe it's actually BSD/open-BSD like but I am not totally sure. I did not know anything about Finger until now so the first thing I searched was "finger enumeration"; the first thing that came up was a page on [Pentest Monkey](http://pentestmonkey.net/tools/user-enumeration/finger-user-enum). I prefer to download from GitHub so I can use `git clone` so I found the same tool on [Pentest Monkeys GitHub](https://github.com/pentestmonkey/finger-user-enum) page by typing "finger enumeration github"

### Finger User Enumeration

Next I started enumerating port 79/finger by using the script I mentioned above. I'd never used this tool before but it was actually pretty easy to use anyway so I won't cover how to use it, the command I run was the following `finger -U /usr/share/seclists/Usernames/Names/names.txt -t 10.10.10.76 | less -S` at first I ran without the pipe to `less` however it was spitting back a lot of very long output so I used `less -S` to clean it up and chop the long lines.

```
access@10.10.10.76: access No Access User                     < .  .  .  . >..nobody4  SunOS 4.x NFS Anonym               < .  .  .  . >..
bin@10.10.10.76: bin             ???                         < .  .  .  . >..
dee dee@10.10.10.76: Login       Name               TTY         Idle    When    Where..dee                   ???..dee                   ???..
jo ann@10.10.10.76: Login       Name               TTY         Idle    When    Where..jo                    ???..ann                   ???..
la verne@10.10.10.76: Login       Name               TTY         Idle    When    Where..la                    ???..verne                 ???..
line@10.10.10.76: Login       Name               TTY         Idle    When    Where..lp       Line Printer Admin                 < .  .  .  . >..
message@10.10.10.76: Login       Name               TTY         Idle    When    Where..smmsp    SendMail Message Sub               < .  .  .  . >..
miof mela@10.10.10.76: Login       Name               TTY         Idle    When    Where..miof                  ???..mela                  ???..
sammy@10.10.10.76: sammy                 pts/2        <Apr 24, 2018> 10.10.14.4          ..
sunny@10.10.10.76: sunny                 pts/3        <Apr 24, 2018> 10.10.14.4          ..
sys@10.10.10.76: sys             ???                         < .  .  .  . >..
zsa zsa@10.10.10.76: Login       Name               TTY         Idle    When    Where..zsa                   ???..zsa                   ???..
```

As you can see there are two very interesting users in the list above `sammy` and `sunny` these are more interesting than the rest because they are the only two that actually have logins, the rest do not. Interestingly, we don't see a root user though, this is odd. Because I couldn't see one in the list I ran another scan just for the username `root` to see what I could see.

``` finger -u "root" -t 10.10.10.76
root@10.10.10.76: root     Super-User            pts/3        <Apr 24, 2018> sunday              ..
```

So we do have a root user, interestingly we see the hostname "sunday" from this we can infer the admin potentially logged in with this locally and the hostname of the device is "sunday". Given that HTB is a bit CTF like, I tried a few username/password combinations for SSH, to my surprise I got in!

### User Exploitation

No real exploitation was/is required for the user flag on this box. There are two ways of doing this however, firstly you can guess the password (like I did) or you could brute-force your way in. I opted with guessing my way in because the word "sunny sunday" was very, very obvious and that got me in over SSH. It is worth noting that when SSHing in you're going to get a key-exchange error, this is because the OS is quite old but don't worry the SSH manpages has a solution to this!

```
ssh sunny@10.10.10.76 -p 22022
Unable to negotiate with 10.10.10.76 port 22022: no matching key exchange method found. Their offer: gss-group1-sha1-toWM5Slw5Ew8Mqkay+al2g==,diffie-hellman-group-exchange-sha1,diffie-hellman-group1-sha1
```

As you can see it gives us a list of key offerings the device has, that's very helpful, I selected `diffie-hellman-group1-sha1` as the key and used the `-okexAlgorithms` flag from the SSH manpages. 

```
ssh -okexAlgorithms=+diffie-hellman-group1-sha1 sunny@10.10.10.76 -p 22022
The authenticity of host '[10.10.10.76]:22022 ([10.10.10.76]:22022)' can't be established.
RSA key fingerprint is SHA256:TmRO9yKIj8Rr/KJIZFXEVswWZB/hic/jAHr78xGp+YU.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[10.10.10.76]:22022' (RSA) to the list of known hosts.
Password: 
Last login: Tue Apr 24 10:48:11 2018 from 10.10.14.4
Sun Microsystems Inc.   SunOS 5.11      snv_111b        November 2008
sunny@sunday:~$ 
```

Yay! We got a login. Now we need to work on our privilege escalation, as we saw before there is another account "sammy" so let's try and get access to that before we go for root.

### Privilege Escalation

