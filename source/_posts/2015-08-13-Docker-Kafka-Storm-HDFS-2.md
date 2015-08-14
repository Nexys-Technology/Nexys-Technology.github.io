---
layout: post
title: Docker, Kafka, Storm et Hadoop HDFS 2éme partie
description: "Création d'un environnement Kafka, Storm & Hadoop avec Docker 2: Création des containers"
modified: 2015-08-13
categories: [hadoop, docker, kafka, storm]
comments: true
lang: fr
---

Dans cette partie nous allons déployer les containers dockers pour Kafka, Storm 
et Hadoop avec l'aide de docker-compose.

<!--more-->

## Docker Compose


Nous allons décrire l'ensemble des services sous la forme d'une appliaction docker compose.
C'est la configuration de plusieurs container Docker décrit dans un fichier docker-compose.yml.

Voila le fichier docker-compose.yml:

```  yml docker-compose.yml
zookeeper:
  image: wurstmeister/zookeeper
  ports: 
    - "2181:2181"
    - "22222:22"
hadoop:
  build: ./hadoop-docker
  #image: sequenceiq/hadoop-docker:2.7.0
  expose:
    - "9000"
    - "50010"
  ports:
    # HDFS Ports
#    - "9000:9000"
#    - "50010:50010"
#    - "50020:50020"
#    - "50070:50070"
#    - "50075:50075"
#    - "50090:50090"
    # MapRed ports
#    - "19888"
    # Yarn Ports
#    - "8030"
#    - "8031"
#    - "8032"
#    - "8033"
#    - "8040"
#    - "8042"
#    - "8088"
    # Ssh
    - "22227:2122"
    # Other
#    - "49707"
kafka:
  image: wurstmeister/kafka:0.8.2.0
  ports:
    - "9092:9092"
    - "22223:22"
  links:
    - zookeeper:zk
  environment:
    KAFKA_ADVERTISED_HOST_NAME: 192.168.99.100
  volumes:
    - /var/run/docker.sock:/var/run/docker.sock
nimbus:
  image: wurstmeister/storm-nimbus
  ports:
    - "49773:3773"
    - "49772:3772"
    - "49627:6627"
    - "22224:22"
  links: 
    - zookeeper:zk
    - kafka:kf
    - hadoop:hp
supervisor:
  image: wurstmeister/storm-supervisor
  ports:
    - "8000"
    - "22225:22"
    - "6703:6703"
  links: 
    - nimbus:nimbus
    - zookeeper:zk
    - kafka:kf
    - hadoop:hp
ui:
  image: wurstmeister/storm-ui
  ports:
    - "49080:8080"
    - "22226:22"
  links: 
    - nimbus:nimbus
    - zookeeper:zk
```

Récuperer le repository git:

``` bash
$ git clone https://github.com/Nexys-Technology/docker-kafka-storm-hdfs.git
$ cd docker-kafka-storm-hdfs
```

Démarrer les containers avec docker-compose. Lors du premier lancement docker va récupérer les images par téléchargement donc c'est le bon moment pour une pause ;):

