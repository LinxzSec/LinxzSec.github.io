---
layout: post
title:  "Samba CVE2007-2447 "
categories: [Technical]
tags: [command execution, exploit, python]
---

### Introduction

On May 7th 2007, a vulnerability found within Samba was reported via email to their security@samba.com mail alias. This bug was initially reported against the anonymous calls to the `SamrChangePassword()` MS-RPC function in combination with the "username map script" `smb.conf` option (which is not enabled by default)

However, after investigation from developers at Samba they found the bug to be much broader noting that it also impacted remote printer & file share management. The root cause is due to passing unfiltered user input provided by MS-RPC calls to `/bin/sh` when invoking external scripts defined in `smb.conf`. Interestingly the vulnerability found with the "username map script" can be exploited without authenticatiton unlike the bug found in remote printer & file share management scripts, those require an authenticated session.

### So, how does it work?


### Code Review


### Exploit


#### Manual Exploitation


#### Metasploit Exploitation


### Python Exploitation



### References

