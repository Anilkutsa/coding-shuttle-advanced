# Configuring Kafka with Spring Boot

This guide demonstrates how to set up Apache Kafka with Spring Boot to establish a simple producer-consumer model between two services: `user-service` and `notification-service`.

## Prerequisites

- **Java Development Kit (JDK) 8 or higher**
- **Apache Kafka**: Installed and running on your local machine. For installation instructions, refer to [Installing Kafka and Kafka Visualization Tool](https://www.codingshuttle.com/spring-boot-hand-book/installing-kafka-and-kafka-visiualisation-tool).
- **Spring Boot**: Basic understanding of Spring Boot applications.

## Project Structure

We'll create two Spring Boot applications:

1. **user-service**: Acts as the Kafka producer.
2. **notification-service**: Acts as the Kafka consumer.

## Step 1: Initialize Spring Boot Projects

Use [Spring Initializr](https://start.spring.io/) to generate two separate projects with the following dependencies:

- **Spring for Apache Kafka**
- **Spring Web**

Download and extract the generated projects.

## Step 2: Configure Kafka Properties

### user-service (Producer)

In `src/main/resources/application.yml`, configure the Kafka producer properties:

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
```

### notification-service (Consumer)

In `src/main/resources/application.yml`, configure the Kafka consumer properties:

```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    consumer:
      group-id: notification-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

## Step 3: Implement Kafka Producer

In `user-service`, create a service to send messages to a Kafka topic.

```java
package com.example.userservice.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class UserService {

    private final KafkaTemplate<String, String> kafkaTemplate;
    private static final String TOPIC = "user-topic";

    @Autowired
    public UserService(KafkaTemplate<String, String> kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    public void sendMessage(String message) {
        kafkaTemplate.send(TOPIC, message);
    }
}
```

Create a REST controller to expose an endpoint for sending messages.

```java
package com.example.userservice.controller;

import com.example.userservice.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserService userService;

    @Autowired
    public UserController(UserService userService) {
        this.userService = userService;
    }

    @PostMapping("/publish")
    public String publishMessage(@RequestParam("message") String message) {
        userService.sendMessage(message);
        return "Message published: " + message;
    }
}
```

## Step 4: Implement Kafka Consumer

In `notification-service`, create a listener to consume messages from the Kafka topic.

```java
package com.example.notificationservice.listener;

import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

@Service
public class NotificationListener {

    @KafkaListener(topics = "user-topic", groupId = "notification-group")
    public void listen(String message) {
        System.out.println("Received message: " + message);
        // Process the message
    }
}
```

## Step 5: Run the Applications

1. Ensure that the Kafka server is running on `localhost:9092`.
2. Start the `notification-service` application.
3. Start the `user-service` application.
4. Use a tool like `curl` or Postman to send a POST request to the `user-service` endpoint:

   ```bash
   curl -X POST "http://localhost:8080/api/users/publish?message=HelloKafka"
   ```

5. Check the console output of `notification-service` to verify that the message has been received.

## Conclusion

By following these steps, you've set up a basic producer-consumer model using Kafka and Spring Boot. For more advanced configurations, such as sending complex messages like JSON objects, refer to [Advance Kafka Configuration with Spring Boot](https://www.codingshuttle.com/spring-boot-hand-book/advance-kafka-configuration-with-spring-boot).
