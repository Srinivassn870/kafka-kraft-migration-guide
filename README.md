# ZooKeeper-based Kafka cluster to KRaft-based architecture Migration Guide 

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
| Phase | Activity |
|---|---|
| Phase 1 | Environment Preparation |
| Phase 2 | Controller Migration |
| Phase 3 | Broker Migration |
| Phase 4 | KRaft Finalization |
| Phase 5 | Zookeeper Decommissioning|
| Phase 6 | Validation & Monitoring |
## Phase 1 – Preparation

Upgraded Kafka cluster to version 3.9.1 (KRaft supported).

Validated application compatibility.

Cluster tested in POC environment.

Zookeeper metadata backup taken.

Step 1) Enable TRACE level logging for the migration 

Step 2)Retrieve the cluster ID of your Kafka cluster 

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


## Phase 3: Reconfigure Brokers for Migration Mode
Step - 6 Reconfigure a server. properties file.

At a minimum, each broker requires the following additional configuration:

Inter-broker protocol version set to version 3.9.1

The migration enabled flag

Controller configuration that matches the controller nodes

A quorum of controller voters

Step -7 Restart the updated broker one at a time and Check that Kafka is running:

NOTE: Once the final ZK broker has been restarted with the necessary configuration, the migration will automatically begin. When the migration is complete, an INFO level log can be observed on the active controller as below:<img width="1576" height="196" alt="image" src="https://github.com/user-attachments/assets/5018828b-2f2b-439f-bad3-87f54ab1254a" />

## Phase 4 : Switch each broker to KRaft mode
Step -8 Update the broker configuration in the server.properties file:

Replace the broker.id with a node.id using the same ID

Add a broker KRaft role for the broker

Remove the inter-broker protocol version (inter.broker.protocol.version)

Remove the migration enabled flag (zookeeper.metadata.migration.enable)

Remove ZooKeeper configuration including Jaas file.

Remove the listener for controller and broker communication (control.plane.listener.name)

Restart the broker one at a time, each broker is restarted with a Kraft configuration until the entire cluster is running in KRaft mode.

## Phase 5 : Finalizing the migration (Switch each controller out of migration mode).

Step-9 Stop the controller in the same way as the broker and update the controller configuration in the controller.properties file:

Remove the ZooKeeper connection details

Remove the zookeeper.metadata.migration.enable property

Remove inter.broker.listener.name

Restart the controller in the same way as the broker.

NOTE - Once the migration has been finalized, you can safely deprovision your ZooKeeper cluster, assuming you are not using it for anything else. After this point, it is no longer possible to revert to ZooKeeper mode.

## Phase 6 : Zookeeper Decommissioning

Shut down Zookeepers nodes.

Removed ZK configs from Kafka.

Confirmed clean operation in Kraft mode.

## Phase  7 : Testing and Monitoring of KAFKA-KRAFT cluster.

Once the migration to KRaft mode was completed, a comprehensive round of testing and monitoring was carried out to validate the stability, correctness, and performance of the new setup. The objective of this phase was to ensure that the Kafka cluster was fully functional without ZooKeeper, resilient under load, and properly observable for long-term operations.

Objectives

Validate cluster quorum and leadership election in KRaft mode.

Ensure all brokers and controllers are stable and connected.

Verify that topics, ACLs, and client operations (produce/consume) function correctly.

Monitor JVM health (heap/non-heap, GC, class loading, thread activity).

Confirm no errors in logs and smooth failover handling.

Assess performance and stability under typical workload.

# Performance Benchmark: ZooKeeper vs. KRaft Mode

Key Findings

✅ Double the throughput in KRaft mode compared to ZooKeeper.

✅ ~70% lower latencies across all percentiles.

✅ Better tail performance (P95–P99.9), confirming improved stability under load.

✅ Kafka is now more scalable, responsive, and efficient without ZooKeeper.


<img width="1404" height="541" alt="image" src="https://github.com/user-attachments/assets/8ea0cb76-b74f-4cef-ac84-340341b4d40b" />


✅ Cluster handled 1 million messages smoothly.

✅ No lag observed between producers and consumers.

✅ Performance consistent with pre-migration levels, validating stability.

✅ Confirms KRaft controllers are managing metadata efficiently.

✅ Memory usage is very stable (~100 MB used out of ~106 MB committed).

✅ Very few Old Gen collections → confirms no memory leaks or unusual object promotion.

✅ JVM is running smoothly with minimal GC overhead in non-heap.

✅ Confirms healthy Kafka KRaft controller runtime.

✅ No indication of memory leaks — Kafka runtime modules (network, replication, Raft, controller services) are loaded and stable after migration.

✅ Indicates Kafka controllers/brokers manage metadata and client connections efficiently.

# Why We Chose Co-Located Approach?

There are 2 types of approaches to migrate zk based kafka cluster  to kraft cluster.

1-Co-located Approach

2-Lift-and-Shift

We adopted a co-located approach where each Kafka Broker also runs as a Controller.

# Key Justifications

Cost Savings

Avoids the need for separate dedicated controller nodes.

Each broker contributes to the KRaft quorum, reducing infra by ~30–40%.

Simplicity

Single process per node (Broker+Controller) with clear role separation.

Easier deployment and scaling in our existing infrastructure.

Sufficient for Current Workload

Depends on throughput and topic/partition counts can be handled comfortably by co-located controllers.

Downtime -Entire transition occurred with zero impact or downtime for our data producers and consumers, preserving business continuity

 Client changes - Minimal to no client changes for the migration itself (if using the official migration path with dual-writing metadata).

# Conclusion

The transition of Kafka ecosystem from the ZooKeeper-based architecture to KRaft (Kafka Raft Metadata mode) is a critical, forward-looking strategic milestone. This document outlines not just a simple upgrade, but a fundamental architectural shift that aligns our platform with the future of  Apache Kafka .

✅ Key Achievements:

Successfully migrated from ZooKeeper to KRaft.

Simplified operations.

Ensured stability and scalability.

the entire transition occurred with zero impact or downtime for our data producers and consumers, preserving business continuity.

Delivering higher performance, stability, and cost savings.

Continuous monitoring confirms stable runtime behavior and healthy controller quorum.


## Disclaimer

This repository is created for learning and portfolio purposes.



High availability validated: Controllers automatically failover without impacting operations.