```
~/docker-kafka-storm-hdfs $ docker-compose
Creating ksh_zookeeper_1...
Creating ksh_hadoop_1...
Creating ksh_kafka_1...
Creating ksh_nimbus_1...
Creating ksh_ui_1...
Creating ksh_supervisor_1...
Attaching to ksh_zookeeper_1, ksh_hadoop_1, ksh_kafka_1, ksh_nimbus_1, ksh_ui_1, ksh_supervisor_1
zookeeper_1  | JMX enabled by default
zookeeper_1  | Using config: /opt/zookeeper-3.4.6/bin/../conf/zoo.cfg
zookeeper_1  | 2015-08-14 15:34:53,120 [myid:] - INFO  [main:QuorumPeerConfig@103] - Reading configuration from: /opt/zookeeper-3.4.6/bin/../conf/zoo.cfg
zookeeper_1  | 2015-08-14 15:34:53,137 [myid:] - INFO  [main:DatadirCleanupManager@78] - autopurge.snapRetainCount set to 3
zookeeper_1  | 2015-08-14 15:34:53,142 [myid:] - INFO  [main:DatadirCleanupManager@79] - autopurge.purgeInterval set to 1
zookeeper_1  | 2015-08-14 15:34:53,149 [myid:] - WARN  [main:QuorumPeerMain@113] - Either no config or no quorum defined in config, running  in standalone mode
zookeeper_1  | 2015-08-14 15:34:53,170 [myid:] - INFO  [PurgeTask:DatadirCleanupManager$PurgeTask@138] - Purge task started.
zookeeper_1  | 2015-08-14 15:34:53,203 [myid:] - INFO  [main:QuorumPeerConfig@103] - Reading configuration from: /opt/zookeeper-3.4.6/bin/../conf/zoo.cfg
hadoop_1     | /
hadoop_1     | Starting sshd: [  OK  ]
zookeeper_1  | 2015-08-14 15:34:53,422 [myid:] - INFO  [PurgeTask:DatadirCleanupManager$PurgeTask@144] - Purge task completed.
zookeeper_1  | 2015-08-14 15:34:53,438 [myid:] - INFO  [main:ZooKeeperServerMain@95] - Starting server
ui_1         | /usr/lib/python2.7/dist-packages/supervisor/options.py:295: UserWarning: Supervisord is running as root and it is searching for its configuration file in default locations (including its current working directory); you probably want to specify a "-c" argument specifying an absolute path to a configuration file for improved security.
ui_1         |   'Supervisord is running as root and it is searching '
nimbus_1     | /usr/lib/python2.7/dist-packages/supervisor/options.py:295: UserWarning: Supervisord is running as root and it is searching for its configuration file in default locations (including its current working directory); you probably want to specify a "-c" argument specifying an absolute path to a configuration file for improved security.
ui_1         | 2015-08-14 15:34:53,324 CRIT Supervisor running as root (no user in config file)
nimbus_1     |   'Supervisord is running as root and it is searching '
ui_1         | 2015-08-14 15:34:53,325 WARN Included extra file "/etc/supervisor/conf.d/storm-ui.conf" during parsing
```

Si docker-compose à réussi à démarrer les containers on peux vérifier qu'ils sont bien lancés par la commande suivante:

```
~/docker-kafka-storm-hdfs $ docker-compose ps
                   Name                                           Command                                          State                                           Ports
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
ksh_hadoop_1                                    /etc/bootstrap.sh -d                            Up                                              19888/tcp, 0.0.0.0:22227->2122/tcp,
                                                                                                                                                49707/tcp, 50010/tcp, 50020/tcp, 50070/tcp,
                                                                                                                                                50075/tcp, 50090/tcp, 8030/tcp, 8031/tcp,
                                                                                                                                                8032/tcp, 8033/tcp, 8040/tcp, 8042/tcp,
                                                                                                                                                8088/tcp, 9000/tcp
ksh_kafka_1                                     /bin/sh -c start-kafka.sh                       Up                                              0.0.0.0:22223->22/tcp, 0.0.0.0:9092->9092/tcp
ksh_nimbus_1                                    /bin/sh -c /usr/bin/start- ...                  Up                                              0.0.0.0:22224->22/tcp,
                                                                                                                                                0.0.0.0:49772->3772/tcp,
                                                                                                                                                0.0.0.0:49773->3773/tcp,
                                                                                                                                                0.0.0.0:49627->6627/tcp
ksh_supervisor_1                                /bin/sh -c /usr/bin/start- ...                  Up                                              0.0.0.0:22225->22/tcp, 6700/tcp, 6701/tcp,
                                                                                                                                                6702/tcp, 0.0.0.0:6703->6703/tcp,
                                                                                                                                                0.0.0.0:32772->8000/tcp
ksh_ui_1                                        /bin/sh -c /usr/bin/start- ...                  Up                                              0.0.0.0:22226->22/tcp,
                                                                                                                                                0.0.0.0:49080->8080/tcp
ksh_zookeeper_1                                 /bin/sh -c /usr/sbin/sshd  ...                  Up                                              0.0.0.0:2181->2181/tcp,
                                                                                                                                                0.0.0.0:22222->22/tcp, 2888/tcp, 3888/tcp
```

