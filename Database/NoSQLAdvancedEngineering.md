# 🍃 SECTION A — MONGODB ADVANCED ENGINEERING

---

# A1. Aggregation Pipeline Mastery

Aggregation pipelines in MongoDB are equivalent to:

* SQL `GROUP BY`
* joins
* analytics
* transformations
* reporting systems

Think of pipelines like:

```text
Input Documents
   ↓
Stage 1 ($match)
   ↓
Stage 2 ($group)
   ↓
Stage 3 ($sort)
   ↓
Final Result
```

Each stage transforms the previous output.

---

# 🧠 Common Dataset

```js id="4y9c59"
orders = {
  _id,
  user_id,
  amount,
  status,
  created_at,
  items: [
    {
      product_id,
      qty,
      price
    }
  ]
}
```

---

# 1. `$match` — Filtering Stage

Equivalent to SQL `WHERE`

---

## Example

```js id="7dc4h6"
db.orders.aggregate([
  {
    $match: {
      status: "PAID"
    }
  }
]);
```

---

## Use Cases

* filter active users
* filter date ranges
* reduce dataset before expensive stages

---

## Best Practice

👉 Always place `$match` early.

Reason:

* reduces documents processed later
* improves performance

---

# 2. `$group` — Aggregation Stage

Equivalent to SQL `GROUP BY`

---

## Example — revenue per user

```js id="s10vsy"
db.orders.aggregate([
  {
    $group: {
      _id: "$user_id",
      totalRevenue: {
        $sum: "$amount"
      }
    }
  }
]);
```

---

## Common Operators

| Operator | Purpose       |
| -------- | ------------- |
| `$sum`   | total         |
| `$avg`   | average       |
| `$max`   | max value     |
| `$min`   | min value     |
| `$push`  | collect array |
| `$first` | first item    |
| `$last`  | last item     |

---

## Example — collect order IDs

```js id="t2j1b7"
{
  $group: {
    _id: "$user_id",
    orders: {
      $push: "$_id"
    }
  }
}
```

---

# 3. `$project` — Field Selection / Transformation

Equivalent to SQL `SELECT`

---

## Example

```js id="cwk0je"
db.orders.aggregate([
  {
    $project: {
      amount: 1,
      status: 1,
      tax: {
        $multiply: ["$amount", 0.18]
      }
    }
  }
]);
```

---

## Use Cases

* rename fields
* compute values
* hide fields
* transform output

---

# 4. `$sort`

Equivalent to SQL `ORDER BY`

---

## Example

```js id="d8szym"
db.orders.aggregate([
  {
    $sort: {
      amount: -1
    }
  }
]);
```

---

## Notes

* `1` = ascending
* `-1` = descending

---

## Performance Tip

👉 Sorting large collections without indexes is expensive.

---

# 5. `$limit`

Equivalent to SQL `LIMIT`

---

## Example

```js id="l89z1f"
db.orders.aggregate([
  {
    $sort: { amount: -1 }
  },
  {
    $limit: 5
  }
]);
```

---

## Common Use Cases

* top customers
* latest orders
* pagination

---

# 6. `$skip`

Used for pagination.

---

## Example

```js id="8k83b0"
db.orders.aggregate([
  {
    $skip: 20
  },
  {
    $limit: 10
  }
]);
```

---

## Problem

Large skips become slow.

---

## Better Approach

👉 Cursor-based pagination.

Example:

```js id="g3wh57"
created_at < last_seen_date
```

---

# 7. `$lookup` — JOIN Equivalent

MongoDB join stage.

---

## Example

```js id="6w7f0z"
db.orders.aggregate([
  {
    $lookup: {
      from: "users",
      localField: "user_id",
      foreignField: "_id",
      as: "user"
    }
  }
]);
```

---

## SQL Equivalent

```sql id="q7uh9u"
SELECT *
FROM orders
JOIN users
ON orders.user_id = users.id;
```

