---
type: dashboard
status: active
tags: [home]
---

# 🕸️ Knowledge Map

The vault is designed around connected concepts. Use Obsidian's graph view alongside this map.

```text
                    Java Enterprise Backend
                              │
          ┌───────────────────┼───────────────────┐
        Java                Spring              Backend
          │                   │                   │
       Core Java          Spring Core        HTTP / REST
       Collections        Spring Boot        Security
       Concurrency        JPA/Hibernate      Kafka / Redis
       JVM                Transactions       Resilience
          └───────────────────┼───────────────────┘
                          Databases
                              │
                    Distributed Systems
                              │
                     System Design (HLD/LLD)
                              │
                     Production Engineering
```

## Example concept clusters

```text
HashMap → equals/hashCode → Hashing → ConcurrentHashMap → Collections
@Transactional → Spring AOP → Propagation → Isolation Levels → DB Transactions
Kafka → Event-Driven Architecture → Idempotency → Outbox Pattern → Microservices
```

## Section MOCs

- [[Java-MOC]] · [[Spring-MOC]] · [[Backend-MOC]] · [[Database-MOC]]
- [[System-Design-MOC]] · [[DSA-MOC]] · [[Interview-MOC]] · [[Projects-MOC]]

---
Related: [[Home]]
