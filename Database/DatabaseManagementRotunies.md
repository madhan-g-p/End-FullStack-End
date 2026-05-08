# 🛠️ SECTION C — DATABASE OPERATIONS, MAINTENANCE & RECOVERY

This section focuses on:

* database operations
* maintenance routines
* backups
* migrations
* failure recovery
* scaling operations
* production safety
* monitoring
* operational best practices

This is the layer that keeps production systems:

* stable
* recoverable
* scalable
* observable

---

# C1. BACKUP STRATEGIES

Backups are not optional.

A production database without tested backups is:

```text id="kt1e0v"
a future outage waiting to happen
```

---

# 🧠 Core Backup Goals

| Goal                   | Meaning                        |
| ---------------------- | ------------------------------ |
| Recovery               | restore deleted/corrupted data |
| Disaster Recovery      | survive infra failure          |
| Point-in-time recovery | restore exact timestamp        |
| Compliance             | legal retention                |
| Rollback               | undo bad deployments           |

---

# SQL vs Mongo Backup Comparison

| Area             | SQL (Postgres/MySQL)   | MongoDB         |
| ---------------- | ---------------------- | --------------- |
| Full Backup      | `pg_dump`, `mysqldump` | `mongodump`     |
| Incremental      | WAL/binlogs            | oplogs          |
| PITR             | WAL replay             | oplog replay    |
| Snapshot Backups | EBS/LVM snapshots      | Atlas snapshots |
| Replica Backups  | preferred              | preferred       |
| Online Backups   | supported              | supported       |

---

# 1. Full Backups

Complete database copy.

---

## PostgreSQL

```bash id="7l75a0"
pg_dump mydb > backup.sql
```

---

## MongoDB

```bash id="h2e7ju"
mongodump --db ecommerce
```

---

| Pros              | Cons              |
| ----------------- | ----------------- |
| easiest restore   | large storage     |
| simplest strategy | slow for huge DBs |

---

# 2. Incremental Backups

Only changed data stored.

---

## SQL

Uses:

* WAL logs
* binlogs

---

## MongoDB

Uses:

* oplog replication entries

---

| Pros              | Cons                  |
| ----------------- | --------------------- |
| storage efficient | recovery more complex |
| faster backups    |                       |

---

# 3. Point-in-Time Recovery (PITR)

Restore DB to exact timestamp.

---

## Example Scenario

```text id="by7b63"
Bad deployment at 2:15 PM
Restore database to 2:14 PM
```

---

## PostgreSQL Flow

```text id="e3w7wi"
Base Backup
  +
WAL replay
```

---

## MongoDB Flow

```text id="8c6x65"
Snapshot
  +
Oplog replay
```

---

# 4. Backup Best Practices

---

# ✅ Use Replica Backups

Never backup primary directly.

Reason:

* backup load affects production traffic

---

# ✅ Automate Backups

Example schedule:

```text id="vc6n4u"
Daily full backup
Hourly incremental
5-min WAL shipping
```

---

# ✅ Test Restores Regularly

Most companies backup successfully.

Many fail restoring successfully.

---

# ✅ Encrypt Backups

Protect:

* user data
* credentials
* payment info

---

# ✅ Multi-region Backup Storage

Store backups:

* separate region
* separate cloud account ideally

---

# C2. FAILURE RECOVERY & FAILOVER

Production systems WILL fail.

Goal:

```text id="mb0c4f"
reduce downtime + prevent data loss
```

---

# Types of Failures

| Failure             | Example           |
| ------------------- | ----------------- |
| Hardware            | disk crash        |
| Network             | partition         |
| Application         | bad deployment    |
| Human Error         | accidental delete |
| Replication Failure | replica lag       |
| Data Corruption     | broken indexes    |

---

# 1. Replication-based Failover

---

# PostgreSQL

```text id="k5mz7d"
Primary
   ↓
Replica
```

If primary fails:

* promote replica
* redirect traffic

---

# MongoDB Replica Sets

