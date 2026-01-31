
## ✅ Updated Full SQL Script for PostgreSQL with Covering Index

```sql
CREATE TABLE employee(
id SERIAL PRIMARY KEY,
name varchar(30),
salary int 
)

select * from employee

INSERT INTO employee (name, salary)
SELECT 
  'Employee_' || g::text AS name,
  (1000 + floor(random() * 9000))::int AS salary  -- Random salary between 1000–9999
FROM generate_series(1, 5000000) AS g;

-- Normal Index 

create index emp_idx on employee(salary);
drop index emp_idx;

-- Covering Index 

create index c_index on employee(salary) INCLUDE(name);
drop index c_index;

Explain Analyze select name from employee where salary=308;





```

---

## 🧠 Why This Is a Covering Index

- The index is **built on `salary`** (used in the `WHERE` clause)
    
- It **includes `name`** (the column selected)
    
- This allows **PostgreSQL to use the index alone** (no table access), enabling **index-only scan** if conditions are met (e.g., visibility map is updated)
    

---

## 🔍 Want to Check Index-Only Scan Happened?

Look for this line in `EXPLAIN ANALYZE` output:

```
Index Only Scan using emp_salary_cover_idx on employee
```

That confirms PostgreSQL used the covering index **without touching the main table**.

---

Let me know if you want the **MySQL equivalent** or to **benchmark performance** with vs without the index.