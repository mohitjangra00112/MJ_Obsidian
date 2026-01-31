
## **1️⃣ What is an Index?**

Think of an **index** as a **separate data structure** (like a sorted list or a map) that PostgreSQL keeps **outside your table** to make searches faster — just like the index at the back of a book.

Without an index → PostgreSQL has to check **every row** in the table (**sequential scan**).  
With an index → PostgreSQL jumps directly to the matching rows.

---

## POSTGRES Example code BY MJ

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

create index emp_idx on employee(salary);

drop index emp_idx;

Explain Analyze select * from employee where salary=308;

```


## **2️⃣ How PostgreSQL Stores Indexes**

By default, PostgreSQL uses a **B-Tree** index (Balanced Tree):

- **Balanced** → All leaf nodes are at the same depth (so searches take the same time).
    
- **Tree** → Stores keys in sorted order.
    
- **Leaf Nodes** → Contain **pointers to actual table rows**.
    

📌 **Note:** Other index types exist: `GIN`, `GiST`, `BRIN`, `Hash`, but B-Tree is the default.

---

## **3️⃣ The Flow of Index Usage**

Let’s say we have:

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name TEXT,
    salary NUMERIC
);

CREATE INDEX idx_salary ON employees(salary);
```

---

### **Step 1: You run a query**

```sql
SELECT * FROM employees WHERE salary = 50000;
```

---

### **Step 2: Query Parser**

- PostgreSQL **parses** your SQL into a query tree.
    

---

### **Step 3: Planner / Optimizer**

- Checks **statistics** (stored in `pg_stats`) to see:
    
    - How many rows have `salary = 50000`?
        
    - Is using the index worth it or should it scan the whole table?
        
- Decides to use **index scan** or **sequential scan**.
    

---

### **Step 4: Index Scan**

- PostgreSQL **searches the B-Tree** for `50000`.
    
- B-Tree search is **O(log n)** (very fast even with millions of rows).
    

Structure inside the index:

```
[ 30000 ] -> pointer to rows
[ 40000 ] -> pointer to rows
[ 50000 ] -> pointer to rows
[ 60000 ] -> pointer to rows
```

---

### **Step 5: Fetch Rows from Table**

- Each matching entry in the index has a **TID** (tuple identifier) = `(block_number, row_number)` in the actual table file.
    
- PostgreSQL **jumps** directly to those rows in the table and fetches them.
    

📌 This is why indexes speed up filtering — we skip reading irrelevant rows.

---

## **4️⃣ The Role of a Covering Index**

Normally, after finding a match in the index, PostgreSQL **still has to go to the table** to get the other column values.

**Covering Index** → We store **extra columns** in the index itself so PostgreSQL doesn’t need to read the table.

Example:

```sql
CREATE INDEX idx_salary_name ON employees(salary) INCLUDE(name);
```

Now:

- If you query `salary` and `name`, PostgreSQL **never touches the table** — it reads directly from the index (**Index Only Scan**).
    

---

## **5️⃣ When PostgreSQL Decides NOT to Use Index**

- If table is **small**, sequential scan might be faster.
    
- If `WHERE` condition matches **most rows** (low selectivity), scanning the index is useless.
    
- If `ANALYZE` stats are outdated.
    

---

## **6️⃣ Flow Summary**

**Query execution with index:**

```
You → SQL → Parser → Planner → Chooses Index Scan → 
Search B-Tree → Get TIDs → Fetch Table Rows → Return Result
```

**Query execution without index:**

```
You → SQL → Parser → Planner → Sequential Scan → Read All Rows → Return Result
```

---

Alright Mohit, let’s go step-by-step and actually **create an example of a B-Tree index for a salary column** in PostgreSQL, so you understand both the **concept** and **flow** from beginner level.

---

## **1️⃣ Scenario Setup**

Suppose you have an `employees` table:

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name TEXT,
    salary INT
);
```

Let’s insert some sample data:

```sql
INSERT INTO employees (name, salary) VALUES
('Aman', 35000),
('Bhavna', 42000),
('Chirag', 25000),
('Deepak', 55000),
('Esha', 30000),
('Farhan', 70000),
('Geeta', 45000);
```

---

## **2️⃣ Creating a B-Tree Index**

```sql
CREATE INDEX idx_salary ON employees (salary);
```

📌 This creates a **B-Tree index** (default type in PostgreSQL).

---

## **3️⃣ How the B-Tree is Built Internally**

B-Tree stores **sorted keys** (here: `salary`) with **pointers** to the actual table rows.

Example structure (simplified):

```
         [42000]
        /      \
 [30000]       [55000]
 /    \        /    \
25000 35000  45000  70000
```

**Explanation:**

- **Internal Nodes** hold key values for navigation.
    
- **Leaf Nodes** hold:
    
    - The `salary` value
        
    - A **row pointer** (TID) to the actual row in the table.
        

---

## **4️⃣ How PostgreSQL Uses It**

If you run:

```sql
EXPLAIN SELECT * FROM employees WHERE salary = 45000;
```

Flow:

1. PostgreSQL **checks the index** (`idx_salary`) instead of scanning the whole table.
    
2. In the B-Tree, it starts from the root → follows branches → reaches leaf where `salary=45000`.
    
3. Uses **row pointer (TID)** to directly fetch that row from the table (heap).
    
4. Returns the result.
    

---

## **5️⃣ Benefit**

Without index → PostgreSQL checks **all rows** (sequential scan).  
With index → It jumps **directly** to matching rows (logarithmic time complexity `O(log n)`).

---

## **6️⃣ Visualization of Flow**

Query: `salary = 45000`

1. Start at root node: `[42000]`
    
    - `45000` is greater → go right branch.
        
2. Node: `[55000]`
    
    - `45000` is less → go left branch.
        
3. Leaf node: `45000 → Row ID`
    
4. Jump to the table row via TID.
    

---

Okay Mohit, let’s break it down step by step — so we compare **search complexity** for:

1. **Without index** (full table scan)
    
2. **With B-Tree index** on `salary`
    

---

## **1️⃣ Without Index (Full Table Scan)**

- Imagine you have a table `employees` with **N** rows.
    
- If you run:
    
    ```sql
    SELECT * FROM employees WHERE salary = 60000;
    ```
    
- **PostgreSQL** will check **each row one by one**.
    
- **Time Complexity** → **O(N)** (linear search)
    
- Example:
    
    - If you have **1 million** rows, PostgreSQL will compare **salary** for all **1 million** rows before finding matches.
        

---

## **2️⃣ With B-Tree Index on `salary`**

We create:

```sql
CREATE INDEX idx_salary ON employees(salary);
```

### **How B-Tree Search Works**

1. **B-Tree** is a balanced tree structure.
    
2. Data is **sorted** by `salary` in index pages.
    
3. PostgreSQL:
    
    - Starts at **root node**.
        
    - Compares target `salary` with node’s keys.
        
    - Moves **left** or **right** child node.
        
4. When it reaches **leaf node**, it finds the exact row pointers.
    
5. Then it fetches those rows from the table (heap).
    

### **Complexity**

- Searching in a B-Tree → **O(log N)**
    
- Example:
    
    - If you have **1 million** rows:
        
        - **Without index:** ~1,000,000 checks.
            
        - **With index:** ~log₂(1,000,000) ≈ **20 checks** to find matches.
            

---

## **Quick Comparison Table**

|Method|Complexity|Example (1M rows)|
|---|---|---|
|**Full Scan**|O(N)|~1,000,000 checks|
|**B-Tree Index**|O(log N)|~20 checks|

---

If you want, I can **draw a small B-Tree for salary values** so you see the **search path visually**.  
It will make the concept crystal clear for you.