```text id="ny9qj7"
Primary election occurs automatically
```

---

# Redis Sentinel

Monitors:

* master health
* failover

---

# Best Practice

| Practice               | Why                   |
| ---------------------- | --------------------- |
| automatic failover     | faster recovery       |
| health checks          | detect failures early |
| replication monitoring | avoid stale replicas  |
| failover drills        | validate process      |

---

# 2. Recovery Approaches

---

# A. Snapshot Restore

Restore complete backup.

Best for:

* catastrophic failures

---

# B. PITR Restore

Restore to precise timestamp.

Best for:

* accidental deletes
* bad deployments

---

# C. Logical Recovery

Recover:

* specific table
* collection
* rows

---

# Example

```sql id="x1z9ff"
RESTORE users table only
```

---

# 3. Recovery Metrics

| Metric | Meaning              |
| ------ | -------------------- |
| RPO    | acceptable data loss |
| RTO    | acceptable downtime  |

---

## Example

```text id="8s4fxw"
RPO = 5 min
RTO = 15 min
```

Means:

* max 5 min data loss
* service restored within 15 min

---

# C3. DATABASE MIGRATIONS

Migrations evolve schemas safely in production.

---

# SQL Migration Types

| Type           | Example          |
| -------------- | ---------------- |
| Additive       | add column       |
| Destructive    | drop column      |
| Transformative | datatype changes |

---

# 1. Safe Migration Strategy

---

# Step 1 — Add Nullable Column

```sql id="2iy56m"
ALTER TABLE users
ADD COLUMN age INT NULL;
```

---

# Step 2 — Deploy App

Application starts writing:

```text id="l7j7xg"
old + new schema
```

---

# Step 3 — Backfill Data

```sql id="t1k3jg"
UPDATE users
SET age = 18
WHERE age IS NULL;
```

---

# Step 4 — Make Required

```sql id="n50o1f"
ALTER TABLE users
ALTER COLUMN age SET NOT NULL;
```

---

# Why This Matters

Bad migrations can:

* lock huge tables
* crash APIs
* break deployments

---

# MongoDB Migration Strategy

Mongo schema changes are:

```text id="dwg3rj"
application-driven
```

---

# Example

Old:

```json id="j4xuq8"
{
  "name": "John"
}
```

New:

```json id="pukjtv"
{
  "full_name": "John"
}
```

---

# Migration Flow

```text id="r4k4pn"
deploy app supporting both fields
↓
background migration
↓
remove old field later
```

---

# Best Practices

| Practice                        | Why               |
| ------------------------------- | ----------------- |
| backward compatible changes     | safer deploys     |
| avoid destructive changes first | rollback safety   |
| batch migrations                | avoid DB overload |
| use feature flags               | gradual rollout   |

---

# C4. DATABASE SEEDING

Seeding creates initial or test data.

---

# Types of Seeds

| Type        | Example                  |
| ----------- | ------------------------ |
| Static      | countries table          |
| Test        | fake users               |
| Development | local demo data          |
| Staging     | production-like datasets |

---

# SQL Seed Example

```sql id="ukzr34"
INSERT INTO roles(name)
VALUES ('ADMIN');
```

---

# Mongo Seed Example

```js id="c86sn9"
db.roles.insertOne({
  role: "ADMIN"
});
```

---

# Best Practices

---

# ✅ Idempotent Seeds

Safe to rerun.

Bad:

```text id="2fzq8o"
duplicate inserts
```

Better:

```sql id="l2m7ji"
INSERT ... WHERE NOT EXISTS
```

---

# ✅ Separate Dev vs Prod Seeds

Production should NOT receive:

* fake users
* test payments

---

# C5. SCALING STRATEGIES

Scaling is about handling:

* more users
* more reads
* more writes
* larger datasets

---

# 1. Vertical Scaling

Increase server resources.

```text id="jm0j5r"
more CPU
more RAM
faster disks
```

---

| Pros           | Cons            |
| -------------- | --------------- |
| easy initially | expensive       |
|                | hardware limits |

