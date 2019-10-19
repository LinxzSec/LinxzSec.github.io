---
layout: post
title:  "XXE Injection"
categories: Web App
tags: []
draft: true
---

# XXE Injection

XXE Injection is XML External Entity Injection is a web security vulnerability that allows an attacker to interfere with an application's processing of XML data. Often it allows an attacker to view files and interact with backend or external systems that the application can access. In some cases an attacker might be able to escalate an XXE injection to compromise the server or other backend infrastructure by leveraging the XXE to perform server-side forgery (SSRF).

## XML Entities

To understand how XXE works and how it occurs, we must first understand what XML is, what XML entities are, what a custom XML entity is and what an external XML entity is. We will cover this and once understood & covered we will continue on with our understanding of XXE injection.

### What is XML?

XML is Extensible Markup Language - XML is designed for the storing & transportation of data, it's relatively old and JSON is much more favoured in modern applications. XML is a bit like HTML in the sense that it uses a tree-like structure of tags & data however unlike HTML it does not use predefined tags and thus tags can be given names that describe the data being stored for example `<bigMeme>`

### What is an XML Entity?

XML entities are a way of representing an item of data within an XML document instead of using data itself. Various entities are built in to the specification of the XML language for example the entities `&lt;` and `&gt;` represent the characters `<and>`. These are "metacharacters" used to denote XML tags.

### What is Document Type Definition?

Before we discuss XML custom entities and external entities, we need to understand what Document Type Definition or DTD is. So, the XML DTD contains declarations that can define the structure of an XML document and the types of data values it can contain. We declare the DTD within the `<DOCTYPE>` element at the start of an XML document. The DTD can be fully self-contained within the document itself this is known as "internal DTD", it could equally be loaded from somewhere else known as "external DTD" it could also be the hybrid of the two.

### What are XML Custom Entities?

So now that we know what the DTD is, let's talk about custom XML entities. Let's look at an example of a custom entity.

`<!DOCTYPE foo [ <!ENTITY linxz "linxz is a memer" > ]>`

We can now call this custom entity with `&linxz;` within the XML document and it will be replaced with the value of `linxz is a memer` pretty cool!

### What are XML External Entities?

An external entity is like a custom entity however it is defined outside of the DTD unlike a custom entity which we define in the DTD. This allows us to define the entity somewhere else and use a URL to access it - pretty cool! However, external entities are the reason we have XXE based attacks which is not so cool!

We declare an external entity using the `SYSTEM` keyword followed by a URL from where the entity should be loaded an example would be.

`<!DOCTYPE foo [ <!ENTITY ext SYSTEM "http://linxz.co.uk" > ]>`

We could also load from a file using the `file://path` method and it works the same way just its loaded from a file instead :p

## How does an XXE Vulnerability occur?

Some web applications use XML to transmit data between the browser and the server, this is typically done either via a standard library or API which will help to process the XML data. XXEs arise because the XML specification contains various potentially dangerous features and standard parsers support these features even if they are not typically used by the application.

As mentioned, external entities are a type of custom entity which are defined outside of the DTD where they are declared. In short the attack occurs when XML input containing a reference to an external entity is processed by a poorly configured XML parser. There's a few types of XXE attacks, here we will talk about:
    - Exploiting XXE to retrieve files.
    - Exploiting XXE to perform SSRF attacks.
    - Exploiting blind XXE to exfiltrate data out-of-band.
    - Exploiting blind XXE to retrieve data via error messages.

### Exploiting XXE to Retrieve Files

In order to use an XXE Injection attack to retrieve an arbitrary file from the server we need to modify the submitted XML in two ways:
    1. Introduce/Edit a `DOCTYPE` element that defines an external entity containing the path to the file
    2. Edit a data value in the XML that is returned to the applications response in order to make use of the external entity in step 1.

Lets look at an example of XXE against an application that has no defences against an XXE injection attack.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<stockCheck><productId>381</productId></stockCheck>
```

Here we have an XML file which is storing the stock level of an item in the XML, this value will be checked by the application when someone tries to buy an item for example. But for us, we can easily take advantage of this by sending the following to the server.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<stockCheck><productId>&xxe;</productId></stockCheck> 
```

As you can see in the above XML, what we're doing is asking the XML (using an external entity) to give us `/etc/passwd/` back and we're calling the custom entity in the `<productID>` value. In this case our web app is going to return the the following data!

```
Invalid product ID: root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
```

As you can see, we now have the contents of `/etc/passwd` that's good! This type of attack can be quite powerful and is fairly simple to execute! It is worth noting that in the real world an XML application will have **a lot** of data. In order to test for XXE in these cases we must test each data node in the XML individually by making use of our defined external entity.

### Exploiting XXE to perform SSRF Attacks

We've never talked about Server-Side Request Forgery attacks before but simply put an SSRF allows us to induce a server-side application to make HTTP requests to any URL that the server can access, this can be very dangerous and so XXEs that allow us to perform SSRFs can in turn be very dangerous. In order to carry out an XXE to perform an SSRF we use an external entity that defines the URL that we want to target and then use that defined entry in a data value. If we can do this within a data value that is returned in the applications response then we will be able to view the response from the URL and thus gain a two-way interaction with the backend system. If not, we may only be able to perform blind SSRF attacks, which are still critical however they are a bit more difficult for an attacker as he does not get to see the response.

```xml
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "http://internal.linxz.com/"> ]> 
```

In the above example you can see we are defining an external entity which will cause the server to make a HTTP request to an internal system, in this case "internal.linxz.com" within our infrastructure. You can probably now see/understand why a non-blind version of this can be absolutely devestating!

