# Go & SQL — A Complete Self-Study Library

A comprehensive, hands-on book series for learning Go programming and SQL from zero to production-ready applications. Each book builds on the previous one, progressing from fundamentals to building a full e-commerce system.

> **Bahasa Indonesia?** Semua buku tersedia dalam [Bahasa Indonesia](id/).

---

## Book Overview

| # | Book | Topics | Duration |
|---|------|--------|----------|
| 1 | [SQL for Developers](database/sql-book1-developer-deep.md) | PostgreSQL, schema design, queries, window functions | 4 phases |
| 2 | [Go Fundamentals](golang/1.go-fundamental-book.md) | Syntax, types, structs, interfaces, concurrency, generics | 6 weeks |
| 3 | [Standard Library & Tooling](golang/2.standard-library&tooling.md) | Files, JSON, HTTP, databases, context, CLI tools | 6 weeks |
| 4 | [Architecture & Production](golang/3.architecture.md) | Clean architecture, Gin, Redis, Docker, testing | 6 weeks |
| 5 | [E-commerce Project](golang/4.ecommerce-project.md) | Full project: payments, uploads, jobs, observability | 8 weeks |

---

## Learning Path

```
┌───────────────┐
│  Book 1:      │     ┌───────────────┐   ┌─────────────────────┐   ┌─────────────────────┐   ┌──────────────────┐
│  SQL for      │────▶│  Book 3:      │──▶│  Book 4:            │──▶│  Book 5:            │──▶│  E-commerce      │
│  Developers   │     │  Standard     │   │  Architecture &     │   │  E-commerce         │   │  Complete!       │
└───────┬───────┘     │  Library      │   │  Production         │   │  Project            │   └──────────────────┘
        │             └───────┬───────┘   └─────────────────────┘   └─────────────────────┘
        │                     │                6 weeks                    8 weeks
        │             ┌───────┴───────┐
        └────────────▶│  Book 2:      │
                      │  Go           │
                      │  Fundamentals │
                      └───────────────┘
                          6 weeks
```

**Recommended order:**
1. Start with **Book 1** (SQL for Developers) and **Book 2** (Go Fundamentals) in parallel — no prior knowledge needed for either
2. Progress to **Book 3** (Standard Library) after completing both Book 1 and Book 2 — you'll need SQL knowledge for the database chapters
3. Continue to **Book 4** (Architecture) → **Book 5** (E-commerce Project) sequentially

---

## Detailed Table of Contents

### Book 1 — SQL for Developers

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

---

### Book 2 — Go Fundamentals

> *Phase 1: From Zero to Confident Go Developer*

- **Week 1 — Setup, Syntax & Basic Types**
  - [1.1 Installing Go & Your First Program](golang/1.go-fundamental-book.md#11-installing-go--your-first-program)
  - [1.2 Variables & Types](golang/1.go-fundamental-book.md#12-variables--types)
  - [1.3 Control Flow](golang/1.go-fundamental-book.md#13-control-flow)
  - [1.4 Arrays, Slices & Maps](golang/1.go-fundamental-book.md#14-arrays-slices--maps)
  - [1.5 Functions](golang/1.go-fundamental-book.md#15-functions)
  - Week 1 Project: CLI Calculator

- **Week 2 — Structs, Methods & Pointers**
  - [2.1 Structs](golang/1.go-fundamental-book.md#21-structs)
  - [2.2 Pointers](golang/1.go-fundamental-book.md#22-pointers)
  - [2.3 Methods](golang/1.go-fundamental-book.md#23-methods)
  - Week 2 Project: Library Book Manager

- **Week 3 — Interfaces & Error Handling**
  - [3.1 Interfaces](golang/1.go-fundamental-book.md#31-interfaces)
  - [3.2 Type Assertions & Type Switches](golang/1.go-fundamental-book.md#32-type-assertions--type-switches)
  - [3.3 Error Handling](golang/1.go-fundamental-book.md#33-error-handling)
  - Week 3 Project: Shape Calculator with Plugin System

- **Week 4 — Packages, Modules & Testing**
  - [4.1 Packages](golang/1.go-fundamental-book.md#41-packages)
  - [4.2 Go Modules](golang/1.go-fundamental-book.md#42-go-modules)
  - [4.3 Project Structure](golang/1.go-fundamental-book.md#43-project-structure)
  - [4.4 Testing](golang/1.go-fundamental-book.md#44-testing)
  - Week 4 Project: Task Manager Package

- **Week 5 — Goroutines & Channels**
  - [5.1 Goroutines](golang/1.go-fundamental-book.md#51-goroutines)
  - [5.2 Channels](golang/1.go-fundamental-book.md#52-channels)
  - [5.3 select Statement](golang/1.go-fundamental-book.md#53-select-statement)
  - [5.4 Mutexes & Shared State](golang/1.go-fundamental-book.md#54-mutexes--shared-state)
  - Week 5 Project: Concurrent File Processor

- **Week 6 — Consolidation & Capstone**
  - [6.1 Generics](golang/1.go-fundamental-book.md#61-generics-go-118)
  - [6.2 Common Go Patterns](golang/1.go-fundamental-book.md#62-common-go-patterns)
  - [6.3 Capstone Project](golang/1.go-fundamental-book.md#63-capstone-project)

---

### Book 3 — Standard Library & Real-World Development

> *From Fundamentals to Building Real Backend Services*

- **Week 1** — Working with Files & the OS
- **Week 2** — JSON, HTTP Client & External APIs
- **Week 3** — Building HTTP Servers
- **Week 4** — Databases with database/sql
- **Week 5** — Context, Middleware & CLI Tools
- **Week 6** — Capstone: Full REST API (BookStore)

---

### Book 4 — Production-Grade Services

> *Clean Architecture, Gin, Redis, Docker & Background Jobs*

- **Week 1** — Clean Architecture
- **Week 2** — Gin Framework
- **Week 3** — Redis: Caching & Sessions
- **Week 4** — Background Jobs & Workers
- **Week 5** — Docker & Deployment
- **Week 6** — Advanced Testing & Capstone

---

### Book 5 — The E-commerce Project

> *A Complete Real-World Application*

- **Week 1** — Project Bootstrap & Product Catalog
- **Week 2** — File Uploads & Image Management
- **Week 3** — Shopping Cart
- **Week 4** — Stripe Payments & Checkout
- **Week 5** — Order Fulfillment & Notifications
- **Week 6** — Admin API & Inventory Management
- **Week 7** — Observability: Metrics & Tracing
- **Week 8** — Security Hardening & Final Deployment

---

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