---

# 2. Read Replicas

Scale read traffic.

---

## Architecture

```text id="vfpb0h"
Primary
 ↓
Read Replicas
```

---

## Best For

* analytics
* dashboards
* heavy read APIs

---

## Problem

Replication lag.

---

# 3. Partitioning

Split tables into smaller chunks.

---

## Example

```text id="f7l50w"
orders_2024
orders_2025
```

---

## Benefits

* smaller indexes
* faster scans

---

# 4. Sharding

Distribute data across servers.

---

## Example

```text id="2vdaxv"
Shard A → users A-M
Shard B → users N-Z
```

---

## SQL Sharding

Usually application-managed.

---

## MongoDB Sharding

Built-in support.

---

# Choosing Good Shard Keys

Good:

```text id="d9n1db"
high cardinality
uniform distribution
```

Bad:

```text id="44ppd0"
status = active/inactive
```

Causes:

* hotspot shards

---

# C6. DATABASE MAINTENANCE ROUTINES

Maintenance prevents:

* performance degradation
* storage bloat
* stale statistics
* fragmentation

---

# SQL Maintenance

| Task              | Purpose               |
| ----------------- | --------------------- |
| VACUUM            | reclaim dead rows     |
| ANALYZE           | refresh planner stats |
| REINDEX           | rebuild indexes       |
| Partition cleanup | archive old data      |

---

# PostgreSQL Example

```sql id="jlwmcg"
VACUUM ANALYZE orders;
```

---

# MongoDB Maintenance

| Task            | Purpose                 |
| --------------- | ----------------------- |
| Compact         | reclaim space           |
| Reindex         | rebuild indexes         |
| Chunk balancing | even shard distribution |
| Oplog cleanup   | replication stability   |

---

# Maintenance Schedule Example

| Frequency | Task                    |
| --------- | ----------------------- |
| Daily     | backup verification     |
| Weekly    | slow query analysis     |
| Monthly   | index review            |
| Quarterly | disaster recovery drill |

---

# C7. QUERY OPTIMIZATION & PERFORMANCE DEBUGGING

---

# SQL Explain Plan

```sql id="tfr4nf"
EXPLAIN ANALYZE
SELECT *
FROM orders
WHERE user_id = 1;
```

---

# Mongo Explain

```js id="rmpkxr"
db.orders.find({
  user_id: 1
}).explain("executionStats")
```

---

# Key Metrics

| Metric              | Meaning                     |
| ------------------- | --------------------------- |
| Seq Scan / COLLSCAN | full scan (bad large-scale) |
| Index Scan / IXSCAN | indexed access              |
| Rows Examined       | workload size               |
| Execution Time      | latency                     |

---

# Optimization Workflow

```text id="ab95ei"
Slow Query
   ↓
Explain Plan
   ↓
Check Indexes
   ↓
Rewrite Query
   ↓
Benchmark Again
```

---

# Common Performance Problems

| Problem         | Cause             |
| --------------- | ----------------- |
| slow joins      | missing indexes   |
| high CPU        | full scans        |
| memory spikes   | huge aggregations |
| write slowdown  | over-indexing     |
| lock contention | long transactions |

---

# C8. MONITORING & OBSERVABILITY

Monitoring prevents silent failures.

---

# Important Metrics

| Metric           | Why Important     |
| ---------------- | ----------------- |
| QPS              | database load     |
| replication lag  | stale replicas    |
| slow queries     | bottlenecks       |
| connection count | pool exhaustion   |
| cache hit ratio  | Redis efficiency  |
| lock waits       | contention issues |

---

# Monitoring Tools

| Tool                | Purpose                 |
| ------------------- | ----------------------- |
| Prometheus          | metrics collection      |
| Grafana             | dashboards              |
| pg_stat_statements  | Postgres query insights |
| Mongo Atlas Metrics | Mongo monitoring        |

---

# Alerts You MUST Have

