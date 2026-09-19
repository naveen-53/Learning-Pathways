# Java/Spring Boot OutOfMemoryError — Heap Dump Troubleshooting Guide

## 1. Purpose

This guide covers a practical production-style workflow for investigating and resolving:

```text
java.lang.OutOfMemoryError: Java heap space
```

It focuses on:

- JVM heap basics
- Garbage Collection (GC)
- Memory leaks
- `jcmd` commands
- Heap dumps
- Eclipse Memory Analyzer (MAT)
- Finding GC roots
- Finding the source-code problem
- Fixing and verifying the issue

---

# 2. Understand the Problem

A simplified JVM memory model:

```text
                    JVM
                     |
          +----------+----------+
          |                     |
        Heap                 Metaspace
          |
       Objects
```

The **heap** stores Java objects such as:

```java
Loan loan = new Loan();
Customer customer = new Customer();
byte[] data = new byte[1024];
```

If the heap is configured as:

```text
-Xmx128m
```

the JVM can use approximately 128 MB of Java heap.

When the application needs more memory and GC cannot reclaim enough space, the JVM can throw:

```text
java.lang.OutOfMemoryError: Java heap space
```

---

# 3. What Is a Memory Leak?

A Java memory leak usually occurs when objects are no longer logically needed but are still reachable through references.

Example:

```java
private static final List<byte[]> leakedData = new ArrayList<>();

public void process() {
    byte[] data = new byte[1024 * 1024];
    leakedData.add(data);
}
```

Every request adds another 1 MB:

```text
Request 1 -> 1 MB
Request 2 -> 1 MB
Request 3 -> 1 MB
...
Request 100 -> 100 MB
```

The list still references all the objects:

```text
GC Root
   |
   v
static leakedData
   |
   v
ArrayList
   |
   +-- byte[]
   +-- byte[]
   +-- byte[]
   +-- ...
```

Therefore GC cannot remove them.

---

# 4. Memory Leak vs OutOfMemoryError

These are not exactly the same thing.

### Memory leak

Unnecessary objects remain reachable.

```text
Objects retained
      |
      v
Heap usage increases
```

### OutOfMemoryError

The JVM cannot allocate the memory required for an operation.

```text
Heap almost full
      |
      v
GC cannot free enough
      |
      v
Allocation fails
      |
      v
OutOfMemoryError
```

A memory leak can eventually cause an OOM, but OOM can also occur because the application legitimately needs more memory than the configured heap.

---

# 5. Reproduce the Issue

For a practice application, configure a small heap:

```text
-Xms64m
-Xmx128m
```

If running a JAR:

```powershell
java -Xms64m -Xmx128m -jar target\your-app.jar
```

If using IntelliJ/STS, add the same values as JVM/VM options:

```text
-Xms64m -Xmx128m
```

---

# 6. Example Spring Boot Memory Leak

Create an endpoint that intentionally retains memory:

```java
@RestController
public class LoanLeakController {

    private static final List<byte[]> leakedLoans = new ArrayList<>();

    @GetMapping("/loan")
    public String createLoan() {

        byte[] loanData = new byte[1024 * 1024]; // 1 MB

        leakedLoans.add(loanData);

        return "Loan created";
    }
}
```

Each request retains approximately 1 MB.

---

# 7. Find the Java Process

Open PowerShell:

```powershell
jps -l
```

Example:

```text
25536 com.example.loan.LoanApplication
```

Here:

```text
PID = 25536
```

The PID can change after restarting the application, so always check it again.

---

# 8. Check Heap Information

Run:

```powershell
jcmd 25536 GC.heap_info
```

Example:

```text
garbage-first heap
total 131072K, used 129931K
```

This means approximately:

```text
Heap capacity = 128 MB
Used          = 127 MB
```

If you see:

```text
total 131072K
used 129931K
```

the heap is almost full.

---

# 9. Run Garbage Collection for Diagnosis

Run:

```powershell
jcmd 25536 GC.run
```

Then check again:

```powershell
jcmd 25536 GC.heap_info
```

Example:

