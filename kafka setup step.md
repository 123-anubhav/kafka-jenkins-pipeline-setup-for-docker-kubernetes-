---

# 🚀 Kafka & Zookeeper Setup on Amazon Linux (AWS EC2)

This guide walks through setting up an **Amazon Linux EC2 VM**, installing **Docker & Docker Compose**, and running **Kafka + Zookeeper** using **Docker Compose**.

---

## 🖥️ Step 1: Create Amazon Linux EC2 VM

* Login to **AWS Cloud Account**
* Launch **Amazon Linux** EC2 instance
* Instance type: **t2.medium**
* Connect to the VM using **MobaXterm**

---

## 🐧 Step 2: Install Docker on Amazon Linux

Run the following commands on the EC2 instance:

```bash
sudo yum update -y
sudo yum install docker -y
sudo service docker start
sudo usermod -aG docker ec2-user
exit
```

➡️ **Logout and login again** for Docker group permissions to apply.

---

## 🧩 Step 3: Docker Compose Setup

Install Docker Compose:

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

---

## 🧪 Step 4: Verify Installation

```bash
docker --version
docker-compose version
```

---

## 🧱 Step 5: Kafka + Zookeeper Using Docker Compose (Recommended)

### Create `docker-compose.yml`

```yaml
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
```

---

## ▶️ Step 6: Start Kafka & Zookeeper

From the same directory:

```bash
docker-compose down -v
docker-compose up -d
```

Check running containers:

```bash
docker ps
```

---

## 🧪 Step 7: Kafka Testing

### Create Topic

```bash
docker exec -it kafka kafka-topics \
  --create \
  --topic user_data_topics \
  --bootstrap-server kafka:9092 \
  --partitions 1 \
  --replication-factor 1
```

### List Topics

```bash
docker exec -it kafka kafka-topics \
  --list \
  --bootstrap-server kafka:9092
```

### Produce Messages

```bash
docker exec -it kafka kafka-console-producer \
  --topic test-topic \
  --bootstrap-server localhost:9092
```

### Consume Messages

```bash
docker exec -it kafka kafka-console-consumer \
  --topic user_data_topics \
  --from-beginning \
  --bootstrap-server localhost:9092
```

---

## 🧹 Step 8: Stop & Clean Up

Stop services:

```bash
docker-compose down
```

Remove containers and volumes:

```bash
docker-compose down -v
```

---

✨ **Your Kafka + Zookeeper environment is now ready on AWS EC2!**
