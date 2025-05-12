---
layout: post
title: An Introduction to Kafka
permalink: /articles/kafka/introduction
---

# Apache Kafka Demystified: A Practical Guide for Developers and Data Engineers

## Introduction

### What is Apache Kafka?

Apache Kafka is a distributed event streaming platform that enables high-performance data pipelines, streaming
analytics, data integration, and mission-critical applications. Originally developed by LinkedIn, it's now an
open-source system that provides a unified, high-throughput, low-latency platform for handling real-time data feeds.
Kafka combines three key capabilities:

1. Publish and subscribe to streams of records
2. Store streams of records in a fault-tolerant way
3. Process streams of records as they occur


### Why Kafka is important in modern data pipelines

Modern data pipelines need to handle massive amounts of data in real-time, and Kafka has become a cornerstone technology
for several reasons:

- **Scalability**: Can handle millions of messages per second
- **Reliability**: Offers fault tolerance and data replication
- **Real-time processing**: Enables stream processing with minimal latency
- **Decoupling**: Allows separation between data producers and consumers
- **Integration**: Connects easily with most modern data tools and frameworks
- **Persistence**: Maintains data history with configurable retention


### Real-world use cases:
 
- **LinkedIn**: Uses Kafka to track user interactions and provide real-time analytics
- **Uber**: Processes millions of messages per second for real-time trip updates
- **Netflix**: Handles real-time streaming data and monitoring across their global infrastructure

## Kafka Basics

Key concepts:

- **Topics**: The fundamental unit of data organization in Kafka. A topic is a category or feed name to which records are
  published. Topics are divided into partitions for scalability.

- **Producers**: Applications that publish (write) records to Kafka topics. Producers automatically know which broker and
  partition to write to.

- **Consumers**: Applications that subscribe to (read) topics and process the published records. Consumers keep track of
  which records they have already consumed by tracking offsets.

- **Brokers**: Kafka servers that store the published records. A Kafka cluster consists of one or more brokers that handle
  topic partitions.

- **Partitions**: Each topic is divided into partitions, which are ordered, immutable sequences of records. Partitions allow
  Kafka to scale horizontally and provide redundancy.

- **Consumer Groups**: A group of consumers working together to consume records from topics. Each partition is consumed by
  exactly one consumer within each consumer group.

- **Zookeeper**: An open-source distributed coordination service used by Kafka to maintain broker metadata, handle
  leader election, and manage cluster state. Zookeeper stores configuration data, provides distributed synchronization,
  and maintains service registry for the Kafka cluster.

- **KRaft**: KRaft (Kafka Raft metadata) is Kafka's built-in consensus mechanism that replaces ZooKeeper for managing
  metadata. It uses the Raft consensus protocol to maintain the cluster state, handle leader elections, and manage
  broker metadata. This simplifies Kafka's architecture by removing the external dependency on ZooKeeper and improves
  scalability and performance.

## Quickstart Example (with Docker)

Kafka recently introduced KRaft (Kafka Raft) mode which eliminates the need for ZooKeeper. Below is a docker Compose
setup for three Kafka brokers using KRaft:

```yaml
version: '3.8'

services:
  kafka1:
    image: confluentinc/cp-kafka:7.3.0
    container_name: kafka1
    ports:
      - "9092:9092"
    volumes:
      - ./configs/kafka1.properties:/etc/kafka/kafka.properties
      - ./scripts/start-kafka.sh:/start-kafka.sh
      - ./data/kafka1:/tmp/kraft-combined-logs
    command: ["bash", "-c", "/start-kafka.sh"]
    networks:
      - kafka-network

  kafka2:
    image: confluentinc/cp-kafka:7.3.0
    container_name: kafka2
    ports:
      - "9094:9092"
    volumes:
      - ./configs/kafka2.properties:/etc/kafka/kafka.properties
      - ./scripts/start-kafka.sh:/start-kafka.sh
      - ./data/kafka2:/tmp/kraft-combined-logs
    command: ["bash", "-c", "/start-kafka.sh"]
    networks:
      - kafka-network

  kafka3:
    image: confluentinc/cp-kafka:7.3.0
    container_name: kafka3
    ports:
      - "9096:9092"
    volumes:
      - ./configs/kafka3.properties:/etc/kafka/kafka.properties
      - ./scripts/start-kafka.sh:/start-kafka.sh
      - ./data/kafka3:/tmp/kraft-combined-logs
    command: ["bash", "-c", "/start-kafka.sh"]
    networks:
      - kafka-network

networks:
  kafka-network:
    driver: bridge
```

