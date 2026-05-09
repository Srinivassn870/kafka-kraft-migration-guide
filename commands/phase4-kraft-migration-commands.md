Switch to Full KRaft Mode
# Verify no ZooKeeper references remain in server.properties

grep -i "zookeeper" /opt/kafka/config/server.properties



# Verify no ZooKeeper references remain in controller.properties

grep -i "zookeeper" /opt/kafka/config/kraft/controller.properties



# Stop broker

/opt/kafka/bin/kafka-server-stop.sh



# Wait for full stop

sleep 10



# Start broker in full KRaft mode using nohup

cd /opt/kafka/bin

nohup /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties > /opt/logs/kafka-broker/server.log 2>&1 &



# Verify broker process running

ps -ef | grep server.properties | grep -v grep



# Stop controller

kill -15 $(ps -ef | grep controller.properties | grep -v grep | awk '{print $2}')



# Wait for full stop

sleep 10



# Start controller in full KRaft mode using nohup

cd /opt/kafka/bin

nohup /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/kraft/controller.properties > /opt/logs/kafka-controller/controller.log 2>&1 &



# Verify controller process running

ps -ef | grep controller.properties | grep -v grep



# Check quorum healthy after full KRaft switch

/opt/kafka/bin/kafka-metadata-quorum.sh --bootstrap-controller broker1.example.com:9093 describe --status



# Check under replicated partitions — must be zero

/opt/kafka/bin/kafka-topics.sh --bootstrap-server broker1.example.com:9092 --describe --under-replicated-partitions



# Check offline partitions — must be zero

/opt/kafka/bin/kafka-topics.sh --bootstrap-server broker1.example.com:9092 --describe --unavailable-partitions



# Create test topic — confirms KRaft managing metadata

/opt/kafka/bin/kafka-topics.sh --bootstrap-server broker1.example.com:9092 --create --topic kraft-test --partitions 3 --replication-factor 3



# Produce test message

echo "kraft-test-message" | /opt/kafka/bin/kafka-console-producer.sh --bootstrap-server broker1.example.com:9092 --topic kraft-test



# Consume test message — confirms end to end working

/opt/kafka/bin/kafka-console-consumer.sh --bootstrap-server broker1.example.com:9092 --topic kraft-test --from-beginning --max-messages 1
