---
layout: post
title: Docker, Kafka, Storm et Hadoop HDFS 1
description: "Création d'un environnement Kafka, Storm & Hadoop avec Docker 1: Préparation de l'environnement"
modified: 2015-08-12
categories: [hadoop, docker, kafka, storm]
comments: true
---


Le but de cete suite de posts est de décrire l'installation d'un environnement fonctionnel de Kafka, Storm & Hadoop avec docker.
Nous utiliserons la version 1.8 de docker avec graylog 2 pour la centralisation des logs entre les containers


## Initialisation de l'environnement Docker

- Télécharger DockerToolbox: [DockerToolbox](https://www.docker.com/toolbox)
- Installer l'application en executer le programme d'installation
- Lancer le shell Docker Quickstart Terminal ou dans le cas d'un shell existant
- ou lancer la commande suivante dans un shell existant pour initialiser les variables d'nevironnements docker: `eval $(docker-machine env default --shell=zsh)` (changer le zsh en bash en fonction de votre shell)
  
Vérifier que docker fonctionne correctement:

``` bash
$ docker version
Client:
 Version:      1.8.0
 API version:  1.20
 Go version:   go1.4.2
 Git commit:   0d03096
 Built:        Tue Aug 11 17:17:40 UTC 2015
 OS/Arch:      darwin/amd64

Server:
 Version:      1.8.0
 API version:  1.20
 Go version:   go1.4.2
 Git commit:   0d03096
 Built:        Tue Aug 11 17:17:40 UTC 2015
 OS/Arch:      linux/amd64

```

## Création d'un machine virtuel avec docker-machine

Nous allons récréer la VM default avece 4G de Ram et 60Gb de disque:

``` bash
$ docker-machine stop default
$ docker-machine rm default
$ docker-machine create -d virtualbox --virtualbox-memory 4096 --virtualbox-disk-size 60000 default
$ eval $($DOCKER_MACHINE env default --shell=zsh)
$ docker-machine ls
NAME      ACTIVE   DRIVER       STATE     URL                         SWARM
default   *        virtualbox   Running   tcp://192.168.99.101:2376
$
```

## Création du container Graylog pour centraliser les logs

Nous allons utiliser Graylog 2 pour centraliser les logs des difféerents containers.
Utilisons le container standard pour déployer un système Graylog 2 complet:

``` bash
$ docker run -t -p 19000:9000 -p 12201:12201/udp graylog2/allinone
```

Se connecter à l'interface GrayLog 2: `open http://$(docker-machine ip default):19000`

Aller dnas le menu  System/input et créer un input de type GELF UDP donner lui un nom (ex: Docker Log). Ensuite essayons un container avec la reidrection des lofs vers GrayLog:

```bash
docker run --log-driver=gelf --log-opt gelf-address=udp://$(docker-machine ip default):12201 busybox echo Hello Graylog
```

Vour devriez avoir un nouveau message dans GrayLog:

![GrayLog Message]({{ site.url }}/images/messages.png)
{: .image-left}




