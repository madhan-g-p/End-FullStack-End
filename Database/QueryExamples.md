
---

#  PHASE 3 — PART 1: 30 SQL QUERY PATTERNS

We’ll use 3 datasets:

---

##  EMPLOYEE TABLE

```sql
employee(id, name, dept, salary, manager_id, join_date)
```

---

##  ECOMMERCE TABLES

```sql
users(id, name)
orders(id, user_id, amount, status, created_at)
order_items(order_id, product_id, qty, price)
products(id, name, category, price)
```

---

##  SALES TABLE

```sql
sales(id, region, product, amount, sale_date)
```

---

#  A. EMPLOYEE QUERY PATTERNS (1–10)

---

## 1. Second highest salary

```sql id="q1"
SELECT MAX(salary)
FROM employee
WHERE salary < (SELECT MAX(salary) FROM employee);
```

---

## 2. Top 3 salaries

```sql id="q2"
SELECT DISTINCT salary
FROM employee
ORDER BY salary DESC
LIMIT 3;
```

---

## 3. Department-wise max salary

```sql id="q3"
SELECT dept, MAX(salary)
FROM employee
GROUP BY dept;
```

---

## 4. Employees earning above average

```sql id="q4"
SELECT *
FROM employee
WHERE salary > (SELECT AVG(salary) FROM employee);
```

---

## 5. Manager–employee hierarchy (self join)

```sql id="q5"
SELECT e.name, m.name AS manager
FROM employee e
LEFT JOIN employee m ON e.manager_id = m.id;
```

---

## 6. Employees with same salary

```sql id="q6"
SELECT salary, COUNT(*)
FROM employee
GROUP BY salary
HAVING COUNT(*) > 1;
```

---

## 7. Latest joiners

```sql id="q7"
SELECT *
FROM employee
ORDER BY join_date DESC
LIMIT 5;
```

---

## 8. Department with highest average salary

```sql id="q8"
SELECT dept
FROM employee
GROUP BY dept
ORDER BY AVG(salary) DESC
LIMIT 1;
```

---

## 9. Rank employees by salary

```sql id="q9"
SELECT name, salary,
RANK() OVER (ORDER BY salary DESC)
FROM employee;
```

---

## 10. Employees without manager

```sql id="q10"
SELECT * FROM employee
WHERE manager_id IS NULL;
```

---

# 🛒 B. ECOMMERCE QUERIES (11–20)

---

## 11. Total revenue

```sql id="q11"
SELECT SUM(amount) FROM orders;
```

---

## 12. Revenue per user

```sql id="q12"
SELECT user_id, SUM(amount)
FROM orders
GROUP BY user_id;
```

---

## 13. Top 5 customers

```sql id="q13"
SELECT user_id, SUM(amount) AS total
FROM orders
GROUP BY user_id
ORDER BY total DESC
LIMIT 5;
```

---

## 14. Orders with no items (data issue detection)

```sql id="q14"
SELECT o.id
FROM orders o
LEFT JOIN order_items i ON o.id = i.order_id
WHERE i.order_id IS NULL;
```

---

## 15. Most sold product

```sql id="q15"
SELECT product_id, SUM(qty)
FROM order_items
GROUP BY product_id
ORDER BY SUM(qty) DESC
LIMIT 1;
```

---

## 16. Cart abandonment simulation

```sql id="q16"
SELECT *
FROM orders
WHERE status = 'PENDING';
```

---

## 17. Average order value

```sql id="q17"
SELECT AVG(amount) FROM orders;
```

---

## 18. Daily revenue

```sql id="q18"
SELECT DATE(created_at), SUM(amount)
FROM orders
GROUP BY DATE(created_at);
```

---

## 19. Users with no orders

```sql id="q19"
SELECT u.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.user_id IS NULL;
```

---

## 20. Product revenue

```sql id="q20"
SELECT product_id, SUM(price * qty)
FROM order_items
GROUP BY product_id;
```

---

#  C. SALES QUERIES (21–30)

---

## 21. Total sales by region

```sql id="q21"
SELECT region, SUM(amount)
FROM sales
GROUP BY region;
```

---

## 22. Best performing region

```sql id="q22"
SELECT region
FROM sales
GROUP BY region
ORDER BY SUM(amount) DESC
LIMIT 1;
```

---

## 23. Monthly sales trend

```sql id="q23"
SELECT DATE_FORMAT(sale_date, '%Y-%m'), SUM(amount)
FROM sales
GROUP BY DATE_FORMAT(sale_date, '%Y-%m');
```

---

## 24. Top product by revenue

```sql id="q24"
SELECT product, SUM(amount)
FROM sales
GROUP BY product
ORDER BY SUM(amount) DESC
LIMIT 1;
```

---

## 25. Sales above average

```sql id="q25"
SELECT *
FROM sales
WHERE amount > (SELECT AVG(amount) FROM sales);
```

---

## 26. Cumulative sales (window function)

```sql id="q26"
SELECT sale_date, amount,
SUM(amount) OVER (ORDER BY sale_date)
FROM sales;
```

---

## 27. Running average sales

```sql id="q27"
SELECT sale_date,
AVG(amount) OVER (ORDER BY sale_date)
FROM sales;
```

---

## 28. Region-wise ranking

```sql id="q28"
SELECT region, amount,
RANK() OVER (PARTITION BY region ORDER BY amount DESC)
FROM sales;
```

---

## 29. Duplicate detection

```sql id="q29"
SELECT product, sale_date, COUNT(*)
FROM sales
GROUP BY product, sale_date
HAVING COUNT(*) > 1;
```

---

## 30. Peak sales day

```sql id="q30"
SELECT sale_date
FROM sales
GROUP BY sale_date
ORDER BY SUM(amount) DESC
LIMIT 1;
```

---

#  PHASE 3 — PART 2: MONGODB AGGREGATION MASTERY

---

## 1. Total sales per user

```js id="m1"
db.orders.aggregate([
  { $group: { _id: "$user_id", total: { $sum: "$amount" } } }
]);
```

---

## 2. Top customers

```js id="m2"
db.orders.aggregate([
  { $group: { _id: "$user_id", total: { $sum: "$amount" } } },
  { $sort: { total: -1 } },
  { $limit: 5 }
]);
```

---

## 3. Monthly revenue

```js id="m3"
db.orders.aggregate([
  { $group: {
      _id: { $month: "$created_at" },
      revenue: { $sum: "$amount" }
  }}
]);
```

---

## 4. Lookup (JOIN equivalent)

```js id="m4"
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

## 5. Filter + group pipeline

```js id="m5"
db.orders.aggregate([
  { $match: { status: "PAID" } },
  { $group: { _id: "$user_id", total: { $sum: "$amount" } } }
]);
```

---

#  WHAT YOU NOW HAVE (PHASE 3 OUTPUT)

You now understand:

### SQL mastery:

* 30 real query patterns
* joins, ranking, aggregation
* ecommerce + sales + employee logic

### Mongo mastery:

* aggregation pipelines
* lookup joins
* grouping + filtering pipelines

---
