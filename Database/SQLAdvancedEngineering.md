# 💽 SECTION B — ADVANCED SQL ENGINEERING

SQL becomes truly powerful when you move beyond:

* basic CRUD
* simple joins
* single-table queries

This section focuses on:

* joins
* ranking
* window functions
* analytical querying
* performance behavior
* production query engineering

---

# B1. SQL JOIN MASTERY

Joins are the foundation of relational databases.

They allow data spread across multiple tables to be queried together.

---

# 🧱 Example Dataset

---

## users

```sql id="u9g8d2"
users(
  id,
  name,
  city
)
```

---

## orders

```sql id="7jy8k1"
orders(
  id,
  user_id,
  amount,
  status
)
```

---

# 1. INNER JOIN

Returns:
👉 only matching records from both tables

---

## Query

```sql id="s9dk21"
SELECT users.name, orders.amount
FROM users
INNER JOIN orders
ON users.id = orders.user_id;
```

---

## Output Concept

```text id="1t5d3p"
Only users having orders
```

---

## Use Cases

* users with purchases
* students with enrollments
* employees with departments

---

| Pros                             | Cons                                     |
| -------------------------------- | ---------------------------------------- |
| fast                             | unmatched records disappear              |
| simple                           | may unintentionally lose data visibility |
| most common join                 |                                          |
| optimized heavily by SQL engines |                                          |

---

# 2. LEFT JOIN

Returns:

* all left table rows
* matching right rows
* NULL if no match

---

## Query

```sql id="k7f2m0"
SELECT users.name, orders.amount
FROM users
LEFT JOIN orders
ON users.id = orders.user_id;
```

---

## Output Concept

```text id="p0w4ex"
All users shown
Even users without orders
```

---

## Use Cases

* finding inactive users
* optional relationships
* audit reports

---

## Example — users without orders

```sql id="r4vn2q"
SELECT users.*
FROM users
LEFT JOIN orders
ON users.id = orders.user_id
WHERE orders.id IS NULL;
```

---

| Pros                       | Cons                            |
| -------------------------- | ------------------------------- |
| preserves all primary data | can produce many NULL rows      |
| excellent for reporting    | larger datasets than INNER JOIN |

---

# 3. RIGHT JOIN

Opposite of LEFT JOIN.

---

## Query

```sql id="h9c5xa"
SELECT users.name, orders.amount
FROM users
RIGHT JOIN orders
ON users.id = orders.user_id;
```

---

## Reality

Rarely used in production.

Most teams rewrite as LEFT JOIN for readability.

---

# 4. FULL OUTER JOIN

Returns:

* all rows from both tables
* matched where possible

---

## Query

```sql id="h7rz8q"
SELECT *
FROM users
FULL OUTER JOIN orders
ON users.id = orders.user_id;
```

---

## Use Cases

* reconciliation systems
* migration validation
* audit comparisons

---

| Pros | Cons |
|------| ----- |
| complete visibility | expensive |
| | huge result sets|
| |not supported in MySQL directly|

---

# 5. CROSS JOIN

Creates cartesian product.

---

## Query

```sql id="x4v3pq"
SELECT *
FROM users
CROSS JOIN products;
```

---

## Example

```text id="e3s8gi"
100 users × 100 products
=
10,000 rows
```

---

## Use Cases

* matrix generation
* combinations
* scheduling systems

---

## Danger

Accidental CROSS JOINs can crash databases.

---

# 6. SELF JOIN

Table joined with itself.

---

# Employee Hierarchy Example

```sql id="7x9h1c"
employee(
  id,
  name,
  manager_id
)
```

---

## Query

```sql id="9qv8xm"
SELECT e.name AS employee,
       m.name AS manager
FROM employee e
LEFT JOIN employee m
ON e.manager_id = m.id;
```

---

## Use Cases

* organization trees
* referral systems
* category hierarchies

---

# 7. Multi-table JOIN

Production systems often join:

* users
* orders
* products
* payments

---

## Example

```sql id="q3k7t2"
SELECT u.name,
       o.id,
       p.name
FROM users u
JOIN orders o ON u.id = o.user_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.id;
```

---

# ⚠️ JOIN Performance Reality

---

## 1. Too Many Joins

```text id="dxv5u9"
7–10 joins
```

Usually indicates:

* over-normalization
* poor schema design

---

## Problems

* slow execution
* memory pressure
* massive intermediate datasets

---

# 8. JOIN Algorithms (Internal DB Behavior)

---

# A. Nested Loop Join

DB loops through rows repeatedly.

---

## Good For

* small datasets
* indexed lookups

---

## Bad For

* large tables

---

# B. Hash Join

Creates in-memory hash table.

---

| Pros | Cons |
|------|------|
| large joins | memory-heavy|
| equality conditions | |

---

# C. Merge Join

Requires sorted inputs.

---

## Very Fast For

* already indexed/sorted datasets

---

# 9. JOIN Optimization Strategies

---

# ✅ Index JOIN Columns

```sql id="0eqi0s"
CREATE INDEX idx_orders_user
ON orders(user_id);
```

---

# ✅ Filter Early

Bad:

```sql id="yfc5f5"
JOIN huge tables
then WHERE filter
```

Better:

```sql id="d5c0s6"
filter first
then join
```

---

# ✅ Avoid SELECT *

Bad:

```sql id="9l0j06"
SELECT *
```

Reason:

* unnecessary data transfer
* larger memory usage

---

# B2. SQL RANKING FUNCTIONS

Ranking functions are part of:
👉 Window Functions

They are heavily used in:

* analytics
* dashboards
* leaderboards
* reporting systems

---

# Dataset

```sql id="mev7r5"
employee(
  id,
  name,
  dept,
  salary
)
```

---