---

## Problems with `$lookup`

* expensive at scale
* memory-heavy
* avoid deep joins

---

## Best Practice

👉 MongoDB prefers denormalization.

---

# 8. `$unwind` — Array Expansion

Used for arrays.

---

## Input

```json id="9jlwmn"
{
  items: [
    { product: "A" },
    { product: "B" }
  ]
}
```

---

## Pipeline

```js id="njlwmf"
{
  $unwind: "$items"
}
```

---

## Output

```json id="13shg5"
{ item: "A" }
{ item: "B" }
```

---

## Use Cases

* order item analytics
* nested array filtering
* event streams

---

# 9. `$facet` — Multi Queries in One Pipeline

Runs parallel aggregations.

---

## Example

```js id="e79u0q"
db.orders.aggregate([
  {
    $facet: {
      revenue: [
        {
          $group: {
            _id: null,
            total: { $sum: "$amount" }
          }
        }
      ],
      topOrders: [
        {
          $sort: { amount: -1 }
        },
        {
          $limit: 5
        }
      ]
    }
  }
]);
```

---

## Use Cases

* dashboard APIs
* analytics pages
* multiple widgets from one query

---

# 10. `$bucket`

Groups values into ranges.

---

## Example

```js id="kvt3rt"
db.orders.aggregate([
  {
    $bucket: {
      groupBy: "$amount",
      boundaries: [0, 100, 500, 1000],
      default: "Other"
    }
  }
]);
```

---

## Use Cases

* salary ranges
* pricing analytics
* histogram reports

---

# 11. `$addFields`

Adds computed fields.

---

## Example

```js id="ihs4if"
{
  $addFields: {
    totalWithTax: {
      $multiply: ["$amount", 1.18]
    }
  }
}
```

---

# 12. `$count`

Counts documents.

---

## Example

```js id="z9ktwi"
db.orders.aggregate([
  {
    $match: { status: "PAID" }
  },
  {
    $count: "paidOrders"
  }
]);
```

---

# 13. `$merge`

Writes pipeline output into another collection.

---

## Example

```js id="vh7xvt"
db.orders.aggregate([
  {
    $group: {
      _id: "$user_id",
      total: { $sum: "$amount" }
    }
  },
  {
    $merge: "user_summary"
  }
]);
```

---

## Use Cases

* materialized views
* analytics caching
* precomputed dashboards

---

# 14. Aggregation Pipeline Optimization

---

# ✅ Correct Order

```text
$match
 ↓
$project
 ↓
$group
 ↓
$sort
 ↓
$limit
```

---

# ❌ Bad Pipeline

```text
$group first
then $match
```

Reason:

* groups unnecessary data
* huge memory usage

---

# 15. Aggregation Performance Problems

---

## Problem 1 — Large `$lookup`

```text
orders JOIN users JOIN products JOIN inventory
```

Very expensive.

---

## Problem 2 — Massive `$group`

Grouping millions of records:

* high RAM usage
* disk spilling

---

## Problem 3 — Unindexed `$match`

Causes:

```text
COLLSCAN
```

(collection scan)

---

# 16. Aggregation Explain Plan

---

## Example

```js id="tehnrf"
db.orders.aggregate([...]).explain("executionStats")
```

---

## Important Metrics

| Metric              | Meaning           |
| ------------------- | ----------------- |
| `COLLSCAN`          | bad (full scan)   |
| `IXSCAN`            | good (index scan) |
| `nReturned`         | rows returned     |
| `totalDocsExamined` | scanned docs      |

---

# A2. MongoDB Schema Design Patterns

MongoDB schema design is about:

👉 Embed vs Reference

This is the MOST important MongoDB design decision.

---

# 1. Embedded Documents

Store related data together.

---

## Example

```json id="pn6c5i"
{
  name: "John",
  addresses: [
    {
      city: "Bangalore"
    }
  ]
}
```

---

