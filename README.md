# ApacheKafkaLearnings

Article from https://www.instaclustr.com/education/apache-kafka/spring-boot-with-apache-kafka-tutorial-and-best-practices/ 

What is Apache Kafka?
Apache Kafka is a distributed event streaming platform for high-throughput, fault-tolerant, and real-time data processing. It allows applications to publish, subscribe to, store, and process event streams. Kafka is used for building event-driven architectures, log aggregation, and real-time analytics.

Kafka operates with producers (sending messages), topics (message categories), brokers (servers managing messages), and consumers (receiving messages). It ensures data durability and fault tolerance through replication across multiple nodes. Due to its scalability and performance, Kafka is a popular choice for messaging and event-driven applications.

What is Spring Boot?
Spring Boot is a framework built on top of the Spring framework that simplifies the development of Java-based applications. It removes the need for extensive configuration by providing default settings and embedded application servers.

Spring Boot enables rapid development through features like auto-configuration, dependency management, and an opinionated approach to application setup. It is used for building microservices, REST APIs, and enterprise applications with minimal boilerplate code.

Why integrate Kafka with Spring Boot?
Integrating Apache Kafka with Spring Boot simplifies the development of event-driven applications by leveraging Spring’s abstraction and Kafka’s messaging capabilities. This combination enables efficient data streaming, fault-tolerant processing, and real-time analytics.

Key reasons to integrate Kafka with Spring Boot:

Simplified configuration: Spring Boot provides auto-configuration and easy integration with Kafka through spring-kafka dependency.
Event-driven architecture: Kafka enables asynchronous, decoupled communication between microservices, improving scalability.
High throughput and fault tolerance: Kafka ensures reliable message delivery with replication and partitioning, while Spring Boot handles message processing efficiently.
Built-in consumer and producer support: Spring Boot’s KafkaTemplate simplifies message publishing, and @KafkaListener makes message consumption straightforward.
Scalability and load balancing: Kafka partitions help distribute workload across multiple consumers, and Spring Boot manages processing efficiently.
Integration with other Spring modules: Works well with Spring Cloud, Spring Data, and Spring Security for building enterprise-grade applications.
This integration is used for log processing, event sourcing, and microservices communication.

Tutorial: Getting started with Spring for Apache Kafka
This tutorial covers how to integrate Apache Kafka with a Spring Boot application to send and receive messages. We’ll walk through the steps of setting up Kafka, adding dependencies, and implementing a producer and consumer.

Prerequisites
Before starting, ensure you have the following installed:

Apache Kafka (installed and running)
Java 17 or higher
Maven or Gradle
Spring Boot (if not using start.spring.io, manually add dependencies)
To add Kafka dependencies, use:

Maven:
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
Gradle:
dependencies {
    implementation 'org.springframework.kafka:spring-kafka'
}
If using Spring Boot, the correct Kafka version will be resolved automatically.

Setting up a Spring Boot Kafka consumer
A Kafka consumer subscribes to a topic and listens for messages. Below is a minimal consumer setup:

Consumer application

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
 
    @Bean
    public NewTopic topic() {
        return TopicBuilder.name("mytopic")
                .partitions(10)
                .replicas(1)
                .build();
    }
 
    @KafkaListener(id = "exampleId", topics = "mytopic")
    public void listen(String message) {
        System.out.println("Received message: " + message);
    }
}
Kakfa Spring Boot setup screenshot

Explanation:

@KafkaListener(id = "exampleId", topics = "mytopic") listens to messages from topic1.
NewTopic bean ensures the topic exists (not needed if the topic is already created).
Messages received from Kafka are printed to the console.
Configuration (application.properties):

spring.kafka.consumer.auto-offset-reset=earliest
This ensures messages are consumed from the beginning if no offset is present.

Setting up a Spring Boot Kafka producer
A Kafka producer sends messages to a topic.

Producer application

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
 
    @Bean
    public NewTopic topic() {
        return TopicBuilder.name("mytopic")
                .partitions(10)
                .replicas(1)
                .build();
    }
 
    @Bean
    public ApplicationRunner runner(KafkaTemplate<String, String> template) {
        return args -> {
            template.send("mytopic", "Hello, Kafka!");
        };
    }
}
Kakfa Spring Boot setup screenshot