Dans une autre fenétre terminal démarrer un shell Kafka dans un container avec la commande suivante et créer le topic Kafka:

```
~/docker-kafka-storm-hdfs $ ./start-kafka-shell.sh $(docker-machine ip default) $(docker-machine ip default):2181
root@09c715741c84:/# $KAFKA_HOME/bin/kafka-topics.sh --create --topic truckevent --partitions 1 --zookeeper $ZK --replication-factor 1
Created topic "truckevent".
root@09c715741c84:/# $KAFKA_HOME/bin/kafka-topics.sh --describe --topic truckevent --zookeeper $ZK
Topic:truckevent    PartitionCount:1    ReplicationFactor:1 Configs:
    Topic: truckevent   Partition: 0    Leader: 9092    Replicas: 9092  Isr: 9092
root@09c715741c84:/#
```

Transférer le traitement Storm dans le container Nimbus (le mot de passe est *wurstmeister*):

```
~/docker-kafka-storm-hdfs $ scp -P 22224 TruckEvents/Tutorials-master/target/Tutorial-1.0-SNAPSHOT.jar root@$(docker-machine ip default):/tmp
Tutorial-1.0-SNAPSHOT.jar                                                                                                                                      100%  102MB 101.7MB/s   00:01
~/docker-kafka-storm-hdfs $ 
```

Entrer dans un shell du container Storm Numbus

