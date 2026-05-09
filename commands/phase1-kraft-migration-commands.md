# Kafka KRaft Migration — Commands Reference

## Phase-wise commands 

## PHASE 1 — Preparation & Validation

# Check current Kafka version on broker

/opt/kafka/bin/kafka-broker-api-versions.sh --bootstrap-server broker1.example.com:9092 | grep "id:" or

/opt/kafka/bin kafka-topics --version

# Check all brokers are online

/opt/kafka/bin/kafka-broker-api-versions.sh --bootstrap-server broker1.example.com:9092

# Check offline partitions — must be zero before starting

/opt/kafka/bin/kafka-topics.sh --bootstrap-server broker1.example.com:9092 --describe --unavailable-partitions

# Check under replicated partitions — must be zero before starting

/opt/kafka/bin/kafka-topics.sh --bootstrap-server broker1.example.com:9092 --describe --under-replicated-partitions

# Get existing cluster ID from ZooKeeper — save this ID!

/opt/zookeeper/bin/zookeeper-shell.sh zk1.example.com:2181 get /PT/QA/cluster/id

# Backup all topics list

/opt/kafka/bin/kafka-topics.sh --bootstrap-server broker1.example.com:9092 --list > /tmp/topics-backup.txt

# Backup all consumer groups

/opt/kafka/bin/kafka-consumer-groups.sh --bootstrap-server broker1.example.com:9092 --list > /tmp/consumer-groups-backup.txt

# Enable TRACE logging for migration visibility — add to log4j.properties

echo "log4j.logger.org.apache.kafka.metadata.migration=TRACE" >> /opt/kafka/config/log4j.properties

echo "log4j.logger.org.apache.kafka.controller=DEBUG" >> /opt/kafka/config/log4j.properties
