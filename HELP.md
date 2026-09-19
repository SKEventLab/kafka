# Spring Boot Apache Kafka Event Processing with Aiven

This repository contains a **Spring Boot** application integrated with **Apache Kafka** hosted on **Aiven**. The system processes incoming event messages through an API controller, publishes them via a Kafka producer, routes them through a data processor, and safely reads them via an active consumer listener.

## 🚀 Features
* **REST API Endpoint:** Send message payloads dynamically.
* **Aiven Kafka Integration:** Secured cloud-hosted event streaming with SSL certificates.
* **Producer-Consumer Architecture:** Decoupled real-time data flow inside Spring Boot.

---

## 🛠️ Step 1: Creating Your Kafka Topic on Aiven

Before running the application, you must provision your cluster and topics on the Aiven console.

1. **Log in to Aiven:** Go to the [Aiven Console](https://aiven.io) and select or create your **Apache Kafka** service.
2. **Download SSL Credentials:** 
   * Navigate to the **Overview** tab of your Kafka service.
   * Scroll down to the **Connection information** -> **Certificates** section.
   * Download the Access Certificate (`service.cert`), Access Key (`service.key`), and CA Certificate (`ca.pem`).
3. **Navigate to Topics:** Click on the **Topics** tab in the left-hand sidebar menu.
4. **Create a Topic:**
   * Click the **Create topic** button.
   * Enter your desired topic name (e.g., `sps-kafka-topic`).
   * Define your **Partitions** (Default: `3`) and **Replication Factor** (Default: `3`).
   * Click **Create topic**.

---

## ⚙️ Step 2: Local Application Setup

### 1. Security Certificates Placement
Place your downloaded Aiven connection certificates inside your project resources directory so your Spring Boot application can authenticate via SSL:
* Place `ca.pem` directly into `src/main/resources/ca.pem`.

If your local setup requires a standard Java Keystore/Truststore generated from the keys, run these commands in your resource folder:
```bash
openssl pkcs12 -export -in service.cert -inkey service.key -out client.p12 -name client -CAfile ca.pem -caname root
keytool -importkeystore -deststorepass mypassword -destkeystore client.truststore.jks -srckeystore client.p12 -srcstoretype PKCS12 -srcstorepass mypassword
keytool -import -file ca.pem -alias AivenCA -keystore client.truststore.jks -storepass mypassword
```

### 2. Environment Configurations
Configure your Aiven connection variables inside `src/main/resources/application.yml` or `application.properties`:

```yaml
spring:
  application:
    name: prj
  kafka:
    bootstrap-servers: YOUR_AIVEN_KAFKA_BOOTSTRAP_URL:YOUR_PORT
    properties:
      security.protocol: SSL
      ssl.truststore.location: src/main/resources/client.truststore.jks
      ssl.truststore.password: mypassword
      ssl.keystore.location: src/main/resources/client.truststore.jks
      ssl.keystore.password: mypassword
      ssl.key.password: mypassword
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
    consumer:
      group-id: sps-consumer-group
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
```

---

## 🏃 Compilation & Execution

Build and run your application using the included Maven wrapper:

```bash
# For Linux/macOS
./mvnw clean spring-boot:run

# For Windows
mvnw.cmd clean spring-boot:run
```

---

## 🧪 Testing the Pipeline

Once the application logs show a successful connection to the Aiven broker, send a sample POST payload using cURL or Postman to your messaging endpoint:

```bash
curl -X POST http://localhost:8080/api/messages \
     -H "Content-Type: application/json" \
     -d '{"message": "Hello Aiven Kafka Streaming!"}'
```

Check your application's terminal console logs to watch the event flow instantly transition from the **Producer** through the **Processor** layout to the **Consumer** listener!
