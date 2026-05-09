# Format KRaft metadata storage on each broker node — run one at a time

# Replace YOUR-CLUSTER-UUID with ID saved from Phase 1

/opt/kafka/bin/kafka-storage.sh format -t YOUR-CLUSTER-UUID -c /opt/kafka/config/kraft/controller.properties

# Verify formatting was successful

ls -lh /opt/data/kraft/meta-logs

# Start KRaft controller using nohup on each node — run one at a time

cd /opt/kafka/bin

nohup /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/kraft/controller.properties > /opt/logs/kafka-controller/controller.log 2>&1 &

# Verify controller process is running

ps -ef | grep controller.properties | grep -v grep

# Watch controller logs — check for errors

tail -f /opt/logs/kafka-controller/controller.log

# Check KRaft quorum status — run after all 3 controllers started

/opt/kafka/bin/kafka-metadata-quorum.sh --bootstrap-controller broker1.example.com:9093 describe --status

# Check quorum replication details

/opt/kafka/bin/kafka-metadata-quorum.sh --bootstrap-controller broker1.example.com:9093 describe --replication


expected output-
LeaderId:             101
MaxFollowerLag:       0
MaxFollowerLagTimeMs: 0
CurrentVoters:        [101, 102, 103]
