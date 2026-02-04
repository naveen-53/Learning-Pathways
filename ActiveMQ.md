# Apache ActiveMQ – Complete Guide
## 🚀 What is ActiveMQ?

- Apache ActiveMQ is an **open-source message broker** that enables applications to communicate with each other asynchronously using queues and topics.
- It is maintained by the Apache Software Foundation.

In simple terms:

" ActiveMQ helps different services send messages to each other reliably without being directly connected. "

---

## Why Do We Use ActiveMQ?

Without messaging systems:

- Apps must call each other directly
- Failures break workflows
- Systems don’t scale well

With ActiveMQ:

- ✅ Loose coupling
- ✅ High reliability
- ✅ Async processing
- ✅ Scalability

---

## ⚙️ What Should ActiveMQ Do?

ActiveMQ typically handles:

- Function	Purpose
- Message Queuing	Store & forward messages
- Pub/Sub	Broadcast messages
- Load balancing	Distribute work
- Fault tolerance	Prevent data loss
- Async communication	Improve performance

---

## 🌍 Real World Use Cases

- 🛒 E-commerce
  - Order service sends order
  - Payment service processes
  - Shipping service receives update

- 🏦 Banking

  - Transactions queued
  - Fraud detection consumes events

- 📦 Logistics
  - Shipment updates
  - Inventory notifications

- 📡 Microservices
  - Services communicate without direct APIs
 
---
 
## 📌 Maven Dependency
```xml
<dependency>
  <groupId>org.apache.activemq</groupId>
  <artifactId>activemq-client</artifactId>
  <version>5.18.2</version>
</dependency>
```

---

## ☕ ActiveMQ Java Example (Step by Step)


### 📤 Message Producer (Send Message)
```java
import jakarta.jms.*;
import org.apache.activemq.ActiveMQConnectionFactory;

public class Producer {

    public static void main(String[] args) throws Exception {

        // 1. Connect to ActiveMQ broker
        ActiveMQConnectionFactory factory =
            new ActiveMQConnectionFactory("tcp://localhost:61616");

        Connection connection = factory.createConnection();
        connection.start();

        // 2. Create session
        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

        // 3. Create queue
        Destination queue = session.createQueue("order.queue");

        // 4. Create producer
        MessageProducer producer = session.createProducer(queue);

        // 5. Create message
        TextMessage message = session.createTextMessage("Order Created!");

        // 6. Send message
        producer.send(message);

        System.out.println("Message sent!");

        // 7. Close resources
        session.close();
        connection.close();
    }
}
```

### 🔍 Line-by-Line Explanation
| Line	| What it does |
|-----|------|
| ActiveMQConnectionFactory |	Creates broker connection |
| createConnection() | Opens connection | 
| start()	| Starts message flow |
| createSession()	| Creates context |
| createQueue()	| Defines message channel |
| createProducer()	| Sends messages |
| createTextMessage()	| Builds message |
| send()	| Delivers message |
| close()	| Releases resources |
---

### 📥 Message Consumer (Receive Message)
```java
import jakarta.jms.*;
import org.apache.activemq.ActiveMQConnectionFactory;

public class Consumer {

    public static void main(String[] args) throws Exception {

        ActiveMQConnectionFactory factory =
            new ActiveMQConnectionFactory("tcp://localhost:61616");

        Connection connection = factory.createConnection();
        connection.start();

        Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);

        Destination queue = session.createQueue("order.queue");

        MessageConsumer consumer = session.createConsumer(queue);

        Message message = consumer.receive();

        TextMessage textMessage = (TextMessage) message;

        System.out.println("Received: " + textMessage.getText());

        session.close();
        connection.close();
    }
}
```

### 🔍 Consumer Explanation
| Step	| Purpose |
|-----|------|
| createConsumer()	| Listens to queue |
| receive()	| Waits for message |
| cast to TextMessage	| Reads message |
| getText()	| Retrieves content |

---

## ✅ Advantages

- ✔ Reliable messaging
- ✔ Decouples systems
- ✔ High performance
- ✔ Supports clustering
- ✔ Works with many languages
- ✔ Handles spikes well

---

## ❌ Disadvantages

- ❗ Adds system complexity
- ❗ Requires broker management
- ❗ Message duplication possible (if misconfigured)
- ❗ Learning curve
- ❗ Broker downtime affects flow

---

## 📊 When Should You Use ActiveMQ?

Use it when:

- ✅ Microservices architecture
- ✅ High traffic systems
- ✅ Asynchronous processing
- ✅ Event-driven apps

Avoid when:

- ❌ Simple monolithic apps
- ❌ Low message volume

## 🧠 Summary

ActiveMQ is a powerful messaging system that:

- ✔ Improves scalability
- ✔ Prevents tight coupling
- ✔ Handles async workflows

It’s a backbone for modern distributed systems.

---
