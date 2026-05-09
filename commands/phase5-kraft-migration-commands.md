# Final quorum check before removing ZooKeeper
/opt/kafka/bin/kafka-metadata-quorum.sh --bootstrap-controller broker1.example.com:9093 describe --status
 
# ZooKeeper Decommissioning

# Final topic list check — compare with backup from Phase 1
/opt/kafka/bin/kafka-topics.sh --bootstrap-server broker1.example.com:9092 --list
 
# Stop ZooKeeper on each node — one at a time
/opt/zookeeper/bin/zkServer.sh stop
 
# Check ZooKeeper stopped
/opt/zookeeper/bin/zkServer.sh status
 
# Verify Kafka still healthy after each ZooKeeper stop
/opt/kafka/bin/kafka-broker-api-versions.sh --bootstrap-server broker1.example.com:9092 | grep "id:"
 
# Confirm no ZooKeeper in server.properties — must return no output
grep -i "zookeeper" /opt/kafka/config/server.properties
 
# Confirm no ZooKeeper in controller.properties — must return no output
grep -i "zookeeper" /opt/kafka/config/kraft/controller.properties
 
# Create new topic without ZooKeeper — final validation
/opt/kafka/bin/kafka-topics.sh --bootstrap-server broker1.example.com:9092 --create --topic zk-removed-test --partitions 3 --replication-factor 3
 
# Produce message — final validation
echo "zookeeper-removed-success" | /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server broker1.example.com:9092 --topic zk-removed-test
 
# Consume message — final validation
/opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server broker1.example.com:9092 --topic zk-removed-test --from-beginning --max-messages 1
 
# Check ACLs still working
/opt/kafka/bin/kafka-acls.sh --bootstrap-server broker1.example.com:9092 --list