Explanation:

KafkaTemplate<String, String> is used to send messages to Kafka.
The ApplicationRunner bean sends a message to mytopic when the application starts.
Tips from the expert
Andrew Mills
Andrew Mills

Senior Solution Architect

Andrew Mills is an industry leader with extensive experience in open source data solutions and a proven track record in integrating and managing Apache Kafka and other event-driven architectures.

In my experience, here are tips that can help you better integrate Spring Boot with Apache Kafka:

Use consumer lag monitoring for performance tuning: Monitor consumer lag using JMX metrics or Kafka’s built-in tools (e.g., kafka-consumer-groups.sh). High lag indicates slow processing, requiring either consumer scaling or better optimization.
Optimize producer throughput with batching and compression: Configure linger.ms to delay sending messages slightly, allowing batching for efficiency. Enable compression.type=zstd or gzip for improved network performance.
Use dead letter topics for fault handling: Instead of retrying indefinitely, configure a Dead Letter Queue (DLQ) topic for messages that consistently fail. Spring Boot’s @KafkaListener supports error handling with SeekToCurrentErrorHandler.
Leverage Confluent Schema Registry for schema evolution: Instead of manually handling schema changes, use Avro/Protobuf with Confluent Schema Registry. This prevents compatibility issues when evolving message structures.
Tune consumer thread count for parallel processing: Spring Boot’s ConcurrentKafkaListenerContainerFactory allows you to configure multiple threads (setConcurrency(n)) to parallelize message consumption across partitions.
Spring for Apache Kafka: 3 tips and best practices
Here are some recommendations for using Spring Boot with Apache Kafka.

1. Manually assigning all partitions
In some cases, you may need to read all records from all partitions, such as when using a compacted topic to load a distributed cache. Instead of relying on Kafka’s group management, you can manually assign partitions. However, this approach becomes complex if the number of partitions changes over time, as you would need to update your application accordingly.

A more flexible approach is using a SpEL expression to dynamically determine partition assignments at runtime. The following example achieves this by creating a partition list when the application starts:

@KafkaListener(topicPartitions = @TopicPartition(topic = "compacted",
        partitions = "#{@finder.partitions('compacted')}",
        partitionOffsets = @PartitionOffset(partition = "*", initialOffset = "0")))
public void listen(@Header(KafkaHeaders.RECEIVED_KEY) String key, String payload) {
    // Process the message
    System.out.println("Received message with key: " + key + " and payload: " + payload);
}
Here, PartitionFinder dynamically retrieves partitions for the topic:

@Bean
public PartitionFinder finder(ConsumerFactory<String, String> consumerFactory) {
    return new PartitionFinder(consumerFactory);
}
 
public static class PartitionFinder {
    private final ConsumerFactory<String, String> consumerFactory;
 
    public PartitionFinder(ConsumerFactory<String, String> consumerFactory) {
        this.consumerFactory = consumerFactory;
    }
 
    public String[] partitions(String topic) {
        try (Consumer<String, String> consumer = consumerFactory.createConsumer()) {
            return consumer.partitionsFor(topic).stream()
                .map(pi -> "" + pi.partition())
                .toArray(String[]::new);
        }
	  catch (Exception e) {
		logger.error("Error retrieving partitions for topic: {}", topic, e);
	  }
    }
}
Setting spring.kafka.consumer.auto-offset-reset=earliest ensures that messages are consumed from the beginning each time the application starts. Additionally, you should set the container’s acknowledgment mode to MANUAL to prevent automatic offset commits when using manual partition assignment.

2. Kafka Transactions with other transaction managers
Spring Boot supports transactional operations involving both Kafka and a database. A common use case is ensuring that a database update and Kafka message publishing are executed as part of a single atomic transaction.

The following example demonstrates a transactional setup where a database transaction commits first, followed by a Kafka transaction:

@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
 
    @Bean
    public ApplicationRunner runner(KafkaTemplate<String, String> template) {
        return args -> template.executeInTransaction(t -> t.send("mytopic", "test"));
    }
 
    @Bean
    public DataSourceTransactionManager dstm(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
 
    @Component
    public static class Listener {
        private final JdbcTemplate jdbcTemplate;
        private final KafkaTemplate<String, String> kafkaTemplate;
 
        public Listener(JdbcTemplate jdbcTemplate, KafkaTemplate<String, String> kafkaTemplate) {
            this.jdbcTemplate = jdbcTemplate;
            this.kafkaTemplate = kafkaTemplate;
        }
 
        @KafkaListener(id = "mygroup", topics = "mytopic")
        @Transactional("dstm")
        public void listen1(String in) {
            this.kafkaTemplate.send("myothertopic", in.toUpperCase());
            this.jdbcTemplate.execute("insert into mytable (data) values ('" + in + "')");
        }
 
        @KafkaListener(id = "myothergroup", topics = "myothertopic")
        public void listen2(String in) {
            System.out.println(in);
        }
    }
 
    @Bean
    public NewTopic mytopic() {
        return TopicBuilder.name("mytopic").build();
    }
 
    @Bean
    public NewTopic myothertopic() {
        return TopicBuilder.name("myothertopic").build();
    }
}
Configuration properties:

spring.datasource.url=jdbc:mysql://localhost/integration?serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=your_password
 
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
 
spring.kafka.consumer.auto-offset-reset=earliest
spring.kafka.consumer.enable-auto-commit=false
spring.kafka.consumer.properties.isolation.level=read_committed
 
spring.kafka.producer.transaction-id-prefix=tx-
This setup ensures that the database transaction commits first. If the Kafka transaction fails, the record will be redelivered, making the database update idempotent.

If you want to commit the Kafka transaction first and only commit the database transaction if the Kafka transaction succeeds, use nested @Transactional methods:

@Transactional("dstm")
public void someMethod(String in) {
    this.jdbcTemplate.execute("insert into mytable (data) values ('" + in + "')");
    sendToKafka(in);
}
 
@Transactional("kafkaTransactionManager")
public void sendToKafka(String in) {
    this.kafkaTemplate.send("topic2", in.toUpperCase());
}
Here, sendToKafka() runs in a separate Kafka transaction, ensuring that the Kafka message is committed before the database update.

3. Customizing the JsonSerializer and JsonDeserializer
Kafka’s JsonSerializer and JsonDeserializer can be customized using properties or by subclassing them. If you need a custom ObjectMapper, you can extend JsonSerializer:

public class CustomJsonSerializer extends JsonSerializer<Object> {
 
    // Constructor initializes the serializer with a customized ObjectMapper.
    public CustomJsonSerializer() {
        super(customizedObjectMapper());
    }
 
    // Creates and configures an ObjectMapper instance with custom settings.
    private static ObjectMapper customizedObjectMapper() {
        // Utilize a pre-configured enhanced ObjectMapper from JacksonUtils.
        ObjectMapper mapper = JacksonUtils.enhancedObjectMapper();
        // Configure the mapper to serialize dates in a readable format rather than timestamps.
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        return mapper;
    }
}
This allows you to modify JSON serialization behavior while still leveraging Spring Kafka’s automatic serialization mechanisms.

Why choose NetApp Instaclustr for your Spring Boot and Apache Kafka Integration
NetApp Instaclustr for Apache Kafka provides the perfect foundation for seamless and efficient integrations, such as pairing Spring Boot with Apache Kafka. Designed with reliability and performance in mind, Instaclustr’s managed Kafka service handles the operational heavy lifting like the complexities of setup, authentication, and cluster maintenance, letting developers focus on building impactful applications rather than managing infrastructure.

Whether you’re navigating the initial setup or optimizing for peak performance, Instaclustr offers the expertise and tools to empower your success. If you’re aiming to simplify your Kafka architecture while maintaining flexibility and scalability, NetApp Instaclustr is your trusted partner to make it happen. Get started today and unlock the true potential of Spring Boot and Apache Kafka in your applications.
