Setup Linux VM (Amazon Linux )

Login into AWS Cloud account [take t2.medium]

Create Linux VM and connect to it using MobaXterm

Install Docker In Amazon Linux VM
==================================

sudo yum update -y 
sudo yum install docker -y
sudo service docker start
sudo usermod -aG docker ec2-user
exit


====================================================
Docker compose setup
=======================


sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version

====================================================

Kafka + Zookeeper using Docker Compose (Recommended)
====================================

1) verify software at ec2 vm

$ docker --version
$ docker-compose version

2) Create docker-compose.yml

----------------------------------WORKING docker-compose.yml----------------------
version: '2.2'

services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.6.0
    container_name: zookeeper
    ports:
      - "2181:2181"
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000

  kafka:
    image: confluentinc/cp-kafka:7.6.0
    container_name: kafka
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181

      KAFKA_LISTENERS: PLAINTEXT://0.0.0.0:9092
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:9092

      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      KAFKA_TRANSACTION_STATE_LOG_MIN_ISR: 1



----------------------------------------------------------------------------

3. Start Kafka & Zookeeper


From the same directory:

Restart cleanup

$ docker-compose down -v
$ docker-compose up -d


Check running containers:

$ docker ps

🧪 Test Kafka
docker exec -it kafka kafka-topics \
  --create \
  --topic user_data_topics \
  --bootstrap-server kafka:9092 \
  --partitions 1 \
  --replication-factor 1

To see list of topics
docker exec -it kafka kafka-topics \
  --list \
  --bootstrap-server kafka:9092

Produce
docker exec -it kafka kafka-console-producer \
  --topic test-topic \
  --bootstrap-server localhost:9092

Consume
docker exec -it kafka kafka-console-consumer \
  --topic user_data_topics \
  --from-beginning \
  --bootstrap-server localhost:9092

5. Stop & Clean Up
docker compose down


To remove volumes as well:

docker compose down -v

