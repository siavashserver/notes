---
title: Kafka
---

## Introduction

Apache Kafka is a distributed, durable, high‑throughput messaging and streaming platform.

## Core Components

- Broker: A Kafka server that stores partitions, handles client requests and
  coordinates things like partition assignments and detecting broker failures.
  Brokers form a cluster.
- Topic: A named stream or category of messages, partitioned for scalability.
  Think of it as a log.
- Partition: Each topic is split into multiple partitions; immutable,
  append-only logs with uniquely ordered offsets per message. Ordering is
  guaranteed within a partition, but not across partitions. More partitions =
  more parallel consumers. Each partition has a _leader_ broker handling all
  reads/writes; the other replicas are _followers_, which replicate from the
  leader. **Only the leader handles client I/O**.
- Producer: Publishes records to topics, choosing partitions via key-based
  hashing, explicit partition number, and round-robin distribution (if no key
  provided).
- Consumer & Consumer Group: Consumers subscribe to topics and **pull**
  messages, tracking their position via offsets. In a _consumer group_, each
  partition is assigned to a single consumer to enable parallel processing;
  extra consumers simply stay idle. Offsets are committed to the internal
  `__consumer_offsets` topic, allowing consumers to resume from where they left
  off.
- Zookeeper / KRaft (KIP‑500): Manages cluster metadata, leader elections,
  broker coordination. Modern Kafka uses its internal Raft-based metadata
  (KRaft), replacing Zookeeper.

### Workflow

- _Producer_ writes to a topic -> assigned to a partition on a broker (the
  leader).
- _Follower_ brokers replicate the partition log.
- _Consumer_ in a group pulls messages from leader via offset tracking.
- On broker failure, _ZooKeeper_ (legacy) or _KRaft_ (modern) elects a new
  leader from the ISR.

## Replication & Durability

- Each partition can have multiple _replicas_ across brokers; one is the leader,
  others are followers.
- ISR (In-Sync Replica): Only replicas caught up with the leader are eligible
  for failover.
- High Watermark (HW): Position up to which all replicas have received messages.
  Consumers only read up to HW.
- Producers specify `acks` and `min.insync.replicas` to control durability
  guarantees.

## Retention & Compaction

- You can configure data retention by time (`log.retention.ms`) or size
  (`log.retention.bytes`), and decide between delete vs. compact policies.
- Compaction keeps latest record per key—useful for changelog or state topics.

## APIs beyond the basics

- Kafka Connect: Standard interface and connectors for integrating with external
  systems.
- Kafka Streams: Library for real-time processing and aggregations, supports
  stateful operations with RocksDB.
- ksqlDB: SQL-like interface for stream processing queries.

## Exactly-Once Semantics (EOS)

- Idempotent producers use message sequence number + producer ID to avoid
  duplicates. The broker deduplicates based on PID + sequence, ensuring each
  message is recorded only once within that session and partition.
- Transactions coordinate writes across partitions and offset commits so that
  Kafka can guarantee exactly-once semantics (when
  `isolation.level=read_committed`).

### Limitations

- Only deduplicates _within a single partition and producer session_
- If the producer restarts (loses PID), duplicates can occur

### Read Isolation: Committed vs Uncommitted

Consumers control visibility via `isolation.level`:

- `read_uncommitted`: sees even in-flight or aborted messages.
- `read_committed`: only sees messages from fully committed transactions and
  skips aborted ones.

---

## Interview Questions

### How do partitions and consumer groups enable scalability?

- Partitions allow a topic to be split across brokers, enabling parallelism.
- In a consumer group, each partition is consumed by a single consumer, so the
  number of consumers scales with partitions.

### Explain replication and how Kafka handles failures

- Each partition has multiple replicas across brokers: one leader, multiple
  followers.
- Followers fetch data from the leader and maintain ISRs (In-Sync Replicas).
- If the leader fails, one ISR is promoted to leader.
- Producers use `acks=all` and `min.insync.replicas` to ensure durability.

### Describe data retention and log compaction

- You can delete old data via time/size-based policies.
- Log compaction retains only the latest value per key, cleaning up older ones.
  Useful for stateful evolutions.