|  Pros | Cons |
--------|--------
| very fast reads |  document growth |
| single query fetch | duplication risk |
| fewer joins | update complexity |

---

## Best For

* profile settings
* small nested data
* tightly coupled data

---

# 2. Reference Pattern

Store relations separately.

---

## Example

```json id="sykgdx"
{
  order_id: 1,
  user_id: ObjectId("...")
}
```

---

| Pros              | Cons               |
| ----------------- | ------------------ |
| scalable          | requires `$lookup` |
| reusable data     | more queries       |
| smaller documents |                    |

---

## Best For

* ecommerce orders
* large reusable entities
* many-to-many relationships

---

# 3. Bucket Pattern

Group time-series data.

---

## Bad

```text
1 document per sensor reading
```

---

## Better

```json id="6zc3d6"
{
  sensor: "A",
  readings: [
    ...
  ]
}
```

---
| Pros             | Used For      |
| ---------------- | ------------- |
| smaller indexes  | IoT           |
| fewer documents  | metrics       |
| efficient writes | event streams |

---

# 4. Computed Pattern

Store precomputed values.

---

## Example

```json id="fpgw3y"
{
  user_id: 1,
  total_spent: 25000
}
```

---

| Pros | Cons |
|------|------|
| ultra-fast reads | consistency maintenance |

---

# 5. Polymorphic Pattern

Different document shapes in same collection.

---

## Example

```json id="3jkgvb"
{
  type: "video"
}

{
  type: "article"
}
```

---

# 6. Outlier Pattern

Move giant documents separately.

---

## Example

```text
normal posts → posts collection
viral post comments → separate collection
```

---

# A3. MongoDB Performance Tuning

---

# 1. Indexing Strategy

---

## Single Field Index

```js id="4v09tb"
db.users.createIndex({
  email: 1
});
```

---

## Compound Index

```js id="x4mf7z"
db.orders.createIndex({
  status: 1,
  created_at: -1
});
```

---

## Rule

Order matters.

```text
(status, created_at)
≠
(created_at, status)
```

---

# 2. Covered Queries

Query fully satisfied from index.

---

## Example

```js id="t74dk5"
db.users.find(
  { email: "a@b.com" },
  { email: 1 }
)
```

---

# 3. Avoid Large Documents

Mongo document limit:

```text
16 MB
```

---

## Problems

* slower updates
* relocation overhead
* memory waste

---

# 4. Replica Sets

Provides:

* failover
* redundancy
* transactions

---

## Structure

```text
Primary
  ↓
Secondary
  ↓
Secondary
```

---

# 5. Read Preferences

Choose read source.

---

## Example

```text
primary
secondary
nearest
```

---

# 6. Sharding

Horizontal scaling mechanism.

---

## Example

```text
Shard 1 → users A-M
Shard 2 → users N-Z
```

---

# 7. Choosing Good Shard Keys

---

## Good Shard Key

* high cardinality
* evenly distributed

---

## Bad Example

```text
status = active/inactive
```

Only few values → hotspot issue.

---

# 8. Slow Query Monitoring

---

## Enable profiling

```js id="6l2l2l"
db.setProfilingLevel(1)
```

---

## Analyze

```js id="9fuk17"
db.system.profile.find()
```

---

# 9. Connection Pooling

Avoid opening connections repeatedly.

---

## Node.js Example

```js id="2cq70h"
maxPoolSize: 50
```

---

# 10. Common Production Problems

---

## Over-indexing

Too many indexes:

* slower writes
* more RAM

---

## Large `$lookup`

MongoDB not optimized for relational-heavy systems.

---

## Jumbo Chunks

Bad shard key causes uneven shards.

---

## Replication Lag

Secondary nodes become stale.

---

# ✅ END OF SECTION A

Covered:

* aggregation pipelines
* schema patterns
* embed vs reference
* MongoDB scaling
* indexing
* sharding
* replica sets
* performance tuning
* production operational issues
