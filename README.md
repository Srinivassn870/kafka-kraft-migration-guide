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