The Docker Compose file defines a Kafka cluster setup with three broker nodes using KRaft (Kafka Raft) mode, which eliminates the need for ZooKeeper. Here are the key components:

1. **Services**: The file defines three Kafka broker services (`kafka1`, `kafka2`, and `kafka3`), each using the image. `confluentinc/cp-kafka:7.3.0`
2. **Port Mapping**: Each broker exposes different ports:
  - kafka1: 9092:9092
  - kafka2: 9094:9092
  - kafka3: 9096:9092

3. **Volumes**: Each service mounts three volumes:
  - Configuration files `kafka{1,2,3}.properties`
  - A start-up script `start-kafka.sh`
  - Data directory for Kraft logs `/tmp/kraft-combined-logs`

4. **Networking**: All services are connected through a custom bridge network named `kafka-network`
5. **Command**: Each service uses a bash command to execute the script for initialization `start-kafka.sh`

**Note**: Before starting, edit your **/etc/hosts** file to include the Kafka broker hostnames, as both Docker Compose and the Java
applications will need to resolve these hostnames correctly.

This would be a production-ready setup that provides a fault-tolerant Kafka cluster with three nodes, suitable for distributed streaming applications.

Before you start make sure to prepare the following configuration files:
    
    ./kafka1.properties
    ./kafka2.properties
    ./kafka3.properties

