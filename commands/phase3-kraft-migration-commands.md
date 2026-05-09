# PHASE 3 — Broker Migration Mode (Dual Write)
# Stop broker gracefully — run one broker at a time
/opt/kafka/bin/kafka-server-stop.sh
 
# Verify broker process stopped
ps -ef | grep server.properties | grep -v grep
 
# Start broker in migration mode using nohup
cd /opt/kafka/bin
nohup /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties > /opt/logs/kafka-broker/server.log 2>&1 &
 
# Verify broker process is running
ps -ef | grep server.properties | grep -v grep
 
# Watch broker logs — check for migration messages
tail -f /opt/logs/kafka-broker/server.log
 
# Grep for migration completion message — must see this after each broker restart
grep "Completed migration of metadata from ZooKeeper to KRaft" /opt/kafka/bin/nohup_kraft_broker.log
 
# Check under replicated partitions after each broker restart — must be zero
/opt/kafka/bin/kafka-topics.sh --bootstrap-server broker1.example.com:9092 --describe --under-replicated-partitions
 
# Check quorum still healthy after each broker restart
/opt/kafka/bin/kafka-metadata-quorum.sh --bootstrap-controller broker1.example.com:9093 describe --status

# Check Expected Migration log message 
INFO [KRaftMigrationDriver id=102] Completed migration of metadata
from ZooKeeper to KRaft. 71 records generated in 316 ms.
TOPIC_RECORD=5, PARTITION_RECORD=62, CONFIG_RECORD=3
Saw 3 brokers in migrated metadata.
