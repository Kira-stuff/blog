---
title: "Space Heroes CTF 2023 - Forensics[Conspiracy Nuts]"
date: 2023-04-24T21:11:46+02:00
author: "Kira"
description: "Writeups du challenge Conspiracy Nuts du Space Heroes CTF 2023"
images: []
resources:
featuredImage: "images/spaceheroesctf/banner_2023.png"
featuredImagePreview: "images/spaceheroesctf/banner_2023.png"

tags: ["writeups", "ctf", "spaceheroesctf"]
categories: ["writeups"]

lightgallery: true

toc:
  auto: false
---


<!--more-->
## Conspiracy Nuts

![Conspiracy Nuts Description](images/spaceheroesctf/conspiracy/title.png "Conspiracy Nuts Description")
Pour ce challenge, nous avons à notre disposition un dump mémoire à analyser afin de retrouver la preuve de cette conspiration 😄 
Le Challenge a eu très peu de solve, ce qui montre qu’il avait une certaine difficulté. Il a également demandé énormément de temps d’analyse(car beaucoup de fausses pistes). Je vais donc synthétiser ici l’analyse.

### Introduction
Je vais utiliser Volatility pour commencer à analyser ce dump. 
J’utilise ici la version Windows standalone de volatility 2.6.

### Vérification du profil de l'image

```cmd
.\volatility_2.6_win64_standalone.exe -f conspiracy_nut.dump imageinfo
```

![Conspiracy Nuts Description](images/spaceheroesctf/conspiracy/vol1.png "Profile Suggestions")
On peut donc partir ici sur un Profil Win7SP1x64

### Début d'investigation

Analyse des process : 
Il semble y avoir quelques processus intéressants comme firefox ou notepad à première vue.

```cmd
.\volatility_2.6_win64_standalone.exe -f conspiracy_nut.dump --profile Win7SP1x64 pslist
```

![Conspiracy Nuts Description](images/spaceheroesctf/conspiracy/vol2.png "Liste des processus")
Mais je décide de d’abord faire une recherche sur les différents fichiers présent.

### Analyse des fichiers du dump avec filescan

Je commence par vérifier avec le module filescan de volatility si le mot flag nous retournes quelques choses d’intéressant :

```cmd
.\volatility_2.6_win64_standalone.exe -f conspiracy_nut.dump --profile Win7SP1x64 filescan
```

Je repère directement le path de l’utilisateur “\Users\tinfoil\”. 
Je décide donc d’améliorer mon filtre pour récupérer les fichiers se trouvant dans les dossiers de l’utilisateur.

En vérifiant le dossier Downloads par exemple :
```cmd
.\volatility_2.6_win64_standalone.exe -f conspiracy_nut.dump --profile Win7SP1x64 filescan | Select-String -Pattern "tinfoil\\Downloads"
```

Cela nous ressort quelques fichiers .jpg intéressant :
![Conspiracy Nuts Description](images/spaceheroesctf/conspiracy/vol3.png "Fichiers .jpg intéressants")

En faisant le dump des fichiers pour vérifier ceux-ci, comme celui-ci par exemple : 

```cmd
.\volatility_2.6_win64_standalone.exe -f conspiracy_nut.dump --profile Win7SP1x64 dumpfiles -Q 0x000000017f0d42b0 -n -u --dump-dir output/
```

Je peux constater que pas mal de fichiers ont l’air corrompus. Cependant, après avoir essayé de les ouvrir avec notepad ou avec strings on peut remarquer ceci à l’intérieur : 

```texte
[ZoneTransfer]
ZoneId=3
ReferrerUrl=http://57.135.219.202:9000/
HostUrl=http://57.135.219.202:9000/Patterson%E2%80%93Gimlin_film_frame_352.jpg
```

Une URL suspecte à première vue qui va orienter mes recherches pendant de longs moments sans pouvoir trouver quelques choses de concret.
Je décide donc de refaire une recherche sur l’extension .jpg afin de voir si d’autres images sont intéressantes à analyser.

```cmd
.\volatility_2.6_win64_standalone.exe -f conspiracy_nut.dump --profile Win7SP1x64 filescan | Select-String -Pattern ".jpg"
```

Un fichier attire mon attention : TranscodeWallpaper.jpg
![Conspiracy Nuts Description](images/spaceheroesctf/conspiracy/vol4.png "Fichiers .jpg Suspect")

En effectuant un dump de celui-ci, je vois que l’image semble corrompue : 
![Conspiracy Nuts Description](images/spaceheroesctf/conspiracy/export_broken.jpg "Fichiers .jpg corrompu")

Le texte visible sur le tableau “Potentially Flat” est assez parlant et me fait penser à la conspiration sur le fait que la terre soit plate. Après quelques tests, je n’arrive malheureusement pas à réparer l’image.

### Dump des processus

Je commence à me dire qu’il serait possible de récupérer cette image pour autant qu’elle ait pu être manipulée sur un des processus existant.

Je commence donc par dump tous les processus firefox pour commencer 1 par 1.
J’utilise la commande suivante pour dump un process (-p = le pid du process) : 
```cmd
.\volatility_2.6_win64_standalone.exe -f conspiracy_nut.dump --profile Win7SP1x64 memdump -p 880 --dump-dir output/
```

Il me fallait maintenant un moyen d’exporter les images de mon dump provenant du processus.
Après quelques recherches sur internet, je tombe sur un script intéressant qui me permettrait d’extraire des images au format jpg d’un dump.

{{< link "https://github.com/stevesbrain/python-image-extractor/blob/master/extract.py" >}}

une fois le script lancé sur mon dump du process firefox ( et après de nombreux essais sur les autres processus firefox, je tombe enfin sur le bon qui est le 880)
On peut voir ici que la même image trouvée tout à l’heure semble bien visible : 
![Conspiracy Nuts Description](images/spaceheroesctf/conspiracy/extract.png "Extraction jpg")

Une fois ouverte … Bingo Le Flag se trouve sur celle-ci 🙂
![Conspiracy Nuts Description](images/spaceheroesctf/conspiracy/flag.png "The Flag")