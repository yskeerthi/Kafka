
---

# 📘 Introduction to Kafka – A Helicopter View

## 📍 What is Apache Kafka?

* **Kafka** is a **distributed event store** and a **streaming platform**.
* It originated as an internal **LinkedIn project** and is now widely used by organizations like **Netflix**, **Uber**, and many others.
* Kafka is a critical component of modern **event-driven architectures** and **data pipelines**.

### ✅ Why Kafka?

* **Reliability** – Ensures message delivery and durability.
* **Scalability** – Handles vast amounts of data across many systems.

---

## 🧩 Core Concepts of Kafka

---

### 1. 📦 Kafka Messages and Batches

* **Message**: The basic unit of data in Kafka (like a row in a database).

  * Stored as a **byte array** (schema-free by default).
  * Can use **JSON**, **XML**, or **Apache Avro** for structured data.

* **Batch**: A collection of messages sent together to a topic/partition.

  * ✅ Reduces overhead (fewer network round trips).
  * ✅ Allows compression → efficient transfer/storage.
  * ⚖️ Trade-off: Larger batches = higher throughput but higher latency.

---

### 2. 📁 Kafka Topics and Partitions

* **Topic**: A category or feed name where messages are published (similar to a database table).
* **Partition**: A subset of a topic, ordered and immutable sequence of records.

📊 **Example:**

```
Topic: numbers
├── Partition 0
├── Partition 1
└── Partition 2
```

* Messages are **appended** to partitions → **ordered** within a partition.
* **No global ordering** across partitions.
* ✅ **Scalability**: Each partition can reside on a different broker.
* ✅ **Redundancy**: Partitions can be **replicated** across brokers for fault tolerance.

📸 **Diagram – Topics & Partitions:**

```
Topic: Orders
  ├─ Partition 0: [msg1, msg2, msg3]
  ├─ Partition 1: [msg4, msg5]
  └─ Partition 2: [msg6, msg7, msg8]
```

---

### 3. 👨‍💻 Kafka Producers and Consumers

* **Producer**: Creates and sends messages to Kafka.

  * Specifies the topic.
  * Can control partition assignment (default: round-robin).

* **Consumer**: Reads messages from Kafka.

  * Works as part of a **consumer group**.
  * Each partition is read by **only one consumer** in a group (ownership).
  * If one consumer fails, others take over.

📸 **Consumer Group Example:**

```
Topic: Logs
  ├─ Partition 0 ─► Consumer A
  ├─ Partition 1 ─► Consumer B
  └─ Partition 2 ─► Consumer C
```

✅ **Horizontal scalability**: Add more consumers to process more data in parallel.

---

### 4. 🖥️ Kafka Brokers and Clusters

* **Broker**: A single Kafka server.

  * **Producer side tasks**: Receive messages, assign offsets, write to disk.
  * **Consumer side tasks**: Handle fetch requests and deliver messages.

* **Cluster**: A group of brokers working together.

  * One broker is elected **controller** – manages metadata, partitions, and broker failures.
  * A broker can handle **millions of messages/second**.

📸 **Basic Architecture:**

```
Producers ─► [Broker 1 | Broker 2 | Broker 3] ◄─ Consumers
                   (Kafka Cluster)
```

---

### 5. 🔁 Kafka Cluster Message Replication

* Each partition has:

  * **Leader** – handles all reads/writes.
  * **Followers** – replicate data for fault tolerance.

* **Failover**: If a leader fails, a follower becomes the new leader automatically.

📸 **Replication Example:**

```
Partition 0 (Topic A)
  ├─ Broker 1 (Leader)
  └─ Broker 2 (Follower)

Partition 1 (Topic A)
  ├─ Broker 2 (Leader)
  └─ Broker 3 (Follower)
```

* Producers always write to the **leader**.
* Consumers can read from **leader or follower**.

---

## 🚀 Advantages of Using Kafka

✅ **Multiple Producers Support**

* Aggregate data from multiple sources into unified streams.

✅ **Multiple Consumers Support**

* Multiple consumers can read the same message stream independently.
* Or, they can form a **group** to share workload.

✅ **Disk-Based Retention**

* Messages stored with retention policies (e.g., 7 days).
* Consumers can restart anytime without data loss.

✅ **Horizontal Scalability**

* Start small and scale to hundreds of brokers.
* Scale **online** with no downtime.

---

## ⚠️ Disadvantages of Kafka

❌ **Complex Configuration**

* Many settings → steep learning curve.

❌ **Weak Tooling**

* Command-line tools are inconsistent and limited.

❌ **Limited Client Support**

* Best clients: Java, C. Others (e.g., Python, Go) still maturing.

❌ **No True Multi-Tenancy**

* Difficult to isolate logical clusters within one physical cluster.

---

## 🧠 Summary

