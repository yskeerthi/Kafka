
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
```
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
```
```
**PART1:** 
Kafka Core Concepts: Clear & Concise Explanations
What is Apache Kafka?
"Apache Kafka is a distributed event streaming platform designed for high-throughput, fault-tolerant, real-time data streaming. At its core, Kafka functions as a distributed commit log, durably storing sequences of events that can be read by multiple consumers at their own pace."

Why Kafka vs Other Alternatives?
"We chose Kafka over alternatives like RabbitMQ, ActiveMQ, and traditional ESBs for several key reasons:

Durability: Kafka persists messages to disk, allowing replay and recovery
Scalability: Handles millions of messages per second through partitioning
Retention: Stores messages for configurable periods, not just until consumption
Multi-consumer: Enables multiple applications to read the same data independently
Ordering: Guarantees message order within partitions
While RabbitMQ excels at complex routing patterns and ActiveMQ offers JMS compatibility, Kafka's combination of high throughput, durability, and scalability makes it optimal for our event-driven architecture."

Core Kafka Concepts
1. Producers
"Producers are applications that publish (write) events to Kafka topics. They can send messages to specific topics and optionally specify which partition to use. Our TransportDocumentService acts as a producer when it sends document events to Kafka."

2. Topics
"Topics are categories or feeds to which records are published. They're similar to tables in a database but optimized for streaming. In our system, 'transport-document-events' is a topic that streams all document-related events."

3. Partitions
"Each topic is divided into partitions - ordered, immutable sequence of records. Partitioning allows Kafka to scale horizontally and process data in parallel. Messages with the same key always go to the same partition, ensuring ordered processing for related events."

4. Brokers
"Brokers are the Kafka servers that store data and handle client requests. Multiple brokers form a Kafka cluster. Each broker manages certain topic partitions and handles read/write requests for those partitions."

5. Clusters
"A Kafka cluster consists of multiple brokers working together. The cluster provides redundancy and increased capacity. If one broker fails, others can take over its responsibilities, ensuring high availability."

6. Consumers
"Consumers read (subscribe to) events from topics. They can read independently at their own pace. Multiple consumers can be organized into consumer groups for parallel processing, with each partition being read by exactly one consumer in each group."

7. Zookeeper
"Zookeeper is the coordination service for Kafka (in traditional deployments). It maintains broker metadata, tracks cluster membership, elects controllers, and handles broker failover. Newer Kafka versions can use KRaft mode instead, eliminating Zookeeper dependency."

Message Flow in Kafka
"The message flow in Kafka follows these steps:

Production: Producer sends message to a topic
Distribution: Kafka broker assigns message to a specific partition
Storage: Message is written to the partition's commit log on disk
Replication: Message is replicated to other brokers for fault tolerance
Consumption: Consumers read messages from partitions at their own pace
Offset Management: Consumers track their position (offset) in each partition
This decoupled flow enables high throughput and resilience, as producers aren't blocked by consumer processing speed."

```
```
****Part 2: Our Architecture Deep Dive****
```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│    Database     │────▶│  MiniProducer   │────▶│     Kafka       │
│    (MySQL)      │     │  Application    │     │                 │
└─────────────────┘     └─────────────────┘     └─────────────────┘
Our system follows a clear three-layer architecture that separates concerns effectively:

Data Layer: JPA entities and repositories
Service Layer: Business logic and transformations
Messaging Layer: Kafka integration and event publishing"
Database Design & Entity Relationships
"We designed our database with a one-to-many relationship between transport documents and references:

┌─────────────────────────┐       ┌────────────────────────┐
│ transport_documents     │       │ document_references    │
├─────────────────────────┤       ├────────────────────────┤
│ id                      │       │ id                     │
│ transport_document_number│       │ transport_document_id  │──┐
│ event_trigger           │       │ reference_type_code    │  │
│ create_source           │       │ reference_value        │  │
└─────────────────────────┘       └────────────────────────┘  │
        │                                                     │
        └─────────────────────────────────────────────────────┘
 ```java
@Entity
@Table(name = "transport_documents")
public class TransportDocument {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "transport_document_number", unique = true)
    private String transportDocumentNumber;
    
    @Column(name = "event_trigger")
    private String eventTrigger;
    
    // Other fields...
    
    @OneToMany(mappedBy = "transportDocument", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<Reference> references = new ArrayList<>();
    
    // Getters and setters...
}
```

```java
@Entity
@Table(name = "document_references")
public class Reference {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "reference_id", nullable = false)
    private String referenceId;
    
    @Column(name = "reference_type_code", nullable = false)
    private String referenceTypeCode;
    
    @Column(name = "reference_value", nullable = false)
    private String referenceValue;
    
    // Other fields...
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "transport_document_id", nullable = false)
    private TransportDocument transportDocument;
    
    // Getters and setters...
}
```
```
This design allows:

Each transport document to have multiple references
Different types of references (BL numbers, booking numbers, etc.)
Efficient querying of documents with their associated references"
Avro Schemas & Data Conversion
"We define our message structure using Avro schemas in .avsc files:

The Maven Avro plugin generates Java classes from these schemas during build:
```
<plugin>
    <groupId>org.apache.avro</groupId>
    <artifactId>avro-maven-plugin</artifactId>
    <version>${avro.version}</version>
    <executions>
        <execution>
            <phase>generate-sources</phase>
            <goals>
                <goal>schema</goal>
            </goals>
            <configuration>
                <sourceDirectory>${project.basedir}/src/main/resources/avro</sourceDirectory>
                <outputDirectory>${project.build.directory}/generated-sources/avro</outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```
```
A key architectural decision was separating our database entities from our message format, giving us flexibility to evolve each independently."