Below is the content of **kafka1.properties**. You can refer [here](https://github.com/borispopicbusiness/portfolio-advanced-examples/kafka-intro/docker) to see what kafka2.properties and kafka3.properties should look like.

Once the files are in place, you can start the cluster with:

    docker compose up --build

```properties
# Node and roles
node.id=1
process.roles=broker,controller
controller.listener.names=CONTROLLER

# Listeners
listeners=PLAINTEXT://kafka1:9092,CONTROLLER://kafka1:9093
advertised.listeners=PLAINTEXT://kafka1:9092
listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT
inter.broker.listener.name=PLAINTEXT

# Controller quorum voters
controller.quorum.voters=1@kafka1:9093,2@kafka2:9093,3@kafka3:9093

# Storage
log.dirs=/tmp/kraft-combined-logs

# Performance and defaults
num.network.threads=3
num.io.threads=8
num.partitions=1
default.replication.factor=1
min.insync.replicas=1
log.retention.hours=168
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000
group.initial.rebalance.delay.ms=0
```

This is the content of **./docker/scripts/start-kafka.sh**

```shell
#!/bin/bash

KAFKA_CONFIG="/etc/kafka/kafka.properties"
CLUSTER_ID="DvTS59yoTtClH_4jP3Dqzw"  # must be same across all brokers
LOG_DIR="/tmp/kraft-combined-logs"

if [ ! -f "$KAFKA_CONFIG" ]; then
  echo "Missing Kafka config at $KAFKA_CONFIG"
  exit 1
fi

# Format storage only if meta.properties doesn't exist
if [ ! -f "$LOG_DIR/meta.properties" ]; then
  echo "Formatting storage with cluster ID: $CLUSTER_ID"
  kafka-storage format -t "$CLUSTER_ID" -c "$KAFKA_CONFIG"
fi

# Start Kafka
exec kafka-server-start "$KAFKA_CONFIG"
```

First, the script defines three important variables:

- Path to Kafka configuration file `KAFKA_CONFIG`
- A unique identifier that must be identical across all brokers in the cluster `CLUSTER_ID`
- Directory where Kafka will store its logs `LOG_DIR`

```shell
# Format storage only if meta.properties doesn't exist
if [ ! -f "$LOG_DIR/meta.properties" ]; then
  echo "Formatting storage with cluster ID: $CLUSTER_ID"
  kafka-storage format -t "$CLUSTER_ID" -c "$KAFKA_CONFIG"
fi
```

This is a critical section that:
1. Checks if file exists in the log directory `meta.properties`
2. If it doesn't exist (first time setup), formats the storage with the specified cluster ID
3. This step is necessary for KRaft mode initialization

```shell
exec kafka-server-start "$KAFKA_CONFIG"
```

Finally, the script starts the Kafka server using the configuration file.

This script is designed to handle both initial setup and subsequent starts of a Kafka broker in KRaft mode, ensuring proper initialization of storage and consistent cluster ID across all brokers.

### Producing and Consuming Messages

For working with Kafka in Java, you'll need the following Maven dependencies:

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.7.0</version>
</dependency>
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>2.0.12</version>
</dependency>
```

🧠 **SimpleKafkaProducer Explanation**

This Java class defines a basic Kafka producer using Apache Kafka’s Producer API. It connects to a Kafka cluster and sends messages to a specified topic with optional delivery confirmation and error handling.

**Class**: `SimpleKafkaProducer`

`Fields`:

- `Properties props`: Holds Kafka producer configuration settings.

- `KafkaProducer<String, String> producer`: The actual Kafka producer instance used to send messages with String keys and values.

`Constructor`:

Sets Kafka producer configurations such as:

- `bootstrap.servers`: List of Kafka brokers to connect to.

- `key.serializer` & `value.serializer`: Define how keys and values are serialized (here, both are `StringSerializer`).

- `enable.idempotence`: Ensures exactly-once delivery semantics by avoiding duplicate message delivery.

Instantiates the `KafkaProducer` with the given properties and logs the configuration.

`send(String topic, String message)` method:

- Creates a `ProducerRecord` for the specified topic and message.

- Sends the message asynchronously using `producer.send(...)`, and registers a callback that:

  - Logs an error message and stack trace if the send fails.

  - Logs metadata (`topic`, `partition`, `offset`) if the send succeeds.

- Calls `producer.flush()` to ensure all buffered records are sent immediately.

`close()` method:

- Closes the Kafka producer gracefully, releasing all associated resources.

- Logs success or error messages accordingly.

⚠️ **Notes**:

- The line `props.put("acks", "all");` is commented out. If you want stronger delivery guarantees, consider uncommenting it.

- Calling `producer.flush()` after every send ensures immediate delivery but may reduce throughput. For high-performance use cases, batching is recommended.

- `enable.idempotence = true` is a good practice for production environments to avoid duplicate message delivery.


```java
public class SimpleKafkaProducer {
  private final Properties props;
  private final KafkaProducer<String, String> producer;

  public SimpleKafkaProducer() {
    this.props = new Properties();
    props.put("bootstrap.servers", "kafka1:9092,kafka2:9094,kafka3:9096");
    props.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
    props.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
    //props.put("acks", "all");
    props.put("enable.idempotence", "true");

    this.producer = new KafkaProducer<>(props);
    System.out.println("Producer created with properties: " + props);
  }

  public void send(String topic, String message) {
    try {
      System.out.println("Attempting to send message: '" + message + "' to topic: " + topic);
      ProducerRecord<String, String> record = new ProducerRecord<>(topic, message);

      producer.send(record, (metadata, exception) -> {
        if (exception != null) {
          System.err.println("Error sending message: " + exception.getMessage());
          exception.printStackTrace();
        } else {
          System.out.printf("Message sent successfully to topic=%s partition=%d offset=%d%n",
                  metadata.topic(),
                  metadata.partition(),
                  metadata.offset());
        }
      });

      producer.flush();
    } catch (Exception e) {
      System.err.println("Error in send method: " + e.getMessage());
      e.printStackTrace();
    }
  }

  public void close() {
    try {
      producer.close();
      System.out.println("Producer closed successfully");
    } catch (Exception e) {
      System.err.println("Error closing producer: " + e.getMessage());
      e.printStackTrace();
    }
  }
}
```

🧠 **SimpleKafkaConsumer** Explanation

This Java class defines a basic Kafka consumer using Apache Kafka's consumer API. It subscribes to a topic and continuously polls for new messages.

**Class**: `SimpleKafkaConsumer`

`Fields`:

- Properties props: Holds Kafka configuration settings.

- KafkaConsumer<String, String> consumer: The actual Kafka consumer instance that reads messages with String keys and values.

`Constructor`:

Sets Kafka consumer configurations such as:

- `bootstrap.servers`: List of Kafka brokers to connect to.

- `group.id`: Consumer group identifier (used for load balancing and offset tracking).

- `key.deserializer` & `value.deserializer`: Define how keys and values are deserialized (here, both are String).

- `auto.offset.reset`: Tells the consumer to start at the beginning of the topic if no previous offset is found.

- `enable.auto.commit`: Enables automatic offset committing.

Instantiates the KafkaConsumer with the given properties.

`consume(String topic)` method:

- Subscribes the consumer to the test-topic (ignores the input parameter topic).

- Continuously polls Kafka for new messages using `poll(Duration.ofMillis(100))`.

- For each record:

  - Logs the received message’s value, partition, and offset.

  - Commits the offset synchronously using `commitSync()`

- Runs forever unless interrupted.

`close()` method:

- Closes the Kafka consumer gracefully.

⚠️ **Notes**: 
  - The topic parameter is unused; the method hardcodes test-topic instead. You might want to replace `List.of("test-topic")` with `List.of(topic)`.
  - `enable.auto.commit` is set to `true`, but `commitSync()` is called manually. This is redundant and potentially confusing.


```java
public class SimpleKafkaConsumer {
  private Properties props;
  private KafkaConsumer<String, String> consumer;

  public SimpleKafkaConsumer() {
    this.props = new Properties();

    props.put("bootstrap.servers","kafka1:9092,kafka2:9094,kafka3:9096");
    props.put("group.id","my-consumer-group2");
    props.put("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
    props.put("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
    props.put("auto.offset.reset","earliest");
    props.put("enable.auto.commit","true");

    this.consumer = new KafkaConsumer<>(props);
  }
  
  public void consume(String topic) {
    consumer.subscribe(List.of("test-topic"));

    while(true){
      ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
      for( ConsumerRecord<String, String> record :records){
        try {
          System.out.println("Received: " + record.value() + " from partition " + record.partition() + " at offset " + record.offset());
          consumer.commitSync();
        } catch (Exception e){
          e.printStackTrace();
        }
      }
    }
  }

  public void close() {
    this.consumer.close();
  }
}
```

## Kafka in Real Projects

- How Kafka fits into microservices

  In microservices architectures, Kafka serves as a backbone for asynchronous communication between services. It
  enables:
  - Event-driven architectures where services react to events
  - Service decoupling through publish-subscribe patterns
  - Data consistency across services using event sourcing
  - Scalable message processing with consumer groups
  - Reliable service communication with message persistence


- Integration with Spring Boot or Spark

  **Spring Boot Integration:**
  - Spring Kafka provides native integration with Kafka
  - Simplified configuration using application.properties
  - Annotation-based message handling (@KafkaListener)
  - Built-in serialization/deserialization support

  **Spark Integration:**
  - Spark Streaming can directly consume from Kafka topics
  - Structured Streaming provides DataFrame API for Kafka
  - Built-in exactly-once processing semantics
  - Scalable stream processing with Kafka partitions

## Key Takeaways

- Kafka is not just a message queue, it’s a distributed log

  While traditional message queues delete messages after consumption, Kafka maintains messages as an append-only log
  for a configurable retention period. This means multiple consumers can read the same messages independently, and
  messages can be replayed from any point in history. This distributed log architecture makes Kafka ideal for event
  sourcing, audit trails, and stream processing applications.

- Use cases include event sourcing, real-time analytics, and decoupling services

  Event sourcing with Kafka allows applications to maintain a complete history of state changes. Real-time analytics
  can process millions of events per second to provide immediate insights. Service decoupling enables building
  maintainable microservices architectures where services communicate through Kafka topics instead of direct calls.

- Easy to scale, hard to misuse if you understand the basics

  Kafka's partition-based architecture allows horizontal scaling by adding more brokers to the cluster. Each partition
  can be replicated across multiple brokers for fault tolerance. The producer/consumer model with consumer groups
  enables parallel processing while maintaining message ordering within partitions. Understanding these fundamentals
  helps avoid common pitfalls and ensures robust implementations.

### Practical Considerations

- Start with fewer partitions than you think you need - you can always add more later

  Starting with fewer partitions is recommended because:
    - Partitions consume system resources (files, memory, CPU)
    - Too many partitions can impact performance and leader election times
    - Each partition requires memory and open file handles on brokers
    - More partitions mean longer recovery time after broker failures

  You can add partitions in two ways:

  1. Using the kafka-topics.sh command line tool:
  ```shell
      # Increase partitions to 4 for topic "my-topic"
      bin/kafka-topics --bootstrap-server kafka1:9092 \
        --alter --topic my-topic \
        --partitions 4
  ```
  2. Using the AdminClient API programmatically:
  ```java 
  public class KafkaPartitionManager {
    public static void main(String[] args) {
        // Step 1: Define AdminClient configuration
        Properties config = new Properties();
        config.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "kafka1:9092,kafka2:9094,kafka3:9096");
      
        // Step 2: Create AdminClient
        try (AdminClient adminClient = AdminClient.create(config)) {
            // Step 3: Define the topic and new partition count
            String topicName = "test-topic";
            int newPartitionCount = 4;
      
            // Step 4: Prepare request
            Map<String, NewPartitions> newPartitionsMap = new HashMap<>();
            newPartitionsMap.put(topicName, NewPartitions.increaseTo(newPartitionCount));
      
            // Step 5: Send request to Kafka cluster
                  
            adminClient.createPartitions(newPartitionsMap).all().get();
            System.out.println("Successfully increased partitions for topic: " + topicName);
        } catch (ExecutionException e) {
            System.err.println("Failed to increase partitions: " + e.getCause().getMessage());
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            System.err.println("Partition update was interrupted");
        }
    }
  }
  ```

    After increasing the number of partitions for a Kafka topic using the AdminClient, the new partitions are immediately available. Here's how to utilize them effectively for both producers and consumers.

    - 📨 Producers (KafkaProducer)

      Kafka producers automatically distribute messages across available partitions using a partitioning strategy.

      1. Let Kafka Decide (Default Behavior)
      If no key is specified, Kafka uses a round-robin approach to distribute messages:

      ```java
      ProducerRecord<String, String> record = new ProducerRecord<>("test-topic", "my message");
      producer.send(record);
      ```

      💡 With 4 partitions, Kafka distributes messages among partitions 0–3.

      2. Use Keys to Control Partitioning
      Kafka uses the hash of the key to determine the partition:

      ```java
      ProducerRecord<String, String> record = new ProducerRecord<>("test-topic", "user123", "profile updated");
      producer.send(record);
      ```

      💡 Messages with the same key (e.g., "user123") go to the same partition.

      3. Specify Partition Explicitly
      You can manually target a partition:

      ```java
      ProducerRecord<String, String> record = new ProducerRecord<>("test-topic", 2, null, "explicit partition message");
      producer.send(record);
      ```

  - 📥 Consumers (KafkaConsumer)

    Consumers in the same group automatically rebalance when partition count changes.

    - If partitions > consumers → some consumers read from multiple partitions.
    - If consumers > partitions → some consumers remain idle.

    💡 Consumers should use:

    ```java
    consumer.subscribe(List.of("test-topic"));
    ```

    Restarting consumers can help ensure proper rebalance after increasing partitions.

  - ⚠️ Important Notes

    - Increasing partitions is irreversible — you cannot decrease the number of partitions.
    - Existing data is not redistributed; new messages will utilize new partitions.
    - Consumers must handle rebalances gracefully.

    | Role      | Action Post-Partition Increase                                          |
    |-----------|-------------------------------------------------------------------------|
    | Producer  | Continue sending — Kafka uses all partitions automatically              |
    | Consumer  | Use `subscribe()`; restart to rebalance if needed                       |
    | Monitoring| Use Kafka tools (e.g. `kafka-consumer-groups.sh`) to inspect assignments |
 
      **Note**: You can only increase the number of partitions, never decrease them. The partition increase is done without affecting existing data.

- Use appropriate key distribution to avoid partition hotspots

  When producing messages to Kafka, the partition selection is determined by the message key's hash (if a key is
  provided)
  or round-robin (if no key is provided). Poor key selection can lead to partition hotspots where some partitions
  receive
  significantly more data than others. For example:

  ```java
  // Bad: Using constant key
  producer.send(new ProducerRecord<>("topic", "constant-key", "message"));  // Always goes to same partition
  
  // Better: Using diverse keys
  producer.send(new ProducerRecord<>("topic", "user-" + userId, "message")); // Distributes across partitions
  ```

  Common strategies for good key distribution:
    - Use unique identifiers (user IDs, transaction IDs)
    - Combine multiple fields to create composite keys
    - Add random components for very high-volume single keys
    - Consider time-based prefixes for time-series data

  **Remember**: Messages with the same key always go to the same partition to maintain ordering guarantees within a key.


- Consider message compression for better network and storage efficiency

  Kafka supports several compression types (none, gzip, snappy, lz4, zstd) that can significantly reduce storage space
  and network bandwidth usage. Messages are compressed in batches by producers and decompressed by consumers.

  To enable compression in your producer:

  ```java
  // Enable compression in producer config
  Properties props = new Properties();
  props.put("compression.type", "snappy");  // Options: none, gzip, snappy, lz4, zstd
  
  // Create producer with compression enabled
  Producer<String, String> producer = new KafkaProducer<>(props);
  ```

  Compression considerations:
    - **gzip**: Highest compression ratio but more CPU intensive
    - **snappy**: Good balance between compression and speed
    - **lz4**: Fastest compression/decompression with decent ratios
    - **zstd**: Better compression than gzip with better performance

  Best practices:
    - Use snappy or lz4 for most use cases
    - Consider gzip for low-throughput scenarios where storage/bandwidth savings are critical
    - Monitor compression ratios and CPU usage to find the optimal setting
    - Larger batch sizes generally improve compression ratios


- Implement proper error handling and dead letter queues

  Error handling in Kafka consumers is crucial for maintaining system reliability. When a message cannot be processed,
  it's important to handle it gracefully and avoid blocking the entire consumer group. Dead Letter Queues (DLQ) provide
  a way to handle problematic messages by moving them to a separate topic for later analysis or reprocessing.

  Example implementation of error handling with DLQ:

  ```java
  Properties consumerProps = new Properties();
  consumerProps.put("bootstrap.servers", "kafka1:9092");
  consumerProps.put("group.id", "your-consumer-group-id");
  consumerProps.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
  consumerProps.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
  consumerProps.put("auto.offset.reset", "earliest");
  consumerProps.put("enable.auto.commit", "false");

  KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProps);
  Producer<String, String> dlqProducer = new KafkaProducer<>(producerProps);

  consumer.subscribe(List.of("test-topic"));
  try {
    while (true) {
        ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
        for (ConsumerRecord<String, String> record : records) {
            try {
                processRecord(record);  // Your business logic
                consumer.commitSync(Map.of(
                    new TopicPartition(record.topic(), record.partition()),
                    new OffsetAndMetadata(record.offset() + 1)
                ));
            } catch (ProcessingException e) {
                ProducerRecord<String, String> dlqRecord = new ProducerRecord<>("dlq-topic", record.key(), record.value());
                record.headers().forEach(h -> dlqRecord.headers().add(h));
                dlqProducer.send(dlqRecord).get();

                consumer.commitSync(Map.of(
                    new TopicPartition(record.topic(), record.partition()),
                    new OffsetAndMetadata(record.offset() + 1)
                ));
                log.error("Failed to process record. Sent to DLQ.", e);
            }
        }
    }
  } catch (Exception e) {
    log.error("Consumer error", e);
  } finally {
    consumer.close();
    dlqProducer.close();
  }
  ```

  Best practices for error handling:
    - Use manual commit mode for better control over message acknowledgment
    - Implement retry logic with exponential backoff for transient failures
    - Include original message metadata when sending to DLQ
    - Monitor DLQ size and processing failures
    - Implement a process to analyze and potentially reprocess DLQ messages


- Regular monitoring of broker health and partition leadership

  Monitoring Kafka clusters is essential for maintaining a healthy system. Key metrics to monitor include:

  **Broker Health Metrics:**
    - CPU, memory, and disk usage
    - Network throughput and latency
    - Number of under-replicated partitions
    - Request handler and network processor thread pool usage

  Example JMX monitoring configuration:
  ```java
  // Enable JMX monitoring in server.properties
  Properties props = new Properties();
  props.put("jmx.port", "9999");
  props.put("kafka.metrics.polling.interval.secs", "60");
  ```

  **Partition Leadership:**
    - Monitor partition leadership distribution
    - Watch for leadership skew (too many leaders on one broker)
    - Track partition reassignment events
    - Monitor replica lag and ISR shrinks/expands

  Common monitoring tools:
    - Kafka Manager (CMAK)
    - Prometheus with JMX Exporter
    - Confluent Control Center
    - Custom JMX monitoring

  Best practices:
    - Set up alerts for critical metrics
    - Monitor both broker-level and topic-level metrics
    - Track historical trends for capacity planning
    - Implement automated failover testing

## Administration and Security

### Security Features

Kafka provides several security features:

- **Authentication**: Supports SASL (PLAIN, SCRAM, GSSAPI/Kerberos) and SSL/TLS client authentication
- **Authorization**: ACL-based authorization for topic operations
- **Encryption**: SSL/TLS encryption for client-broker and inter-broker communication
- **Audit**: Logging of client operations and access attempts

Example SSL configuration for producers:

