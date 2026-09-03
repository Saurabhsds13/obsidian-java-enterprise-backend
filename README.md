# Java Enterprise Backend Engineering

> Enterprise-grade Java backend engineering knowledge base covering Core Java, JVM, Spring Boot, databases, distributed systems, microservices, system design, testing, security, performance, and backend interview preparation.

This repository is a long-term engineering knowledge base for becoming a strong **pure Java + enterprise Spring Boot backend developer**, while also preparing for senior-level backend interviews.

It is designed to feel like a professional engineer's handbook, not a collection of random interview questions:

- Java knowledge base
- Spring Boot handbook
- Backend engineering handbook
- System design notebook
- Interview preparation system
- Project architecture journal
- Revision system
- Daily learning system

---

## Who this is for

Developers targeting roles such as Java Backend Engineer, Spring Boot Developer, Senior Java Developer, Java Microservices Engineer, or Enterprise Application Developer, progressing from junior to staff/architect thinking.

## Learning philosophy

Every note aims to answer:

- How does this work?
- Why is it designed this way?
- When should I use it (and when not)?
- What can go wrong in production?
- How would I explain this in an interview?

Principles: understand before memorizing, prefer simple designs, know the trade-offs, connect theory to production, measure before optimizing, and security by default.

---

## Repository structure

```text
00-Home/                 Dashboards, roadmap, weekly review
01-Java/                 Core Java, collections, concurrency, JVM
02-Spring/               Spring Core, Boot, MVC, Data/JPA, Security
03-Backend-Engineering/  HTTP, REST, messaging, caching, resilience
04-Database/             SQL, indexes, transactions, scaling
05-System-Design/        Fundamentals, distributed systems, HLD, LLD
06-DSA/                  Data structures, algorithms, patterns
07-Interview/            Interview question bank by category
08-Projects/             Project docs and architecture decisions
09-Engineering-Practices/ Clean code, SOLID, testing, CI/CD
10-Daily-Notes/          Daily learning log
90-Templates/            Reusable note templates
99-Archive/              Retired notes
```

## Topic coverage

- **Java**: language, collections, generics, streams, concurrency, JVM internals, GC, memory model
- **Spring**: IoC/DI, Boot auto-configuration, MVC, Data/JPA, Hibernate, transactions, security, AOP
- **Backend**: HTTP, REST, API design, auth, caching, Redis, Kafka, resilience, observability
- **Database**: SQL, indexing, transactions/ACID, isolation levels, replication, sharding
- **System design**: scalability, CAP/PACELC, distributed systems, HLD and LLD case studies
- **DSA**: data structures, algorithms, and problem-solving patterns
- **Interview prep**: category question banks emphasizing depth over trivia
- **Engineering practices**: clean code, SOLID, testing, code review, CI/CD, production readiness

---

## How to use this repository

1. Open the folder as an Obsidian vault.
2. Start at [[Home]] (`00-Home/Home.md`).
3. Follow the [[Learning-Roadmap]] in the order that suits your level.
4. Create new notes from the templates in `90-Templates/`.
5. Link related concepts as you learn so the knowledge graph grows.

### Obsidian usage

This vault relies on Obsidian's built-in features (backlinks, graph view, properties, templates). No community plugins are required. Notes use YAML frontmatter (Obsidian Properties) for metadata and `[[wikilink]]` style cross-references.

---

## Contribution guidelines

This is primarily a personal engineering knowledge base. When contributing:

- Keep notes technically accurate and concise
- Include practical examples and explain trade-offs
- Link related concepts instead of duplicating content
- Do not include copyrighted material verbatim
- Never include credentials or sensitive information

## Security

This repository contains knowledge and examples only. Never commit passwords, API keys, tokens, JWT secrets, cloud credentials, or `.env` secrets. Use placeholders such as `YOUR_API_KEY`, `YOUR_DATABASE_PASSWORD`, `YOUR_JWT_SECRET`. For the hosted repository, enable Dependabot alerts, secret scanning, push protection, and code scanning where appropriate.

## License

See [`LICENSE`](LICENSE). This repository contains personal learning notes and educational material.
