---
title: Create Producer and Consumer Using spring-kafka  
date: 2024-04-21 17:26:00 +0800  
categories: [Technology, A Simple Application Using spring-kafka]  
tags: [java, kafka]  
---
The Spring for Apache Kafka (spring-kafka) project applies core Spring concepts to the development of Kafka-based messaging solutions. It provides a "template" as a high-level abstraction for sending messages. It also provides support for Message-driven POJOs with @KafkaListener annotations and a "listener container". These libraries promote the use of dependency injection and declarative.  
This article will demonstrate creating kafka producer/consumer and respective unit test using an embedded instance of Kafka. All the codes of the article can be downloaded from [here](https://github.com/hivsuper/learning-journey/tree/master/demos/kafka-demo).
## 1. Prerequisites
- Java 21+
- Kafka topic is ready. See [Install-Kafka-in-KRaft-Mode-Using-Docker](/posts/Install-Kafka-in-KRaft-Mode-Using-Docker/)

## 2. Dependencies
Add the standard spring-kafka dependency to pom.xml
```xml
<dependency>
  <groupId>org.springframework.kafka</groupId>
  <artifactId>spring-kafka</artifactId>
  <version>3.1.4</version>
</dependency>
```
Add one more dependency for test.
```xml
<dependency>
  <groupId>org.springframework.kafka</groupId>
  <artifactId>spring-kafka-test</artifactId>
  <version>3.1.4</version>
</dependency>
```
## 3. A Simple Kafka Producer Application
Below is the producer application entry point:
```java
@SpringBootApplication
public class Bootstrap implements CommandLineRunner {
    private static final Logger LOGGER = LoggerFactory.getLogger(Bootstrap.class);
    private KafkaProducer kafkaProducer;

    @Inject
    public Bootstrap(KafkaProducer kafkaProducer) {
        this.kafkaProducer = kafkaProducer;
    }

    public static void main(String[] args) {
        SpringApplication.run(Bootstrap.class, args);
    }

    @Override
    public void run(String... args) {
        final ExecutorService executor = Executors.newSingleThreadExecutor();
        // Start a separate thread to read input from System.in
        executor.submit(() -> {
            try {
                while (true) {
                    Thread.sleep(1000); // Sleep for 1 second
                    System.out.println("Enter a message:");
                    Scanner scanner = new Scanner(System.in);
                    String message = scanner.nextLine();
                    kafkaProducer.sendMessage(message);
                }
            } catch (InterruptedException e) {
                LOGGER.error(e.getMessage(), e);
            }
        });
    }
}
```
The messages ending with an `Enter` key should be sent in the console.
### 3.1 Application Properties
Configure the kafka server and topic information correctly.
```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      properties:
        linger:
          ms: 0 # 0 means the producer will send messages immediately without waiting
        request:
          timeout:
            ms: 30000 # the time the producer waits for a response from the broker for each individual request.
        delivery:
          timeout:
            ms: 120000 # It’s the total time allotted for the entire send operation, including retries. default 120000
test:
  topic: test-topic
```
### 3.2 Producer Setup
The producer bean can be used to send messages to a given Kafka topic:
```java
@Component
public class KafkaProducer {
  private static final Logger LOGGER = LoggerFactory.getLogger(KafkaProducer.class);
  private final KafkaTemplate<String, String> kafkaTemplate;
  private final String topicName;

  @Inject
  public KafkaProducer(KafkaTemplate<String, String> kafkaTemplate, @Value("${test.topic}") String topicName) {
    this.kafkaTemplate = kafkaTemplate;
    this.topicName = topicName;
  }

  public void sendMessage(String message) {
    String messageId = UUID.randomUUID().toString();
    CompletableFuture<SendResult<String, String>> future = kafkaTemplate.send(topicName, messageId, message);
    future.whenComplete((result, exception) -> {
      if (Objects.isNull(exception)) {
        LOGGER.info("Message key={} value={} has been successfully sent to the topic {}, partition {}, offset {}",
          messageId, message, topicName, result.getRecordMetadata().partition(), result.getRecordMetadata().offset());
      } else {
        LOGGER.error("Failed to send message {} to the topic {}", message, topicName);
      }
    });
  }
}
```
### 3.3 Application Properties for Test
```yaml
spring:
  kafka:
    consumer:
      auto-offset-reset: earliest
      group-id: baeldung
test:
  topic: test-topic-embedded
```
The consumer property `auto-offset-reset` is set to `earliest`, which ensures that the consumer group gets the messages sent by tests because the container might start after the sends have completed.
### 3.4 Testing Using Embedded Kafka
Define the inner class `TestConsumer` for receiving message and verification.
```java
@SpringBootTest
@DirtiesContext
@EmbeddedKafka(partitions = 1, brokerProperties = {"listeners=PLAINTEXT://localhost:19092", "port=19092"})
@Import(KafkaProducerTest.TestConsumer.class)
public class KafkaProducerTest {
    private final TestConsumer consumer;
    private final KafkaProducer producer;

    @Inject
    public KafkaProducerTest(TestConsumer consumer, KafkaProducer producer) {
        this.consumer = consumer;
        this.producer = producer;
    }

    @Test
    public void sendMessage() throws Exception {
        String data = "Sending with our own simple KafkaProducer";

        producer.sendMessage(data);

        boolean messageConsumed = consumer.getLatch().await(10, TimeUnit.SECONDS);
        assertTrue(messageConsumed);
        assertThat(consumer.getPayload(), containsString(data));
    }

    public static class TestConsumer {
        private final CountDownLatch latch = new CountDownLatch(1);
        private String payload;

        @KafkaListener(topics = "${test.topic}")
        public void receive(ConsumerRecord<?, ?> consumerRecord) {
            payload = consumerRecord.toString();
            latch.countDown();
        }

        public CountDownLatch getLatch() {
            return latch;
        }

        public String getPayload() {
            return payload;
        }
    }
}
```
## 4. <span id="4">A Simple Kafka Consumer Application with BATCH AckMode and Auto Commit</span>
Below is the consumer application entry point:
```java
@SpringBootApplication
public class Bootstrap {
    public static void main(String[] args) {
        SpringApplication.run(Bootstrap.class, args);
    }
}
```
And a message handler
```java
@Service
public class MessageHandler {
    private static final Logger LOGGER = LoggerFactory.getLogger(MessageHandler.class);

    public void handle(String message) {
        LOGGER.info("MessageHandler:{}", message);
    }
}
```
### 4.1 Application Properties
Configure the kafka server and topic information correctly. `spring.kafka.listener.type` is set to `BATCH` as it has to pair with the ack mode.
```yaml
spring:
  kafka:
    consumer:
      group-id: test-topic-events
      bootstrap-servers: localhost:9092
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      properties:
        spring:
          json:
            trusted:
              packages: org.lxp
    listener:
      type: BATCH
      ack-mode: BATCH
test:
  topic: test-topic
```
### 4.2 Consumer Setup
It receives `ConsumerRecords` instead of `ConsumerRecord` as its ack mode is `BATCH`.
```java
@Component
public class KafkaConsumer {
    private static final Logger LOGGER = LoggerFactory.getLogger(KafkaConsumer.class);
    private final MessageHandler messageHandler;

    @Inject
    public KafkaConsumer(MessageHandler messageHandler) {
        this.messageHandler = messageHandler;
    }

    @KafkaListener(topics = "${test.topic}")
    public void handle(ConsumerRecords<String, String> records) {
        records.forEach(record -> {
            LOGGER.info("receive message key={} topic={} partition={}, offset={}",
                    record.key(), record.topic(), record.partition(), record.offset());
            messageHandler.handle(record.value());
        });
    }
}
```
### 4.3 Application Properties for Test
```yaml
spring:
  kafka:
    consumer:
      auto-offset-reset: earliest
      group-id: baeldung
    listener:
      type: BATCH
      ack-mode: BATCH
test:
  topic: test-topic-embedded
```
The consumer property `auto-offset-reset` is set to `earliest`, which ensures that the consumer group gets the messages sent by tests because the container might start after the sends have completed.
### 4.4 <span id="4-4">Testing Using Embedded Kafka</span>
Define the inner class `TestProducer` to send message and `MyMessageHandler` for verification.
```java
@SpringBootTest
@DirtiesContext
@EmbeddedKafka(partitions = 1, brokerProperties = {"listeners=PLAINTEXT://localhost:19092", "port=19092"})
@Import({KafkaConsumerTest.Config.class, KafkaConsumerTest.TestProducer.class})
public class KafkaConsumerTest {
  private static final Logger LOGGER = LoggerFactory.getLogger(KafkaConsumerTest.class);
  private static final CountDownLatch MESSAGE_HANDLER_LATCH = new CountDownLatch(1);
  @Inject
  private MessageHandler messageHandler;
  @Inject
  private TestProducer producer;

  @Test
  public void handle() throws InterruptedException {
    final String messageBody = "test-message";
    producer.sendMessage(messageBody);
    boolean messageSent = producer.getLatch().await(10, TimeUnit.SECONDS);
    boolean messageConsumed = MESSAGE_HANDLER_LATCH.await(10, TimeUnit.SECONDS);
    assertTrue(messageSent);
    assertTrue(messageConsumed);
    verify(messageHandler, times(1)).handle(messageBody);
  }

  public static class Config {
    @Primary
    @Bean
    public MessageHandler messageHandler() {
      return Mockito.spy(new MyMessageHandler());
    }
  }

  public static class MyMessageHandler extends MessageHandler {
    @Override
    public void handle(String message) {
      super.handle(message);
      MESSAGE_HANDLER_LATCH.countDown();
      LOGGER.info("MyMessageHandler count down");
    }
  }

  public static class TestProducer {
    private CountDownLatch latch = new CountDownLatch(1);
    private final KafkaTemplate<String, String> kafkaTemplate;
    private final String topicName;


    @Inject
    public TestProducer(KafkaTemplate<String, String> kafkaTemplate, @Value("${test.topic}") String topicName) {
      this.kafkaTemplate = kafkaTemplate;
      this.topicName = topicName;
    }

    public void sendMessage(String message) {
      try {
        kafkaTemplate.send(topicName, UUID.randomUUID().toString(), message).get();
        latch.countDown();
      } catch (InterruptedException | ExecutionException e) {
        throw new RuntimeException(e);
      }
    }

    public CountDownLatch getLatch() {
      return latch;
    }
  }
}
```
## 5. A Simple Kafka Consumer Application with Manual Commit
Refer to [4. A Simple Kafka Consumer Application with BATCH AckMode and Auto Commit](#4) for its entry point and message handler.
### 5.1 Application Properties
Configure the kafka server and topic information correctly. `spring.kafka.consumer.enable-auto-commit` has to be `false` when the ack mode is `MANUAL` or `MANUAL_IMMEDIATE`.
```yaml
spring:
  application:
    name: kafka-manual-commit-consumer
  kafka:
    consumer:
      group-id: test-topic-manual-commit-events
      bootstrap-servers: localhost:9092
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      properties:
        spring:
          json:
            trusted:
              packages: org.lxp
      enable-auto-commit: false
    listener:
      ack-mode: MANUAL_IMMEDIATE
test:
  topic: test-topic
```
### 5.2 Consumer Setup
It receives `Acknowledgment` as the ack mode is `spring.kafka.consumer.enable-auto-commit` is `false`.
```java
@Component
public class KafkaConsumer {
  private static final Logger LOGGER = LoggerFactory.getLogger(KafkaConsumer.class);
  private final MessageHandler messageHandler;

  @Inject
  public KafkaConsumer(MessageHandler messageHandler) {
    this.messageHandler = messageHandler;
  }

  @KafkaListener(topics = "${test.topic}")
  public void handle(ConsumerRecord<String, String> record, Acknowledgment acknowledgment) {
    LOGGER.info("receive message key={} topic={} partition={}, offset={}",
      record.key(), record.topic(), record.partition(), record.offset());
    messageHandler.handle(record.value());
    acknowledgment.acknowledge();
  }
}
```
### 5.3 Application Properties for Test
```yaml
spring:
  kafka:
    consumer:
      auto-offset-reset: earliest
      group-id: baeldung
      enable-auto-commit: false
    listener:
      ack-mode: MANUAL_IMMEDIATE
test:
  topic: test-topic-embedded
```
The consumer property `auto-offset-reset` is set to `earliest`, which ensures that the consumer group gets the messages sent by tests because the container might start after the sends have completed.
### 5.4 Testing Using Embedded Kafka
The test class is same as [4.4 Testing Using Embedded Kafka](#4-4)
## 6. Test the Producer and Consumers
Before starting the producer and consumers, ensure the kafka-server is running locally and the topic `test-topic` has been created.
### 6.1 Start the Producer and Consumers
Start them by running their `Bootstarp` respectively.
### 6.2 Produce and Consume Messages
- Enter any message in the producer's console. Producer log sample:
```
2024-04-22T17:30:04.271+08:00  INFO 29304 --- [kafka-producer] [           main] org.lxp.producer.Bootstrap               : Started Bootstrap in 0.862 seconds (process running for 1.154)
Enter a message:
test 1234 abc 8&&^%%$#
...
2024-04-22T17:30:53.725+08:00  INFO 29304 --- [kafka-producer] [ad | producer-1] org.lxp.producer.KafkaProducer           : Message key=05a849ed-fe70-4545-8a46-56a7150596a9 value=test 1234 abc 8&&^%%$# has been successfully sent to the topic test-topic, partition 1, offset 12
```
- Consumer with auto commit log sample:
```
2024-04-22T17:30:53.722+08:00  INFO 35268 --- [kafka-consumer] [ntainer#0-0-C-1] org.lxp.auto.consumer.KafkaConsumer      : receive message key=05a849ed-fe70-4545-8a46-56a7150596a9 topic=test-topic partition=1, offset=12
2024-04-22T17:30:53.722+08:00  INFO 35268 --- [kafka-consumer] [ntainer#0-0-C-1] org.lxp.auto.consumer.MessageHandler     : MessageHandler:test 1234 abc 8&&^%%$#
```
- Consumer with manual commit log sample:
```
2024-04-22T17:30:53.722+08:00  INFO 35236 --- [kafka-manual-commit-consumer] [ntainer#0-0-C-1] org.lxp.manual.consumer.KafkaConsumer    : receive message key=05a849ed-fe70-4545-8a46-56a7150596a9 topic=test-topic partition=1, offset=12
2024-04-22T17:30:53.722+08:00  INFO 35236 --- [kafka-manual-commit-consumer] [ntainer#0-0-C-1] org.lxp.manual.consumer.MessageHandler   : MessageHandler:test 1234 abc 8&&^%%$#
```

## 7. References
- [Kafka Producer in Spring Boot Microservice](https://www.appsdeveloperblog.com/kafka-producer-in-spring-boot-microservice/)
- [Creating Kafka Consumer in Spring Boot Microservice](https://www.appsdeveloperblog.com/kafka-consumer-in-spring-boot/)
- [线上问题:SpringBoot中kafka消费者自动提交关闭无效](https://blog.csdn.net/Xin_101/article/details/123138756)
- [Spring Kafka消费模式（single, batch）及确认模式（自动、手动）示例](https://blog.csdn.net/luo15242208310/article/details/122285689)
- [Testing Kafka and Spring Boot](https://www.baeldung.com/spring-boot-kafka-testing)
