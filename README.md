#  Lab 2 — Hands-On Kafka (CLI)

In this lab, you will interact directly with **Apache Kafka** using its **command-line tools**.
The goal is to understand Kafka's **core concepts** by experimenting and exploring the CLI.

---

## Learning Objectives

By the end of this lab, you should understand and be able to explain:

- What a **topic** is and why it is a logical stream
- How **partitions** enable parallelism
- What a **producer** does
- What a **consumer** does
- How **offsets** track message positions
- What a **consumer group** is and how it shares workload
- What a **broker** is

---

## Kafka Installation

Kafka must be run using **Docker**.

### Prerequisites
- Docker Desktop installed and running
- Basic familiarity with the terminal

---

## Lab Tasks & Commands

---

### Task 1 — Start Kafka

**Start the Kafka broker and ZooKeeper using Docker Compose:**

```bash
docker-compose up -d
```

**Verify that both containers are running:**

```bash
docker ps
```

You should see two containers running:
- `zookeeper`
- `kafka`

**Check Kafka logs to confirm it started correctly:**

```bash
docker logs kafka
```

**Open a shell inside the Kafka container (needed for all CLI tasks below):**

```bash
docker exec -it kafka bash
```

> All commands from Task 2 onwards are run **inside** this shell.

---

### Task 2 — Topics

**Create a topic named `transactions` with 3 partitions:**

```bash
kafka-topics --bootstrap-server localhost:9092 \
  --create \
  --topic transactions \
  --partitions 3 \
  --replication-factor 1
```

**List all existing topics:**

```bash
kafka-topics --bootstrap-server localhost:9092 --list
```

**Inspect the topic configuration (partitions, replication, offsets):**

```bash
kafka-topics --bootstrap-server localhost:9092 \
  --describe \
  --topic transactions
```

**Expected output:**

```
Topic: transactions   PartitionCount: 3   ReplicationFactor: 1
  Partition: 0   Leader: 1   Replicas: 1   Isr: 1
  Partition: 1   Leader: 1   Replicas: 1   Isr: 1
  Partition: 2   Leader: 1   Replicas: 1   Isr: 1
```

**What we observe:**
The topic `transactions` has **3 partitions**, each assigned to broker 1.
Each partition has its own offset counter starting at 0.

---

### Task 3 — Producers

**Start a Kafka producer for the `transactions` topic:**

```bash
kafka-console-producer --bootstrap-server localhost:9092 \
  --topic transactions
```

**Once the `>` prompt appears, type messages one per line:**

```
1,101,49.99,2024-01-01 10:00:05
2,102,120.00,2024-01-01 10:00:20
3,103,8.50,2024-01-01 10:01:10
4,101,299.99,2024-01-01 10:02:00
5,104,15.75,2024-01-01 10:02:45
6,102,55.20,2024-01-01 10:03:30
7,105,200.00,2024-01-01 10:04:10
8,103,32.10,2024-01-01 10:05:00
9,106,500.00,2024-01-01 10:05:40
10,101,22.00,2024-01-01 10:06:15
```

Press `Ctrl+C` to stop the producer.

**What we observe:**
Each line typed becomes one **Kafka message**. The producer sends it to the broker,
which writes it to one of the 3 partitions. Messages are distributed across
partitions using a round-robin strategy (no key specified).

---

### Task 4 — Consumers

**Start a consumer that reads from the beginning of the topic:**

```bash
kafka-console-consumer --bootstrap-server localhost:9092 \
  --topic transactions \
  --from-beginning
```

You should see all 10 messages printed in the terminal.

**Stop the consumer:**

```bash
Ctrl+C
```

**Restart the consumer without `--from-beginning`:**

```bash
kafka-console-consumer --bootstrap-server localhost:9092 \
  --topic transactions
```

**What we observe:**
- With `--from-beginning`: the consumer reads **all messages** stored in the topic,
  starting from offset 0 of each partition.
- Without `--from-beginning`: the consumer starts at the **latest offset** and only
  receives **new messages** produced after it started.
- Message order may differ from production order because messages are spread
  across 3 partitions — order is guaranteed **within** a partition, not across them.

---

### Task 5 — Offsets

**Check the current offsets for all partitions of the topic:**

```bash
kafka-run-class kafka.tools.GetOffsetShell \
  --bootstrap-server localhost:9092 \
  --topic transactions
```

