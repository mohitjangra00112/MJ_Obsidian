Here’s a clear and concise set of **Node.js + PostgreSQL** notes to help you get started or quickly reference how to connect, query, and manage PostgreSQL databases using Node.js.

---

## 🗒️ Node.js + PostgreSQL Notes

### 🔧 Requirements

- **Node.js** installed
    
- **PostgreSQL** installed
    
- Node packages:
    
    - `pg` (node-postgres)
        
    - (Optional: `dotenv` for environment variables)
        

```bash
npm install pg dotenv
```

---

### 📁 Project Structure Example

```
my-app/
├── db.js           # DB connection setup
├── index.js        # Main app file
├── .env            # Environment variables
└── package.json
```

---

### 🌐 .env Example

```env
PGHOST=localhost
PGUSER=your_db_user
PGPASSWORD=your_db_password
PGDATABASE=your_db_name
PGPORT=5432
```

---

### 🧩 db.js – PostgreSQL Connection

```js
const { Pool } = require('pg');
require('dotenv').config();

const pool = new Pool({
  host: process.env.PGHOST,
  user: process.env.PGUSER,
  password: process.env.PGPASSWORD,
  database: process.env.PGDATABASE,
  port: process.env.PGPORT,
});

module.exports = pool;
```

---

### 🚀 index.js – Example Queries

```js
const pool = require('./db');

// GET all users
const getUsers = async () => {
  try {
    const res = await pool.query('SELECT * FROM users');
    console.log(res.rows);
  } catch (err) {
    console.error(err.message);
  }
};

// INSERT a user
const addUser = async (name, email) => {
  try {
    const res = await pool.query(
      'INSERT INTO users(name, email) VALUES($1, $2) RETURNING *',
      [name, email]
    );
    console.log(res.rows[0]);
  } catch (err) {
    console.error(err.message);
  }
};

getUsers();
addUser('Alice', 'alice@example.com');
```

---

### 🔄 Common SQL Queries (with `pg`)

```js
// SELECT
const res = await pool.query('SELECT * FROM table_name');

// INSERT
const res = await pool.query('INSERT INTO table(col1, col2) VALUES($1, $2)', [val1, val2]);

// UPDATE
const res = await pool.query('UPDATE table SET col1 = $1 WHERE id = $2', [val1, id]);

// DELETE
const res = await pool.query('DELETE FROM table WHERE id = $1', [id]);
```

---

### 🔒 Prevent SQL Injection

Use **parameterized queries** with `$1`, `$2`, etc. like this:

```js
pool.query('SELECT * FROM users WHERE email = $1', [email]);
```

---

### 📦 Transactions Example

```js
const runTransaction = async () => {
  const client = await pool.connect();
  try {
    await client.query('BEGIN');
    await client.query('UPDATE accounts SET balance = balance - 100 WHERE id = 1');
    await client.query('UPDATE accounts SET balance = balance + 100 WHERE id = 2');
    await client.query('COMMIT');
  } catch (err) {
    await client.query('ROLLBACK');
    console.error('Transaction failed:', err.message);
  } finally {
    client.release();
  }
};
```

---

### 🧪 Testing the Connection

```js
pool.connect()
  .then(() => console.log('Connected to PostgreSQL'))
  .catch(err => console.error('Connection error', err.stack));
```

---

### 📚 Additional Notes

- `pg.Pool` is preferred for handling multiple connections efficiently.
    
- Always use `async/await` or `.then()` to handle promises.
    
- Use `try/catch` for error handling.
    
- Keep your `.env` file out of version control (`.gitignore` it).
    

---

Would you like this exported as a **Markdown file**, **PDF**, or something else?