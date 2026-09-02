# Obsidian Java Enterprise Backend

> A structured Obsidian knowledge base for becoming a production-ready, enterprise-grade Java backend engineer.

This repository contains my personal engineering knowledge base covering **Core Java, JVM internals, Spring Boot, databases, microservices, distributed systems, system design, DSA, security, performance, testing, observability, and backend interview preparation**.

The goal is not to collect random interview questions.

The goal is to build a **deep, connected understanding of Java backend engineering** that can be applied to real production systems and senior-level technical interviews.

---

## 🎯 Goals

This knowledge base is designed to help me:

* Master Core Java
* Understand JVM internals
* Build production-grade Spring Boot applications
* Understand Spring internals instead of relying on annotations blindly
* Design reliable REST APIs
* Build secure backend services
* Work effectively with SQL and relational databases
* Understand JPA and Hibernate internals
* Design microservices
* Work with Kafka and event-driven systems
* Use Redis and caching effectively
* Understand distributed systems
* Design scalable backend architectures
* Practice High-Level Design (HLD)
* Practice Low-Level Design (LLD)
* Improve DSA problem-solving
* Prepare for Java backend interviews
* Understand production failures and troubleshooting
* Develop senior-level engineering judgment

---

# 🧭 Knowledge Roadmap

```text
                    Java Enterprise Backend
                              │
          ┌───────────────────┼───────────────────┐
          │                   │                   │
        Java                Spring              Backend
          │                   │                   │
       Core Java          Spring Core        HTTP / REST
       Collections        Spring Boot        Security
       Generics           Spring MVC         APIs
       Streams            Spring Data        Kafka
       Concurrency        JPA/Hibernate      Redis
       JVM                Security           Resilience
       Performance        Transactions       Observability
          │                   │                   │
          └───────────────────┼───────────────────┘
                              │
                         Databases
                              │
                   SQL / Transactions
                   Indexes / Optimization
                   Replication / Sharding
                              │
                              ▼
                    Distributed Systems
                              │
                    System Design / HLD
                              │
                              ▼
                    Production Engineering
                              │
                              ▼
                       Senior Engineer
```

---

# 📚 Repository Structure