| Alert                 | Reason            |
| --------------------- | ----------------- |
| replication lag spike | stale reads       |
| disk near full        | outage prevention |
| backup failure        | disaster risk     |
| high query latency    | performance issue |
| connection exhaustion | app failure       |

---

# C9. SECURITY & ACCESS CONTROL

---

# SQL Security

| Feature            | Purpose             |
| ------------------ | ------------------- |
| Roles              | grouped permissions |
| Grants             | access control      |
| Row-level security | tenant isolation    |

---

# PostgreSQL RLS Example

```sql id="6esqwy"
CREATE POLICY tenant_policy
ON orders
USING (tenant_id = current_setting('app.tenant'));
```

---

# MongoDB Security

| Feature | Purpose            |
| ------- | ------------------ |
| RBAC    | permission control |
| TLS     | encrypted traffic  |
| SCRAM   | authentication     |

---

# Redis Security

| Feature        | Purpose             |
| -------------- | ------------------- |
| AUTH           | password protection |
| ACLs           | user permissions    |
| Protected mode | network safety      |

---

# Security Best Practices

- ✅ Never expose DB publicly - Only app servers should access DBs.
- ✅ Rotate credentials regularly
- ✅ Encrypt backups
- ✅ Principle of least privilege - Applications should get only minimum required permissions


---

# C10. OPERATIONAL BEST PRACTICES

---

# Deployment Checklist

| Check             | Why                 |
| ----------------- | ------------------- |
| backup verified   | recovery safety     |
| rollback plan     | deployment safety   |
| migration tested  | schema safety       |
| replica healthy   | HA readiness        |
| alerts configured | incident visibility |

---

# Production Golden Rules

---

# ✅ Avoid long-running transactions

Reason:

* locks
* blocking
* deadlocks

---

# ✅ Avoid SELECT *

Reason:

* unnecessary I/O

---

# ✅ Review indexes quarterly

Old indexes:

* waste RAM
* slow writes

---

# ✅ Separate OLTP & analytics workloads

Heavy analytics can:

```text id="jxy3hx"
kill production performance
```

---

# ✅ Benchmark before scaling

Sometimes:

```text id="ltme1w"
query optimization
>
adding servers
```

---

# ✅ Test disaster recovery regularly

A backup is only useful if:

```text id="9lclax"
restoration actually works
```

---

# ✅ END OF SECTION C

Covered:

* backups
* PITR
* migrations
* failure recovery
* failover
* scaling
* maintenance
* monitoring
* optimization
* security
* production operational routines
* SQL + Mongo operational differences


So Far In database engineering foundation we have covered:

* SQL fundamentals → advanced analytics
* MongoDB engineering
* aggregation pipelines
* schema design
* joins/window functions
* indexing/query optimization
* scaling/sharding
* backups/recovery
* migrations/maintenance
* operational production practices

This is already enough to:

* handle backend interviews confidently
* design production-ready schemas
* debug slow queries
* understand scaling tradeoffs
* maintain real systems safely

A very strong next layer after this would typically be one of:

1. **Advanced PostgreSQL Engineering**

   * WAL internals
   * MVCC
   * vacuum internals
   * query planner deep dive
   * GIN/GiST indexes
   * JSONB optimization

2. **Redis Deep Dive**

   * eviction policies
   * persistence (RDB vs AOF)
   * distributed locks
   * pub/sub
   * streams
   * rate limiting
   * caching patterns

3. **Database-heavy System Design**

   * Instagram feed DB
   * Uber location storage
   * payment systems
   * multi-tenant SaaS databases
   * event-driven architectures

4. **Data Engineering Layer**

   * Kafka
   * CDC (Change Data Capture)
   * Debezium
   * warehouses
   * ETL pipelines
   * ClickHouse
   * BigQuery
   * Snowflake

5. **ORMs & Query Builders**

   * Prisma
   * TypeORM
   * Sequelize
   * Drizzle
   * Mongoose
   * query optimization through ORMs
   * N+1 ORM issues

We've now built a solid “raw database engineering handbook” structure that can be expanded topic-by-topic later.
