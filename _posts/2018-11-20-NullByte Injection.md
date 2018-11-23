<<<<<<< HEAD
---
layout: post
title:  "NullByte Injection"
categories: [Attacks]
tags: []
draft: false
---

### Null Byte Injection

Null Byte Injection is an exploit technique whereby an attacker tries to bypass sanity checking filters in web applications. By sending what's known as a "null byte" such as `%00` into the user supplied data. This process can alter the intended logic of the application and allow a malicious adversary to perform tasks that are otherwise not possible.

### What Is A Null Byte?

In C a Null Byte is a reserved character to signify the end of a string, this is typically known as a "null-terminated string". The advantge of this is that it allows a string to be of any length with only the overheard of one byte. You can find more advtanges/disadvtanges of [null-terminatd strings here.](https://en.wikipedia.org/wiki/Null-terminated_string)

In modern character sets the null character has a code point value of zero, for instance in UTF-8 the null character is a single zero byte. If you fail to correctly sanitize user input then it's very likely there could be the chance to perform Null Byte Injection. 

Before we can understand how to leverage a Null Byte to maniuplate a web app, we need to really dive into how Null Bytes work, we're going to use C to explain this so you have a really good idea of how they work. C does not support strings as a distinct primitive data type - in other words, to create a string in C we need to use an array below is an example of what I mean by this.

```
Char Tmp[4]:
Tmp [0] = 'h':
Tmp [1] = 'e':
Tmp [2] = 'h':
Tmp [3] = 'e':
Tmp [4] = '\0':
```

Alternatively, we can use other functions in C like `strcpy` to populate the array however the concept is the same. As C handles strings as a character array, it needs a way to define the last character of a string, this is exactly what we use a null byte for.

```
#include <string.h>
char string[10];
int main() {
  strcpy(string, "foo");
}
```
The actual contents of `char string[10];` would look like the following.

```
string[0] = 'f';
string[1] = 'o';
string[2] = 'o';
string[3] = '\0';
string[4] = null;
string[5] = null;
string[6] = null;
string[7] = null;
string[8] = null;
string[9] = null;
```

As you can see, we have a total of 10 characters, you could say only three of them are "English" that would be 0, 1 & 2, the 3rd character (4th) is our null byte and the rest are set to null. All this "null" means is that we have not explicity written any data to those memory locations, but it doesn't mean they don't contain any value.

As I mentioned earlier, we can use a function like `strcpy` which you can see an example of above. Going back to my first example, when the program runs, it will start reading the string from the first character until it hits the null byte, this could become an issue because some functions might handle the null byte in different & potentially unexpected ways, thus potentially allowing an attacker to leverage poor handling, this is why user input sanitization is so important! (Yeah I'm talking to you, yeah you guys the devs! You know who you are, don't look at your code look at me, get it right!)

### How Can We Execute a Null Byte Injection?

Well, that's a good question! In a URL, a Null Byte is represented with `%00` this is known as a "percent code" or [Percent Encoding](https://en.wikipedia.org/wiki/Percent-encoding). You may be wondering why the web application reacts so badly to this, I mean, we did just say it represents the end of a string. Hah! Well, you'd be right to question that, in the case of web applications they fail to understand (without being told) when to terminate strings and thus vulnerable apps might start to execute things they shouldn't!

Let's look at the following PHP code as an example of how we can exploit a vulnerability using Null Byte Injection. Below you can see some PHP code which will load an image we call.

```
$file = $_GET['file'];
require_once("/var/www/images/$file.jpg");
```

Upon first glance you might think this is relatively secure, I mean after all we are forcing the extension to be `.jpg` in this case our URL extension would look like this ` .php?file=file.jpg` however, what happens if we insert a null byte into this? Now our URL looks like this - `.php?file=[file inclusion here]%00` so why is this code vulnerable? Well, the `.jpg` extension will be appended to the end of your request, however because we've inserted a null byte it will ignore anything that comes after the null byte so all of a sudden the `.jpg` extension becomes futile, if for example we sent this request to the server `.php?file=../../../../../../etc/passwd%00` now we'd have access to that file because it will append the `.jpg` but due to the null byte the extension will simply be well, ignored. For the most part it's hard to carry out a null byte injection in something like PHP6 , it's a lot less likely to be a problem but this attack can work in other languages, Perl is another language we can leverage null byte injections in but I won't talk about that here.

### How Can We Defend Against Null Byte Injection?

Good question! In short; sanitize user input, this is very, very important - users should not dictate what they can and can't input, you should be doing that or rather your code should be doing that to stop issues like this one! A very simple way to combat null byte injection in PHP would be to use the below code.

```
$input = str_replace(chr(0), '', $input);
```

All this will do is simply strip any null bytes out of the user input, it's pretty basic but also could get very tedious to do manually, although it is worth noting some PHP functions strip out null bytes for you whereas other functions do not, it's probably quite important to know which ones do strip null bytes and which ones do not strip null bytes without explicit string manipulation. There is a nice list of vulnerable functions at the bottom of [this blog post.](http://www.madirish.net/401)

### References

[NullByte-WonderHowTo](https://null-byte.wonderhowto.com/how-to/null-byte-injections-work-history-our-namesake-0130141/)

[Vulnerable Functions](http://www.madirish.net/401)





=======
---
layout: post
title:  "NullByte Injection"
categories: [Attacks]
tags: []
draft: false
---

### Null Byte Injection

Null Byte Injection is an exploit technique whereby an attacker tries to bypass sanity checking filters in web applications. By sending what's known as a "null byte" such as `%00` into the user supplied data. This process can alter the intended logic of the application and allow a malicious adversary to perform tasks that are otherwise not possible.

### What Is A Null Byte?

In C a Null Byte is a reserved character to signify the end of a string, this is typically known as a "null-terminated string". The advantge of this is that it allows a string to be of any length with only the overheard of one byte. You can find more advtanges/disadvtanges of [null-terminatd strings here.](https://en.wikipedia.org/wiki/Null-terminated_string)

In modern character sets the null character has a code point value of zero, for instance in UTF-8 the null character is a single zero byte. If you fail to correctly sanitize user input then it's very likely there could be the chance to perform Null Byte Injection. 

Before we can understand how to leverage a Null Byte to maniuplate a web app, we need to really dive into how Null Bytes work, we're going to use C to explain this so you have a really good idea of how they work. C does not support strings as a distinct primitive data type - in other words, to create a string in C we need to use an array below is an example of what I mean by this.

```
Char Tmp[4]:
Tmp [0] = 'h':
Tmp [1] = 'e':
Tmp [2] = 'h':
Tmp [3] = 'e':
Tmp [4] = '\0':
```

Alternatively, we can use other functions in C like `strcpy` to populate the array however the concept is the same. As C handles strings as a character array, it needs a way to define the last character of a string, this is exactly what we use a null byte for.

```
#include <string.h>
char string[10];
int main() {
  strcpy(string, "foo");
}
```
The actual contents of `char string[10];` would look like the following.

```
string[0] = 'f';
string[1] = 'o';
string[2] = 'o';
string[3] = '\0';
string[4] = null;
string[5] = null;
string[6] = null;
string[7] = null;
string[8] = null;
string[9] = null;
```

As you can see, we have a total of 10 characters, you could say only three of them are "English" that would be 0, 1 & 2, the 3rd character (4th) is our null byte and the rest are set to null. All this "null" means is that we have not explicity written any data to those memory locations, but it doesn't mean they don't contain any value.

As I mentioned earlier, we can use a function like `strcpy` which you can see an example of above. Going back to my first example, when the program runs, it will start reading the string from the first character until it hits the null byte, this could become an issue because some functions might handle the null byte in different & potentially unexpected ways, thus potentially allowing an attacker to leverage poor handling, this is why user input sanitization is so important! (Yeah I'm talking to you, yeah you guys the devs! You know who you are, don't look at your code look at me, get it right!)

### How Can We Execute a Null Byte Injection?

Well, that's a good question! In a URL, a Null Byte is represented with `%00` this is known as a "percent code" or [Percent Encoding](https://en.wikipedia.org/wiki/Percent-encoding). You may be wondering why the web application reacts so badly to this, I mean, we did just say it represents the end of a string. Hah! Well, you'd be right to question that, in the case of web applications they fail to understand (without being told) when to terminate strings and thus vulnerable apps might start to execute things they shouldn't!

Let's look at the following PHP code as an example of how we can exploit a vulnerability using Null Byte Injection. Below you can see some PHP code which will load an image we call.

```
$file = $_GET['file'];
require_once("/var/www/images/$file.jpg");
```

Upon first glance you might think this is relatively secure, I mean after all we are forcing the extension to be `.jpg` in this case our URL extension would look like this `.php?file=file.jpg` however, what happens if we insert a null byte into this? Now our URL looks like this `.php?file=[file inclusion here]%00` so why is this code vulnerable? Well, the `.jpg` extension will be appended to the end of your request, however because we've inserted a null byte it will ignore anything that comes after the null byte so all of a sudden the `.jpg` extension becomes futile, if for example we sent this request to the server `.php?file=../../../../../../etc/passwd%00` now we'd have access to that file because it will append the `.jpg` but due to the null byte the extension will simply be well, ignored. For the most part it's hard to carry out a null byte injection in something like PHP6 , it's a lot less likely to be a problem but this attack can work in other languages, Perl is another language we can leverage null byte injections in but I won't talk about that here.

### How Can We Defend Against Null Byte Injection?

Good question! In short; sanitize user input, this is very, very important - users should not dictate what they can and can't input, you should be doing that or rather your code should be doing that to stop issues like this one! A very simple way to combat null byte injection in PHP would be to use the below code.

```
$input = str_replace(chr(0), '', $input);
```

All this will do is simply strip any null bytes out of the user input, it's pretty basic but also could get very tedious to do manually, although it is worth noting some PHP functions strip out null bytes for you whereas other functions do not, it's probably quite important to know which ones do strip null bytes and which ones do not strip null bytes without explicit string manipulation. There is a nice list of vulnerable functions at the bottom of [this blog post.](http://www.madirish.net/401)

### References

[NullByte-WonderHowTo](https://null-byte.wonderhowto.com/how-to/null-byte-injections-work-history-our-namesake-0130141/)

[Vulnerable Functions](http://www.madirish.net/401)





>>>>>>> 999c5cf39297b2bf32a07ffc575f9d9dd319def6
