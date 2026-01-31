# Quick Reference Guide

## 🚀 Sequelize Cheat Sheet

This is your go-to reference for common Sequelize operations. Bookmark this page!

## 📋 Table of Contents

- [Connection & Setup](#connection--setup)
- [Model Definition](#model-definition)
- [Data Types](#data-types)
- [Associations](#associations)
- [CRUD Operations](#crud-operations)
- [Querying](#querying)
- [Operators](#operators)
- [Validation](#validation)
- [Hooks](#hooks)
- [Transactions](#transactions)
- [CLI Commands](#cli-commands)

## 🔌 Connection & Setup

### Basic Connection
```javascript
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize('database', 'username', 'password', {
    host: 'localhost',
    dialect: 'postgres',
    port: 5432
});

// Test connection
await sequelize.authenticate();
```

### Environment-based Config
```javascript
const sequelize = new Sequelize(process.env.DATABASE_URL, {
    dialect: 'postgres',
    logging: false,
    pool: {
        max: 5,
        min: 0,
        acquire: 30000,
        idle: 10000
    }
});
```

## 🏗️ Model Definition

### Basic Model
```javascript
const User = sequelize.define('User', {
    id: {
        type: DataTypes.INTEGER,
        primaryKey: true,
        autoIncrement: true
    },
    firstName: {
        type: DataTypes.STRING(50),
        allowNull: false
    },
    email: {
        type: DataTypes.STRING(100),
        unique: true,
        validate: { isEmail: true }
    },
    isActive: {
        type: DataTypes.BOOLEAN,
        defaultValue: true
    }
}, {
    tableName: 'users',
    timestamps: true,
    underscored: true
});
```

### Model with All Options
```javascript
const User = sequelize.define('User', {
    // attributes
}, {
    tableName: 'users',
    timestamps: true,
    paranoid: true,
    underscored: true,
    freezeTableName: true,
    version: true,
    defaultScope: { where: { isActive: true } },
    scopes: {
        active: { where: { isActive: true } }
    },
    indexes: [{ fields: ['email'] }],
    hooks: {
        beforeCreate: (user) => { /* logic */ }
    }
});
```

## 📊 Data Types

### Common Types
```javascript
// Strings
DataTypes.STRING           // VARCHAR(255)
DataTypes.STRING(100)      // VARCHAR(100)
DataTypes.TEXT             // TEXT
DataTypes.CHAR(2)          // CHAR(2)

// Numbers
DataTypes.INTEGER          // INTEGER
DataTypes.BIGINT           // BIGINT
DataTypes.FLOAT            // FLOAT
DataTypes.DOUBLE           // DOUBLE PRECISION
DataTypes.DECIMAL(10,2)    // DECIMAL(10,2)

// Dates
DataTypes.DATE             // TIMESTAMP
DataTypes.DATEONLY         // DATE
DataTypes.TIME             // TIME

// Others
DataTypes.BOOLEAN          // BOOLEAN
DataTypes.JSON             // JSON
DataTypes.JSONB            // JSONB (PostgreSQL)
DataTypes.UUID             // UUID
DataTypes.ENUM('a','b','c') // ENUM
DataTypes.ARRAY(DataTypes.STRING) // ARRAY
```

### Virtual Fields
```javascript
fullName: {
    type: DataTypes.VIRTUAL,
    get() {
        return `${this.firstName} ${this.lastName}`;
    },
    set(value) {
        const names = value.split(' ');
        this.setDataValue('firstName', names[0]);
        this.setDataValue('lastName', names[1]);
    }
}
```

## 🔗 Associations

### One-to-One
```javascript
User.hasOne(Profile, { foreignKey: 'userId', as: 'profile' });
Profile.belongsTo(User, { foreignKey: 'userId', as: 'user' });
```

### One-to-Many
```javascript
User.hasMany(Post, { foreignKey: 'userId', as: 'posts' });
Post.belongsTo(User, { foreignKey: 'userId', as: 'author' });
```

### Many-to-Many
```javascript
Post.belongsToMany(Tag, { 
    through: 'PostTags', 
    foreignKey: 'postId',
    otherKey: 'tagId',
    as: 'tags' 
});
Tag.belongsToMany(Post, { 
    through: 'PostTags', 
    foreignKey: 'tagId',
    otherKey: 'postId',
    as: 'posts' 
});
```

### Self-Reference
```javascript
User.hasMany(User, { as: 'subordinates', foreignKey: 'managerId' });
User.belongsTo(User, { as: 'manager', foreignKey: 'managerId' });
```

## 🔄 CRUD Operations

### Create
```javascript
// Single
const user = await User.create({ firstName: 'John', email: 'john@example.com' });

// Bulk
const users = await User.bulkCreate([
    { firstName: 'John', email: 'john@example.com' },
    { firstName: 'Jane', email: 'jane@example.com' }
]);

// Find or Create
const [user, created] = await User.findOrCreate({
    where: { email: 'john@example.com' },
    defaults: { firstName: 'John' }
});

// Upsert
const [user, created] = await User.upsert({
    id: 1,
    firstName: 'John',
    email: 'john@example.com'
});
```

### Read
```javascript
// Find all
const users = await User.findAll();

// Find one
const user = await User.findOne({ where: { email: 'john@example.com' } });

// Find by primary key
const user = await User.findByPk(1);

// Find and count
const { count, rows } = await User.findAndCountAll({
    where: { isActive: true },
    limit: 10,
    offset: 0
});

// Count
const count = await User.count({ where: { isActive: true } });
```

### Update
```javascript
// Instance update
const user = await User.findByPk(1);
user.firstName = 'Updated';
await user.save();

// Direct update
await user.update({ firstName: 'Updated' });

// Bulk update
const [affectedRows] = await User.update(
    { isActive: false },
    { where: { lastLoginAt: { [Op.lt]: new Date('2023-01-01') } } }
);

// Increment
await user.increment('loginCount');
await user.increment({ loginCount: 1, score: 10 });
```

### Delete
```javascript
// Instance delete
const user = await User.findByPk(1);
await user.destroy();

// Bulk delete
const deletedRows = await User.destroy({
    where: { isActive: false }
});

// Truncate
await User.truncate();
```

## 🔍 Querying

### Basic Queries
```javascript
// With conditions
const users = await User.findAll({
    where: { isActive: true, age: { [Op.gte]: 18 } }
});

// Select specific attributes
const users = await User.findAll({
    attributes: ['id', 'firstName', 'email']
});

// Exclude attributes
const users = await User.findAll({
    attributes: { exclude: ['passwordHash'] }
});

// Ordering
const users = await User.findAll({
    order: [['firstName', 'ASC'], ['createdAt', 'DESC']]
});

// Pagination
const users = await User.findAll({
    limit: 10,
    offset: 20
});
```

### Advanced Queries
```javascript
// Raw queries
const [results] = await sequelize.query('SELECT * FROM users WHERE age > ?', {
    replacements: [18],
    type: QueryTypes.SELECT
});

// Aggregation
const stats = await User.findAll({
    attributes: [
        'role',
        [sequelize.fn('COUNT', sequelize.col('id')), 'count'],
        [sequelize.fn('AVG', sequelize.col('age')), 'avgAge']
    ],
    group: ['role']
});

// Subqueries
const users = await User.findAll({
    where: {
        id: {
            [Op.in]: sequelize.literal('(SELECT user_id FROM posts WHERE status = "published")')
        }
    }
});
```

### Includes (Joins)
```javascript
// Basic include
const users = await User.findAll({
    include: [{ model: Post, as: 'posts' }]
});

// Nested includes
const users = await User.findAll({
    include: [{
        model: Post,
        as: 'posts',
        include: [{ model: Comment, as: 'comments' }]
    }]
});

// Include with conditions
const users = await User.findAll({
    include: [{
        model: Post,
        as: 'posts',
        where: { status: 'published' },
        required: false // LEFT JOIN
    }]
});

// Include with specific attributes
const users = await User.findAll({
    include: [{
        model: Post,
        as: 'posts',
        attributes: ['id', 'title']
    }]
});
```

## ⚡ Operators

### Comparison Operators
```javascript
const { Op } = require('sequelize');

// Basic operators
{ id: 1 }                          // = 1
{ id: { [Op.ne]: 1 } }            // != 1
{ id: { [Op.gt]: 1 } }            // > 1
{ id: { [Op.gte]: 1 } }           // >= 1
{ id: { [Op.lt]: 1 } }            // < 1
{ id: { [Op.lte]: 1 } }           // <= 1
{ id: { [Op.between]: [1, 10] } }  // BETWEEN 1 AND 10
{ id: { [Op.notBetween]: [1, 10] } } // NOT BETWEEN 1 AND 10
{ id: { [Op.in]: [1, 2, 3] } }    // IN (1, 2, 3)
{ id: { [Op.notIn]: [1, 2, 3] } } // NOT IN (1, 2, 3)
```

### String Operators
```javascript
{ name: { [Op.like]: '%John%' } }     // LIKE '%John%'
{ name: { [Op.notLike]: '%John%' } }  // NOT LIKE '%John%'
{ name: { [Op.iLike]: '%john%' } }    // ILIKE '%john%' (case insensitive)
{ name: { [Op.regexp]: '^[a-z]+$' } } // REGEXP '^[a-z]+$'
```

### Logical Operators
```javascript
{
    [Op.and]: [
        { id: { [Op.gt]: 1 } },
        { name: 'John' }
    ]
}

{
    [Op.or]: [
        { id: 1 },
        { name: 'John' }
    ]
}

{ name: { [Op.not]: 'John' } }
```

### Date Operators
```javascript
{ createdAt: { [Op.gte]: new Date() } }
{ createdAt: { [Op.between]: [startDate, endDate] } }
```

### Array/JSON Operators (PostgreSQL)
```javascript
{ tags: { [Op.contains]: ['javascript'] } }     // Array contains
{ metadata: { [Op.contains]: { key: 'value' } } } // JSON contains
{ 'metadata.key': 'value' }                     // JSON key equals
```

## ✅ Validation

### Built-in Validators
```javascript
{
    email: {
        type: DataTypes.STRING,
        validate: {
            isEmail: true,              // Email format
            isUrl: true,                // URL format
            isIP: true,                 // IP address
            isAlpha: true,              // Only letters
            isAlphanumeric: true,       // Letters and numbers
            isNumeric: true,            // Only numbers
            isInt: true,                // Integer
            isFloat: true,              // Float
            isDecimal: true,            // Decimal
            isDate: true,               // Date
            isCreditCard: true,         // Credit card
            len: [2, 50],              // Length between 2-50
            min: 0,                     // Minimum value
            max: 100,                   // Maximum value
            notEmpty: true,             // Not empty
            notNull: true,              // Not null
            is: /^[a-z]+$/i,           // Regex match
            not: /[^a-z]/i,            // Regex not match
            contains: 'hello',          // Contains substring
            notContains: 'world',       // Doesn't contain
            isIn: [['admin', 'user']],  // In array
            notIn: [['banned']]         // Not in array
        }
    }
}
```

### Custom Validators
```javascript
{
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
}

// Model-level validation
{
    validate: {
        passwordsMatch() {
            if (this.password !== this.confirmPassword) {
                throw new Error('Passwords do not match');
            }
        }
    }
}
```

## 🪝 Hooks

### Available Hooks
```javascript
// Creation hooks
beforeValidate, afterValidate
beforeCreate, afterCreate
beforeSave, afterSave

// Update hooks
beforeUpdate, afterUpdate
beforeUpsert, afterUpsert

// Deletion hooks
beforeDestroy, afterDestroy

// Bulk operation hooks
beforeBulkCreate, afterBulkCreate
beforeBulkUpdate, afterBulkUpdate
beforeBulkDestroy, afterBulkDestroy
```

### Hook Examples
```javascript
// Instance hooks
User.beforeCreate((user, options) => {
    user.email = user.email.toLowerCase();
});

User.afterCreate((user, options) => {
    console.log('User created:', user.id);
});

// Model hooks
User.addHook('beforeValidate', (user, options) => {
    if (user.firstName) {
        user.firstName = user.firstName.trim();
    }
});

// Conditional hooks
User.beforeSave((user, options) => {
    if (user.changed('email')) {
        user.emailVerified = false;
    }
});
```

## 💾 Transactions

### Basic Transaction
```javascript
const transaction = await sequelize.transaction();

try {
    const user = await User.create({ name: 'John' }, { transaction });
    const profile = await Profile.create({ userId: user.id }, { transaction });
    
    await transaction.commit();
} catch (error) {
    await transaction.rollback();
    throw error;
}
```

### Auto-managed Transaction
```javascript
await sequelize.transaction(async (t) => {
    const user = await User.create({ name: 'John' }, { transaction: t });
    const profile = await Profile.create({ userId: user.id }, { transaction: t });
    
    // Auto-commit on success, auto-rollback on error
});
```

### Transaction Types
```javascript
// Read committed (default)
const transaction = await sequelize.transaction({
    isolationLevel: Transaction.ISOLATION_LEVELS.READ_COMMITTED
});

// Serializable
const transaction = await sequelize.transaction({
    isolationLevel: Transaction.ISOLATION_LEVELS.SERIALIZABLE
});
```

## 🖥️ CLI Commands

### Sequelize CLI Setup
```bash
npm install -g sequelize-cli
sequelize init
```

### Migration Commands
```bash
# Create migration
sequelize migration:generate --name create-users-table

# Run migrations
sequelize db:migrate

# Undo last migration
sequelize db:migrate:undo

# Undo all migrations
sequelize db:migrate:undo:all

# Migration status
sequelize db:migrate:status
```

### Seeder Commands
```bash
# Create seeder
sequelize seed:generate --name demo-users

# Run seeders
sequelize db:seed:all

# Run specific seeder
sequelize db:seed --seed 20231201000000-demo-users.js

# Undo seeders
sequelize db:seed:undo:all
```

### Model/Migration Generation
```bash
# Generate model and migration
sequelize model:generate --name User --attributes firstName:string,lastName:string,email:string

# Generate migration only
sequelize migration:generate --name add-age-to-users
```

### Database Commands
```bash
# Create database
sequelize db:create

# Drop database
sequelize db:drop
```

## 🔧 Common Patterns

### Repository Pattern
```javascript
class UserRepository {
    static async findById(id) {
        return await User.findByPk(id);
    }
    
    static async findByEmail(email) {
        return await User.findOne({ where: { email } });
    }
    
    static async create(userData) {
        return await User.create(userData);
    }
    
    static async update(id, updates) {
        const [affectedRows] = await User.update(updates, { where: { id } });
        return affectedRows > 0;
    }
    
    static async delete(id) {
        const deletedRows = await User.destroy({ where: { id } });
        return deletedRows > 0;
    }
}
```

### Service Layer
```javascript
class UserService {
    static async createUser(userData) {
        const existingUser = await UserRepository.findByEmail(userData.email);
        if (existingUser) {
            throw new Error('Email already exists');
        }
        
        return await UserRepository.create(userData);
    }
    
    static async getUserWithPosts(id) {
        return await User.findByPk(id, {
            include: [{ model: Post, as: 'posts' }]
        });
    }
}
```

### Error Handling
```javascript
try {
    await User.create(userData);
} catch (error) {
    if (error.name === 'SequelizeValidationError') {
        // Handle validation errors
        const errors = error.errors.map(e => e.message);
        console.error('Validation errors:', errors);
    } else if (error.name === 'SequelizeUniqueConstraintError') {
        // Handle unique constraint errors
        console.error('Duplicate entry:', error.errors[0].path);
    } else if (error.name === 'SequelizeForeignKeyConstraintError') {
        // Handle foreign key errors
        console.error('Foreign key constraint failed');
    } else {
        // Handle other errors
        console.error('Database error:', error.message);
    }
}
```

## 📱 Environment Variables

### .env File
```bash
# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp_development
DB_USER=postgres
DB_PASSWORD=password
DB_DIALECT=postgres

# Test Database
TEST_DB_NAME=myapp_test

# Production
DATABASE_URL=postgres://user:password@host:port/database

# Application
NODE_ENV=development
PORT=3000
JWT_SECRET=your_jwt_secret
```

### Config Usage
```javascript
require('dotenv').config();

const config = {
    development: {
        username: process.env.DB_USER,
        password: process.env.DB_PASSWORD,
        database: process.env.DB_NAME,
        host: process.env.DB_HOST,
        port: process.env.DB_PORT,
        dialect: process.env.DB_DIALECT
    }
};
```

## 🎯 Performance Tips

### Query Optimization
```javascript
// Good: Select only needed fields
User.findAll({ attributes: ['id', 'name', 'email'] });

// Good: Use indexes
{ where: { email: 'john@example.com' } } // If email is indexed

// Good: Limit results
User.findAll({ limit: 10, offset: 20 });

// Good: Use includes instead of separate queries
User.findAll({ include: [{ model: Post, as: 'posts' }] });

// Avoid: N+1 queries
const users = await User.findAll();
for (const user of users) {
    const posts = await user.getPosts(); // N+1 query problem
}
```

### Connection Pool
```javascript
const sequelize = new Sequelize(database, username, password, {
    pool: {
        max: 20,     // Maximum connections
        min: 0,      // Minimum connections
        acquire: 30000, // Maximum time to get connection
        idle: 10000  // Maximum idle time
    }
});
```

### Indexes
```javascript
// In model definition
{
    indexes: [
        { fields: ['email'] },
        { fields: ['firstName', 'lastName'] },
        { unique: true, fields: ['username'] }
    ]
}

// In migration
await queryInterface.addIndex('users', ['email']);
```

---

## 🔗 Related Topics
- [[01-Introduction|Introduction]]
- [[04-Sequelize-Introduction|Sequelize Basics]]
- [[09-CRUD-Operations|CRUD Operations]]
- [[11-Advanced-Queries|Advanced Queries]]
