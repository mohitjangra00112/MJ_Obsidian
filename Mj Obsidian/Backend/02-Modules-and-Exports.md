# Modules and Exports

**Navigation**: [[01-Node-JS-Fundamentals]] | [[00-Main-Index]] | Next: [[03-Event-Loop-and-Threading]]

---

## 🎯 Node.js Module System

Node.js uses the **CommonJS** module system for organizing code into reusable modules.

---

## 📦 require() Function

### Basic Syntax
```javascript
const moduleeName = require('module-name');
```

### Built-in Modules
```javascript
const fs = require('fs');
const path = require('path');
const http = require('http');
const url = require('url');
```

### Local Modules
```javascript
// Relative paths
const myModule = require('./myModule');
const utils = require('../utils/helpers');

// From node_modules
const express = require('express');
const mongoose = require('mongoose');
```

---

## 📤 module.exports

### Basic Export
```javascript
// math.js
function add(a, b) {
    return a + b;
}

function subtract(a, b) {
    return a - b;
}

// Export single function
module.exports = add;

// Or export an object
module.exports = {
    add: add,
    subtract: subtract
};

// Shorthand
module.exports = { add, subtract };
```

### Using Exported Module
```javascript
// main.js
const math = require('./math');

console.log(math.add(5, 3)); // 8
console.log(math.subtract(5, 3)); // 2
```

---

## 🔗 Named Exports

### exports Object
```javascript
// utils.js
exports.formatDate = function(date) {
    return date.toISOString().split('T')[0];
};

exports.capitalize = function(str) {
    return str.charAt(0).toUpperCase() + str.slice(1);
};

exports.PI = 3.14159;
```

### Using Named Exports
```javascript
// app.js
const utils = require('./utils');

console.log(utils.formatDate(new Date()));
console.log(utils.capitalize('hello'));
console.log(utils.PI);
```

### Destructuring Imports
```javascript
// app.js
const { formatDate, capitalize } = require('./utils');

console.log(formatDate(new Date()));
console.log(capitalize('hello'));
```

---

## 🆚 module.exports vs exports

### Key Differences
```javascript
// ✅ This works
exports.myFunction = function() {};

// ✅ This works
module.exports = {
    myFunction: function() {}
};

// ❌ This doesn't work as expected
exports = {
    myFunction: function() {}
};

// ✅ This works
module.exports.myFunction = function() {};
```

### Why exports = {} doesn't work
```javascript
// When you do this:
exports = { myFunction: function() {} };

// You're reassigning the local variable 'exports'
// but module.exports remains unchanged
```

---

## 📁 Module Patterns

### 1. Function Export
```javascript
// logger.js
module.exports = function(message) {
    console.log(`[${new Date().toISOString()}] ${message}`);
};

// Usage
const log = require('./logger');
log('Hello World');
```

### 2. Class Export
```javascript
// person.js
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    
    greet() {
        return `Hello, I'm ${this.name}`;
    }
}

module.exports = Person;

// Usage
const Person = require('./person');
const john = new Person('John', 30);
console.log(john.greet());
```

### 3. Object Export
```javascript
// config.js
module.exports = {
    database: {
        host: 'localhost',
        port: 5432,
        name: 'myapp'
    },
    server: {
        port: 3000,
        host: '0.0.0.0'
    }
};

// Usage
const config = require('./config');
console.log(config.database.host);
```

### 4. Mixed Export
```javascript
// api.js
function get(url) {
    // GET request logic
}

function post(url, data) {
    // POST request logic
}

const BASE_URL = 'https://api.example.com';

module.exports = {
    get,
    post,
    BASE_URL
};
```

---

## 🔍 Module Resolution

### Resolution Order
1. **Core modules**: `fs`, `http`, `path`
2. **Local files**: `./`, `../`
3. **node_modules**: Current directory, then parent directories

### File Extensions
```javascript
// These are equivalent
require('./module');
require('./module.js');
require('./module.json');
require('./module.node');
```

### Directory Modules
```javascript
// If requiring a directory, Node.js looks for:
// 1. package.json with "main" field
// 2. index.js
// 3. index.json
// 4. index.node

require('./myModule'); // Looks for myModule/index.js
```

---

## 🌐 ES6 Modules vs CommonJS

### ES6 Modules (Modern)
```javascript
// math.js (ES6)
export function add(a, b) {
    return a + b;
}

export default function multiply(a, b) {
    return a * b;
}

// main.js (ES6)
import multiply, { add } from './math.js';
```

### CommonJS (Node.js Default)
```javascript
// math.js (CommonJS)
function add(a, b) {
    return a + b;
}

module.exports = { add };

// main.js (CommonJS)
const { add } = require('./math');
```

---

## 📝 Best Practices

### 1. Consistent Export Style
```javascript
// ✅ Good - consistent object export
module.exports = {
    create,
    update,
    delete: deleteItem
};

// ❌ Avoid mixing styles
exports.create = create;
module.exports.update = update;
```

### 2. Clear Module Purpose
```javascript
// ✅ Good - single responsibility
// userService.js
module.exports = {
    createUser,
    getUserById,
    updateUser,
    deleteUser
};
```

### 3. Use Descriptive Names
```javascript
// ✅ Good
const userController = require('./controllers/userController');
const emailService = require('./services/emailService');

// ❌ Avoid
const uc = require('./controllers/userController');
const es = require('./services/emailService');
```

---

## 🛠️ Practical Examples

### Creating a Utility Module
```javascript
// utils/helpers.js
const crypto = require('crypto');

exports.generateId = function() {
    return crypto.randomBytes(16).toString('hex');
};

exports.formatCurrency = function(amount) {
    return new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD'
    }).format(amount);
};

exports.delay = function(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
};
```

### Database Module
```javascript
// database/connection.js
const { Pool } = require('pg');

const pool = new Pool({
    user: 'username',
    host: 'localhost',
    database: 'myapp',
    password: 'password',
    port: 5432,
});

module.exports = {
    query: (text, params) => pool.query(text, params),
    getClient: () => pool.connect()
};
```

---

## 🔗 Related Topics
- [[01-Node-JS-Fundamentals]] - Basic Node.js concepts
- [[05-Express-Framework]] - Using modules in Express
- [[19-MVC-Pattern]] - Organizing modules with MVC

---

*Back to [[00-Main-Index]]*