### What is exactly-once delivery, and how is it achieved?

- Achieved with _idempotent producers_ (sequence numbers + PID) and transactions
  across partitions.
- Consumers read only committed messages (`read_committed`).
- Ensures no duplicates and full atomicity.

### What is the role of Schema Registry?

- Stores and enforces Avro/Protobuf/JSON schemas.
- Provides schema compatibility checks (backward, forward, full).
- Ensures producers and consumers agree on message structure.

### Kafka vs. RabbitMQ?

- Kafka focuses on durable, append‑only logs with replayability and high
  throughput.
- RabbitMQ follows traditional queue semantics (ack‑delete), less suited for
  large-scale streaming.
- Kafka decouples producers/consumers and supports stream processing; RabbitMQ
  prioritizes complex routing and real-time delivery.

---

## Sample Producer/Consumer Application

### Producer

```csharp
using System;
using System.Collections.Generic;
using Confluent.Kafka;

class TransactionalProducer
{
    public static void Main(string[] args)
    {
        var producerConfig = new ProducerConfig
        {
            BootstrapServers = "localhost:9092",
            // Enable idempotence and transactions
            EnableIdempotence = true,
            Acks = Acks.All,
            TransactionalId = "txn-producer-1",  // unique across instances
            MessageSendMaxRetries = 5,
            RetryBackoffMs = 100
        };

        using var producer = new ProducerBuilder<string, string>(producerConfig)
            .SetKeySerializer(Serializers.Utf8)
            .SetValueSerializer(Serializers.Utf8)
            .Build();

        // Initialize transactions
        producer.InitTransactions(TimeSpan.FromSeconds(10));

        try
        {
            // Begin transaction
            producer.BeginTransaction();

            // Example: produce to output topic
            producer.Produce("output-topic", new Message<string, string>
            {
                Key = "key1",
                Value = "processed-value-1"
            });

            // Suppose you consumed and processed offsets separately:
            // var offsets = new List<TopicPartitionOffset> {
            //     new TopicPartitionOffset("input-topic", 0, 1234)
            // };
            // Send those consumed offsets to the transaction:
            // producer.SendOffsetsToTransaction(
            //     offsets,
            //     consumerGroupId: "input-consumer-group",
            //     transactionTimeout: TimeSpan.FromSeconds(10));

            // Commit the transaction (both the produced messages and offsets)
            producer.CommitTransaction();
            Console.WriteLine("Transaction committed successfully.");
        }
        catch (ProduceException<string, string> e)
        {
            Console.WriteLine($"Produce failed: {e.Error.Reason}");
            producer.AbortTransaction();
            Console.WriteLine("Transaction aborted.");
        }
    }
}
```

### Consumer

```csharp
using System;
using Confluent.Kafka;

class CommittedOnlyConsumer
{
    public static void Main(string[] args)
    {
        var consumerConfig = new ConsumerConfig
        {
            BootstrapServers = "localhost:9092",
            GroupId = "input-consumer-group",
            AutoOffsetReset = AutoOffsetReset.Earliest,
            EnableAutoCommit = false,
            IsolationLevel = IsolationLevel.ReadCommitted  // ONLY see committed transactions
        };

        using var consumer = new ConsumerBuilder<string, string>(consumerConfig)
            .SetKeyDeserializer(Deserializers.Utf8)
            .SetValueDeserializer(Deserializers.Utf8)
            .Build();

        consumer.Subscribe("output-topic");

        Console.CancelKeyPress += (_, e) =>
        {
            e.Cancel = true;
            consumer.Close();
        };

        Console.WriteLine("Consuming committed messages...");
        while (true)
        {
            try
            {
                var msg = consumer.Consume();
                Console.WriteLine($"Key: {msg.Message.Key}, Value: {msg.Message.Value}, Offset: {msg.Offset}");
                // After processing:
                consumer.Commit(msg);
            }
            catch (ConsumeException e)
            {
                Console.WriteLine($"Consume error: {e.Error.Reason}");
            }
        }
    }
}
```
