### 🔄 Stored Procedures in PostgreSQL

In **PostgreSQL**, a **stored procedure** is a set of SQL statements saved in the database and executed as a single unit. They are typically used for:

- Reusable business logic
    
- Data manipulation
    
- Performance optimization
    
- Encapsulating complex operations
    

---

### 🧩 Difference Between Stored **Functions** and **Procedures**

| Feature             | **Function**             | **Procedure**                  |
| ------------------- | ------------------------ | ------------------------------ |
| Returns value?      | Yes                      | No (can return via OUT params) |
| Call syntax         | `SELECT function_name()` | `CALL procedure_name()`        |
| Transaction control | ❌ Not allowed            | ✅ Allowed                      |

PostgreSQL added **stored procedures** in **version 11** — before that, only functions were available.

---

### 🛠️ How to Create a Stored Procedure

#### ✅ Basic Syntax:

```sql
CREATE PROCEDURE procedure_name(param1 type, param2 type)
LANGUAGE plpgsql
AS $$
BEGIN
  -- your SQL logic here
END;
$$;
```

---

### ✅ Example: Stored Procedure in PostgreSQL

#### 🔹 Create a procedure to update a user's name:

```sql
CREATE PROCEDURE update_user_name(user_id INT, new_name TEXT)
LANGUAGE plpgsql
AS $$
BEGIN
  UPDATE users
  SET name = new_name
  WHERE id = user_id;
END;
$$;
```

#### 🔹 Call the procedure:

```sql
CALL update_user_name(1, 'John Doe');
```

---

### 🔁 Using OUT Parameters (to "return" data)

```sql
CREATE PROCEDURE get_user_email(IN user_id INT, OUT user_email TEXT)
LANGUAGE plpgsql
AS $$
BEGIN
  SELECT email INTO user_email
  FROM users
  WHERE id = user_id;
END;
$$;
```

#### Call and fetch result:

```sql
CALL get_user_email(1, email_output);
```

> Note: You usually call this inside a client that can capture `OUT` values.

---

### 📌 Key Notes

- You can **start/commit/rollback transactions** inside procedures (not allowed in functions).
    
- Use **`plpgsql`** for procedural logic (loops, conditionals, etc.).
    
- You can use **IN, OUT, INOUT** parameters.
    

---

### 🧪 Tools for Working with Procedures

- **pgAdmin**: GUI to write/test procedures
    
- **psql**: CLI
    
- ORMs like Sequelize don’t have great built-in support for calling procedures, but you can use raw queries:
    

```javascript
await sequelize.query('CALL update_user_name(:id, :name)', {
  replacements: { id: 1, name: 'Jane Doe' }
});
```

---

Would you like an example with conditional logic or loops in a stored procedure?