Service Layer & Entity-to-Avro Conversion
"Our TransportDocumentService handles the core business logic, including database access and conversion to Avro objects:
```
```
@Service
public class TransportDocumentService {
    private final TransportDocumentRepository transportDocumentRepository;
    private final CustomerProducer customerProducer;
    private final String topicName;
    
    // Constructor injection...
    
    @Transactional
    public void publishDocumentToKafka(Long documentId) {
        TransportDocument document = transportDocumentRepository
            .findByIdWithReferences(documentId)
            .orElseThrow(() -> new ResourceNotFoundException("Document not found: " + documentId));
            
        TransportDocumentEvent event = convertToAvroEvent(document);
        customerProducer.sendMessage(topicName, event);
    }
    
    private TransportDocumentEvent convertToAvroEvent(TransportDocument document) {
        TransportDocumentEvent event = new TransportDocumentEvent();
        event.setTransportDocumentNumber(document.getTransportDocumentNumber());
        event.setEventTrigger(document.getEventTrigger());
        
        List<com.example.miniproducer.avro.Reference> avroReferences = new ArrayList<>();
        
        for (Reference entityReference : document.getReferences()) {
            com.example.miniproducer.avro.Reference avroRef = new com.example.miniproducer.avro.Reference();
            avroRef.setReferenceId(entityReference.getReferenceId());
            avroRef.setReferenceTypeCode(entityReference.getReferenceTypeCode());
            avroRef.setReferenceValue(entityReference.getReferenceValue());
            // Map other fields...
            
            avroReferences.add(avroRef);
        }
        
        event.setReferences(avroReferences);
        return event;
    }
    
    // More methods...
}
```
This service:

Retrieves documents with their references using a single optimized query
Converts database entities to Avro objects
Sends the Avro event to Kafka via the producer
The separation between entity and Avro classes gives us flexibility to change our database schema independently from our message format."

Kafka Integration
"Our Kafka integration is configured in several key components:
```
```
**1.Topic Configuration**
```
```java
@Configuration
public class KafkaTopicConfig {
    @Value("${topic.name}")
    private String topicName;
    
    @Bean
    public NewTopic transportDocumentTopic() {
        return TopicBuilder.name(topicName)
                .partitions(3)
                .replicas(1)
                .build();
    }
}
```
****2.Producer Service:****
```java
@Service
public class CustomerProducer {
    private final KafkaTemplate<String, TransportDocumentEvent> kafkaTemplate;
    private final Logger log = LoggerFactory.getLogger(CustomerProducer.class);
    
    public CustomerProducer(KafkaTemplate<String, TransportDocumentEvent> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }
    
    public void sendMessage(String topic, TransportDocumentEvent event) {
        log.info("Producing message to topic {}: {}", topic, event.getTransportDocumentNumber());
        
        // Using document number as key ensures same document always goes to same partition
        kafkaTemplate.send(topic, event.getTransportDocumentNumber(), event);
    }
}
```
```
**3.Auto-Producer for Startup Publishing:**

```

```java
@Service
public class AutoProducerService {
    private final TransportDocumentService transportDocumentService;
    private final Logger log = LoggerFactory.getLogger(AutoProducerService.class);
    
    @EventListener(ApplicationReadyEvent.class)
    public void startAutomaticProduction() {
        log.info("🚀 Auto-Producer Service starting...");
        produceAllMessages();
    }
    
    public void produceAllMessages() {
        transportDocumentService.publishAllDocumentsToKafka();
    }
}
```
```
**4.Kafka Configuration in application.properties:**

```
```java
# Kafka Producer Configuration
spring.kafka.bootstrap-servers=localhost:9092
spring.kafka.producer.key-serializer=org.apache.kafka.common.serialization.StringSerializer
spring.kafka.producer.value-serializer=io.confluent.kafka.serializers.KafkaAvroSerializer
spring.kafka.producer.properties.schema.registry.url=http://localhost:8081

# Producer Performance Tuning
spring.kafka.producer.batch-size=16384
spring.kafka.producer.linger.ms=5
spring.kafka.producer.compression-type=snappy

# Topic Configuration
topic.name=transport-document-events
```

Our Kafka integration carefully handles:

Schema serialization with Avro
Performance optimization with batching and compression
Key-based partitioning for ordering by document number
Automated startup publishing"
REST API for Manual Control
"We provide a REST API for manual control of document publishing:
```
```java
@RestController
@RequestMapping("/api/transport-documents")
public class TransportDocumentController {
    private final TransportDocumentService transportDocumentService;
    
    public TransportDocumentController(TransportDocumentService transportDocumentService) {
        this.transportDocumentService = transportDocumentService;
    }
    
    @PostMapping("/{id}/publish")
    public ResponseEntity<String> publishDocument(@PathVariable Long id) {
        transportDocumentService.publishDocumentToKafka(id);
        return ResponseEntity.ok("Document published successfully");
    }
    
    @PostMapping("/publish-all")
    public ResponseEntity<String> publishAllDocuments() {
        transportDocumentService.publishAllDocumentsToKafka();
        return ResponseEntity.ok("All documents published successfully");
    }
}
```

```
This API allows manual trigger of document publishing for testing or recovery scenarios."

Key Architectural Decisions
"To summarize our key architectural decisions:

Database Design: One-to-many relationship between documents and references
Entity-Avro Separation: Decoupled database schema from message format
Lazy Loading with Optimized Queries: Used FetchType.LAZY with specific eager fetch queries
Avro Schema Design: Schema evolution with default values for backward compatibility
Kafka Topic Design: 3 partitions with document number as key for balanced scaling
Automatic Publishing: Event listener for startup publishing
Performance Optimization: Batching, compression, and key-based partitioning
These decisions balance performance, maintainability, and flexibility for our transport document processing needs."
```
