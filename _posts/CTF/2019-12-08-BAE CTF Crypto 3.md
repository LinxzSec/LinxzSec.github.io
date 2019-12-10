---
layout: post
title:  "BAE x BSides Cheltenham CTF"
categories: [CTF]
tags: [crypto, cryptanalysis]
draft: true
---

## Introduction

BAE hosted a CTF the day before BSides Cheltenham for the community and I played with our new team elRaptor (formerly DAE). There was a crypto challenge I saw a fair few people struggling with (I think it only got 3 solves in the end and I got the first solve) so I thought I'd write a blog post about how I did it. Also, shout out to the person that drew out ECB mode on paper, although it wasn't the right method that's some serious dedication, haha!

## The Challenge

So, the challenge was some what reminisient of the ECB penguin problem in the sense that we had two picture files in .bmp format, one was unencrypted and the other was encrypted. The respective file names were as follows: `banner.bmp` and `newbanner.bmp.aes`. We were given a `description.txt` file which said the following.

> The administrator of a popular cricketing website was recently updating images on their homepage. During the updates, they accidentally uploaded an image we believe may help the England squad continue their domination of the sport. Unfortunately, the image is encrypted. We do still have a copy of the image that was replaced, if it's any help.
> Original file: banner.bmp
> New file: newbanner.bmp.aes
> He will have used a strong encryption key and we don't have time to brute-force it, but we're not sure what encryption mode he's used. He does seem pretty obsessed with the English Cricket Board!
> Is there any way you can get the information we need?

Our goal was to somehow decrypt that .bmp file and retrieve the flag, it's actually much simpler than you might think! If we do the following on the unencrypted file:

```Bash
hexdump -C banner.bmp | head -n2
00000000  42 4d 76 5c 02 00 00 00  00 00 36 04 00 00 28 00  |BMv\......6...(.|
00000010  00 00 5b 05 00 00 70 00  00 00 01 00 08 00 00 00  |..[...p.........|
```

We see that the image has a `BM` header as it should. If we do the same thing on the encrypted version like below:

```Bash
hexdump -C newbanner.bmp.aes | head -n2
00000000  0e a4 77 dd 6a 96 6a 76  c0 f3 69 18 ac 55 91 e4  |..w.j.jv..i..U..|
00000010  2e 98 dc 4f bc 65 b3 8a  27 bc 6f 67 6f 2c a9 ab  |...O.e..'.ogo,..|
```

We see no such header, let's simply just try adding the header with `dd if=banner.bmp of=newbanner.aes.bmp bs=1 count=54 conv=notrunc` - this will add the `BM` header in for us and we should now be able to open `newbanner.aes.bmp` and the flag is inside the image :)