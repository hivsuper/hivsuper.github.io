---
title: Install Kafka in KRaft Mode Using Docker  
date: 2024-04-19 18:14:00 +0800  
categories: [Technology, Kafka Learning Journey]  
tags: [kafka]  
---
Apache Kafka® Raft (KRaft) is the consensus protocol that was introduced to remove Kafka’s dependency on ZooKeeper for metadata management. This greatly simplifies Kafka’s architecture by consolidating responsibility for metadata into Kafka itself.   
This tutorial demonstrates the process of installing kafka in KRaft mode and simple tests.
## Prerequisites
- Java 11+
- Docker environment is ready. See [Windows 10安装Docker并使用私钥连接AWS EC2](/posts/Windows-10安装Docker并使用私钥连接AWS-EC2/)

## Kafka Installation
Before starting this process, one has to download the necessary [scripts](https://www.apache.org/dyn/closer.cgi?path=/kafka/3.7.0/kafka_2.13-3.7.0.tgz) and extract it.
### Enter the Extracted Folder
```shell
cd /d/kafka/kafka_2.13-3.7.0/bin
```
### Generate a random-uuid as `CLUSTER_ID`
```shell
sh kafka-storage.sh random-uuid
```
Copy the value returned by the command above.
### Install Kafka
Update the `{CLUSTER_ID}` then run the command below:
```shell
docker run -d \
--name=kafka-kraft \
-h kafka-kraft \
-p 9101:9101 \
-p 9092:9092 \
-e KAFKA_NODE_ID=1 \
-e KAFKA_LISTENER_SECURITY_PROTOCOL_MAP='CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT' \
-e KAFKA_ADVERTISED_LISTENERS='PLAINTEXT://kafka-kraft:29092,PLAINTEXT_HOST://localhost:9092' \
-e KAFKA_JMX_PORT=9101 \
-e KAFKA_JMX_HOSTNAME=localhost \
-e KAFKA_PROCESS_ROLES='broker,controller' \
-e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
-e KAFKA_CONTROLLER_QUORUM_VOTERS='1@kafka-kraft:29093' \
-e KAFKA_LISTENERS='PLAINTEXT://kafka-kraft:29092,CONTROLLER://kafka-kraft:29093,PLAINTEXT_HOST://0.0.0.0:9092' \
-e KAFKA_INTER_BROKER_LISTENER_NAME='PLAINTEXT' \
-e KAFKA_CONTROLLER_LISTENER_NAMES='CONTROLLER' \
-e CLUSTER_ID='{CLUSTER_ID}' \
confluentinc/cp-kafka:7.6.1
```
<span style="color: rgba(255, 0, 0, 1)">It's essential to expose the port</span> `9092` <span style="color: rgba(255, 0, 0, 1)">here, otherwise the host won't be able to connect the kafka server running in the docker.</span>

## Kafka Verification
### In Kafka CLI
1. Create the Kafka topic `test-topic` with 3 partitions and a replication factor of 1
```shell
sh kafka-topics.sh --bootstrap-server localhost:9092 --topic test-topic --create --partitions 3 --replication-factor 1
```
2. Connect the kafka topic `test-topic`
```
sh kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test-topic
msg1
msg2
```
- Send messages when pressing `Enter`
- `Ctrl + C` is used to exit the producer

3. Create a consumer to digest messages
```
sh kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --from-beginning
msg1
msg2
Processed a total of 2 messages
```
- `Ctrl + C` is used to exit the consumer

### In Offset Explorer(Formerly Kafka Tool)
Download [Offset Explorer](https://kafkatool.com/download.html) then install it.
1. Configure the kafka server  
<img src="/assets/img/202404/Offset-Explorer-Add-Cluster.png" width = "800" />
2. Send message  
The key/value can be converted in [string-hex](https://string-functions.com/string-hex.aspx).  
<img src="/assets/img/202404/Offset-Explorer-Send-Message.png" width = "800" />

## References
- [Install Confluent Platform using Docker](https://docs.confluent.io/platform/current/installation/docker/config-reference.html)
- [Guide to Setting Up Apache Kafka Using Docker](https://www.baeldung.com/ops/kafka-docker-setup)
- [APACHE KAFKA QUICKSTART](https://kafka.apache.org/quickstart)
- [Kafka CLI Tutorials](https://www.conduktor.io/kafka/kafka-cli-tutorial/)