```text
obsidian-java-enterprise-backend/
│
├── 00-Home/
│   ├── Home.md
│   ├── Learning-Roadmap.md
│   ├── Weekly-Review.md
│   ├── Engineering-Maturity.md
│   └── Knowledge-Map.md
│
├── 01-Java/
│   ├── Java-MOC.md
│   ├── 01-Core-Java/
│   ├── 02-OOP/
│   ├── 03-Collections/
│   ├── 04-Generics/
│   ├── 05-Exceptions/
│   ├── 06-Strings/
│   ├── 07-Streams-Lambdas/
│   ├── 08-Concurrency/
│   ├── 09-JVM/
│   ├── 10-Memory-Management/
│   ├── 11-IO-NIO/
│   ├── 12-Reflection/
│   ├── 13-Annotations/
│   ├── 14-Functional-Programming/
│   ├── 15-Modern-Java/
│   └── 16-Advanced-Java/
│
├── 02-Spring/
│   ├── Spring-MOC.md
│   ├── 01-Spring-Core/
│   ├── 02-Spring-Container/
│   ├── 03-Dependency-Injection/
│   ├── 04-Spring-Boot/
│   ├── 05-Spring-MVC/
│   ├── 06-Spring-Data/
│   ├── 07-Spring-JPA/
│   ├── 08-Hibernate/
│   ├── 09-Spring-Security/
│   ├── 10-Spring-AOP/
│   ├── 11-Spring-Transactions/
│   ├── 12-Spring-Testing/
│   ├── 13-Spring-Validation/
│   ├── 14-Spring-Actuator/
│   ├── 15-Spring-Cloud/
│   └── 16-Spring-Advanced/
│
├── 03-Backend-Engineering/
│   ├── Backend-MOC.md
│   ├── 01-HTTP/
│   ├── 02-REST/
│   ├── 03-API-Design/
│   ├── 04-Authentication/
│   ├── 05-Authorization/
│   ├── 06-API-Security/
│   ├── 07-Microservices/
│   ├── 08-Messaging/
│   ├── 09-Kafka/
│   ├── 10-Redis/
│   ├── 11-Caching/
│   ├── 12-Resilience/
│   ├── 13-Observability/
│   ├── 14-Logging/
│   ├── 15-Monitoring/
│   ├── 16-Performance/
│   ├── 17-File-Processing/
│   ├── 18-Scheduling/
│   └── 19-Enterprise-Patterns/
│
├── 04-Database/
│   ├── Database-MOC.md
│   ├── 01-SQL/
│   ├── 02-PostgreSQL/
│   ├── 03-MySQL/
│   ├── 04-Database-Design/
│   ├── 05-Indexes/
│   ├── 06-Query-Optimization/
│   ├── 07-Transactions/
│   ├── 08-ACID/
│   ├── 09-Isolation-Levels/
│   ├── 10-Locking/
│   ├── 11-Deadlocks/
│   ├── 12-Replication/
│   ├── 13-Sharding/
│   ├── 14-Partitioning/
│   ├── 15-Connection-Pooling/
│   ├── 16-JPA/
│   └── 17-Hibernate/
│
├── 05-System-Design/
│   ├── System-Design-MOC.md
│   ├── 01-Fundamentals/
│   ├── 02-Scalability/
│   ├── 03-Availability/
│   ├── 04-Reliability/
│   ├── 05-Consistency/
│   ├── 06-Distributed-Systems/
│   ├── 07-Caching/
│   ├── 08-Load-Balancing/
│   ├── 09-Database-Scaling/
│   ├── 10-Messaging/
│   ├── 11-Distributed-Transactions/
│   ├── 12-Distributed-Locks/
│   ├── 13-Rate-Limiting/
│   ├── 14-Event-Driven-Architecture/
│   ├── 15-Service-Discovery/
│   ├── 16-Resilience/
│   ├── 17-HLD/
│   └── 18-LLD/
│
├── 06-DSA/
│   ├── DSA-MOC.md
│   ├── 01-Arrays/
│   ├── 02-Strings/
│   ├── 03-Hashing/
│   ├── 04-Linked-List/
│   ├── 05-Stack/
│   ├── 06-Queue/
│   ├── 07-Heap/
│   ├── 08-Trees/
│   ├── 09-Graphs/
│   ├── 10-Trie/
│   ├── 11-Union-Find/
│   ├── 12-Recursion/
│   ├── 13-Backtracking/
│   ├── 14-Dynamic-Programming/
│   ├── 15-Greedy/
│   ├── 16-Binary-Search/
│   └── 17-Problem-Patterns/
│
├── 07-Interview/
│   ├── Interview-MOC.md
│   ├── Core Java/
│   ├── Spring/
│   ├── Backend/
│   ├── Database/
│   ├── Production-Scenarios/
│   ├── Behavioral/
│   └── Company-Questions/
│
├── 08-Projects/
│   ├── Projects-MOC.md
│   ├── Project-01/
│   ├── Project-02/
│   ├── Project-03/
│   └── Architecture-Decisions/
│
├── 09-Engineering-Practices/
│   ├── Clean-Code/
│   ├── SOLID/
│   ├── Design-Patterns/
│   ├── Refactoring/
│   ├── Testing/
│   ├── Code-Review/
│   ├── Git/
│   ├── CI-CD/
│   ├── Security/
│   ├── Performance/
│   ├── Documentation/
│   └── Production-Readiness/
│
├── 10-Daily-Notes/
│
├── 90-Templates/
│
└── 99-Archive/
```

---

# ☕ Java

The Java section focuses on understanding the language and runtime deeply.

### Core Java

* OOP
* Classes and Objects
* Interfaces
* Abstract Classes
* Encapsulation
* Inheritance
* Polymorphism
* Composition
* Exceptions
* Strings
* Immutability
* `equals()` / `hashCode()`
* `Object`
* `final`
* `static`

### Collections

* ArrayList
* LinkedList
* HashMap
* HashSet
* TreeMap
* TreeSet
* LinkedHashMap
* ConcurrentHashMap
* PriorityQueue
* Deque
* Iterators
* Comparable
* Comparator

### Generics

* Generic classes
* Generic methods
* Wildcards
* Bounds
* PECS
* Type erasure

### Functional Java

* Lambdas
* Functional interfaces
* Streams
* Optional
* Method references
* Collectors

