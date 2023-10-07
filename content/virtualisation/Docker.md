---
title: Introduction à Docker 🐳
enableToc: true
openToc: true
tags:
  - devops
  - docker
  - packaging
  - containers
  - FOSS
  - virtualisation
draft: true
---
**Outils/Compétences**

![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white) 

- - -
# Présentation

Docker est un outil permettant de packager vos applications afin de les rendre partageable et exécutable sur n'importe quelle machine. 

Pour atteindre ce but, il propose d'utiliser les conteneurs qui vont renfermer votre application, c'est notamment lui qui a mis cette technologie sur le devant de la scène en 2013 malgré son existence depuis l'an 2000 avec la création de [FreeBSD Jails](https://www.freebsd.org/doc/handbook/jails.html).

Mais comment ça peut marcher ? Quelles sont les différences entre ça et une machine virtuelle?

> [!info]
> 1) Nous ne parlerons pas de [[docker-compose]] car il s'agit d'un outil utilisant docker avec un but différent de ce dernier.
> 2) Partez du principe que ce qui est écrit dans ce cours par rapport au fonctionnement des conteneurs n'est valable QUE pour Linux, vous comprendrez.

## Les Conteneurs

### Qu'est ce que c'est ?

Un conteneur est en réalité composé de 3 choses: 

- ⚙️ **D'un runtime** permettant de lancer un **PROCESSUS** se retrouvant complètement isolé du reste du système, pensant être le processus 1 (également appelé `init`, le père de tous les processus dans une machine Linux) 
- 🗃️ **Une image contenant les fichiers nécessaires** pour lancer l'application
- 🧰 **D'un ensemble d'outils** pour le manipuler (construction, démarrage, arrêt, etc..)

Donc oui en soit un conteneur est presque semblable à un patient en psychiatrie atteint de schizophrénie et du complexe du Messie.

![[crazy.gif]]

> [!warning] Important
> Concrètement ce qu'il faut retenir c'est cette équation: 
> > **1 Conteneur = 1 image + 1 runtime**
>
>et donc 
>
>> **1 Conteneur = 1 image + 1 processus**
> 
>>[!question]- Pourquoi ?
>>Il est possible d'avoir plusieurs processus dans un seul et même conteneur mais cela implique que:
>>1) Vous devrez vous assurer qu'ils soient tous en vie et prêts avec du scripting (ce qui n'est PAS FUN)
>>2) Gérer comment ils se terminent (TOUJOURS PAS FUN)
>>3) Si vous envisagez de mettre à l'échelle horizontalement votre conteneur (cf. le [[scaling]]), vous risquez de gaspiller des cycles CPU et de la mémoire, car les autres processus vont les utiliser.
>>4) La collecte de logs sera plus compliquée
>>5) Et bien d'autres raisons...
>>
>>> [!quote] Citation
>>> Each container addresses a single concern, and does it well.
>>>
>>>\- [le "single concern principle"](https://medium.com/@bibryam/cloud-native-container-design-principles-144ceaa98dba)
>>
>>Si vous vous posez cette question parceque vous voulez faire communiquer 2 conteneurs entre eux, pas de soucis, on y vient.


Vous avez déjà peut être exécuté un conteneur sans le savoir ! Par exemple avec la commande `chroot`

```bash
tar xf alpine-minirootfs.tar.gz -C /tmp/ctr-image
chroot /tmp/ctr-image /bin/sh
# 👆      👆            👆 le processus
# ||      || L'image
# || Le runtime
```

Ici vous avez bien un processus, son runtime et une image. Le problème c'est qu'il n'est isolé que au niveau des processus, il ne voit rien d'autre mais peut communiquer avec les autres processus de votre machine. Mais comment faire?

## Comment ça marche? 
### Comment est isolé un conteneur?

Le kernel Linux contient une fonctionnalité nommée les namespaces, qui permettent à un ensemble de processus à accéder à un ensemble de ressource exclusivement. Il faut les imaginer comme des cloisons qui ont leur propre vue et ne savent pas ce qui se passe dans les autres. 
Les 6 namespaces les plus courants:
- **PID** isole les processus
- **Network** isole les interfaces réseau, les tables de routage etc.
- **User** sépare lels utilisateurs du système
- **UTS (UNIX Time-Sharing)** permet d'avoir des hostname et des noms de domaines séparés 
- **IPC** isole les mécanismes de communication inter-processus tels que les files de messages et les sémaphores
- **Mount** isole les points de montage du système de fichiers pour avoir des systèmes de fichiers séparés

C'est avec ce mécanisme que les conteneurs peuvent s'isoler du reste du système. 

> [!info]
> C'est pour ça que l'on se penche uniquement sur Linux, car les namespaces ne sont pas disponible sous MacOS et Windows, ces derniers ne lancent pas des conteneurs mais des machines virtuelles.


### Les différences entre les conteneurs et les machines virtuelles

Il existe concrètement 2 différences entre conteneurs et machines virtuelles 

#### La nature de ce qui est émulé
   
#### Le niveau d'isolation
   
   Quand vous exécutez une machine virtuelle, la machine émulée est contrôlée par un hyperviseur, ce dernier va se charger d'exécuter chaque instruction de la machine émulée, de limiter l'accès à la mémoire RAM de votre machine physique, de gérer les instructions dites privilégiées etc. Autrement dit l'hyperviseur supporte les machines et les isole de la machine hôte en faisant tampon. 
   ![](https://lh3.googleusercontent.com/ECBRHeUiHBwMPv7q2Cl_zjb9IvpyOK2_CsLE34KI8_T6sMyrcg4I3bm87tQnlqnhDnb-b5XZAWVxdykoXAwl_m_lJoVC8JL5qA5Eg_mMKTfdIOXUWuBPRWv2pf9UAphQb7YJ-mNBZt9bcLoEx4iH19TYLw=nw)
   
   Les conteneurs eux n'étant pas des OS, mais des processus, ils sont plus faiblement isolés, notamment grâce aux namespaces Linux vu ci dessus. C'est la raison pour laquelle les conteneurs n'ont pas besoin de système d'exploitation car ils exploitent la `libc` de l'hôte, cependant il est donc plus facile de sortir du conteneur et donc accéder à l'OS de l'hôte.

### Machine virtuelle ou conteneur? 



# Docker en lui même

## Les Dockerfiles

## La CLI 

# Le docker hub




