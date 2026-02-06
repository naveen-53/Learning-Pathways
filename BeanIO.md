# BeanIO in Java – Deep Dive Guide 🚀🫛💻⚙️

## 📃Table of Contents

1. Introduction
2. Core Concepts
3. Architecture Overview
4. Installation & Setup
5. Mapping Files Explained
6. Working with CSV (Delimited) Files
7. Working with Fixed-Length Files
8. Using Annotations Instead of XML
9. Error Handling & Validation
10. Advanced Features
11. Performance Tips
12. Testing Strategies
13. Common Pitfalls
14. Complete End-to-End Example

---

## 1. ℹ️Introduction

BeanIO is a Java framework designed to map flat files (CSV, fixed-length, and stream formats) directly to Java objects (beans). It simplifies parsing, validation, marshalling, and unmarshalling without heavy manual parsing logic.

### Why Use BeanIO?

* Strong validation support
* XML or annotation-based configuration
* Supports huge files (streaming)
* Works with legacy fixed-length systems

---

## 2. 🖥️Core Concepts

| Concept | Description              |
| ------- | ------------------------ |
| Stream  | A file format definition |
| Record  | A single row or line     |
| Field   | Column or segment        |
| Bean    | Java object mapped       |
| Layout  | Structure of the file    |

---

## 3. 🗺️Architecture Overview

File → Stream → Record → Field → Java Bean

BeanIO handles:

* Tokenization
* Conversion
* Validation
* Error recovery

---

## 4. 📲Installation & Setup (Maven)

```xml
<dependency>
  <groupId>org.beanio</groupId>
  <artifactId>beanio</artifactId>
  <version>2.1.0</version>
</dependency>
```

---

## 5. 🗺️Mapping Files Explained

Mapping file defines how file data maps to Java classes.

### Example: CSV Mapping

```xml
<beanio xmlns="http://www.beanio.org/2012/03">
  <stream name="userStream" format="csv">
    <record name="user" class="com.app.User">
      <field name="id" />
      <field name="name" />
      <field name="email" />
    </record>
  </stream>
</beanio>
```

---

## 6. 📂Working with CSV Files

### Java Bean

```java
public class User {
  private int id;
  private String name;
  private String email;
}
```

### Reading File

```java
StreamFactory factory = StreamFactory.newInstance();
factory.load("mapping.xml");

BeanReader reader = factory.createReader("userStream", new File("users.csv"));

Object record;
while ((record = reader.read()) != null) {
   User user = (User) record;
}
reader.close();
```

---

## 7. 📁Fixed-Length Files

### Mapping Example

```xml
<stream name="employeeStream" format="fixedlength">
  <record name="emp" class="Employee">
    <field name="id" length="5" />
    <field name="name" length="20" />
    <field name="salary" length="10" />
  </record>
</stream>
```

Each field consumes exact characters.

---

## 8. 🅰️Annotation-Based Mapping

```java
@Record
public class User {

 @Field(at=0)
 private int id;

 @Field(at=1)
 private String name;

 @Field(at=2)
 private String email;
}
```

Enable annotation mode in StreamFactory.

---

## 9. 🚫Error Handling & Validation

### Required Fields

```xml
<field name="email" required="true" />
```

### Handling Errors

```java
if (reader.hasRecordErrors()) {
   for (String error : reader.getRecordErrors())
       System.out.println(error);
}
```

Supports:

* Regex validation
* Length checks
* Numeric ranges

---

## 10. 🪶Advanced Features

### Record Groups

Useful when file has headers + body + footer

```xml
<group name="file">
  <record name="header" />
  <record name="body" occurs="*" />
  <record name="footer" />
</group>
```

### Type Handlers

Custom converters:

```java
public class DateHandler implements TypeHandler {
 public Object parse(String text) { }
}
```

---

## 11. 💯Performance Tips

* Use streaming API
* Avoid loading whole file
* Disable unnecessary validation
* Reuse StreamFactory

---

## 12. 🧪Testing Strategies

* Test mapping files separately
* Validate against malformed files
* Use sample datasets

---

## 13. Common Pitfalls

- ❌ Incorrect field length
- ❌ Misaligned CSV columns
- ❌ Not closing reader
- ❌ Wrong data types

---

## 14. Complete End-to-End Example

### users.csv

```
1,John,john@email.com
2,Alice,alice@email.com
```

### Mapping + Bean + Reader → Object list

BeanIO converts automatically.

---

## 15. ♨️Spring Boot + Spring Batch Integration

BeanIO fits naturally into Spring Batch for large-scale ETL jobs.

### Maven Dependencies

```xml
<dependency>
  <groupId>org.beanio</groupId>
  <artifactId>beanio</artifactId>
  <version>2.1.0</version>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-batch</artifactId>
</dependency>
```

### ItemReader using BeanIO

```java
@Bean
public BeanIOFlatFileItemReader<User> reader() {
   BeanIOFlatFileItemReader<User> reader = new BeanIOFlatFileItemReader<>();
   reader.setStreamName("userStream");
   reader.setResource(new FileSystemResource("users.csv"));
   reader.setMapping(new ClassPathResource("mapping.xml"));
   return reader;
}
```

Spring Batch manages chunk processing, retries, transactions, and scalability.

---

## 16. ✨BeanIO vs OpenCSV vs Jackson CSV

| Feature          | BeanIO   | OpenCSV | Jackson CSV |
| ---------------- | -------- | ------- | ----------- |
| Fixed length     | ✅        | ❌       | ❌           |
| Validation       | ✅ strong | ⚠ basic | ⚠ basic     |
| XML mapping      | ✅        | ❌       | ❌           |
| Streaming        | ✅        | ⚠       | ⚠           |
| Enterprise batch | ✅        | ❌       | ⚠           |

### When to use BeanIO

- ✔ Legacy systems
- ✔ Large files
- ✔ Strict format rules

### When to use OpenCSV/Jackson

- ✔ Simple CSV imports
- ✔ REST-based systems

---

## 17. 🎯Real-World Project Structure

```
beanio-sample-project
│
├── src/main/java
│   └── com/example/beanio
│       ├── Application.java
│       ├── model/
│       │   └── User.java
│       ├── batch/
│       │   └── BatchConfig.java
│       └── service/
│           └── FileProcessor.java
│
├── src/main/resources
│   ├── mapping.xml
│   └── users.csv
│
└── pom.xml
```

Layered design keeps mapping, processing, and business logic clean.

---

## Final Thoughts

BeanIO shines in enterprise batch processing where strict file formats, performance, and validation matter.

Use it when:

- ✔ File layouts are complex
- ✔ Legacy systems are involved
- ✔ Data integrity is critical

---

Next upgrades you could explore:

• Multi-record layouts
• Header/footer validation
• Parallel batch jobs
• Custom type handlers

---