```text
Before GC:
used 129931K

After GC:
used 129736K
```

Only approximately 195 KB was reclaimed.

This is suspicious because most objects remained reachable.

Important:

> `GC.run` is useful for diagnosis. It is not a permanent production fix.

---

# 10. Check the Class Histogram

Run:

```powershell
jcmd 25536 GC.class_histogram
```

This shows which classes consume the most heap.

Example:

```text
num     #instances       #bytes     Class description
------------------------------------------------------
1       100              104857600  [B
2       5000             ...        java.lang.String
3       1                ...        java.util.ArrayList
```

`[B` means:

```text
byte[]
```

This can quickly tell you what type of object is consuming memory.

---

# 11. Create a Heap Dump

Create a directory:

```powershell
mkdir C:\temp
```

Create the heap dump:

```powershell
jcmd 25536 GC.heap_dump C:\temp\loan-heap.hprof
```

This produces:

```text
C:\temp\loan-heap.hprof
```

A heap dump is a snapshot of objects and references in the JVM heap at that moment.

---

# 12. What a Heap Dump Contains

A heap dump can contain information about:

- Java objects
- object counts
- object sizes
- references between objects
- retained memory
- class information
- GC root relationships

It helps answer:

> What is consuming memory?

and:

> Why are those objects still reachable?

---

# 13. Open the Heap Dump in Eclipse MAT

Use **Eclipse Memory Analyzer (MAT)**.

Open:

```text
loan-heap.hprof
```

MAT analyzes the heap dump.