| Kafka Concept   | Description                    |
| --------------- | ------------------------------ |
| **Message**     | Basic data unit                |
| **Batch**       | Group of messages              |
| **Topic**       | Category/feed name             |
| **Partition**   | Ordered, append-only log       |
| **Producer**    | Sends data                     |
| **Consumer**    | Reads data                     |
| **Broker**      | Kafka server                   |
| **Cluster**     | Group of brokers               |
| **Replication** | Data redundancy across brokers |

---

## 📊 Kafka Architecture Diagram (Simplified)

```
[ Producers ] ─► [ Broker 1 (Leader) ] ─► [ Consumers ]
                     │
                     ├─ Replication ─► [ Broker 2 (Follower) ]
                     └─ Replication ─► [ Broker 3 (Follower) ]
```

---


---

Complete Flow of MiniProducer Microservice: Start to End
🏗️ Project Structure & Component Relationships
1️⃣ Database Layer (Foundation)
Files:

entity/TransportDocument.java - Database entity representing shipping documents
entity/Reference.java - Database entity for document references
How they work:

These files use @Entity, @Table, @Column annotations to map Java classes to database tables
They define relationships with @OneToMany and @ManyToOne annotations
Example: @ManyToOne(fetch = FetchType.LAZY) @JoinColumn(name = "transport_document_id") in Reference connects to TransportDocument
2️⃣ Data Access Layer
Files:

repository/TransportDocumentRepository.java - Database access for documents
repository/ReferenceRepository.java - Database access for references
How they work:

They extend JpaRepository interface providing CRUD operations
Custom query methods like findByTransportDocumentNumber() generate SQL automatically
@Repository annotation registers them as Spring beans
Used by Service layer to retrieve data
3️⃣ Message Format Definition
Files:

src/main/resources/avro/*.avsc - Avro schema definitions
avro/TransportDocumentEvent.java - Auto-generated message class
avro/Reference.java - Auto-generated reference class
How they work:

.avsc files define the message structure in JSON format
Maven plugin processes these during build: mvn generate-sources
Generated classes provide serialization for Kafka
4️⃣ Kafka Configuration
Files:

config/KafkaTopicConfig.java - Defines Kafka topics
application.properties - Kafka connection settings
How they work:

@Configuration annotation identifies this as a Spring config class
@Bean methods create topic configurations
@Value annotations inject properties from application.properties
Settings like bootstrap servers and serializers defined in properties
5️⃣ Core Services
Files:

service/TransportDocumentService.java - Core business logic
service/CustomerProducer.java - Kafka message sender
service/AutoProducerService.java - Automated publishing
How they work:

@Service annotations register them as Spring beans
@Autowired annotations inject repositories and other services
CustomerProducer uses KafkaTemplate to send messages
TransportDocumentService converts entities to Avro objects
6️⃣ REST API
Files:

controller/TransportDocumentController.java - HTTP endpoints
How they work:

@RestController annotation identifies this as a REST endpoint provider
@RequestMapping defines URL paths
@GetMapping, @PostMapping annotations define HTTP methods
Injects services via constructor to access business logic
🔄 Complete Data Flow Process
1. Application Startup
Spring Boot starts (MiniproducerApplication.main())
Reads application.properties
Creates database connection using properties
Creates Kafka topics defined in KafkaTopicConfig
2. Automatic Data Publishing
AutoProducerService.startAutomaticProduction() triggered by @EventListener(ApplicationReadyEvent.class)
Calls transportDocumentRepository.findAll() to get all documents
Loops through documents calling transportDocumentService.publishDocumentToKafka(id)
3. Document Publishing Process
TransportDocumentService.publishDocumentToKafka(id) fetches document by ID
Converts database entity to Avro object:
Calls customerProducer.sendMessage(topicName, event)
4. Kafka Message Production
CustomerProducer.sendMessage() uses KafkaTemplate to send message
Message serialized using Avro serializer (configured in application.properties)
Message sent to Kafka topic defined in KafkaTopicConfig
5. Manual Operations via REST API
User can trigger publishing via HTTP endpoints:
POST /api/transport-documents/{id}/publish
POST /api/transport-documents/publish-all
These endpoints call the same service methods as auto-publishing
🔌 Key Annotation References Between Files
@Entity + @Table → Connect Java classes to database tables
@Repository → Register repositories as Spring components
@Service → Register service classes as Spring components
@RestController → Register controller as Spring web component
@Autowired → Inject repositories into services, services into controllers
@Bean → Register Kafka topics with Spring
@Value → Connect properties file values to Java variables
@EventListener → Trigger auto-publishing on application start
🖥️ Commands Used in Development
Initialize Project:

Generate Avro Classes: mvn generate-sources

Build Application: mvn clean package

Run Application:

Test REST Endpoints: curl -X POST http://localhost:8080

View Kafka Messages: kafka-avro-console-consumer --bootstrap-server localhost:9092 --topic transport-document-events --from-beginning


