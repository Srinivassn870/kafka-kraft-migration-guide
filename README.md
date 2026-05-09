# Kafka ZooKeeper to KRaft Migration Guide

### Based on Real Production Experience — 15 Clusters Migrated


![Kafka](https://img.shields.io/badge/Apache%20Kafka-3.9.1-black)

![KRaft](https://img.shields.io/badge/Mode-KRaft-blue)

![AWS](https://img.shields.io/badge/AWS-EC2-orange)

![Status](https://img.shields.io/badge/Migration-Completed-green)


## 📌 Overview



This guide documents a **real production migration** of 15 Apache Kafka

clusters from ZooKeeper-based metadata management to KRaft (Kafka Raft) mode.



### Results Achieved:

- ✅ 45 servers removed across 15 clusters

- ✅ 33-40% infrastructure cost reduction

- ✅ Zero downtime — zero impact on producers and consumers

- ✅ Simplified operations — no more dual maintenance

- ✅ Future-proof — KRaft is Kafka's default path (ZK deprecated in Kafka 4.0)



---



## ❓ Why Migrate to KRaft?



### ZooKeeper Limitations:

- **Operational Complexity** — running and managing ZooKeeper adds extra moving parts

- **Consistency Challenges** — metadata changes must sync between Kafka and ZooKeeper

- **Scaling Issues** — ZooKeeper becomes a bottleneck for large Kafka deployments

- **Dual Maintenance** — two separate systems to monitor, patch and upgrade



### KRaft Advantages:

- No external dependency — Kafka manages its own metadata

- Faster controller failover

- Centralized ACL management — fully inside Kafka

- Native Raft consensus — more stable and reliable

- Reduced admin overhead — only one system to manage

## 🏗️ Architecture Changes ##

### Before — ZooKeeper-based (9 servers per cluster 3 Kafka broker + 3 kafka connect node + 3 ZK) ###

Separate ZooKeeper cluster (3 nodes )

Kafka brokers connect to ZooKeeper for metadata.

Controller election handled by ZooKeeper.

 ## After (KRaft-based Kafka) ##
 
ZooKeeper completely removed.

Kafka has two process roles:

Broker: Handles client requests, topics, partitions, and replication.

Controller: Manages cluster metadata, leader election, and configuration.

Metadata is replicated using the Raft consensus algorithm among controllers.

Brokers communicate with controllers directly (no ZooKeeper).

## Migration Process and Steps - (Apache Kafka)
## Phase 1 – Preparation
Upgraded Kafka cluster to version 3.9.1 (KRaft supported).
Validated application compatibility.
Cluster tested in POC environment.
Zookeeper metadata backup taken.

STEP 1) Enable TRACE level logging for the migration 2)Retrieve the cluster ID of your Kafka cluster 

## Phase 2 - Start KRaft Controller in Migration mode (migration of the cluster metadata from ZooKeeper to the KRaft quorum)

Configure a controller node on each host using the controller.properties file.

Step 3) At a minimum, each controller requires the following configuration:
A unique node ID
The migration enabled flag set to true
Zookeeper connection details
A quorum of controller voters
Listener name for inter-broker communication

Step -4) Set up log directories for each controller node and Format Controller Storage
Step -5) Start each controller one at a time and check the status and check the logs of each controller to ensure that they have successfully joined the KRaft cluster
<img width="954" height="132" alt="image" src="https://github.com/user-attachments/assets/083cc5d7-ca6e-4f6a-94b6-3383ad284feb" />

## Phase 3: Reconfigure Brokers for Migration Mode
Step - 6 Reconfigure a server. properties file.

At a minimum, each broker requires the following additional configuration:
Inter-broker protocol version set to version 3.9.1
The migration enabled flag
Controller configuration that matches the controller nodes
A quorum of controller voters

Step -7 Restart the updated broker one at a time and Check that Kafka is running:

NOTE: Once the final ZK broker has been restarted with the necessary configuration, the migration will automatically begin. When the migration is complete, an INFO level log can be observed on the active controller as below:<img width="1576" height="196" alt="image" src="https://github.com/user-attachments/assets/5018828b-2f2b-439f-bad3-87f54ab1254a" />


