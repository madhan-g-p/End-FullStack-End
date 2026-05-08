Below is **Phase w: Advanced Database Engineering Concepts (Deep but structured, with pseudo queries + tradeoffs)**.

---

#  ADVANCED DATABASE SYSTEMS (PHASE 2)

---

# 1.  Partitioning (Horizontal vs Vertical)

---

## 🔹 Horizontal Partitioning (Sharding-lite concept)

Split **rows across tables**

Example: users split by region

```sql id="hz1"
users_us
users_in
users_eu
```

Or logic-based:

```sql id="hz2"
user_id % 4 = shard_1
user_id % 4 = shard_2
```

---

### Pros

* Scales reads/writes
* Smaller indexes
* Parallel queries

### Cons

* Complex queries across partitions
* Hard joins across shards
* Rebalancing is painful

---

## 🔹 Vertical Partitioning

Split **columns across tables**

```sql id="vp1"
users(id, name, email)
user_profiles(user_id, bio, avatar, preferences)
```

---

### Pros

* Faster reads for hot columns
* Better caching
* Reduces I/O

### Cons

* More joins required
* Complexity in retrieval

---

# 2.  Sharding Strategies (Real Production Core)

---

## 🔹 Hash-based Sharding

```pseudo id="sh1"
shard = hash(user_id) % 8
```

### Pros

* Even distribution
* Simple logic

### Cons

* Range queries become expensive
* Resharding is hard

---

## 🔹 Range-based Sharding

```pseudo id="sh2"
Shard 1 → user_id 1–1M
Shard 2 → 1M–2M
```

### Pros

* Efficient range queries
* Easy pagination

### Cons

* Hotspot risk (new users always go to last shard)

---

## 🔹 Geo-based Sharding

```pseudo id="sh3"
India shard → IN users
US shard → US users
```

### Pros

* Low latency
* Compliance friendly

### Cons

* Uneven load
* Cross-region queries hard

---

# 3.  OLTP vs OLAP Systems

---

## OLTP (Online Transaction Processing)

Used for:

* apps
* payments
* orders

Example:

```sql id="oltp1"
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
```

### Traits:

* Fast writes
* Small queries
* Real-time consistency

---

## OLAP (Online Analytical Processing)

Used for:

* analytics
* dashboards
* BI systems

Example:

```sql id="olap1"
SELECT region, SUM(sales)
FROM orders
GROUP BY region;
```

### Traits:

* heavy reads
* large scans
* slow writes acceptable

---

### Key difference

| OLTP          | OLAP                |
| ------------- | ------------------- |
| users, orders | reports, dashboards |
| row-based     | columnar often      |
| fast writes   | fast aggregations   |

---

# 4.  Event Sourcing

Instead of storing final state → store **events**

---

## Traditional approach:

```json id="es1"
{ "balance": 500 }
```

---

## Event sourcing approach:

```json id="es2"
[
  { "event": "DEPOSIT", "amount": 100 },
  { "event": "WITHDRAW", "amount": 50 }
]
```

---

## Rebuild state:

```pseudo id="es3"
balance = sum(all events)
```

---

### Pros

* Full audit trail
* Debuggable history
* Replay system possible

---

### Cons

* Complex queries
* Storage heavy
* Reconstruction cost

---

# 5.  CQRS (Command Query Responsibility Segregation)

Separate:

* Writes (Command DB)
* Reads (Query DB)

---

## Example architecture:

```pseudo id="cq1"
WRITE → PostgreSQL
READ → MongoDB / Redis / ElasticSearch
```

---

## Write model:

```sql id="cq2"
INSERT INTO orders VALUES (...);
```

---

## Read model (precomputed):

```sql id="cq3"
SELECT * FROM order_summary_view;
```

---

### Pros

* Massive read optimization
* Scalable systems
* Flexible data modeling

---

### Cons

* Data sync complexity
* Eventual consistency issues

---

# 6.  Query Optimization (EXPLAIN Plans)

---

## SQL query:

```sql id="ex1"
SELECT * FROM users WHERE email = 'a@b.com';
```

---

## EXPLAIN output idea:

```pseudo id="ex2"
Index Scan → YES
Rows scanned → 1
Cost → LOW
```

---

## Bad query:

```sql id="ex3"
SELECT * FROM users WHERE LOWER(email) = 'a@b.com';
```

 breaks index usage

---

### Optimization rules:

* avoid functions on indexed columns
* use selective filters early
* avoid SELECT *

---

### Pros

* massive speed improvements

### Cons

* requires deep understanding of query planner

---

# 7.  Index Tuning (Real Production Strategy)

---

## Good index:

```sql id="ix1"
CREATE INDEX idx_user_email ON users(email);
```

---

## Composite index:

```sql id="ix2"
CREATE INDEX idx_user_city_age ON users(city, age);
```

---

## Rule (VERY IMPORTANT)

 Order matters:

```pseudo id="ix3"
(city, age) ≠ (age, city)
```

---

## Covering index:

Query satisfied fully from index:

```sql id="ix4"
SELECT email FROM users WHERE email = 'x';
```

---

### Pros

* ultra-fast reads

### Cons

* slower writes
* memory overhead

---

# 8.  Backup & Disaster Recovery

---

## Types:

### 1. Full backup

```pseudo id="bk1"
Copy entire DB
```

### 2. Incremental backup

```pseudo id="bk2"
Only changes since last backup
```

### 3. Point-in-time recovery (PITR)

```pseudo id="bk3"
Restore DB to exact timestamp
```

---

### Strategy example:

```pseudo id="bk4"
Daily full backup
Hourly incremental backup
WAL logs for recovery
```

---

### Pros

* data safety
* rollback capability

### Cons

* storage cost
* recovery time complexity

---

# 9.  Migration Strategies

---

## Big migration example:

MySQL → PostgreSQL

---

## Approach:

### 1. Dual writes

```pseudo id="mg1"
Write → MySQL + Postgres simultaneously
```

### 2. Backfill data

```pseudo id="mg2"
Copy old data in batches
```

### 3. Switch reads

```pseudo id="mg3"
Read → Postgres
```

---

### Pros

* zero downtime migration

### Cons

* complexity
* sync risk

---

# 10.  Schema Evolution in Production

---

## Problem:

Schema changes break systems

---

## Safe migration example:

```sql id="se1"
ALTER TABLE users ADD COLUMN age INT NULL;
```

Then:

```pseudo id="se2"
Backfill data in batches
Update application
```

---

## Dangerous:

```sql id="se3"
DROP COLUMN email
```

---

### Best practice:

* backward compatible changes only
* additive changes preferred

---

#  FINAL SYSTEM VIEW (HOW IT ALL CONNECTS)

---

## Real architecture looks like:

```pseudo id="arc1"
API Layer
   ↓
Write DB (Postgres)
   ↓
Event Stream (Kafka)
   ↓
Read DB (Mongo / Redis / Elastic)
   ↓
Analytics DB (ClickHouse / BigQuery)
```

---

#  WHAT YOU NOW UNDERSTAND

You now have exposure to:

* scaling strategies (sharding, partitioning)
* advanced architecture patterns (CQRS, event sourcing)
* real production optimization (indexes, EXPLAIN)
* system safety (backups, migration)
* schema evolution at scale
* OLTP vs OLAP separation

---