# 1. RANK()

Assigns ranking with gaps.

---

## Query

```sql id="19qxt0"
SELECT name,
       salary,
       RANK() OVER (ORDER BY salary DESC)
FROM employee;
```

---

## Example Output

```text id="03f8ma"
10000 → Rank 1
9000  → Rank 2
9000  → Rank 2
8000  → Rank 4
```

---

## Use Cases

* competition rankings
* sales rankings

---

| Pros | Cons |
|------|------|
| handles ties naturally | gaps in ranking |

---

# 2. DENSE_RANK()

No gaps in ranking.

---

## Query

```sql id="0ayl6t"
SELECT name,
       salary,
       DENSE_RANK() OVER (ORDER BY salary DESC)
FROM employee;
```

---

## Output

```text id="z3jzn4"
10000 → 1
9000  → 2
9000  → 2
8000  → 3
```

---

## Best For

* leaderboard UI
* grouped rankings

---

# 3. ROW_NUMBER()

Unique row numbering.

---

## Query

```sql id="nq4yn6"
SELECT name,
       ROW_NUMBER() OVER (ORDER BY salary DESC)
FROM employee;
```

---

## Output

```text id="2n5qj8"
10000 → 1
9000  → 2
9000  → 3
```

---

## Use Cases

* pagination
* deduplication
* latest row selection

---

# 4. NTILE()

Divides rows into groups.

---

## Query

```sql id="lm7go5"
SELECT name,
       salary,
       NTILE(4) OVER (ORDER BY salary DESC)
FROM employee;
```

---

## Use Cases

* percentile analysis
* quartiles
* grading systems

---

# B3. WINDOW FUNCTIONS MASTERY

Window functions are among the most powerful SQL features.

Unlike GROUP BY:
👉 window functions do NOT collapse rows.

They calculate values while preserving original rows.

---

# 1. OVER()

Defines window scope.

---

## Example

```sql id="l7m0zq"
SELECT name,
       salary,
       AVG(salary) OVER()
FROM employee;
```

---

## Output

```text id="k76cx7"
Every row shows:
employee salary + overall average
```

---

# Difference vs GROUP BY

GROUP BY:

```text id="i0ylvl"
collapses rows
```

Window Functions:

```text id="8vqv75"
preserves rows
```

---

# 2. PARTITION BY

Creates groups inside windows.

---

## Query

```sql id="jqq1w2"
SELECT name,
       dept,
       salary,
       AVG(salary)
       OVER(PARTITION BY dept)
FROM employee;
```

---

## Meaning

```text id="5u7c2r"
Average salary per department
while preserving all rows
```

---

# 3. Running Totals

Very common production pattern.

---

## Query

```sql id="ngm8uh"
SELECT sale_date,
       amount,
       SUM(amount)
       OVER(ORDER BY sale_date)
FROM sales;
```

---

## Use Cases

* cumulative revenue
* wallet balance history
* analytics dashboards

---

# 4. Moving Average

---

## Query

```sql id="l7zj1f"
SELECT sale_date,
       AVG(amount)
       OVER(
         ORDER BY sale_date
         ROWS BETWEEN 2 PRECEDING
         AND CURRENT ROW
       )
FROM sales;
```

---

## Use Cases

* stock trends
* forecasting
* smoothing noisy data

---

# 5. LAG()

Access previous row.

---

## Query

```sql id="6v6n3u"
SELECT sale_date,
       amount,
       LAG(amount)
       OVER(ORDER BY sale_date)
FROM sales;
```

---

## Use Cases

* growth calculations
* previous transaction comparison

---

# 6. LEAD()

Access next row.

---

## Query

```sql id="4z9xpa"
SELECT sale_date,
       amount,
       LEAD(amount)
       OVER(ORDER BY sale_date)
FROM sales;
```

---

# 7. FIRST_VALUE()

Gets first value in partition.

---

## Query

```sql id="v0c1ti"
SELECT dept,
       name,
       FIRST_VALUE(name)
       OVER(
         PARTITION BY dept
         ORDER BY salary DESC
       )
FROM employee;
```

---

# 8. LAST_VALUE()

Gets last value.

---

# ⚠️ Important

Needs proper frame definition.

---

# 9. Window Function Performance

---

## Problem

Window functions require:

* sorting
* partitioning
* memory

---

## Expensive Operations

```text id="0fokar"
ORDER BY inside OVER()
```

---

## Optimization Strategies

---

# ✅ Index partition/order columns

```sql id="zyhy7v"
CREATE INDEX idx_sales_date
ON sales(sale_date);
```

---

# ✅ Reduce dataset first

Bad:

```text id="y5z7wq"
window over entire 100M rows
```

Better:

```text id="ix1f8g"
filter first
then apply window function
```

---

# 10. Real Production Patterns

---

# A. Latest Record Per User

```sql id="sr4k0m"
SELECT *
FROM (
  SELECT *,
         ROW_NUMBER() OVER(
           PARTITION BY user_id
           ORDER BY created_at DESC
         ) rn
  FROM orders
) t
WHERE rn = 1;
```

---

# B. Top N Per Category

```sql id="u7xv4u"
SELECT *
FROM (
  SELECT *,
         RANK() OVER(
           PARTITION BY dept
           ORDER BY salary DESC
         ) r
  FROM employee
) t
WHERE r <= 3;
```

---

# C. Detect Revenue Growth

```sql id="rm72ut"
SELECT month,
       revenue,
       revenue -
       LAG(revenue)
       OVER(ORDER BY month)
FROM monthly_sales;
```

---

# ✅ END OF SECTION B

Covered:

* joins
* join internals
* ranking functions
* window functions
* analytical querying
* optimization patterns
* production SQL analytics
* performance considerations
* real-world usage patterns
