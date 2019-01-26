---
layout: post
title:  "HackTheBox - Mischief Writeup"
categories: [HackTheBox]
tags: []
draft: true
---

### Enumeration

#### NMAP TCP

Firstly I ran an nmap scan on the TCP ports, the following is the command I used `nmap -sV -sC -Pn 10.10.10.92` to which I got the following response.

```Starting Nmap 7.70 ( https://nmap.org ) at 2018-12-20 16:37 GMT
Nmap scan report for 10.10.10.92
Host is up (0.026s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 2a:90:a6:b1:e6:33:85:07:15:b2:ee:a7:b9:46:77:52 (RSA)
|   256 d0:d7:00:7c:3b:b0:a6:32:b2:29:17:8d:69:a6:84:3f (ECDSA)
|_  256 3f:1c:77:93:5c:c0:6c:ea:26:f4:bb:6c:59:e9:7c:b0 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

This is certainly interesting, just SSH? That's... Odd. Usually we have more than one service to attack, to which I decided that I should also run a UDP scan as I usually do, the problem with UDP scans is the length of time that they take.

#### NMAP UDP

For the UDP scan I used the following `nmap -sV -sU -sC -Pn 10.10.10.92 --min-rate 10000` I won't paste the entire output here as there is way too much text, however it basically returns that port 161 is open which is SNMP.

```Starting Nmap 7.70 ( https://nmap.org ) at 2018-12-20 16:39 GMT
Nmap scan report for 10.10.10.92
Host is up.
Not shown: 999 open|filtered ports
PORT    STATE SERVICE VERSION
161/udp open  snmp    SNMPv1 server; net-snmp SNMPv3 server (public)
| snmp-info: 
|   enterprise: net-snmp
|   engineIDFormat: unknown
|   engineIDData: b6a9f84e18fef95a00000000
|   snmpEngineBoots: 19
|_  snmpEngineTime: 3d19h52m59s
```

#### NMAP All Ports

Before I get onto enumerating SNMP, I ran a final nmap scan to cover **all** ports, this is quite important, by all ports I mean 1-65535, we don't want to miss anything, I do this on every box however like the UDP scan, it takes a little while so I left this running whilst I did the SNMP enumeration steps, you can see my command for all ports on nmap here `nmap -sV -sC -Pn -p1-65535 10.10.10.92` the output is as follows.

```
Starting Nmap 7.70 ( https://nmap.org ) at 2018-12-24 00:30 GMT
Nmap scan report for 10.10.10.92
Host is up (0.026s latency).
Not shown: 65533 filtered ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 2a:90:a6:b1:e6:33:85:07:15:b2:ee:a7:b9:46:77:52 (RSA)
|   256 d0:d7:00:7c:3b:b0:a6:32:b2:29:17:8d:69:a6:84:3f (ECDSA)
|_  256 3f:1c:77:93:5c:c0:6c:ea:26:f4:bb:6c:59:e9:7c:b0 (ED25519)
3366/tcp open  caldav  Radicale calendar and contacts server (Python BaseHTTPServer)
| http-auth: 
| HTTP/1.0 401 Unauthorized\x0D
|_  Basic realm=Test
|_http-server-header: SimpleHTTP/0.6 Python/2.7.15rc1
|_http-title: Site doesn't have a title (text/html).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Oh look, we have a web server open on port 3366, we will enumerate this later, for now let's get back to the SNMP enumeration.

#### SNMP Enumeration

In order to enumerate SNMP we're first going to use snmpwalk to see what information we can extract, the following command will do this `snmpwalk -v1 -c public 10.10.10.92` I'm not going to past the full output here as there is simply too much information, however, we're looking for the following line.

`iso.3.6.1.2.1.25.4.2.1.5.617 = STRING: "-m SimpleHTTPAuthServer 3366 loki:godofmischiefisloki --dir /home/loki/hosted/"`

There isn't really a fast way of looking for this in the snmpwalk output, of course you could do some basic searches such as the port number or whatever, but it really comes down to just seeing the line, as you can see this looks a bit like a username & password, doeesn't it? Let's try navigating to that website now and see if there is perhaps a login box?

#### Website Enumeration

Let's browse to http://10.10.10.92:3366/ when we visit the page we're presented with a table with two entries, featuring what appears to be two account usernames & passwords, there is also a picture, before I started enumerating the picture I ran a dirb scan with the following command `dirb http://10.10.10.92:3366/` and I left that to run for a while, I got the following response back.


```
-----------------
DIRB v2.22    
By The Dark Raver
-----------------
START_TIME: Mon Dec 24 13:24:03 2018
URL_BASE: http://10.10.10.92:3366/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt
-----------------
GENERATED WORDS: 4612 
---- Scanning URL: http://10.10.10.92:3366/ ----
(!) WARNING: All responses for this directory seem to be CODE = 401.
    (Use mode '-w' if you want to scan it anyway)
-----------------
END_TIME: Mon Dec 24 13:24:15 2018
DOWNLOADED: 101 - FOUND: 0
```

As you can see, there doesn't appear to be any other pages or directories on this web-server, now we're going to continue to enumerate the image and see if we can find a place to use these credentials. Next, I downloaded the image that is displayed on the page and I tried to use steghide on it, some of the boxes are quite CTF like and it's not uncommon to sometimes extract stuff out of the images like this, I also used `strings` on this image to see if anything was hidden in there. I won't paste the full output of strings because there is way too much stuff `strings loki.jpg` will return the following.

```
root@KaliAttacker:~/Documents/HTB/Boxes/Mischief# strings loki.jpg 
JFIF
JExif
Created with The GIMP
```

Anyway, there was nothing useful in strings, next we will use steghide in order to check if there are any files hidden in the image

```
root@KaliAttacker:~/Documents/HTB/Boxes/Mischief# steghide --extract -sf loki.jpg 
Enter passphrase: 
steghide: could not extract any data with that passphrase!
root@KaliAttacker:~/Documents/HTB/Boxes/Mischief# steghide --extract -sf loki.jpg 
Enter passphrase: 
steghide: could not extract any data with that passphrase!

```

I tried all of the passwords that we currently have which is `godofmischiefisloki` & `trickeryanddeceit` however, none of these work so there is nothing hidden in the image, at this point I was a little bit stuck, I tried to ssh with the credentials I currently have however I could not login with that either and at this point we had enumerated the entire website, I usually run nikto but as the website was very bland, I did not bother with this. As I mentioned, this is where I got a little bit stuck, I asked a friend for some help and he said I should check the box creators [GitHub](https://github.com/trickster0/) after scanning through it for a few minutes I found a very interesting [repo](https://github.com/trickster0/Enyx) this is an IPv6 enumeration tool, after looking at the rest of the tools on his GitHub I just assumed this one was the correct tool.

#### Further SNMP Enumeration

Now that we have this tool "Enyx" let's use it, I'm presuming there is going to be an IPv6 address we should be using else the hint would be pointless, in order to run this tool we use the following command `python ipv6enum.py 1 public 10.10.10.92` with that we get the following response from the host.

```
root@KaliAttacker:~/Documents/Tools# python ipv6enum.py 1 public 10.10.10.92
###################################################################################
#                                                                                 #
#                      #######     ##      #  #    #  #    #                      #
#                      #          #  #    #    #  #    #  #                       #
#                      ######    #   #   #      ##      ##                        #
#                      #        #    # #        ##     #  #                       #
#                      ######  #     ##         ##    #    #                      #
#                                                                                 #
#                           SNMP IPv6 Enumerator Tool                             #
#                                                                                 #
#                   Author: Thanasis Tserpelis aka Trickster0                     #
#                                                                                 #
###################################################################################


[+] Snmpwalk found.
[+] Grabbing IPv6.
[+] Loopback -> 0000:0000:0000:0000:0000:0000:0000:0001
[+] Unique-Local -> dead:beef:0000:0000:0250:56ff:feb9:7db6
[+] Link Local -> fe80:0000:0000:0000:0250:56ff:feb9:7db6
```

Ah, there's a dead:beef address, let's do some enumeration on this IPv6 address and see what ports are running we can do this using nmap and the `-6` flag.

#### Further NMAP Enumeration

```
root@KaliAttacker:~/Documents/Tools# nmap -6 -sV -Pn -sC dead:beef:0000:0000:0250:56ff:feb9:7db6
Starting Nmap 7.70 ( https://nmap.org ) at 2018-12-24 16:39 GMT
Nmap scan report for dead:beef::250:56ff:feb9:7db6
Host is up (0.029s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 2a:90:a6:b1:e6:33:85:07:15:b2:ee:a7:b9:46:77:52 (RSA)
|   256 d0:d7:00:7c:3b:b0:a6:32:b2:29:17:8d:69:a6:84:3f (ECDSA)
|_  256 3f:1c:77:93:5c:c0:6c:ea:26:f4:bb:6c:59:e9:7c:b0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: 400 Bad Request
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

As you can see, we have SSH & another web-server running, let's try browsing to this web-page and see what we can see on this web-page.

#### Further Website Enumeration

Remember, this is an IPv6 web-page, in order to browse to it we will need to use square brackets like this `http://[dead:beef:0000:0000:0250:56ff:feb9:7db6]` it's worth noting that this box is using EUI-64 because of this, if the box is reset the IPv6 address will change, so don't try to browse to the same address I have here as it's likely it will not work.

Upon visiting the page we have a login box, we already have some passwords so we know that one of these is likely to be the password for the login however, we don't have a username except "loki" let's try `loki:godofmischiefisloki` doesn't work, let's try `loki:trickeryanddeceit` damn, does not work either. At this point, I did some more digging, tried to find some stuff in Burp Suite (which I won't show here because I don't want to fill the whole post with pictures) however, I could not find anything use Burp, so I decided to just start using default credentials like `admin:admin`, `administrator:administrator`, `admin:password`, `administrator:password` however, none of these worked either, obviously it was getting pretty tiring to try all these manually but I did not want to resort to brute-force because HTB boxes rarely ever require that so I took a step back. After coming back from a break, I realised - I've got two passwords but I don't have a username, I already know the username is not Loki so maybe it's just "admininistrator" with one of the passwords I've been given already, I tried a few different combinations and it turned out to be `administrator:trickeryanddeceit` I was happy, I finally logged in, would I get the user flag now? No, no I would not.

I was greeted by a "command execution panel" with a little note that said `In my home directory, i have my password in a file called credentials, Mr Admin` well, that's easy enough! Let's just use `cat` to read the password out of the home directory, we will assume he means `/home/loki` because Loki is the user. So, I sent `cat /home/loki/credentials/` to which the server replied "Command is not allowed"... Great! I cannot use cat, what should I do then? I scatcheed my head at this for a little while, wondered what else I might be able to do, I tried a few other basic commands just to see what I could do, you know things like `ls`, `pwd`, etc and to my surpirse, `ls` was also disallowed, I could use `pwd` however, the server told me "command executed successfully" but nothing was printed, at this point I was a little confused, why was the host not returning anything?! Well, there is a simple explanation for this.

If we use `pwd` the output is going to be redirected to `/dev/null` that's not useful as we need it to be redirected to `stdout` which essentially is us, so by adding a `;` to the end of the command, the output will get redirected to `stdout` and anything after the `;` will be redirected to `/dev/null` it's a nice little detail actually, it certainly fits with the box given that the God of Mischief is Loki :)

So, now when we run `pwd;` we get the response `/var/www/html Command was executed succesfully!` great! So we know that we can now run some basic commands, however, we still need to read out that password for the user account, at least - that's what it tells us, so how can we do that? Well, the answer is, we need to use a character which Linux will completely ignore which is `\` if we use this character when we execute `ls` or in other words `l\s;` (not forgetting the ;) we will get the following response `assets database.php index.php login.php logout.php Command was executed succesfully!` great! So now we can even execute commands we do not have permission to do, that's interesting, does this mean we can use cat? Well, yes it does. If we do the following `c\at /home/loki/credential\s;` then we should get the password and... Yes we do!

```
pass: lokiisthebestnorsegod Command was executed succesfully!
```

So, techinically, we should have the user account now, let's try to ssh to the device on the IPv4 address (10.10.10.92) and see what happens, we will use `loki` as the username with `ssh -l loki 10.10.10.92` and using the password `lokiisthebestnorsegod` and there we have it, we have the user account, now let's `cat user.txt` and submit the flag and move onto privlege escalation.
