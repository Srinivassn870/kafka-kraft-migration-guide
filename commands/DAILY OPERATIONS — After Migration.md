# Start broker
systemctl start kafka.service or
nohup /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties > /opt/logs/kafka-broker/server.log 2>&1 &
 
# Stop broker
systemctl stop kafka.service or
/opt/kafka/bin/kafka-server-stop.sh
 
# Start controller
systemctl start controller.service or
nohup /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/kraft/controller.properties > /opt/logs/kafka-controller/controller.log 2>&1 &
 
# Stop controller
systemctl stop controller.service or
kill -15 $(ps -ef | grep controller.properties | grep -v grep | awk '{print $2}')
 
# Check broker process
ps -ef | grep server.properties | grep -v grep
 
# Check controller process
ps -ef | grep controller.properties | grep -v grep
 
# Check broker logs
tail -f /opt/logs/kafka-broker/server.log
 
# Check controller logs
tail -f /opt/logs/kafka-controller/controller.log
 
# Check quorum health
/opt/kafka/bin/kafka-metadata-quorum.sh --bootstrap-controller broker1.example.com:9093 describe --status
 
# List all topics
/opt/kafka/bin/kafka-topics.sh --bootstrap-server broker1.example.com:9092 --list
 
# Describe specific topic
/opt/kafka/bin/kafka-topics.sh --bootstrap-server broker1.example.com:9092 --describe --topic your-topic-name
 
# Check consumer group lag
/opt/kafka/bin/kafka-consumer-groups.sh --bootstrap-server broker1.example.com:9092 --describe --group your-consumer-group
 
# List all consumer groups
/opt/kafka/bin/kafka-consumer-groups.sh --bootstrap-server broker1.example.com:9092 --list
 
# Check ACLs
/opt/kafka/bin/kafka-acls.sh --bootstrap-server broker1.example.com:9092 --list