### Concurrency

* Threads
* Executors
* Thread pools
* `synchronized`
* `volatile`
* Locks
* Atomics
* CompletableFuture
* Concurrent collections
* Race conditions
* Deadlocks
* Starvation
* Livelock
* Java Memory Model

### JVM

* JVM architecture
* Class loading
* Bytecode
* Heap
* Stack
* Metaspace
* JIT
* Garbage collection
* GC tuning
* Memory leaks
* JVM troubleshooting

---

# 🌱 Spring

The Spring section focuses on both **using Spring Boot and understanding what happens underneath it**.

### Spring Core

* IoC
* Dependency Injection
* ApplicationContext
* BeanFactory
* Bean lifecycle
* Bean scopes
* Component scanning
* Configuration
* Bean post-processors

### Spring Boot

* Auto-configuration
* Starters
* Configuration properties
* Profiles
* Actuator
* Application lifecycle
* Embedded servers
* External configuration

### Spring MVC

* DispatcherServlet
* Controllers
* Request mapping
* Validation
* Exception handling
* Filters
* Interceptors
* Message converters

### Spring Data / JPA / Hibernate

* Entity lifecycle
* Persistence context
* EntityManager
* Repositories
* Lazy loading
* Eager loading
* N+1 queries
* Fetch joins
* Projections
* Dirty checking
* Optimistic locking
* Pessimistic locking
* Cascading

### Transactions

* `@Transactional`
* Propagation
* Isolation
* Rollback
* Transaction boundaries
* Proxy behavior
* Self-invocation

### Security

* Authentication
* Authorization
* Security filter chain
* JWT
* OAuth2
* OpenID Connect
* Roles
* Authorities
* CORS
* CSRF
* Method security

---

# ⚙️ Backend Engineering

The backend section focuses on production engineering.

Topics include:

* HTTP
* REST
* API design
* API versioning
* Pagination
* Idempotency
* Authentication
* Authorization
* API security
* Microservices
* Messaging
* Kafka
* Redis
* Caching
* Retry
* Timeout
* Circuit breaker
* Rate limiting
* Distributed locks
* Event-driven architecture
* Observability
* Logging
* Monitoring
* Performance
* Scheduling
* Enterprise integration patterns

---

# 🗄️ Database Engineering

Database knowledge is treated as a core backend engineering skill.

### SQL

* Joins
* Subqueries
* CTEs
* Window functions
* Aggregations
* Query optimization

### Indexing

* B-tree
* Composite indexes
* Covering indexes
* Selectivity
* Cardinality
* Query plans
* `EXPLAIN`

### Transactions

* ACID
* Isolation levels
* Locking
* Deadlocks
* Optimistic locking
* Pessimistic locking

### Scaling

* Replication
* Read replicas
* Partitioning
* Sharding
* Connection pooling

### Java integration

* JDBC
* JPA
* Hibernate
* Persistence context
* N+1 problem
* Fetch strategies

---

# 🌐 Distributed Systems

The repository focuses heavily on the problems that appear when a backend grows beyond a single process.

Topics include:

* Distributed communication
* Network failures
* Timeouts
* Retries
* Idempotency
* Eventual consistency
* CAP
* PACELC
* Distributed locks
* Leader election
* Service discovery
* Message delivery
* Duplicate messages
* Ordering
* Distributed transactions
* Outbox pattern
* Saga pattern
* Failure recovery

---

# 🏗️ System Design

System design notes follow a consistent framework.

## Requirements

* Functional requirements
* Non-functional requirements
* Scale assumptions

## Architecture

* Components
* APIs
* Data flow
* Storage
* Caching
* Messaging

## Scalability

* Horizontal scaling
* Load balancing
* Caching
* Database scaling
* Partitioning

## Reliability

* Failure scenarios
* Timeouts
* Retries
* Circuit breakers
* Redundancy
* Graceful degradation

## Operations

* Logging
* Metrics
* Tracing
* Alerting
* Health checks

## Trade-offs

Every system design should explain:

> Why this design?

> What alternatives were considered?

> What breaks first?

> How would we scale it?

---

# 🧩 HLD Case Studies

The repository includes system-design exercises such as:

