---
layout: post
title:  "Cross Site Scripting"
categories: [Web-App]
tags: [vulnerabilities, webapp]
draft: true
---

# Cross Site Scripting

Cross Site Scripting is one of the oldest web application attacks known, it dates back to tWhe 90s where it was possible to control frames within a web page through injected code. Ultimately Cross Site Scripting is an attack in which its ultimate purpose is to inject HTML or run code (JavaScript) in a users web browser. XSS attacks are considered to be attack against a user of a vulnerable site.

## Basic Example

Let's consider the following PHP code:

```php
<?php
echo '<h3>Hello ' . $_GET['name'] . '</h3>';
?>
```

The code above will print a welcome message to a user whose name is retrieved from `$_GET`. If you didn't know, the `$_GET` variable stores `<parameter, value>` pairs which are passed through a HTTP GET. The user input will bbe extracted from the query string in the URL for example `http://linxz.co.uk/welcome.php?name=Linxz` when this is passed to the server the `$_GET` variable will contain a `name` parameter with the value of `Linxz`. We call the `?name=Linxz` the querystring in-case you didn't know. When we make this request the web server will return `<h3>Hello Linxz</h3>` to the browser.

So, how could we do something malicious with this basic example? Well, instead of submitting our name let us submit a JavaScript alert into the querystring for example `http://linxz.co.uk/welcome.php?name=<script>alert('Bad code mate');</script>` when we send this to the web server the web server is going to respond with the following `<h4> Hello </h4> <script>alert('Bad code mate');</script>`. This is a basic example of XSS, it injects some JS code into the websites source code and the JS will be executed in the browser within the context of the website.

Ultimately this happened because of poor sanitisation on both the input and/or output, this is why these vulnerabilities occur and again it illustrates just how important input sanitisation actually is. XSS attacks are possible where the user inputs something and the web application then outputs that input somewhere on the site this then allows the attacker control over the sites rendered content to the users of the application.

## How could all this be used?

XSS can be leveraged in order to achieve a lot of different things, some examples are the following:
    - Cookie Hijacking
    - Complete control over a browser
    - Exploitation against browser plugins into system compromise
    - Keylogging
    - Sensitive Information Disclosure

## Types of XSS

There are different ways in which an attacker can exploit XSS. There are a few classifications of XSS which are pretty standard everywhere:
    - Reflected XSS
    - Stored XSS
    - DOM XSS

### Reflected XSS

Reflected XSS attacks are the simplest & most common form of an XSS vulnerability. In short it occurs when a web application receives data in an HTTP request and includes that data within the immediate response in an unsafe way for example includes what they searched for directly in the response. This vulnerability occurs in the server side code and is dangerous. Let's take an example, say we can make searches for items on our site and what we input is instantly returned to us like this `http://linxz.co.uk/search?term=meme` the application will now echo back what we searched for as follows `<p>Search for: meme</p>`, if we assume the site does not further processing of the data an attacker could inject JavaScript into this search and execute code in the browser for example `http://linxz.co.uk/search?term=<script>alert('Bad Code')</script>` the following response would come back from the server `<p>Search for: <script>alert('Bad Code')</alert></p>`. If another user of the application requests the attackers URL then the script supplied by the attacker will execute in their browser as mentioned.

Reflected XSS is not persistent and is classified as Type II under the OWASP. The effects to a user of a reflected XSS is that if they can control a script that is executed in the victims browser this typically will lead to them being able to fully compromise the user.

### Stored XSS

Stored XSS flaws are similar to Reflected XSS attacks however rather then the malicious input being directly reflected into the response, it is stored within the web application itself. Once this attack occurs, it is echoed on the web application and might be available to any visitors of the site - this is far more useful to an attacker! This type of XSS still occurs in the server side code

### DOM XSS
