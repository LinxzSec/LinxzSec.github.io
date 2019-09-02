---
layout: post
title:  "NetSec Focus BSides Cymru AES Challenge"
categories: [CTF]
tags: [crypto, cryptanalysis]
draft: false
---

# Introduction

Tunny over at NetSec Focused released a CTF challenge in order to win a ticket for BSides Cymru, as the challenge was cryptography I decided that I would give it a go a link to the challenge can be found on the [NetSec Github](https://github.com/NetSec-Focus/bsides-cymru-ctf).

## Enumeration

Firstly we need to work out what the challenge entails, we are told we need to decrypt two binary files and let NetSec Focus know what they say, we are also given a bash script of how the files were encrypted, lets analyse this.

```bash
#!/bin/bash

: '
usage ./cyfuno.sh file file

#tidy up files - work on this later
#head -n 4 file1 > discard.txt
#tail -n +5 file1 > 1.bin
'

#create unique passwords

pass1="$(openssl rand -base64 16)"
pass2="$(openssl rand -base64 16)"

#create unique IVs
iv1="$(openssl rand -hex 8)"
iv2="$(openssl rand -hex 8)"

#echo $pass1
#echo $pass2
#echo $iv1
#echo $iv2

#encrypt files with the unique passwords and IVs
openssl enc -AES-128-CTR -S "$iv1" -pass "pass:$pass1" -in "$1" -out "$1.enc"
openssl enc -AES-128-CTR -S "$iv1" -pass "pass:$pass1" -in "$2" -out "$2.enc"
```

We see that for the script we pass two files, then some interesting "file tidy up" takes place (we'll get back to this later). From here the script generates two passwords of 16 byte length using the `rand` function in openssl and outputs them in base64 format. Then we use `rand` again to generate two IVs of 8 byte length in hex format. If we analyse this we should first note that an 8 byte IV is incorrect, it should actually be 16 bytes to match the block size.

Next we see that the files we provide get encrypted using AES-128 in counter mode, taking the IV we generated and using it as a salt for the password that we generated and then outputs the encrypted file. If you look closely we see a major, major problem; that problem is that for both files we re-use the IV & the password, that is very bad! In counter mode you should never, ever reuse the IV or the key!

Continuing on with the enumeration process, we now have spotted the vulnerability - the encrypted files are the victim of IV & key-reuse, however we still don't know what the first part of the script is doing. We also have an additional file we did not mention before, `discard.txt` if we open this file it features some rather weird content and I must admit this is where I got stuck for a very, very long time.

```
P6
# 
1000 1000
255
```

I could not work out for the life of me what this was, I'd never seen this before, it just looked like junk but after A LOT of Googling, I figured it out. If we switch back to the script for a moment we see that the content for this file comes from the use of `head -n 4 file1 > discard.txt` by putting this together along with the content from `discard.txt` and a lot of Googling, I managed to find [this post on crypto stack exchange](https://crypto.stackexchange.com/questions/63145/variation-on-the-ecb-penguin-problem?rq=1) and this is where I felt a big sigh of relief, "of course! the ecb penguin problem!".

## Attack

So you might be wondering from earlier why we are not allowed to re-use an IV or key in AES-CTR and it is quite simple. In short; the attacker can xor together the two ciphertexts and it recovers the plaintext but here's that in math.

CTR mode is computed as $ C = P \oplus F(Key,IV) $ the main problem is that if you encrypt two plaintexts with the same Key & IV values the attacker gets two pairs:

$ C_1 = P_1 \oplus F(Key, IV) $
$ C_2 = P_2 \oplus F(Key, IV) $

If we can see the values $ C_1 \; \text{and} \; C_2 $ then we can compute the following $ C_1 \oplus C_2 = P_1 \oplus P_2 $ and thus returning the value of the two plaintexts xored together. If we apply this logic here then we will be 50% of the way to decrypting the files. We can Xor the two files together by using a [simple Python script](https://www.megabeets.net/xor-files-python/)with `python xor.py 1.bin.enc 2.bin.enc result.bin` from there we then had a file `result.bin` which we will use in the next step.

Now that we have Xored the two files together we have to do one more thing, apply the same logic from the ECB Penguin problem, we do this by using `cat discard.txt result.bin > flag.ppm` and from here we get the flag :D

![Flag](../../assets/images/NetSecFocus-CTF/flag.PNG)

Was a super fun & kinda' infuriating challenge - thanks to @Tunny for creating this and thanks for the ticket sir! Its most kind of you. Also thanks to @Trev for pointing out what I missed >.<
