In **PostgreSQL**, a **trigger** is a database object that automatically executes (or “fires”) a specified function **in response to certain events** (INSERT, UPDATE, DELETE, TRUNCATE) on a table or view.

---

## **Why Use Triggers?**

- Automatically enforce rules or validations.
    
- Keep audit logs (e.g., who updated what and when).
    
- Maintain derived or summary data.
    
- Prevent invalid data changes.
    

---

## **How Triggers Work**

A trigger is **linked to**:

1. **A table or view** — where the event happens.
    
2. **An event type** — INSERT, UPDATE, DELETE, or TRUNCATE.
    
3. **A timing** — BEFORE, AFTER, or INSTEAD OF the event.
    
4. **A function** — the trigger calls a function (written in PL/pgSQL, SQL, etc.) to perform actions.
    

---

### **Trigger Execution Flow**

1. **Event happens** (e.g., row is inserted).
    
2. PostgreSQL **detects** the trigger condition.
    
3. The trigger’s **function is executed**.
    
4. Depending on BEFORE/AFTER:
    
    - **BEFORE trigger** → can modify or stop the operation.
        
    - **AFTER trigger** → happens after data change is committed.
        
    - **INSTEAD OF trigger** → replaces the operation (mostly for views).
        

---

## **Basic Syntax**

We need two things:

1. **Trigger Function** (must return `TRIGGER` type)
    
2. **Trigger Definition** (links function to table & event)
    

---

### **Example — Logging Changes**

```sql
-- 1️⃣ Create a table to log updates
CREATE TABLE employee_audit (
    id SERIAL PRIMARY KEY,
    emp_id INT,
    old_salary NUMERIC,
    new_salary NUMERIC,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2️⃣ Create a trigger function
CREATE OR REPLACE FUNCTION log_salary_changes()
RETURNS TRIGGER AS $$
BEGIN
    -- Insert old and new values into audit table
    INSERT INTO employee_audit(emp_id, old_salary, new_salary)
    VALUES (OLD.id, OLD.salary, NEW.salary);

    RETURN NEW; -- continue with the update
END;
$$ LANGUAGE plpgsql;

-- 3️⃣ Create the trigger
CREATE TRIGGER salary_change_trigger
AFTER UPDATE OF salary ON employees
FOR EACH ROW
WHEN (OLD.salary IS DISTINCT FROM NEW.salary) -- only if salary changes
EXECUTE FUNCTION log_salary_changes();
```

---

## **Key Points**

- **FOR EACH ROW** → fires once for every row affected.
    
- **FOR EACH STATEMENT** → fires once per SQL statement, no matter how many rows are changed.
    
- **WHEN condition** → optional filter for triggering.
    
- Can have **multiple triggers** on the same table for different events.
    

---

## **Common Use Cases**

- **Auditing:** Track changes in another table.
    
- **Validation:** Prevent invalid updates or deletes.
    
- **Cascading actions:** Automatically insert/update related data.
    
- **Soft deletes:** Mark a row as deleted instead of actually deleting.
    

---

Here’s your complete PostgreSQL trigger example, including the **main `employees` table** so the trigger has something to work on:

---

```sql
-- 0️⃣ Create main employees table
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    salary NUMERIC
);

-- Add some sample data
INSERT INTO employees (name, salary) VALUES
('John Doe', 50000),
('Jane Smith', 60000);

-- 1️⃣ Create a table to log updates
CREATE TABLE employee_audit (
    id SERIAL PRIMARY KEY,
    emp_id INT,
    old_salary NUMERIC,
    new_salary NUMERIC,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2️⃣ Create a trigger function
CREATE OR REPLACE FUNCTION log_salary_changes()
RETURNS TRIGGER AS $$
BEGIN
    -- Insert old and new values into audit table
    INSERT INTO employee_audit(emp_id, old_salary, new_salary)
    VALUES (OLD.id, OLD.salary, NEW.salary);

    RETURN NEW; -- continue with the update
END;
$$ LANGUAGE plpgsql;

-- 3️⃣ Create the trigger
CREATE TRIGGER salary_change_trigger
AFTER UPDATE OF salary ON employees
FOR EACH ROW
WHEN (OLD.salary IS DISTINCT FROM NEW.salary) -- only if salary changes
EXECUTE FUNCTION log_salary_changes();

-- 4️⃣ Test the trigger
UPDATE employees
SET salary = 55000
WHERE id = 1;

-- 5️⃣ See the audit log
SELECT * FROM employee_audit;
```

---
In PostgreSQL **triggers**, an **event** is the type of action on a table (or view) that causes the trigger to fire.

---

## **Main Trigger Events in PostgreSQL**

|Event|Description|
|---|---|
|**INSERT**|Fires when a new row is inserted into the table.|
|**UPDATE**|Fires when an existing row is updated (can be restricted to specific columns).|
|**DELETE**|Fires when a row is deleted from the table.|
|**TRUNCATE**|Fires when all rows in the table are removed at once.|

---

### **Examples**

#### **1. INSERT Trigger**

```sql
CREATE TRIGGER after_insert_employee
AFTER INSERT ON employees
FOR EACH ROW
EXECUTE FUNCTION some_function();
```

Runs after a new employee row is inserted.

---

#### **2. UPDATE Trigger (any column)**

```sql
CREATE TRIGGER after_update_employee
AFTER UPDATE ON employees
FOR EACH ROW
EXECUTE FUNCTION some_function();
```

---

#### **3. UPDATE Trigger (specific column)**

```sql
CREATE TRIGGER after_salary_update
AFTER UPDATE OF salary ON employees
FOR EACH ROW
EXECUTE FUNCTION log_salary_changes();
```

Fires **only** when `salary` changes.

---

#### **4. DELETE Trigger**

```sql
CREATE TRIGGER after_delete_employee
AFTER DELETE ON employees
FOR EACH ROW
EXECUTE FUNCTION some_function();
```

---

#### **5. TRUNCATE Trigger**

```sql
CREATE TRIGGER after_truncate_employee
AFTER TRUNCATE ON employees
FOR EACH STATEMENT
EXECUTE FUNCTION some_function();
```

Runs after the table is emptied with `TRUNCATE`.

---

### **Extra: WHEN Clause**

You can filter an event with `WHEN`:

```sql
CREATE TRIGGER salary_increase_only
AFTER UPDATE ON employees
FOR EACH ROW
WHEN (NEW.salary > OLD.salary)
EXECUTE FUNCTION log_salary_changes();
```

This will only fire if salary **increases**, not decreases.

---

