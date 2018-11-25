---
layout: post
title:  "Rejetto HFS Remote Command Execution CVE2014-6287"
categories: [Technical]
tags: [command execution, exploit, python]
draft: true
---

### Introduction

On September 12th 2014, a vulnerability found within HttpFileServer was reported that allowed attackers to remotely execute commands on the affected device, this was caused due to poor validation on user input, which would allow an attacker to execute arbitary commands. The bug was reported in the `findMacroMarker()` function in the `parserLib.pas` file.

### So, How does it work?

By performing whats known as [NullByte Injection](http://projects.webappsec.org/w/page/13246949/Null%20Byte%20Injection) we can bypass the sanity checking written in that function thus allowing us to script over HFS without any restrictions in place. This is because the RegEx does not correclty handle the nullbyte characters which is what allows us to escape the filter.

If we send a nullbyte as a paramter when performing a search on the affected device then we can escape the filter of forbidden characters thus allowing us to execute arbitary commands on the device. As an example, take this request here `http://localhost:80/search=%00{.exec|+C:\Windows\system32\ping.exe+10.10.14.11}` as you can see we've got our nullbyte which allows us to bypass the sanity checking, then we're sending a command to tell the machine to run `ping.exe` and ping that IP address. You can learn more about [HFS Scripting Here.](http://www.rejetto.com/wiki/index.php/HFS:_scripting_commands). Now that we know we can execute commands remotely, we can now leverage this in order to setup a reverse shell to the affected device thus giving us access to it with the same privilege level as the account the software is running on.

### Code Review

There's not a massivee amount of code for us to review here however, because HFS is an open-source program we can look at both the affected & fixed code, which is nice! So, let's first look at the vulnerable function within `parserLib.pas`.

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

#### Manual Exploitation

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