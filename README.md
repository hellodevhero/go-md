# Go & SQL — A Complete Self-Study Library

A comprehensive, hands-on book series for learning Go programming and SQL from zero to production-ready applications. Each book builds on the previous one, progressing from fundamentals to building a full e-commerce system.

---

## Book Overview

| # | Book | Topics | Duration |
|---|------|--------|----------|
| 1 | [Go Fundamentals](golang/1.go-fundamental-book.md) | Syntax, types, structs, interfaces, concurrency, generics | 6 weeks |
| 2 | [Standard Library & Tooling](golang/2.standard-library&tooling.md) | Files, JSON, HTTP, databases, context, CLI tools | 6 weeks |
| 3 | [Architecture & Production](golang/3.architecture.md) | Clean architecture, Gin, Redis, Docker, testing | 6 weeks |
| 4 | [E-commerce Project](golang/4.ecommerce-project.md) | Full project: payments, uploads, jobs, observability | 8 weeks |
| 5 | [SQL for Developers](database/sql-book1-developer-deep.md) | PostgreSQL, schema design, queries, window functions | 4 phases |

---

## Detailed Table of Contents

### Book 1 — Go Fundamentals

> *Phase 1: From Zero to Confident Go Developer*

- **Week 1 — Setup, Syntax & Basic Types**
  - [1.1 Installing Go & Your First Program](golang/1.go-fundamental-book.md#11-installing-go--your-first-program)
  - [1.2 Variables & Types](golang/1.go-fundamental-book.md#12-variables--types) — declarations, zero values, basic types, type conversion, constants, iota
  - [1.3 Control Flow](golang/1.go-fundamental-book.md#13-control-flow) — if/else, for loops, switch, defer
  - [1.4 Arrays, Slices & Maps](golang/1.go-fundamental-book.md#14-arrays-slices--maps)
  - [1.5 Functions](golang/1.go-fundamental-book.md#15-functions) — multiple returns, variadic, closures
  - Week 1 Project: CLI Calculator

- **Week 2 — Structs, Methods & Pointers**
  - [2.1 Structs](golang/1.go-fundamental-book.md#21-structs) — struct literals, comparison, embedded structs, tags
  - [2.2 Pointers](golang/1.go-fundamental-book.md#22-pointers) — why pointers matter, nil pointers
  - [2.3 Methods](golang/1.go-fundamental-book.md#23-methods) — value vs pointer receivers, Stringer interface
  - Week 2 Project: Library Book Manager

- **Week 3 — Interfaces & Error Handling**
  - [3.1 Interfaces](golang/1.go-fundamental-book.md#31-interfaces) — interface values, small interfaces, the golden rule
  - [3.2 Type Assertions & Type Switches](golang/1.go-fundamental-book.md#32-type-assertions--type-switches)
  - [3.3 Error Handling](golang/1.go-fundamental-book.md#33-error-handling) — wrapping, custom errors, panic & recover
  - Week 3 Project: Shape Calculator with Plugin System

- **Week 4 — Packages, Modules & Testing**
  - [4.1 Packages](golang/1.go-fundamental-book.md#41-packages) — naming rules, imports, init functions
  - [4.2 Go Modules](golang/1.go-fundamental-book.md#42-go-modules) — dependency management
  - [4.3 Project Structure](golang/1.go-fundamental-book.md#43-project-structure) — flat vs growing projects
  - [4.4 Testing](golang/1.go-fundamental-book.md#44-testing) — table-driven tests, coverage
  - Week 4 Project: Task Manager Package

- **Week 5 — Goroutines & Channels**
  - [5.1 Goroutines](golang/1.go-fundamental-book.md#51-goroutines) — WaitGroup
  - [5.2 Channels](golang/1.go-fundamental-book.md#52-channels) — buffered vs unbuffered, directional channels
  - [5.3 select Statement](golang/1.go-fundamental-book.md#53-select-statement) — timeout, done channel
  - [5.4 Mutexes & Shared State](golang/1.go-fundamental-book.md#54-mutexes--shared-state) — RWMutex, race detector
  - Week 5 Project: Concurrent File Processor

- **Week 6 — Consolidation & Capstone**
  - [6.1 Generics](golang/1.go-fundamental-book.md#61-generics-go-118) — constraints, generic data structures
  - [6.2 Common Go Patterns](golang/1.go-fundamental-book.md#62-common-go-patterns) — functional options, sync.Once, constructors, sentinels
  - [6.3 Capstone Project](golang/1.go-fundamental-book.md#63-capstone-project): CLI Task Management System

---

### Book 2 — Standard Library & Real-World Development

> *From Fundamentals to Building Real Backend Services*

- **Week 1 — Working with Files & the OS**
  - [1.1 The os Package](golang/2.standard-library&tooling.md#11-the-os-package) — env vars, CLI args, filesystem
  - [1.2 Reading & Writing Files](golang/2.standard-library&tooling.md#12-reading--writing-files)
  - [1.3 The io Package](golang/2.standard-library&tooling.md#13-the-io-package--reading--writing-streams) — Reader, Writer, io.Copy
  - [1.4 The bufio Package](golang/2.standard-library&tooling.md#14-the-bufio-package--buffered-io) — line-by-line, stdin, buffered writing
  - [1.5 Working with Paths](golang/2.standard-library&tooling.md#15-working-with-paths--pathfilepath)
  - [1.6 Encoding: CSV Files](golang/2.standard-library&tooling.md#16-encoding-csv--json-files)
  - Week 1 Project: Log File Analyzer

- **Week 2 — JSON, HTTP Client & External APIs**
  - [2.1 encoding/json](golang/2.standard-library&tooling.md#21-the-encodingjson-package) — marshal, unmarshal, streams, custom marshaling
  - [2.2 net/http Client](golang/2.standard-library&tooling.md#22-the-nethttp-client) — GET, POST, custom client, wrapper
  - [2.3 URL Building & Query Parameters](golang/2.standard-library&tooling.md#23-url-building--query-parameters)
  - Week 2 Project: GitHub API CLI Tool

- **Week 3 — Building HTTP Servers**
  - [3.1 The net/http Server](golang/2.standard-library&tooling.md#31-the-nethttp-server) — request routing, ServeMux
  - [3.2 Building a REST API](golang/2.standard-library&tooling.md#32-building-a-rest-api) — CRUD handlers
  - [3.3 Middleware](golang/2.standard-library&tooling.md#33-middleware) — chaining, response capture
  - Week 3 Project: Bookmark API

- **Week 4 — Databases with database/sql**
  - [4.1 Setting Up PostgreSQL](golang/2.standard-library&tooling.md#41-setting-up-postgresql)
  - [4.2 Connecting to the Database](golang/2.standard-library&tooling.md#42-connecting-to-the-database)
  - [4.3 CRUD Operations](golang/2.standard-library&tooling.md#43-crud-operations)
  - [4.4 Transactions](golang/2.standard-library&tooling.md#44-transactions)
  - [4.5 Using sqlx](golang/2.standard-library&tooling.md#45-using-sqlx--a-thin-wrapper)
  - [4.6 Database Migrations](golang/2.standard-library&tooling.md#46-database-migrations)
  - Week 4 Project: Notes API with PostgreSQL

- **Week 5 — Context, Middleware & CLI Tools**
  - [5.1 The context Package](golang/2.standard-library&tooling.md#51-the-context-package) — cancellation, timeouts, values
  - [5.2 Structured Logging with slog](golang/2.standard-library&tooling.md#52-structured-logging-with-logslog)
  - [5.3 Configuration Management](golang/2.standard-library&tooling.md#53-configuration-management)
  - [5.4 Building CLI Tools with flag](golang/2.standard-library&tooling.md#54-building-cli-tools-with-flag)
  - [5.5 Graceful Shutdown](golang/2.standard-library&tooling.md#55-graceful-shutdown)
  - Week 5 Project: Multi-Command CLI App

- **Week 6 — Capstone: Full REST API**
  - [6.1 Project Overview](golang/2.standard-library&tooling.md#61-project-overview) — BookStore API
  - [6.2 Project Structure](golang/2.standard-library&tooling.md#62-project-structure)
  - [6.3 Database Schema](golang/2.standard-library&tooling.md#63-database-schema)
  - [6.4 Core Components](golang/2.standard-library&tooling.md#64-core-components) — models, JWT, stores, auth middleware
  - [6.5 Testing the API](golang/2.standard-library&tooling.md#65-testing-the-api)

---

### Book 3 — Production-Grade Services

> *Clean Architecture, Gin, Redis, Docker & Background Jobs*

- **Week 1 — Clean Architecture**
  - [1.1 Why Architecture Matters](golang/3.architecture.md#11-why-architecture-matters)
  - [1.2 Layers Explained](golang/3.architecture.md#12-layers-explained) — domain, repository, use cases, infrastructure, delivery
  - [1.3 The Full Project Structure](golang/3.architecture.md#13-the-full-project-structure)
  - [1.4 Dependency Injection](golang/3.architecture.md#14-dependency-injection--wiring-it-together)
  - Week 1 Project: Refactor BookStore to Clean Architecture

- **Week 2 — Gin Framework**
  - [2.1 Why Gin](golang/3.architecture.md#21-why-gin)
  - [2.2 Basic Gin](golang/3.architecture.md#22-basic-gin)
  - [2.3 Routing](golang/3.architecture.md#23-routing)
  - [2.4 Binding & Validation](golang/3.architecture.md#24-binding--validation)
  - [2.5 Middleware in Gin](golang/3.architecture.md#25-middleware-in-gin)
  - [2.6 Standardized Response Format](golang/3.architecture.md#26-standardized-response-format)
  - [2.7 Full Gin Router Setup](golang/3.architecture.md#27-full-gin-router-setup)
  - Week 2 Project: Migrate to Gin

- **Week 3 — Redis: Caching & Sessions**
  - [3.1 What Redis Is For](golang/3.architecture.md#31-what-redis-is-for)
  - [3.2 Connecting to Redis](golang/3.architecture.md#32-connecting-to-redis)
  - [3.3 Basic Redis Operations](golang/3.architecture.md#33-basic-redis-operations)
  - [3.4 Storing Structs in Redis](golang/3.architecture.md#34-storing-structs-in-redis)
  - [3.5 Cache-Aside Pattern](golang/3.architecture.md#35-cache-aside-pattern)
  - [3.6 Redis for Rate Limiting](golang/3.architecture.md#36-redis-for-rate-limiting)
  - [3.7 Redis for Session Management](golang/3.architecture.md#37-redis-for-session-management)
  - [3.8 Redis Hashes](golang/3.architecture.md#38-redis-hashes--storing-structured-data-natively)
  - Week 3 Project: Add Caching & Sessions to BookStore

- **Week 4 — Background Jobs & Workers**
  - [4.1 Why Background Jobs](golang/3.architecture.md#41-why-background-jobs)
  - [4.2 Goroutine-Based Worker Pool](golang/3.architecture.md#42-simple-goroutine-based-worker-pool)
  - [4.3 Redis-Backed Job Queue](golang/3.architecture.md#43-redis-backed-job-queue)
  - [4.4 Job Dispatcher & Handlers](golang/3.architecture.md#44-job-dispatcher--handlers)
  - [4.5 Defining Job Types](golang/3.architecture.md#45-defining-job-types)
  - [4.6 Scheduled Jobs — Cron-style](golang/3.architecture.md#46-scheduled-jobs--cron-style-tasks)
  - [4.7 Wiring Jobs into the Application](golang/3.architecture.md#47-wiring-jobs-into-the-application)
  - Week 4 Project: Background Email & Report Generation

- **Week 5 — Docker & Deployment**
  - [5.1 Why Docker](golang/3.architecture.md#51-why-docker)
  - [5.2 Writing a Dockerfile](golang/3.architecture.md#52-writing-a-dockerfile)
  - [5.3 Docker Compose](golang/3.architecture.md#53-docker-compose--local-development)
  - [5.4 .dockerignore](golang/3.architecture.md#54-dockerignore)
  - [5.5 Makefile](golang/3.architecture.md#55-makefile--developer-convenience)
  - [5.6 Environment Configuration](golang/3.architecture.md#56-environment-configuration-for-production)
  - [5.7 Health Checks & Readiness](golang/3.architecture.md#57-health-checks--readiness)
  - Week 5 Project: Containerize the BookStore

- **Week 6 — Advanced Testing & Capstone**
  - [6.1 Testing Strategies](golang/3.architecture.md#61-testing-strategies-in-go)
  - [6.2 Table-Driven Tests with Subtests](golang/3.architecture.md#62-table-driven-tests-with-subtests)
  - [6.3 Fake Implementations vs Mocks](golang/3.architecture.md#63-fake-implementations-vs-mocks)
  - [6.4 HTTP Handler Tests](golang/3.architecture.md#64-http-handler-tests-with-httptest)
  - [6.5 Integration Tests](golang/3.architecture.md#65-integration-tests-with-a-real-database)
  - [6.6 Capstone: Production-Ready BookStore](golang/3.architecture.md#66-capstone-production-ready-bookstore)

---

### Book 4 — The E-commerce Project

> *A Complete Real-World Application*

- **Architecture Overview**
  - [System Design](golang/4.ecommerce-project.md#system-design)
  - [Domain Model](golang/4.ecommerce-project.md#domain-model)
  - [Full Project Structure](golang/4.ecommerce-project.md#full-project-structure)

- **Week 1 — Project Bootstrap & Product Catalog**
  - [1.1 Database Schema — Full ERD](golang/4.ecommerce-project.md#11-database-schema--full-erd)
  - [1.2 Domain Entities](golang/4.ecommerce-project.md#12-domain-entities) — product, category, user, order, payment
  - [1.3 Repository Interfaces](golang/4.ecommerce-project.md#13-repository-interfaces)
  - [1.4 Product Use Case](golang/4.ecommerce-project.md#14-product-use-case)
  - [1.5 Product HTTP Handler](golang/4.ecommerce-project.md#15-product-http-handler)

- **Week 2 — File Uploads & Image Management**
  - [2.1 MinIO — S3-Compatible Storage](golang/4.ecommerce-project.md#21-minio--s3-compatible-object-storage)
  - [2.2 Storage Service](golang/4.ecommerce-project.md#22-storage-service-implementation)
  - [2.3 Image Processing](golang/4.ecommerce-project.md#23-image-processing-before-upload)
  - [2.4 Image Upload Handler](golang/4.ecommerce-project.md#24-image-upload-handler)

- **Week 3 — Shopping Cart**
  - [3.1 Cart Design Decisions](golang/4.ecommerce-project.md#31-cart-design-decisions)
  - [3.2 Cart Repository (Redis)](golang/4.ecommerce-project.md#32-cart-repository-redis)
  - [3.3 Cart Use Case](golang/4.ecommerce-project.md#33-cart-use-case)
  - [3.4 Cart Handler with Session Cookie](golang/4.ecommerce-project.md#34-cart-handler-with-session-cookie)

- **Week 4 — Stripe Payments & Checkout**
  - [4.1 Stripe Setup](golang/4.ecommerce-project.md#41-stripe-setup)
  - [4.2 Checkout Flow](golang/4.ecommerce-project.md#42-checkout-flow)
  - [4.3 Stripe Package Wrapper](golang/4.ecommerce-project.md#43-stripe-package-wrapper)
  - [4.4 Checkout Use Case](golang/4.ecommerce-project.md#44-checkout-use-case)
  - [4.5 Webhook Handler](golang/4.ecommerce-project.md#45-webhook-handler)

- **Week 5 — Order Fulfillment & Notifications**
  - [5.1 Email Service](golang/4.ecommerce-project.md#51-email-service)
  - [5.2 HTML Email Templates](golang/4.ecommerce-project.md#52-html-email-templates)
  - [5.3 PDF Invoice Generation](golang/4.ecommerce-project.md#53-pdf-invoice-generation)
  - [5.4 Job Handlers](golang/4.ecommerce-project.md#54-job-handlers)

- **Week 6 — Admin API & Inventory**
  - [6.1 Admin Routes](golang/4.ecommerce-project.md#61-admin-routes)
  - [6.2 Admin Order Management](golang/4.ecommerce-project.md#62-admin-order-management)
  - [6.3 Inventory Management](golang/4.ecommerce-project.md#63-inventory-management)
  - [6.4 Analytics Endpoints](golang/4.ecommerce-project.md#64-analytics-endpoints)

- **Week 7 — Observability: Metrics & Tracing**
  - [7.1 Why Observability](golang/4.ecommerce-project.md#71-why-observability)
  - [7.2 Metrics with Prometheus](golang/4.ecommerce-project.md#72-metrics-with-prometheus)
  - [7.3 Distributed Tracing with OpenTelemetry](golang/4.ecommerce-project.md#73-distributed-tracing-with-opentelemetry)
  - [7.4 Structured Logging with Correlation IDs](golang/4.ecommerce-project.md#74-structured-logging-with-correlation-ids)

- **Week 8 — Security & Deployment**
  - [8.1 Security Headers](golang/4.ecommerce-project.md#81-security-headers-middleware)
  - [8.2 Input Sanitization](golang/4.ecommerce-project.md#82-input-sanitization)
  - [8.3 SQL Injection Prevention](golang/4.ecommerce-project.md#83-sql-injection-prevention)
  - [8.4 Rate Limiting](golang/4.ecommerce-project.md#84-rate-limiting-per-endpoint)
  - [8.5 Production Docker Compose](golang/4.ecommerce-project.md#85-production-docker-composeyml)
  - [8.6 CI/CD with GitHub Actions](golang/4.ecommerce-project.md#86-cicd-with-github-actions)
  - [8.7 Final Acceptance Checklist](golang/4.ecommerce-project.md#87-final-acceptance-checklist)

---

### Book 5 — SQL for Developers

> *From Zero to Writing Complex Queries and Designing Real Schemas*

- **Phase 1 — Relational Foundations**
  - [1.1 What a Relational Database Actually Is](database/sql-book1-developer-deep.md#11-what-a-relational-database-actually-is)
  - [1.2 Data Types](database/sql-book1-developer-deep.md#12-data-types) — numeric, text, boolean, date/time, UUID, JSONB
  - [1.3 Constraints](database/sql-book1-developer-deep.md#13-constraints) — NOT NULL, PRIMARY KEY, UNIQUE, CHECK, FOREIGN KEY, DEFAULT
  - [1.4 Normalization](database/sql-book1-developer-deep.md#14-normalization) — 1NF, 2NF, 3NF, when to break rules
  - [1.5 ER Diagrams](database/sql-book1-developer-deep.md#15-er-diagrams)
  - Phase 1 Practice Schema & Exercises

- **Phase 2 — SQL Fundamentals**
  - [2.1 SELECT](database/sql-book1-developer-deep.md#21-select--reading-data)
  - [2.2 WHERE](database/sql-book1-developer-deep.md#22-where--filtering-rows) — comparison, LIKE, IN, BETWEEN, NULL handling
  - [2.3 ORDER BY, LIMIT, OFFSET](database/sql-book1-developer-deep.md#23-order-by-limit-offset) — pagination patterns
  - [2.4 Aggregate Functions](database/sql-book1-developer-deep.md#24-aggregate-functions) — COUNT, SUM, AVG, MIN, MAX
  - [2.5 GROUP BY and HAVING](database/sql-book1-developer-deep.md#25-group-by-and-having)
  - [2.6 JOINs](database/sql-book1-developer-deep.md#26-joins) — INNER, LEFT, RIGHT, FULL OUTER, CROSS, SELF
  - [2.7 INSERT, UPDATE, DELETE, UPSERT](database/sql-book1-developer-deep.md#27-insert-update-delete-upsert)
  - [2.8 Transactions](database/sql-book1-developer-deep.md#28-transactions) — BEGIN, COMMIT, ROLLBACK
  - Phase 2 Exercises

- **Phase 3 — Intermediate SQL**
  - [3.1 Subqueries](database/sql-book1-developer-deep.md#31-subqueries) — scalar, correlated, EXISTS
  - [3.2 CTEs](database/sql-book1-developer-deep.md#32-ctes--common-table-expressions) — recursive CTEs, CTE vs subquery
  - [3.3 Window Functions](database/sql-book1-developer-deep.md#33-window-functions) — ROW_NUMBER, RANK, LAG/LEAD, running totals
  - [3.4 Set Operations](database/sql-book1-developer-deep.md#34-set-operations) — UNION, INTERSECT, EXCEPT
  - [3.5 CASE Expressions](database/sql-book1-developer-deep.md#35-case-expressions)
  - [3.6 Built-in Functions](database/sql-book1-developer-deep.md#36-essential-built-in-functions) — string, date/time, math
  - Phase 3 Exercises

- **Phase 4 — Real-World Schema Design**
  - [4.1 Timestamps](database/sql-book1-developer-deep.md#41-timestamps)
  - [4.2 Soft Deletes](database/sql-book1-developer-deep.md#42-soft-deletes)
  - [4.3 Audit Trails](database/sql-book1-developer-deep.md#43-audit-trails)
  - [4.4 Many-to-Many Relationships](database/sql-book1-developer-deep.md#44-many-to-many-relationships)
  - [4.5 Tree Structures](database/sql-book1-developer-deep.md#45-tree-structures-in-postgresql) — adjacency list, ltree
  - [4.6 JSONB in Practice](database/sql-book1-developer-deep.md#46-jsonb-in-practice)
  - [4.7 Safe Schema Migrations](database/sql-book1-developer-deep.md#47-safe-schema-migrations)
  - [4.8 Common Schema Patterns](database/sql-book1-developer-deep.md#48-common-schema-patterns-reference)
  - Phase 4 Exercise: Online Learning Platform

- **Capstone — Ecommerce Database**
  - [Part 1 — Schema](database/sql-book1-developer-deep.md#part-1--schema)
  - [Part 2 — Seed Data](database/sql-book1-developer-deep.md#part-2--seed-data)
  - [Part 3 — Required Queries](database/sql-book1-developer-deep.md#part-3--required-queries)
  - [Part 4 — Find the Problems](database/sql-book1-developer-deep.md#part-4--find-the-problems)

---

## Learning Path

```
                    ┌─────────────────────────┐
                    │   Book 5: SQL for Devs   │
                    │   (can start anytime)     │
                    └───────────┬─────────────┘
                                │
┌───────────────┐   ┌──────────▼──────────┐   ┌─────────────────────┐   ┌──────────────────┐
│  Book 1:      │──▶│  Book 2:            │──▶│  Book 3:            │──▶│  Book 4:         │
│  Go           │   │  Standard Library   │   │  Architecture &     │   │  E-commerce      │
│  Fundamentals │   │  & Tooling          │   │  Production         │   │  Project         │
└───────────────┘   └─────────────────────┘   └─────────────────────┘   └──────────────────┘
    6 weeks              6 weeks                   6 weeks                   8 weeks
```

**Recommended order:**
1. Start with **Book 1** (Go Fundamentals) — no prior Go knowledge needed
2. **Book 5** (SQL) can be studied in parallel at any point
3. Progress through **Books 2 → 3 → 4** sequentially — each builds on the previous
4. By Book 4, you should have completed both Go Books 1-3 and the SQL book

## Prerequisites

- A computer with macOS, Linux, or Windows
- Basic programming experience in any language
- A terminal/command line
- A code editor (VS Code recommended)

## How to Use This Book

Each book is organized into **weeks** (or **phases** for SQL). Each week contains:
- **Concept sections** with explanations and code examples
- **A hands-on project** to apply what you learned

The best way to learn is to **type every code example yourself** — do not copy-paste. Building muscle memory is part of the process.

---

*Built with the goal of taking you from zero Go/SQL knowledge to building production-ready applications.*