![Alt Text](https://raw.githubusercontent.com/naveen-53/Learning-Pathways/refs/heads/main/Images/Screenshot%202026-09-19%20130233.png)

The most important features to learn are:

```text
1. Histogram
2. Dominator Tree
3. Shallow Heap
4. Retained Heap
5. Path to GC Roots
6. Leak Suspects Report
```

---

# 14. Histogram in MAT

The Histogram shows object types and their memory usage.

Example:

```text
Class                  Objects       Shallow Heap
--------------------------------------------------
byte[]                 100           ~100 MB
java.lang.String       5000          ...
Loan                   1000          ...
ArrayList              1             ...
```

If `byte[]` is consuming most of the heap, investigate why so many byte arrays are retained.

![Histogram](https://raw.githubusercontent.com/naveen-53/Learning-Pathways/refs/heads/main/Images/Screenshot%202026-09-19%20130559.png)

---

# 15. Shallow Heap

**Shallow heap** is approximately the memory directly occupied by an object itself.

For example:

```text
byte[1 MB]
```

has roughly 1 MB of direct memory usage.

But a collection may have a small shallow size while referencing many large objects.

Therefore shallow heap alone isn't enough to identify a memory leak.

---

# 16. Retained Heap

**Retained heap** answers:

> How much memory could become collectible if this object and its retained objects were no longer reachable?

Example:

```text
Cache
 |
 v
HashMap
 |
 +-- Loan
 +-- Loan
 +-- Loan
 +-- ...
```

The HashMap may have:

```text
Small shallow heap
Large retained heap
```

This makes retained heap very useful for finding objects that are keeping large portions of the heap alive.

---

# 17. Dominator Tree

Open the **Dominator Tree** in MAT.

It helps identify objects that retain large amounts of memory.

Example:

```text
LoanCache
    |
    v
HashMap
    |
    +-- Loan
    +-- Loan
    +-- Loan
    +-- ...
```

You may find:

```text
HashMap
Retained Heap = 90 MB
```

This tells you to investigate that HashMap.

---

# 18. Path to GC Roots

This is one of the most important steps.

Suppose MAT finds:

```text
byte[]
Retained Heap = 100 MB
```

Right-click the object and use:

```text
Path to GC Roots
```

You may discover:

```text
GC Root
   |
   v
LoanLeakController.leakedLoans
   |
   v
ArrayList
   |
   +-- byte[]
   +-- byte[]
   +-- byte[]
   +-- ...
```

Now the cause becomes clear:

```text
static leakedLoans
        |
        v
keeps byte[] objects reachable
        |
        v
GC cannot remove them
```

---

# 19. What Is a GC Root?

A GC root is a starting reference used by the JVM's garbage collector when determining whether an object is reachable.

Examples can include:

- active thread references
- static fields
- JNI references
- other JVM roots

If an object is reachable from a GC root, it may remain alive.

Example:

```text
GC Root
   |
   v
static cache
   |
   v
Loan object
```

The Loan object is reachable, so GC cannot simply delete it.

---

# 20. Leak Suspects Report

MAT can generate a:

```text
Leak Suspects Report
```

It can identify suspicious memory-retention patterns.

Example:

```text
A large portion of the heap is retained by:
java.util.HashMap
```

Use this as an investigation starting point.

Do not blindly assume the report is the final root cause.

Verify the references and understand the application code.

---

# 21. Find the Source Code Problem

Suppose MAT shows:

```text
GC Root
   |
   v
LoanLeakController.leakedLoans
   |
   v
ArrayList
   |
   v
byte[]
```

Go back to the code:

```java
private static final List<byte[]> leakedLoans = new ArrayList<>();
```

and:

```java
leakedLoans.add(loanData);
```

Now the root cause is clear.

---

# 22. Fix the Example Leak

Remove the unnecessary long-lived collection.

Instead of:

```java
private static final List<byte[]> leakedLoans = new ArrayList<>();

@GetMapping("/loan")
public String createLoan() {

    byte[] loanData = new byte[1024 * 1024];

    leakedLoans.add(loanData);

    return "Loan created";
}
```

use:

```java
@GetMapping("/loan")
public String createLoan() {

    byte[] loanData = new byte[1024 * 1024];

    // Process the data here.

    return "Loan created";
}
```

After the request finishes, assuming nothing else references `loanData`:

```text
loanData
   |
   v
No remaining reference
   |
   v
Eligible for GC
```

---

# 23. Test the Fix

Restart the application with:

```powershell
java -Xms64m -Xmx128m -jar target\your-app.jar
```

Find the new PID:

```powershell
jps -l
```

Generate many requests:

```powershell
1..1000 | ForEach-Object {
    Invoke-WebRequest http://localhost:8080/loan
}
```

Check:

```powershell
jcmd <PID> GC.heap_info
```

You want memory to behave approximately like:

```text
Request
   |
   v
Memory increases
   |
   v
GC
   |
   v
Memory decreases
   |
   v
More requests
```

rather than:

```text
20%
 |
40%
 |
60%
 |
80%
 |
95%
 |
OOM
```

---

# 24. Production Investigation Workflow

When you receive:

```text
java.lang.OutOfMemoryError: Java heap space
```

use this process:

```text
                  OOM ALERT
                     |
                     v
             Check monitoring
                     |
                     v
             Find JVM process
                     |
                     v
             jcmd GC.heap_info
                     |
                     v
               Check GC behavior
                     |
                     v
             Run GC for diagnosis
                     |
                     v
       Did memory significantly decrease?
             /                  \
           YES                   NO
            |                     |
            v                     v
    High legitimate         Suspect retained
    memory usage            objects / leak
                                  |
                                  v
                         GC.class_histogram
                                  |
                                  v
                            Heap dump
                                  |
                                  v
                             Eclipse MAT
                                  |
                                  v
                          Dominator Tree
                                  |
                                  v
                           Retained Heap
                                  |
                                  v
                         Path to GC Roots
                                  |
                                  v
                            Source code
                                  |
                                  v
                              Fix
                                  |
                                  v
                          Load testing
                                  |
                                  v
                            Monitoring
```

---

# 25. Emergency Recovery vs Permanent Fix

If production is already failing:

```text
Production OOM
      |
      +----------------------+
      |                      |
      v                      v
Immediate recovery       Root-cause analysis
      |                      |
      v                      v
Restart/replace          Heap dump
unhealthy instance      MAT analysis
      |                      |
      v                      v
Restore capacity        Fix code
                             |
                             v
                         Deploy fix
```

A restart is **recovery**, not the root-cause fix.

If the leak remains:

```text
Restart
  |
  v
Memory normal
  |
  v
Traffic continues
  |
  v
Memory grows again
  |
  v
OOM again
```

---

# 26. Common Real-World Causes

## Unbounded collection

```java
static List<Loan> loans = new ArrayList<>();
```

Fix:

- remove unnecessary retention
- limit collection size
- use appropriate lifecycle
- use bounded cache if caching is required

## Unbounded cache

```text
Map
 |
 +-- entry
 +-- entry
 +-- entry
 +-- ...
```

Fix:

- maximum size
- TTL
- eviction policy
- appropriate cache implementation

## ThreadLocal retention

Use:

```java
try {
    context.set(data);

    // processing

} finally {
    context.remove();
}
```

## Large database results

Avoid:

```java
List<Loan> loans = repository.findAll();
```

for very large tables.

Prefer:

```text
Page 1 -> process
Page 2 -> process
Page 3 -> process
```

## Large file/payload handling

Avoid loading huge files completely into memory:

```java
byte[] data = inputStream.readAllBytes();
```

Prefer streaming/chunked processing where appropriate.

## Event/listener retention

Repeated registration without appropriate cleanup can keep objects reachable longer than intended.

---

# 27. Important Commands Cheat Sheet

## Find Java processes

```powershell
jps -l
```

## Show JVM commands

```powershell
jcmd <PID> help
```

## Heap information

```powershell
jcmd <PID> GC.heap_info
```

## Run GC for diagnosis

```powershell
jcmd <PID> GC.run
```

## Object histogram

```powershell
jcmd <PID> GC.class_histogram
```

## Create heap dump

```powershell
jcmd <PID> GC.heap_dump C:\temp\loan-heap.hprof
```

## Thread dump

```powershell
jcmd <PID> Thread.print
```

## Save thread dump

```powershell
jcmd <PID> Thread.print > C:\temp\thread-dump.txt
```

## Alternative thread dump command

```powershell
jstack <PID>
```

---

# 28. What Each Tool Answers

| Tool | Question it answers |
|---|---|
| `jps -l` | Which Java applications are running? |
| `jcmd <PID> GC.heap_info` | How much heap is being used? |
| `jcmd <PID> GC.run` | Can GC reclaim memory? |
| `jcmd <PID> GC.class_histogram` | Which object types consume memory? |
| `jcmd <PID> GC.heap_dump` | Can I capture the heap for deep analysis? |
| MAT Histogram | Which objects/classes are consuming memory? |
| MAT Dominator Tree | Which objects retain the most memory? |
| MAT Retained Heap | How much memory is retained through an object? |
| MAT Path to GC Roots | Why is an object still reachable? |
| `jcmd <PID> Thread.print` | What are JVM threads doing? |

---

# 29. Most Important MAT Concepts

Learn these five first:

```text
Histogram
    ↓
Dominator Tree
    ↓
Shallow Heap
    ↓
Retained Heap
    ↓
Path to GC Roots
```

Mental model:

```text
Histogram
"What is consuming memory?"

Dominator Tree
"Who is retaining a lot of memory?"

Retained Heap
"How much memory is retained because of this object?"

Path to GC Roots
"Why can't GC remove this object?"
```

---

# 30. Final Mental Model

When you see:

```text
OutOfMemoryError: Java heap space
```

think:

```text
             OOM
              |
              v
        Is heap actually full?
              |
              v
          heap_info
              |
              v
       Check GC behavior
              |
              v
    Memory still high after GC?
          /            \
        YES             NO
         |               |
         v               v
  Suspect retention   Legitimate/
         |            temporary usage
         v
  class_histogram
         |
         v
    heap dump
         |
         v
       MAT
         |
         v
  Dominator Tree
         |
         v
  Retained Heap
         |
         v
 Path to GC Roots
         |
         v
    Source code
         |
         v
       Fix
         |
         v
   Load test
         |
         v
     Verify
```

> **The goal is not simply to remove the `OutOfMemoryError`. The goal is to identify which objects are consuming/retaining memory, determine why they remain reachable, fix the code or configuration causing that retention, and verify that memory remains stable under load.**
