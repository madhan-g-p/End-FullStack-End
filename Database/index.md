# 🧠 DATABASE CORE HANDBOOK — PART 1 (20 Essential Concepts)

---

If you intend to do database installation first refer this doc [Database Installation and Setup](./DatabasesInstallationandSetup.md)

# 1. SQL vs NoSQL (Core Mental Model)

## SQL (Relational)

Data stored in **tables (rows + columns)** with strict schema.

* Best for structured data
* Strong consistency
* Supports joins

Example: orders ↔ users ↔ payments

---

## NoSQL (Non-relational)

Data stored as:

* JSON documents (MongoDB)

* key-value pairs (Redis)

* wide-column, graph, etc.

* Flexible schema

* High scale

* Fast writes

---

## Core Rule

👉 If relationships matter → SQL
👉 If flexibility & scale matter → NoSQL

---

| Use Case / Requirement                           | MySQL    | PostgreSQL                | MongoDB                | Redis             |
| ------------------------------------------------ | -------- | ------------------------- | ---------------------- | ----------------- |
| Strong relational data (users, orders, payments) | ✅        | ✅ (best)                  | ❌                      | ❌                 |
| Complex joins & analytics queries                | ⚠️ basic | ✅ best                    | ❌                      | ❌                 |
| Flexible schema / fast iteration                 | ❌        | ⚠️ partial (JSON support) | ✅ best                 | ❌                 |
| High write throughput (logs, events)             | ⚠️       | ⚠️                        | ✅ best                 | ⚠️ (cache writes) |
| Real-time caching                                | ❌        | ❌                         | ⚠️                     | ✅ best            |
| Session storage                                  | ❌        | ❌                         | ❌                      | ✅ best            |
| Leaderboard / counters                           | ❌        | ❌                         | ❌                      | ✅ best            |
| Full-text search                                 | ⚠️       | ✅ (good)                  | ⚠️                     | ❌                 |
| Complex transactions                             | ✅        | ✅ best                    | ⚠️ (limited multi-doc) | ❌                 |
| Horizontal scaling ease                          | ⚠️       | ⚠️                        | ✅ best                 | ✅ (clustered)     |
| Analytics / BI queries                           | ❌        | ⚠️                        | ❌                      | ❌                 |
| Event streaming style data                       | ❌        | ❌                         | ⚠️                     | ⚠️                |

🧠 SIMPLE RULES (INTERVIEW READY)
- MySQL → simple relational apps, legacy systems
- PostgreSQL → default best choice for serious backend systems
- MongoDB → flexible schema, fast development, high write scale
- Redis → cache, speed layer, ephemeral data only

# Further Topics
1. [Database Installations](./DatabasesInstallationandSetup.md)
2. [NoSQL Advanced Engineering](./NoSQLAdvancedEngineering.md)
3. [SQL Advanced Engineering](./SQLAdvancedEngineering.md)
4. [Database Management and Routines](./DatabaseManagementRotunies.md)
5. [Query Examples](./QueryExamples.md)
6. [System Design in Databse](./SystemDesign.md)

# 2. Schema Design (Foundation of Good DB)

## SQL

You must define structure:

```sql
CREATE TABLE users (
  id INT,
  name VARCHAR(100),
  email VARCHAR(255)
);
```

## MongoDB

Schema is implicit:

```json
{
  "name": "John",
  "email": "john@email.com"
}
```

---

## Key Insight

Bad schema design = permanent performance issues later.

---

# 3. Tables vs Collections

| SQL    | MongoDB    |
| ------ | ---------- |
| Table  | Collection |
| Row    | Document   |
| Column | Field      |

---

# 4. Primary Key

Unique identifier for a record.

Example:

```sql
id INT PRIMARY KEY
```

Rules:

* Must be unique
* Cannot be NULL
* Indexed automatically

---

# 5. Foreign Key

Used to link tables.

```sql
orders.user_id → users.id
```

Purpose:

* Enforce relationships
* Maintain data integrity

---

# 6. Indexing (Critical for Performance)

## What it does:

Speeds up search queries by creating a lookup structure.

Without index → full table scan
With index → binary/tree search

---

## Types:

* Primary index
* Unique index
* Composite index (multi-column)

---

## Example:

```sql
CREATE INDEX idx_user_email ON users(email);
```

---

## Tradeoff:

* Faster reads
* Slower writes (updates index too)

---

# 7. Joins (SQL Core Power Feature)

- INNER JOIN - Matching records only
- LEFT JOIN - All left table + matching right
- RIGHT JOIN - Opposite of left join
- FULL JOIN  - Everything from both

---

## Example:

```sql
SELECT users.name, orders.amount
FROM users
JOIN orders ON users.id = orders.user_id;
```

---

# 8. Subqueries

Query inside a query.

```sql
SELECT name FROM users
WHERE id IN (
  SELECT user_id FROM orders
);
```

Use cases:

* filtering
* computed datasets

---

# 9. Normalization (Data Cleanup Strategy)

Goal: avoid redundancy.

- 1NF: - No repeating groups
- 2NF: - No partial dependency
- 3NF: - No transitive dependency
- Boyce-Codd (BCNF): - 3NF and all tables in the database should be only one primary key.
- 4NF: - Tables cannot have multi-valued dependencies on a Primary Key.
- 5NF: - A composite key shouldn't have any cyclic dependencies.

---

## Example problem:

Storing:

```
user_name + user_address in orders table
```

Bad → duplication
Fix → separate users table

---

# 10. Denormalization (Opposite of normalization)

Used when performance matters.

Example:

* storing product name in order table (even if duplicated)

👉 tradeoff = faster reads, more storage

---

# 11. Transactions (ACID)

ACID:

* Atomicity
* Consistency
* Isolation
* Durability

---

## SQL Example:

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100;
UPDATE accounts SET balance = balance + 100;
COMMIT;
```

---

# 12. MongoDB Transactions

Supported but:

* requires replica sets
* heavier than SQL

Used for:

* multi-document updates
* financial consistency cases

---

# 13. Joins Depth Reality

Technically unlimited
Practically:

* 3–5 joins max before performance drops

---

## Problem:

Too many joins → slow queries → redesign schema needed

---

# 14. N+1 Query Problem (Very Important)

## Problem:

1 query fetches users
Then N queries for orders per user

---

## Bad:

```sql
SELECT * FROM users;
SELECT * FROM orders WHERE user_id=1;
SELECT * FROM orders WHERE user_id=2;
```

---

## Fix:

* SQL JOIN
* OR ORM eager loading
* Mongo aggregation

---

# 15. Aggregation (MongoDB Power Feature)

Used for:

* grouping
* filtering
* transformation pipelines

Example:

* total sales per user
* product revenue

---

# 16. Triggers (Automated DB logic)

Runs automatically on events.

## SQL supports:

* BEFORE INSERT
* AFTER UPDATE

Example:

* log audit changes

---

MongoDB:

* no native triggers (Atlas has limited cloud triggers)

---

# 17. Stored Procedures / Functions

Reusable SQL logic inside DB.

Example:

```sql
CREATE FUNCTION get_discount(price INT)
RETURNS INT
```

Used for:

* business logic inside DB

Downside:

* harder to maintain than app code

---

# 18. CAP Theorem (Distributed Systems)

You can only choose 2:

* Consistency
* Availability
* Partition tolerance

---

Real systems:

* MongoDB → AP (usually)
* SQL → CP (strong consistency)

---

# 19. Replication

Copy of database across servers.

Purpose:

* high availability
* failover safety

---

MongoDB needs replica sets for:

* transactions
* failover
* consistency guarantees

---

# 20. Connection Pooling

Instead of opening new DB connection every time:

👉 reuse connections

Benefits:

* faster queries
* less DB load
* prevents overload

---

## SQL example:

* max connections tuning required

## Node.js:

* pg / mysql pool usage

---

# 🧠 SUMMARY OF WHAT YOU NOW HAVE

You now understand:

* SQL vs NoSQL decision logic
* Schema design fundamentals
* Indexing & performance basics
* Joins + subqueries
* Transactions & ACID
* Mongo aggregation model
* Distributed systems basics (CAP, replication)
* Real production issues (N+1, pooling, joins limits)

---