* URL Shortener
* Notification System
* Payment System
* Rate Limiter
* WhatsApp
* Instagram
* Uber
* Food Delivery
* E-Commerce
* File Storage
* Video Streaming
* Search
* Order Management
* Inventory Management

---

# 🧱 LLD Case Studies

Examples include:

* Parking Lot
* Elevator
* Library Management
* Tic Tac Toe
* Chess
* Vending Machine
* Payment System
* Food Ordering
* Notification System
* Expense Sharing

Focus areas:

* OOP
* SOLID
* Design patterns
* Interfaces
* Extensibility
* Maintainability
* Java implementation

---

# 🧠 DSA

DSA is included for technical interview preparation.

### Data Structures

* Arrays
* Strings
* Hashing
* Linked Lists
* Stack
* Queue
* Heap
* Trees
* BST
* Graphs
* Trie
* Union Find

### Algorithms

* Sorting
* Binary Search
* Recursion
* Backtracking
* Greedy
* Dynamic Programming
* BFS
* DFS

### Patterns

* Two Pointers
* Sliding Window
* Fast/Slow Pointers
* Prefix Sum
* Binary Search
* Monotonic Stack
* BFS
* DFS
* Backtracking
* Heap
* Greedy
* DP
* Union Find
* Topological Sort
* Intervals

The focus is on **recognizing patterns and reasoning about complexity**, rather than memorizing solutions.

---

# 🎯 Interview Preparation

Interview preparation is integrated with the engineering knowledge.

Categories include:

* Core Java
* OOP
* Collections
* Generics
* Streams
* Concurrency
* JVM
* Spring
* Spring Boot
* Spring Security
* JPA
* Hibernate
* SQL
* Microservices
* Kafka
* Redis
* System Design
* DSA
* Production scenarios
* Behavioral interviews

Questions should emphasize:

> How?

> Why?

> What happens internally?

> What can go wrong?

> How would you debug it?

> How would you scale it?

> What are the trade-offs?

---

# 🚨 Production Engineering

A strong backend engineer needs to understand production failures.

Examples documented here include:

* High API latency
* High CPU
* Memory leaks
* OutOfMemoryError
* Database connection pool exhaustion
* Slow database queries
* Kafka consumer lag
* Redis failure
* Downstream timeout
* Duplicate requests
* Duplicate messages
* Deadlocks
* Transaction failures
* Cascading failures
* Deployment failures

Each scenario follows:

```text
Symptoms
   ↓
Investigation
   ↓
Root Cause
   ↓
Immediate Mitigation
   ↓
Permanent Fix
   ↓
Prevention
```

---

# 🏛️ Engineering Practices

The repository also covers:

* Clean Code
* SOLID
* Design Patterns
* Refactoring
* Code Review
* Testing
* Unit Testing
* Integration Testing
* Contract Testing
* Testcontainers
* Git
* CI/CD
* Security
* Performance
* Documentation
* Production Readiness

---

# 📝 Note Structure

Technical notes follow a consistent structure.

```text
Definition
    ↓
Why it matters
    ↓
How it works
    ↓
Example
    ↓
Production usage
    ↓
Trade-offs
    ↓
Common mistakes
    ↓
Interview questions
    ↓
Related concepts
```

This keeps the knowledge base useful for both **learning and revision**.

---

# 🔗 Obsidian Knowledge Graph

The vault is designed around connected concepts rather than isolated documents.

For example:

```text
HashMap
   │
   ├── equals / hashCode
   ├── Hashing
   ├── Collections
   └── ConcurrentHashMap

@Transactional
   │
   ├── Spring AOP
   ├── Transaction Propagation
   ├── Isolation Levels
   ├── Database Transactions
   └── Hibernate

Kafka
   │
   ├── Event Driven Architecture
   ├── Idempotency
   ├── Outbox Pattern
   ├── Consumer Groups
   └── Distributed Systems
```

The objective is to understand **relationships between concepts**, not memorize isolated definitions.

---

# 🔄 Learning Workflow

Each note can move through:

```text
inbox
  ↓
learning
  ↓
review
  ↓
mastered
```

### Inbox

New concept that has not been studied.

### Learning

Currently studying and understanding.

### Review

Previously learned but needs reinforcement.

### Mastered

Can explain and apply the concept without relying heavily on notes.

---

# 📅 Daily Workflow