```
~/docker-kafka-storm-hdfs $ docker exec -it ksh_nimbus_1 /bin/bash
root@cd60947fb0aa:/#
root@cd60947fb0aa:/# cd /tmp
root@cd60947fb0aa:/tmp# ls -l
total 104168
-rw-r--r-- 1 root root 106659847 Aug 14 16:58 Tutorial-1.0-SNAPSHOT.jar
drwxr-xr-x 2 root root      4096 Aug 14 16:40 hsperfdata_root
root@cd60947fb0aa:/tmp# storm jar Tutorial-1.0-SNAPSHOT.jar com.hortonworks.tutorials.tutorial3.TruckEventProcessingTopology
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/opt/apache-storm-0.9.4/lib/logback-classic-1.0.13.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/tmp/Tutorial-1.0-SNAPSHOT.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [ch.qos.logback.classic.util.ContextSelectorStaticBinder]
Running: /usr/lib/jvm/java-7-openjdk-amd64/bin/java -client -Dstorm.options= -Dstorm.home=/opt/apache-storm-0.9.4 -Dstorm.log.dir=/opt/apache-storm-0.9.4/logs -Djava.library.path=/usr/local/lib:/opt/local/lib:/usr/lib -Dstorm.conf.file= -cp /opt/apache-storm-0.9.4/lib/carbonite-1.4.0.jar:/opt/apache-storm-0.9.4/lib/tools.macro-0.1.0.jar:/opt/apache-storm-0.9.4/lib/jetty-6.1.26.jar:/opt/apache-storm-0.9.4/lib/reflectasm-1.07-shaded.jar:/opt/apache-storm-0.9.4/lib/clj-stacktrace-0.2.2.jar:/opt/apache-storm-0.9.4/lib/joda-time-2.0.jar:/opt/apache-storm-0.9.4/lib/objenesis-1.2.jar:/opt/apache-storm-0.9.4/lib/hiccup-0.3.6.jar:/opt/apache-storm-0.9.4/lib/ring-jetty-adapter-0.3.11.jar:/opt/apache-storm-0.9.4/lib/asm-4.0.jar:/opt/apache-storm-0.9.4/lib/tools.cli-0.2.4.jar:/opt/apache-storm-0.9.4/lib/tools.logging-0.2.3.jar:/opt/apache-storm-0.9.4/lib/log4j-over-slf4j-1.6.6.jar:/opt/apache-storm-0.9.4/lib/logback-classic-1.0.13.jar:/opt/apache-storm-0.9.4/lib/jgrapht-core-0.9.0.jar:/opt/apache-storm-0.9.4/lib/disruptor-2.10.1.jar:/opt/apache-storm-0.9.4/lib/logback-core-1.0.13.jar:/opt/apache-storm-0.9.4/lib/commons-fileupload-1.2.1.jar:/opt/apache-storm-0.9.4/lib/ring-devel-0.3.11.jar:/opt/apache-storm-0.9.4/lib/compojure-1.1.3.jar:/opt/apache-storm-0.9.4/lib/kryo-2.21.jar:/opt/apache-storm-0.9.4/lib/commons-exec-1.1.jar:/opt/apache-storm-0.9.4/lib/storm-core-0.9.4.jar:/opt/apache-storm-0.9.4/lib/commons-logging-1.1.3.jar:/opt/apache-storm-0.9.4/lib/jline-2.11.jar:/opt/apache-storm-0.9.4/lib/ring-servlet-0.3.11.jar:/opt/apache-storm-0.9.4/lib/core.incubator-0.1.0.jar:/opt/apache-storm-0.9.4/lib/jetty-util-6.1.26.jar:/opt/apache-storm-0.9.4/lib/snakeyaml-1.11.jar:/opt/apache-storm-0.9.4/lib/math.numeric-tower-0.0.1.jar:/opt/apache-storm-0.9.4/lib/slf4j-api-1.7.5.jar:/opt/apache-storm-0.9.4/lib/clj-time-0.4.1.jar:/opt/apache-storm-0.9.4/lib/commons-codec-1.6.jar:/opt/apache-storm-0.9.4/lib/clout-1.0.1.jar:/opt/apache-storm-0.9.4/lib/servlet-api-2.5.jar:/opt/apache-storm-0.9.4/lib/clojure-1.5.1.jar:/opt/apache-storm-0.9.4/lib/commons-lang-2.5.jar:/opt/apache-storm-0.9.4/lib/commons-io-2.4.jar:/opt/apache-storm-0.9.4/lib/json-simple-1.1.jar:/opt/apache-storm-0.9.4/lib/minlog-1.2.jar:/opt/apache-storm-0.9.4/lib/chill-java-0.3.5.jar:/opt/apache-storm-0.9.4/lib/ring-core-1.1.5.jar:Tutorial-1.0-SNAPSHOT.jar:/opt/apache-storm-0.9.4/conf:/opt/apache-storm-0.9.4/bin -Dstorm.jar=Tutorial-1.0-SNAPSHOT.jar com.hortonworks.tutorials.tutorial3.TruckEventProcessingTopology
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/opt/apache-storm-0.9.4/lib/logback-classic-1.0.13.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/tmp/Tutorial-1.0-SNAPSHOT.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [ch.qos.logback.classic.util.ContextSelectorStaticBinder]
835  [main] INFO  backtype.storm.StormSubmitter - Jar not uploaded to master yet. Submitting jar...
846  [main] INFO  backtype.storm.StormSubmitter - Uploading topology jar Tutorial-1.0-SNAPSHOT.jar to assigned location: storm-local/nimbus/inbox/stormjar-d1a5a232-ca72-44b1-a0de-34a7b28e73e8.jar
1351 [main] INFO  backtype.storm.StormSubmitter - Successfully uploaded topology jar to assigned location: storm-local/nimbus/inbox/stormjar-d1a5a232-ca72-44b1-a0de-34a7b28e73e8.jar
1351 [main] INFO  backtype.storm.StormSubmitter - Submitting topology truck-event-processor in distributed mode with conf {"topology.debug":true}
2379 [main] INFO  backtype.storm.StormSubmitter - Finished submitting topology: truck-event-processor
root@cd60947fb0aa:/tmp#
```

Ouvrir Storm UI dans un navigateur Web:

```
~/docker-kafka-storm-hdfs $ open http://192.168.99.100:49080
```

Nous devons avoir une topology visible *truck-event-processor*. Elle ne doit pas avoir d'erreur (cliquer sur lien
pour avoir le détail de la topology dans l'UI).

Maintenant nous allons générer des messages Kafka:

