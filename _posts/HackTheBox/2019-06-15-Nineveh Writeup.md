---
layout: post
title:  "HackTheBox - Nineveh Writeup"
categories: HackTheBox
tags: [pentesting]
draft: true
---

# Introduction

This is a writeup for the machine "Nineveh" (10.10.10.43) on the platform HackTheBox. HackTheBox is a penetration testing labs platform so aspiring pen-testers & pen-testers can practice their hacking skills in a variety of different scenarios.

## Enumeration

### NMAP

As always we're going to start off with our full TCP port scan and UDP scan. In this case our UDP scan didn't return anything so we can move on from that. As you can see on this box we have just two ports, 80 & 443.

```
nmap -sV -sC -p- -oA nmap/tcp 10.10.10.43
Starting Nmap 7.70 ( https://nmap.org ) at 2019-06-15 18:32 BST
Nmap scan report for 10.10.10.43
Host is up (0.037s latency).
Not shown: 998 filtered ports
PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=nineveh.htb/organizationName=HackTheBox Ltd/stateOrProvinceName=Athens/countryName=GR
| Not valid before: 2017-07-01T15:03:30
|_Not valid after:  2018-07-01T15:03:30
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1

```




### Web

