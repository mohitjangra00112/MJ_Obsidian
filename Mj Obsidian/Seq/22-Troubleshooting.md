# Troubleshooting Guide

## 🚨 Common Issues & Solutions

This guide covers the most common problems you'll encounter with PostgreSQL + Sequelize and how to solve them.

## 📋 Table of Contents

- [Connection Issues](#connection-issues)
- [Migration Problems](#migration-problems)
- [Model & Association Errors](#model--association-errors)
- [Query Issues](#query-issues)
- [Performance Problems](#performance-problems)
- [Validation Errors](#validation-errors)
- [Production Issues](#production-issues)
- [Development Workflow](#development-workflow)

## 🔌 Connection Issues

### 1. Connection Refused / ECONNREFUSED

**Error:**
```
SequelizeConnectionRefusedError: connect ECONNREFUSED 127.0.0.1:5432
```

**Solutions:**

```bash
# Check if PostgreSQL is running
# Windows
Get-Service postgresql*
Start-Service postgresql-x64-14

# macOS
brew services start postgresql

# Linux
sudo systemctl status postgresql
sudo systemctl start postgresql
```

**Code fix:**
```javascript
// Add connection retry logic
const sequelize = new Sequelize(config, {
    retry: {
        match: [/ECONNREFUSED/],
        max: 3,
        backoffBase: 1000,
        backoffExponent: 1.5
    }
});
```

### 2. Authentication Failed

**Error:**
```
SequelizeConnectionError: password authentication failed for user "username"
```

**Solutions:**

```sql
-- Reset PostgreSQL password
sudo -u postgres psql
ALTER USER postgres PASSWORD 'newpassword';
\q
```

**Environment check:**
```javascript
// Verify environment variables
console.log('DB Config:', {
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    database: process.env.DB_NAME,
    username: process.env.DB_USER,
    // Don't log password!
});
```

### 3. Database Does Not Exist

**Error:**
```
SequelizeConnectionError: database "myapp_development" does not exist
```

**Solutions:**

```bash
# Create database using CLI
sequelize db:create

# Or manually via psql
createdb myapp_development
# or
psql -U postgres -c "CREATE DATABASE myapp_development;"
```

### 4. SSL Connection Issues

**Error:**
```
SequelizeConnectionError: self signed certificate in certificate chain
```

**Solutions:**

```javascript
// For development (not recommended for production)
const sequelize = new Sequelize(config, {
    dialectOptions: {
        ssl: {
            require: true,
            rejectUnauthorized: false
        }
    }
});

// For production with proper SSL
const sequelize = new Sequelize(config, {
    dialectOptions: {
        ssl: {
            require: true,
            ca: fs.readFileSync('/path/to/ca-certificate.crt').toString()
        }
    }
});
```

## 🗃️ Migration Problems

### 1. Migration Already Exists

**Error:**
```
ERROR: relation "SequelizeMeta" already exists
```

**Solutions:**

```bash
# Check migration status
sequelize db:migrate:status

# If needed, reset migrations
sequelize db:migrate:undo:all
sequelize db:migrate
```

**Manual reset:**
```sql
-- Caution: This drops all data!
DROP TABLE IF EXISTS "SequelizeMeta" CASCADE;
-- Then run migrations again
```

### 2. Column Already Exists

**Error:**
```
ERROR: column "newColumn" of relation "users" already exists
```

**Solutions:**

```javascript
// Check if column exists before adding
module.exports = {
    up: async (queryInterface, Sequelize) => {
        const tableDescription = await queryInterface.describeTable('users');
        
        if (!tableDescription.newColumn) {
            await queryInterface.addColumn('users', 'newColumn', {
                type: Sequelize.STRING,
                allowNull: true
            });
        }
    },
    
    down: async (queryInterface, Sequelize) => {
        await queryInterface.removeColumn('users', 'newColumn');
    }
};
```

### 3. Foreign Key Constraint Failures

**Error:**
```
ERROR: insert or update on table violates foreign key constraint
```

**Solutions:**

```javascript
// Ensure proper order in migrations
module.exports = {
    up: async (queryInterface, Sequelize) => {
        // Create referenced table first
        await queryInterface.createTable('users', { /* ... */ });
        
        // Then create table with foreign key
        await queryInterface.createTable('posts', {
            userId: {
                type: Sequelize.INTEGER,
                references: {
                    model: 'users',
                    key: 'id'
                },
                onUpdate: 'CASCADE',
                onDelete: 'SET NULL'
            }
        });
    }
};

// Or add constraints after table creation
module.exports = {
    up: async (queryInterface, Sequelize) => {
        await queryInterface.createTable('posts', { /* without constraints */ });
        
        await queryInterface.addConstraint('posts', {
            fields: ['userId'],
            type: 'foreign key',
            name: 'posts_userId_fkey',
            references: {
                table: 'users',
                field: 'id'
            },
            onDelete: 'CASCADE',
            onUpdate: 'CASCADE'
        });
    }
};
```

### 4. Migration Stuck or Failed

**Error:**
```
Migration failed, but no specific error
```

**Debug steps:**

```bash
# Check current migration status
sequelize db:migrate:status

# Run with debug logging
DEBUG=sequelize:* sequelize db:migrate

# Manually mark migration as completed (if safe)
psql -d myapp_development
INSERT INTO "SequelizeMeta" (name) VALUES ('20231201000000-problematic-migration.js');
```

## 🏗️ Model & Association Errors

### 1. Model Not Found / Undefined

**Error:**
```
TypeError: Cannot read property 'findAll' of undefined
```

**Solutions:**

```javascript
// Ensure proper model registration
// models/index.js
const models = {};

// Import all models
models.User = require('./User');
models.Post = require('./Post');

// Define associations after all models are loaded
Object.keys(models).forEach(modelName => {
    if (models[modelName].associate) {
        models[modelName].associate(models);
    }
});

module.exports = models;
```

### 2. Association Not Found

**Error:**
```
Error: Association with alias "posts" does not exist on User
```

**Solutions:**

```javascript
// Ensure associations are defined correctly
// models/User.js
User.associate = function(models) {
    User.hasMany(models.Post, {
        foreignKey: 'userId',
        as: 'posts' // This alias must match usage
    });
};

// Usage must match alias
const user = await User.findOne({
    include: [{ model: Post, as: 'posts' }] // Use correct alias
});
```

### 3. Circular Dependency

**Error:**
```
RangeError: Maximum call stack size exceeded
```

**Solutions:**

```javascript
// Avoid circular imports by using association functions
// models/User.js
const User = sequelize.define('User', { /* ... */ });

// Don't import Post directly here
User.associate = function(models) {
    User.hasMany(models.Post, { foreignKey: 'userId', as: 'posts' });
};

module.exports = User;

// models/Post.js
const Post = sequelize.define('Post', { /* ... */ });

Post.associate = function(models) {
    Post.belongsTo(models.User, { foreignKey: 'userId', as: 'author' });
};

module.exports = Post;
```

### 4. Junction Table Issues

**Error:**
```
Error: Unknown column 'PostTag.postId' in 'field list'
```

**Solutions:**

```javascript
// Ensure junction table exists and has correct structure
// Migration for junction table
module.exports = {
    up: async (queryInterface, Sequelize) => {
        await queryInterface.createTable('PostTags', {
            id: {
                allowNull: false,
                autoIncrement: true,
                primaryKey: true,
                type: Sequelize.INTEGER
            },
            postId: {
                type: Sequelize.INTEGER,
                allowNull: false,
                references: {
                    model: 'posts',
                    key: 'id'
                }
            },
            tagId: {
                type: Sequelize.INTEGER,
                allowNull: false,
                references: {
                    model: 'tags',
                    key: 'id'
                }
            },
            createdAt: {
                allowNull: false,
                type: Sequelize.DATE
            },
            updatedAt: {
                allowNull: false,
                type: Sequelize.DATE
            }
        });
    }
};

// Correct association definition
Post.belongsToMany(Tag, {
    through: 'PostTags', // Exact table name
    foreignKey: 'postId',
    otherKey: 'tagId'
});
```

## 🔍 Query Issues

### 1. Cannot Read Property of Undefined

**Error:**
```
TypeError: Cannot read property 'firstName' of null
```

**Solutions:**

```javascript
// Always check if record exists
const user = await User.findByPk(1);
if (!user) {
    throw new Error('User not found');
}
console.log(user.firstName);

// Or use optional chaining (Node.js 14+)
console.log(user?.firstName);

// Or provide default
const firstName = user ? user.firstName : 'Unknown';
```

### 2. Unexpected Token in JSON

**Error:**
```
SyntaxError: Unexpected token in JSON at position 0
```

**Solutions:**

```javascript
// Ensure proper JSON handling
try {
    const metadata = JSON.parse(user.metadata || '{}');
} catch (error) {
    console.error('Invalid JSON in metadata:', user.metadata);
    const metadata = {};
}

// Or use JSONB with validation
const User = sequelize.define('User', {
    metadata: {
        type: DataTypes.JSONB,
        defaultValue: {},
        validate: {
            isValidJSON(value) {
                if (typeof value === 'string') {
                    try {
                        JSON.parse(value);
                    } catch (error) {
                        throw new Error('Invalid JSON format');
                    }
                }
            }
        }
    }
});
```

### 3. Include Association Not Working

**Error:**
```
No data returned for association, but no error thrown
```

**Solutions:**

```javascript
// Check if association is properly defined
const users = await User.findAll({
    include: [{
        model: Post,
        as: 'posts', // Ensure alias matches association definition
        required: false // Use LEFT JOIN instead of INNER JOIN
    }]
});

// Debug the generated SQL
const users = await User.findAll({
    include: [{ model: Post, as: 'posts' }],
    logging: console.log // This will show the generated SQL
});

// Check if foreign key values exist
const user = await User.findByPk(1);
console.log('User ID:', user.id);

const posts = await Post.findAll({
    where: { userId: user.id }
});
console.log('Posts for user:', posts.length);
```

### 4. Op is Not Defined

**Error:**
```
ReferenceError: Op is not defined
```

**Solutions:**

```javascript
// Import Op from Sequelize
const { Op } = require('sequelize');

const users = await User.findAll({
    where: {
        age: {
            [Op.gte]: 18 // Now Op is defined
        }
    }
});
```

## 🐌 Performance Problems

### 1. Slow Queries

**Symptoms:**
- Queries taking several seconds
- High CPU usage
- Timeouts

**Debugging:**

```javascript
// Enable query logging to see actual SQL
const sequelize = new Sequelize(config, {
    logging: (sql, timing) => {
        console.log(`[${timing}ms] ${sql}`);
    },
    benchmark: true
});

// Use EXPLAIN to analyze query performance
const [results, metadata] = await sequelize.query(
    'EXPLAIN ANALYZE SELECT * FROM users WHERE email = $1',
    {
        bind: ['john@example.com'],
        type: QueryTypes.SELECT
    }
);
console.log(results);
```

**Solutions:**

```javascript
// Add indexes for frequently queried columns
// Migration
module.exports = {
    up: async (queryInterface, Sequelize) => {
        await queryInterface.addIndex('users', ['email']);
        await queryInterface.addIndex('posts', ['userId', 'status']);
        await queryInterface.addIndex('posts', ['createdAt']);
    }
};

// Optimize queries - select only needed fields
const users = await User.findAll({
    attributes: ['id', 'firstName', 'email'], // Don't select all fields
    where: { isActive: true },
    limit: 20 // Always limit results
});

// Use pagination for large datasets
async function getPaginatedUsers(page = 1, pageSize = 20) {
    const offset = (page - 1) * pageSize;
    return await User.findAndCountAll({
        limit: pageSize,
        offset: offset,
        order: [['createdAt', 'DESC']]
    });
}
```

### 2. N+1 Query Problem

**Problem:**
```javascript
// This creates N+1 queries (1 for users + N for each user's posts)
const users = await User.findAll();
for (const user of users) {
    const posts = await user.getPosts(); // N additional queries
    console.log(`${user.name} has ${posts.length} posts`);
}
```

**Solutions:**

```javascript
// Use include to fetch related data in one query
const users = await User.findAll({
    include: [{
        model: Post,
        as: 'posts'
    }]
});

users.forEach(user => {
    console.log(`${user.name} has ${user.posts.length} posts`);
});

// Or use separate queries with IN clause
const users = await User.findAll();
const userIds = users.map(user => user.id);
const posts = await Post.findAll({
    where: { userId: { [Op.in]: userIds } }
});

// Group posts by userId
const postsByUser = posts.reduce((acc, post) => {
    if (!acc[post.userId]) acc[post.userId] = [];
    acc[post.userId].push(post);
    return acc;
}, {});
```

### 3. Memory Issues with Large Datasets

**Problem:**
```javascript
// Loading all records into memory
const allUsers = await User.findAll(); // Could be millions of records
```

**Solutions:**

```javascript
// Process in batches
async function processAllUsers(batchSize = 1000) {
    let offset = 0;
    let batch;
    
    do {
        batch = await User.findAll({
            limit: batchSize,
            offset: offset,
            order: [['id', 'ASC']]
        });
        
        // Process batch
        for (const user of batch) {
            await processUser(user);
        }
        
        offset += batchSize;
    } while (batch.length === batchSize);
}

// Use streams for very large datasets
const { QueryTypes } = require('sequelize');
const stream = sequelize.query(
    'SELECT * FROM users ORDER BY id',
    { type: QueryTypes.SELECT, stream: true }
);

stream.on('data', (user) => {
    processUser(user);
});
```

## ✅ Validation Errors

### 1. Validation Error Not Descriptive

**Error:**
```
SequelizeValidationError: Validation error
```

**Better error handling:**

```javascript
try {
    await User.create(userData);
} catch (error) {
    if (error.name === 'SequelizeValidationError') {
        const validationErrors = error.errors.map(err => ({
            field: err.path,
            message: err.message,
            value: err.value
        }));
        
        console.error('Validation failed:', validationErrors);
        
        // Return user-friendly errors
        const errorMessages = validationErrors.map(err => 
            `${err.field}: ${err.message}`
        );
        throw new Error(`Validation failed: ${errorMessages.join(', ')}`);
    }
    throw error;
}
```

### 2. Custom Validation Not Working

**Problem:**
```javascript
// Validation not running
const User = sequelize.define('User', {
    age: {
        type: DataTypes.INTEGER,
        validate: {
            isAdult(value) {
                if (value < 18) {
                    throw new Error('Must be 18 or older');
                }
            }
        }
    }
});
```

**Solutions:**

```javascript
// Ensure validation runs
const user = await User.create(userData, {
    validate: true // Explicitly enable validation
});

// For bulk operations
await User.bulkCreate(usersData, {
    validate: true,
    individualHooks: true
});

// For updates
await user.update(newData, {
    validate: true
});
```

### 3. Async Validation Issues

**Problem:**
```javascript
// Async validation not working properly
validate: {
    async isUniqueEmail(value) {
        const existing = await User.findOne({ where: { email: value } });
        if (existing) {
            throw new Error('Email already exists');
        }
    }
}
```

**Solutions:**

```javascript
// Use proper async validation
validate: {
    async isUniqueEmail(value) {
        if (value) {
            const existing = await User.findOne({
                where: { 
                    email: value,
                    id: { [Op.ne]: this.id || 0 } // Exclude current record for updates
                }
            });
            if (existing) {
                throw new Error('Email already exists');
            }
        }
    }
}

// Or handle uniqueness at database level
email: {
    type: DataTypes.STRING,
    unique: {
        name: 'users_email_unique',
        msg: 'Email address already in use'
    }
}
```

## 🚀 Production Issues

### 1. Connection Pool Exhaustion

**Error:**
```
SequelizeConnectionError: Connection pool is exhausted
```

**Solutions:**

```javascript
// Increase pool size
const sequelize = new Sequelize(config, {
    pool: {
        max: 20,      // Increase max connections
        min: 5,       // Minimum connections
        acquire: 60000, // Increase acquire timeout
        idle: 10000   // Connection idle time
    }
});

// Ensure connections are properly released
async function safeQuery() {
    let transaction;
    try {
        transaction = await sequelize.transaction();
        const result = await User.findAll({ transaction });
        await transaction.commit();
        return result;
    } catch (error) {
        if (transaction) await transaction.rollback();
        throw error;
    }
}
```

### 2. Memory Leaks

**Problem:**
- Gradually increasing memory usage
- Application crashes with out-of-memory errors

**Solutions:**

```javascript
// Avoid keeping references to large objects
async function processUsers() {
    const users = await User.findAll();
    
    // Process users
    for (const user of users) {
        await processUser(user);
    }
    
    // Clear references (usually not needed, but can help)
    users.length = 0;
}

// Use weak references for caches
const userCache = new WeakMap();

// Monitor memory usage
setInterval(() => {
    const usage = process.memoryUsage();
    console.log('Memory usage:', {
        rss: Math.round(usage.rss / 1024 / 1024) + 'MB',
        heapUsed: Math.round(usage.heapUsed / 1024 / 1024) + 'MB'
    });
}, 30000);
```

### 3. Slow Application Startup

**Problem:**
- Application takes too long to start
- Database synchronization issues

**Solutions:**

```javascript
// Don't sync in production
if (process.env.NODE_ENV !== 'production') {
    await sequelize.sync({ alter: true });
}

// Use connection retry with backoff
const sequelize = new Sequelize(config, {
    retry: {
        match: [/ETIMEDOUT/, /EHOSTUNREACH/, /ECONNRESET/, /ECONNREFUSED/],
        max: 5,
        backoffBase: 1000,
        backoffExponent: 1.5
    }
});

// Graceful startup
async function connectWithRetry() {
    let retries = 5;
    while (retries > 0) {
        try {
            await sequelize.authenticate();
            console.log('Database connected successfully');
            break;
        } catch (error) {
            retries--;
            console.log(`Database connection failed. Retries left: ${retries}`);
            if (retries === 0) throw error;
            await new Promise(resolve => setTimeout(resolve, 5000));
        }
    }
}
```

## 🛠️ Development Workflow

### 1. Model Sync Issues

**Problem:**
```javascript
// Model changes not reflected in database
const User = sequelize.define('User', {
    newField: DataTypes.STRING // Added this field
});
```

**Solutions:**

```javascript
// Use sync with alter for development
if (process.env.NODE_ENV === 'development') {
    await sequelize.sync({ alter: true });
}

// Or create a migration
sequelize migration:generate --name add-new-field-to-users

// Migration content
module.exports = {
    up: async (queryInterface, Sequelize) => {
        await queryInterface.addColumn('users', 'newField', {
            type: Sequelize.STRING,
            allowNull: true
        });
    },
    down: async (queryInterface, Sequelize) => {
        await queryInterface.removeColumn('users', 'newField');
    }
};
```

### 2. Seeder Not Running

**Problem:**
```bash
sequelize db:seed:all
# No data appears in database
```

**Debug steps:**

```javascript
// Check seeder file format
module.exports = {
    up: async (queryInterface, Sequelize) => {
        await queryInterface.bulkInsert('users', [
            {
                firstName: 'John',
                lastName: 'Doe',
                email: 'john@example.com',
                createdAt: new Date(),
                updatedAt: new Date()
            }
        ]);
    },
    
    down: async (queryInterface, Sequelize) => {
        await queryInterface.bulkDelete('users', null, {});
    }
};

// Check if seeder was already run
SELECT * FROM "SequelizeData";

// Force re-run seeder
sequelize db:seed:undo:all
sequelize db:seed:all
```

### 3. Environment-Specific Issues

**Problem:**
- Works in development but fails in production
- Different behavior across environments

**Solutions:**

```javascript
// Use environment-specific configs
const configs = {
    development: {
        logging: console.log,
        sync: { alter: true }
    },
    test: {
        logging: false,
        sync: { force: true }
    },
    production: {
        logging: false,
        sync: false,
        pool: { max: 20 }
    }
};

const config = configs[process.env.NODE_ENV] || configs.development;

// Environment validation
function validateEnvironment() {
    const required = ['DB_HOST', 'DB_NAME', 'DB_USER', 'DB_PASSWORD'];
    const missing = required.filter(key => !process.env[key]);
    
    if (missing.length > 0) {
        throw new Error(`Missing environment variables: ${missing.join(', ')}`);
    }
}

validateEnvironment();
```

## 🔧 Debug Tools & Techniques

### 1. Enable SQL Logging

```javascript
// Log all SQL queries
const sequelize = new Sequelize(config, {
    logging: (sql, timing) => {
        console.log(`[${new Date().toISOString()}] [${timing}ms] ${sql}`);
    },
    benchmark: true
});

// Log only slow queries
const sequelize = new Sequelize(config, {
    logging: (sql, timing) => {
        if (timing > 1000) { // Only log queries > 1 second
            console.warn(`SLOW QUERY [${timing}ms]: ${sql}`);
        }
    },
    benchmark: true
});
```

### 2. Query Analysis

```javascript
// Analyze query performance
async function analyzeQuery(sql, replacements = []) {
    console.log('Analyzing query:', sql);
    
    const [results] = await sequelize.query(
        `EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) ${sql}`,
        { replacements, type: QueryTypes.SELECT }
    );
    
    console.log('Query plan:', JSON.stringify(results[0], null, 2));
}

// Usage
await analyzeQuery(
    'SELECT * FROM users WHERE email = ?',
    ['john@example.com']
);
```

### 3. Connection Monitoring

```javascript
// Monitor connection pool
setInterval(() => {
    const pool = sequelize.connectionManager.pool;
    console.log('Connection pool status:', {
        total: pool.size,
        used: pool.used,
        waiting: pool.pending
    });
}, 10000);

// Monitor active queries
const activeQueries = new Set();

const originalQuery = sequelize.query;
sequelize.query = function(sql, options) {
    const queryId = Math.random().toString(36);
    activeQueries.add(queryId);
    
    console.log(`[${queryId}] Starting query: ${sql.substring(0, 100)}...`);
    
    const promise = originalQuery.call(this, sql, options);
    
    promise.finally(() => {
        activeQueries.delete(queryId);
        console.log(`[${queryId}] Query completed`);
    });
    
    return promise;
};
```

## 🎯 Best Practices for Avoiding Issues

1. **Always use transactions** for multi-step operations
2. **Handle errors gracefully** with proper try-catch blocks
3. **Validate environment variables** on application startup
4. **Use migrations** instead of model sync in production
5. **Monitor query performance** and add indexes as needed
6. **Implement proper logging** for debugging
7. **Test with realistic data volumes** during development
8. **Use connection pooling** appropriately
9. **Keep Sequelize and PostgreSQL versions updated**
10. **Regular database maintenance** (vacuum, analyze, reindex)

---

Remember: When in doubt, enable SQL logging to see exactly what queries Sequelize is generating!

## 🔗 Related Topics
- [[02-Setup-Installation|Setup & Installation]]
- [[15-Indexes-Performance|Performance Optimization]]
- [[19-Production-Deployment|Production Deployment]]
- [[20-Quick-Reference|Quick Reference Guide]]
