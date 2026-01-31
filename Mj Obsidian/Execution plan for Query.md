Alright — let’s go step-by-step on **query execution plans in PostgreSQL** and how they help you **optimize queries**.

---

## **1️⃣ What is an Execution Plan?**

An **execution plan** is PostgreSQL’s roadmap for how it will run your query:

- Which indexes it will use (if any)
    
- How it will scan tables (sequential scan, index scan, etc.)
    
- The estimated cost and row counts
    
- The order of operations (joins, filters, sorts)
    

We get it using:

```sql
EXPLAIN SELECT * FROM employees WHERE salary > 50000;
```

Or, to see actual runtime:

```sql
EXPLAIN ANALYZE SELECT * FROM employees WHERE salary > 50000;
```

---

## **2️⃣ How to Read an Execution Plan**

Example:

```sql
EXPLAIN SELECT * FROM employees WHERE salary > 50000;
```

Output:

```
Seq Scan on employees  (cost=0.00..18.10 rows=3 width=64)
  Filter: (salary > 50000)
```

- **Seq Scan** → PostgreSQL will scan all rows in the table.
    
- **cost=0.00..18.10** → Estimated startup cost and total cost.
    
- **rows=3** → Estimated number of matching rows.
    
- **width=64** → Estimated size of each row in bytes.
    
- **Filter** → Condition applied to rows.
    

---

## **3️⃣ Example with Index**

If we create an index:

```sql
CREATE INDEX idx_salary ON employees(salary);
```

Then:

```sql
EXPLAIN SELECT * FROM employees WHERE salary > 50000;
```

Might output:

```
Index Scan using idx_salary on employees  (cost=0.15..8.25 rows=3 width=64)
  Index Cond: (salary > 50000)
```

Now:

- **Index Scan** → Uses index instead of scanning the entire table.
    
- Lower cost → More efficient.
    

---

## **4️⃣ Common Scan Types**

|Scan Type|When It’s Used|Notes|
|---|---|---|
|**Seq Scan**|No useful index, small table, or condition not selective.|Scans every row.|
|**Index Scan**|Index exists for filter/join columns.|Efficient for selective queries.|
|**Index Only Scan**|Index has all needed columns.|Avoids accessing the table.|
|**Bitmap Index Scan**|Many rows match; combines index lookups into batches.|Efficient for medium selectivity.|

---

## **5️⃣ JOIN Strategies**

When joining tables:

- **Nested Loop** → Good for small sets.
    
- **Merge Join** → Good when both sides are sorted.
    
- **Hash Join** → Good for large, unsorted datasets.
    

Example:

```sql
EXPLAIN SELECT e.name, d.name
FROM employees e
JOIN departments d ON e.dept_id = d.id;
```

---

## **6️⃣ Tips for Query Optimization**

1. **Check Execution Plan** with `EXPLAIN ANALYZE`.
    
2. **Add Indexes** for frequent `WHERE`, `JOIN`, `ORDER BY` columns.
    
3. **Use Index Only Scans** by including all required columns in the index.
    
4. **Avoid Functions on Indexed Columns** in WHERE clauses:
    
    ```sql
    WHERE LOWER(name) = 'john' -- ❌ breaks index usage
    ```
    
5. **Analyze Table Statistics**:
    
    ```sql
    ANALYZE employees;
    ```
    
6. **Reduce Row Counts Early** — filter before joining if possible.
    
7. **Use Proper Data Types** — mismatches can prevent index usage.
    
8. **Partition Large Tables** if data is huge and time-based.
    

---

Alright, let’s do a **hands-on query optimization example** in PostgreSQL.  
We’ll start with a slow query, check its execution plan, then optimize it step-by-step.

---

## **1️⃣ Initial Setup**

```sql
-- Employees table
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name TEXT,
    dept_id INT,
    salary NUMERIC
);

-- Departments table
CREATE TABLE departments (
    id SERIAL PRIMARY KEY,
    name TEXT
);

-- Insert 1 million employees
INSERT INTO employees (name, dept_id, salary)
SELECT 'Emp' || i, (i % 100) + 1, (50000 + (i % 1000) * 10)
FROM generate_series(1, 1000000) s(i);

-- Insert 100 departments
INSERT INTO departments (name)
SELECT 'Dept' || i FROM generate_series(1, 100) s(i);
```

---

## **2️⃣ Slow Query**

```sql
EXPLAIN ANALYZE
SELECT e.name, d.name, e.salary
FROM employees e
JOIN departments d ON e.dept_id = d.id
WHERE e.salary > 95000
ORDER BY e.salary DESC
LIMIT 10;
```

**Possible output (slow):**

```
Limit  (cost=... rows=10 width=...)
  ->  Sort  (cost=... rows=...)
        Sort Key: e.salary DESC
        ->  Hash Join
              Hash Cond: (e.dept_id = d.id)
              ->  Seq Scan on employees e
                    Filter: (salary > 95000)
              ->  Seq Scan on departments d
```

**Problem:**

- Sequential scan on `employees` — scanning **all 1 million rows**.
    
- Sorting large result set instead of using index.
    

---

## **3️⃣ Optimization Step 1 — Add Index on `salary`**

```sql
CREATE INDEX idx_salary ON employees(salary DESC);
```

Run again:

```sql
EXPLAIN ANALYZE
SELECT e.name, d.name, e.salary
FROM employees e
JOIN departments d ON e.dept_id = d.id
WHERE e.salary > 95000
ORDER BY e.salary DESC
LIMIT 10;
```

Now PostgreSQL may do:

```
Limit
  ->  Nested Loop
        ->  Index Scan using idx_salary on employees e
              Index Cond: (salary > 95000)
        ->  Index Scan using departments_pkey on departments d
```

✅ Uses `Index Scan` — avoids scanning all rows, fetches in sorted order.

---

## **4️⃣ Optimization Step 2 — Covering Index**

If we want an **Index Only Scan** (even faster), create:

```sql
CREATE INDEX idx_salary_cover ON employees(salary DESC, name, dept_id);
```

Now:

```
Index Only Scan using idx_salary_cover on employees e
```

✅ No need to fetch extra columns from the table — everything is in the index.

---

## **5️⃣ Optimization Step 3 — Analyze**

After bulk inserts, run:

```sql
ANALYZE employees;
ANALYZE departments;
```

✅ Updates PostgreSQL statistics so the planner can choose the best execution plan.

---

## **Result**

- **Before:** Sequential scan → hash join → sort (very slow on large data).
    
- **After:** Index scan → nested loop (fast, uses indexes) → no large sort needed.
    

---

If you want, I can **add a second example where the query uses a `WHERE LOWER(name)` condition** and show you how to optimize it so the index is still used.  
That’s a common mistake people make.