A typical learning session:

```text
1. Choose today's topic
        ↓
2. Study concept
        ↓
3. Create/update note
        ↓
4. Link related concepts
        ↓
5. Solve coding problem
        ↓
6. Practice interview question
        ↓
7. Review older concept
        ↓
8. Record confusion
        ↓
9. Plan next session
```

---

# 📆 Weekly Review

Every week review:

* New concepts learned
* Concepts forgotten
* Weak Java topics
* Weak Spring topics
* Weak database concepts
* DSA patterns
* System design problems
* Interview questions
* Production concepts
* Open questions

The goal is continuous improvement rather than simply increasing the number of notes.

---

# 🛠️ Tools and Technologies

Primary ecosystem:

* Java
* Spring
* Spring Boot
* Spring Data
* JPA
* Hibernate
* Spring Security
* SQL
* PostgreSQL
* MySQL
* Kafka
* Redis
* Docker
* Git
* CI/CD
* Observability tooling
* Obsidian

---

# 📐 Engineering Principles

This repository follows several principles:

### Understand before memorizing

Know why something exists and how it works.

### Prefer simple designs

Do not introduce distributed systems complexity without a real requirement.

### Know the trade-offs

Every architectural choice has costs.

### Production over theory

Connect concepts to real systems.

### Measure before optimizing

Use evidence such as metrics, profiles, query plans, and traces.

### Security by default

Never store real credentials, API keys, passwords, tokens, or private secrets in this repository.

### Documentation should remain useful

Prefer concise, linked notes over giant documents.

---

# 🔐 Security

This repository contains knowledge and examples only.

**Never commit:**

* API keys
* Passwords
* JWT secrets
* Cloud credentials
* Private certificates
* `.env` files containing secrets
* Production configuration containing sensitive information

Use placeholders such as:

```text
YOUR_API_KEY
YOUR_DATABASE_PASSWORD
YOUR_JWT_SECRET
```

---

# 🚀 How to Use This Repository

## 1. Open the vault

Open this repository as an Obsidian vault.

## 2. Start at Home

Open:

```text
00-Home/Home.md
```

## 3. Choose a learning path

Recommended order:

```text
Core Java
   ↓
Collections
   ↓
Concurrency
   ↓
JVM
   ↓
Spring Core
   ↓
Spring Boot
   ↓
Spring MVC
   ↓
JPA / Hibernate
   ↓
Transactions
   ↓
Spring Security
   ↓
SQL / Databases
   ↓
REST / API Design
   ↓
Kafka / Redis
   ↓
Microservices
   ↓
Distributed Systems
   ↓
System Design
   ↓
Production Engineering
```

## 4. Use the templates

Create new notes using the templates in:

```text
90-Templates/
```

## 5. Link everything

When learning a concept, connect it to related concepts.

---

# 📈 Long-Term Objective

The ultimate goal is to progress from:

```text
Java Developer
      ↓
Java Backend Developer
      ↓
Spring Boot Developer
      ↓
Enterprise Backend Engineer
      ↓
Senior Backend Engineer
      ↓
Backend / System Design Engineer
```

The repository is therefore designed to grow continuously.

It is not intended to be "finished."

New engineering lessons, production experiences, architecture decisions, interview questions, and system-design insights should continue to be added over time.

---

# 🤝 Contributions

This is primarily a personal engineering knowledge base.

Suggestions, corrections, and improvements are welcome.

When contributing:

* Keep notes technically accurate
* Prefer concise explanations
* Include practical examples
* Explain trade-offs
* Avoid unnecessary duplication
* Link related concepts
* Do not include copyrighted material copied verbatim
* Never include credentials or sensitive information

---

# 📄 License

This repository contains personal learning notes and educational material.

See `LICENSE` for licensing information.

---

# ⭐ Philosophy

> **Don't just learn how to use Java and Spring Boot. Learn how Java and Spring Boot work, why enterprise systems are designed the way they are, what fails in production, and how to make better engineering decisions.**

---

## Repository

**Name:** `obsidian-java-enterprise-backend`

**Focus:** Java Enterprise Backend Engineering

**Primary use:** Obsidian knowledge management + backend engineering + interview preparation

**Level:** Beginner → Intermediate → Advanced → Senior/Architect thinking