```
~/docker-kafka-storm-hdfs $ java -cp TruckEvents/Tutorials-master/target/Tutorial-1.0-SNAPSHOT.jar com.hortonworks.tutorials.tutorial1.TruckEventsProducer $(docker-machine ip default):9092 $(docker-machine ip default):2181
15/08/14 19:07:20 INFO utils.VerifiableProperties: Verifying properties
15/08/14 19:07:20 INFO utils.VerifiableProperties: Property metadata.broker.list is overridden to 192.168.99.100:9092
15/08/14 19:07:20 WARN utils.VerifiableProperties: Property zk.connect is not valid
15/08/14 19:07:20 INFO utils.VerifiableProperties: Property request.required.acks is overridden to 1
15/08/14 19:07:20 INFO utils.VerifiableProperties: Property serializer.class is overridden to kafka.serializer.StringEncoder
1
1
1
1
15/08/14 19:07:20 INFO tutorial1.TruckEventsProducer: Sending Messge #: route17: 0, msg:2015-08-14 19:07:20.962|1|11|Normal|-79.762351999999964|42.131190999999944
15/08/14 19:07:21 INFO client.ClientUtils$: Fetching metadata from broker id:0,host:192.168.99.100,port:9092 with correlation id 0 for 1 topic(s) Set(truckevent)
15/08/14 19:07:21 INFO producer.SyncProducer: Connected to 192.168.99.100:9092 for producing
15/08/14 19:07:21 INFO producer.SyncProducer: Disconnecting from 192.168.99.100:9092
15/08/14 19:07:21 INFO producer.SyncProducer: Connected to 192.168.99.100:9092 for producing
15/08/14 19:07:22 INFO tutorial1.TruckEventsProducer: Sending Messge #: route17k: 0, msg:2015-08-14 19:07:22.194|2|12|Normal|-74.021358|41.500703
15/08/14 19:07:23 INFO tutorial1.TruckEventsProducer: Sending Messge #: route208: 0, msg:2015-08-14 19:07:23.201|3|13|Normal|-74.192248999999947|41.333772999999915
15/08/14 19:07:24 INFO tutorial1.TruckEventsProducer: Sending Messge #: route27: 0, msg:2015-08-14 19:07:24.207|4|14|Lane Departure|-73.995427999999947|40.667374000000109
15/08/14 19:07:25 INFO tutorial1.TruckEventsProducer: Sending Messge #: route17: 1, msg:2015-08-14 19:07:25.215|1|11|Normal|-79.74982399999999|42.129476999999952
15/08/14 19:07:26 INFO tutorial1.TruckEventsProducer: Sending Messge #: route17k: 1, msg:2015-08-14 19:07:26.222|2|12|Lane Departure|-74.021477|41.500715
15/08/14 19:07:27 INFO tutorial1.TruckEventsProducer: Sending Messge #: route208: 1, msg:2015-08-14 19:07:27.227|3|13|Normal|-74.190990000000056|41.335225999999
```

La vérifions que nous des fichiers dans la partie HDFS:

```
~/docker-kafka-storm-hdfs $ docker exec -it ksh_hadoop_1 /bin/bash
bash-4.1# cd $HADOOP_PREFIX
bash-4.1# bin/hdfs dfs -ls /truck-events-v4/staging
Found 4 items
-rw-r--r--   3 root supergroup         57 2015-08-14 13:07 /truck-events-v4/staging/truckEventshdfsBolt-2-0-1439571724879.txt
-rw-r--r--   3 root supergroup          0 2015-08-14 13:08 /truck-events-v4/staging/truckEventshdfsBolt-2-1-1439572042257.txt
-rw-r--r--   3 root supergroup         75 2015-08-14 13:07 /truck-events-v4/staging/truckEventshdfsBolt-3-0-1439571724873.txt
-rw-r--r--   3 root supergroup          0 2015-08-14 13:08 /truck-events-v4/staging/truckEventshdfsBolt-3-1-1439572041979.txt
bash-4.1# bin/hdfs dfs -cat truck-events-v4/staging/truckEventshdfsBolt-2-0-1439571724879.txt
12,2,2015-08-14 19:07:22.194,Normal,-74.021358,41.500703
bash-4.1#
```
