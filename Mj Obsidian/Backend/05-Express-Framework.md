# Express Framework

**Navigation**: [[04-HTTP-Server]] | [[00-Main-Index]] | Next: [[06-Express-Router]]

---

## 🚀 Introduction to Express.js

Express.js is a **minimal and flexible** Node.js web application framework that provides a robust set of features for web and mobile applications.

---

## 📦 Installation and Setup

### Installing Express
```bash
# Initialize a new project
npm init -y

# Install Express
npm install express

# Install development dependencies
npm install --save-dev nodemon
```

### Basic Express App
```javascript
const express = require('express');
const app = express();
const PORT = 3000;

// Basic route
app.get('/', (req, res) => {
    res.send('Hello World!');
});

app.listen(PORT, () => {
    console.log(`Server is running on http://localhost:${PORT}`);
});
```

### Package.json Scripts
```json
{
  "scripts": {
    "start": "node app.js",
    "dev": "nodemon app.js"
  }
}
```

---

## 🛤️ Basic Routing

### HTTP Methods
```javascript
const express = require('express');
const app = express();

// Middleware to parse JSON bodies
app.use(express.json());

// GET route
app.get('/', (req, res) => {
    res.json({ message: 'GET request to homepage' });
});

// POST route
app.post('/users', (req, res) => {
    const user = req.body;
    res.status(201).json({ 
        message: 'User created',
        user: user 
    });
});

// PUT route
app.put('/users/:id', (req, res) => {
    const id = req.params.id;
    const updates = req.body;
    res.json({ 
        message: `User ${id} updated`,
        updates: updates 
    });
});

// DELETE route
app.delete('/users/:id', (req, res) => {
    const id = req.params.id;
    res.json({ message: `User ${id} deleted` });
});

// PATCH route
app.patch('/users/:id', (req, res) => {
    const id = req.params.id;
    const updates = req.body;
    res.json({ 
        message: `User ${id} partially updated`,
        updates: updates 
    });
});

app.listen(3000);
```

---

## 📍 Route Parameters

### URL Parameters
```javascript
const express = require('express');
const app = express();

// Single parameter
app.get('/users/:id', (req, res) => {
    const userId = req.params.id;
    res.json({ userId: userId });
});

// Multiple parameters
app.get('/users/:userId/posts/:postId', (req, res) => {
    const { userId, postId } = req.params;
    res.json({ 
        message: `Post ${postId} from User ${userId}`,
        params: req.params 
    });
});

// Optional parameters with ?
app.get('/posts/:year/:month?', (req, res) => {
    const { year, month } = req.params;
    res.json({ 
        year: year,
        month: month || 'all months' 
    });
});

// Parameter with specific pattern
app.get('/users/:id(\\d+)', (req, res) => {
    // Only matches if id is a number
    res.json({ userId: req.params.id });
});

app.listen(3000);
```

### Query Parameters
```javascript
const express = require('express');
const app = express();

// Query parameters: /search?q=node&limit=10&sort=date
app.get('/search', (req, res) => {
    const query = req.query.q;
    const limit = parseInt(req.query.limit) || 10;
    const sort = req.query.sort || 'relevance';
    const page = parseInt(req.query.page) || 1;
    
    res.json({
        query: query,
        limit: limit,
        sort: sort,
        page: page,
        allQuery: req.query
    });
});

// Array query parameters: /tags?categories=tech&categories=web
app.get('/tags', (req, res) => {
    const categories = req.query.categories;
    // categories could be string or array
    const categoriesArray = Array.isArray(categories) ? categories : [categories];
    
    res.json({
        categories: categoriesArray,
        count: categoriesArray.length
    });
});

app.listen(3000);
```

---

## 🔧 Middleware

### Built-in Middleware
```javascript
const express = require('express');
const app = express();

// Parse JSON bodies
app.use(express.json({ limit: '10mb' }));

// Parse URL-encoded bodies
app.use(express.urlencoded({ extended: true }));

// Serve static files
app.use(express.static('public'));

// Custom static folder with prefix
app.use('/assets', express.static('assets'));

app.listen(3000);
```

### Custom Middleware
```javascript
const express = require('express');
const app = express();

// Logger middleware
const logger = (req, res, next) => {
    const timestamp = new Date().toISOString();
    console.log(`[${timestamp}] ${req.method} ${req.url}`);
    next(); // Important: call next() to continue
};

// Request timing middleware
const timer = (req, res, next) => {
    req.startTime = Date.now();
    
    // Override res.end to calculate duration
    const originalEnd = res.end;
    res.end = function(...args) {
        const duration = Date.now() - req.startTime;
        console.log(`Request took ${duration}ms`);
        originalEnd.apply(this, args);
    };
    
    next();
};

// Authentication middleware
const authenticate = (req, res, next) => {
    const token = req.headers.authorization;
    
    if (!token) {
        return res.status(401).json({ error: 'No token provided' });
    }
    
    // Verify token (simplified)
    if (token === 'Bearer valid-token') {
        req.user = { id: 1, name: 'John' };
        next();
    } else {
        res.status(401).json({ error: 'Invalid token' });
    }
};

// Apply middleware globally
app.use(logger);
app.use(timer);

// Apply middleware to specific routes
app.get('/public', (req, res) => {
    res.json({ message: 'Public route' });
});

app.get('/protected', authenticate, (req, res) => {
    res.json({ 
        message: 'Protected route',
        user: req.user 
    });
});

app.listen(3000);
```

---

## 🎯 Request and Response Objects

### Request Object Properties
```javascript
const express = require('express');
const app = express();

app.use(express.json());

app.all('/info', (req, res) => {
    const requestInfo = {
        // Basic properties
        method: req.method,
        url: req.url,
        originalUrl: req.originalUrl,
        path: req.path,
        
        // Headers
        headers: req.headers,
        userAgent: req.get('User-Agent'),
        
        // Parameters
        params: req.params,
        query: req.query,
        body: req.body,
        
        // Network info
        ip: req.ip,
        ips: req.ips,
        hostname: req.hostname,
        protocol: req.protocol,
        secure: req.secure,
        
        // Useful methods
        contentType: req.get('Content-Type'),
        accepts: req.accepts(['json', 'html']),
        
        // Custom properties (added by middleware)
        timestamp: new Date().toISOString()
    };
    
    res.json(requestInfo);
});

app.listen(3000);
```

### Response Object Methods
```javascript
const express = require('express');
const app = express();

app.get('/response-demo', (req, res) => {
    // Set status code
    res.status(200);
    
    // Set headers
    res.set('X-API-Version', '1.0.0');
    res.set({
        'Cache-Control': 'no-cache',
        'X-Custom-Header': 'value'
    });
    
    // Different response methods
    const format = req.query.format;
    
    switch (format) {
        case 'json':
            res.json({ message: 'JSON response' });
            break;
            
        case 'text':
            res.send('Plain text response');
            break;
            
        case 'html':
            res.send('<h1>HTML Response</h1>');
            break;
            
        case 'redirect':
            res.redirect('/');
            break;
            
        case 'download':
            res.download('package.json');
            break;
            
        case 'attachment':
            res.attachment('data.json');
            res.json({ data: 'example' });
            break;
            
        default:
            res.json({ 
                availableFormats: ['json', 'text', 'html', 'redirect', 'download', 'attachment'],
                usage: 'Add ?format=json to the URL'
            });
    }
});

// Content negotiation
app.get('/negotiate', (req, res) => {
    res.format({
        'text/plain': () => {
            res.send('Hey');
        },
        'text/html': () => {
            res.send('<p>Hey</p>');
        },
        'application/json': () => {
            res.json({ message: 'Hey' });
        },
        'default': () => {
            res.status(406).send('Not Acceptable');
        }
    });
});

app.listen(3000);
```

---

## 🏗️ Application Structure

### Modular Express App
```javascript
// app.js
const express = require('express');
const userRoutes = require('./routes/users');
const postRoutes = require('./routes/posts');

const app = express();

// Global middleware
app.use(express.json());
app.use(express.static('public'));

// Custom middleware
app.use((req, res, next) => {
    console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
    next();
});

// Routes
app.use('/api/users', userRoutes);
app.use('/api/posts', postRoutes);

// Health check
app.get('/health', (req, res) => {
    res.json({ 
        status: 'OK', 
        timestamp: new Date().toISOString(),
        uptime: process.uptime()
    });
});

// 404 handler
app.use('*', (req, res) => {
    res.status(404).json({ 
        error: 'Route not found',
        method: req.method,
        url: req.url
    });
});

// Error handler
app.use((err, req, res, next) => {
    console.error(err.stack);
    res.status(500).json({ 
        error: 'Something went wrong!',
        message: err.message
    });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});

module.exports = app;
```

---

## 📊 Complete CRUD API Example

### User Management API
```javascript
const express = require('express');
const app = express();

app.use(express.json());

// In-memory data store
let users = [
    { id: 1, name: 'John Doe', email: 'john@example.com', age: 30 },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com', age: 25 }
];
let nextId = 3;

// Validation middleware
const validateUser = (req, res, next) => {
    const { name, email } = req.body;
    
    if (!name || !email) {
        return res.status(400).json({
            error: 'Name and email are required'
        });
    }
    
    if (!email.includes('@')) {
        return res.status(400).json({
            error: 'Invalid email format'
        });
    }
    
    next();
};

// GET /users - Get all users with pagination
app.get('/users', (req, res) => {
    const page = parseInt(req.query.page) || 1;
    const limit = parseInt(req.query.limit) || 10;
    const startIndex = (page - 1) * limit;
    const endIndex = page * limit;
    
    const paginatedUsers = users.slice(startIndex, endIndex);
    
    res.json({
        users: paginatedUsers,
        pagination: {
            page: page,
            limit: limit,
            total: users.length,
            pages: Math.ceil(users.length / limit)
        }
    });
});

// GET /users/:id - Get user by ID
app.get('/users/:id', (req, res) => {
    const id = parseInt(req.params.id);
    const user = users.find(u => u.id === id);
    
    if (!user) {
        return res.status(404).json({ error: 'User not found' });
    }
    
    res.json({ user });
});

// POST /users - Create new user
app.post('/users', validateUser, (req, res) => {
    const { name, email, age } = req.body;
    
    // Check if email already exists
    const existingUser = users.find(u => u.email === email);
    if (existingUser) {
        return res.status(409).json({ error: 'Email already exists' });
    }
    
    const newUser = {
        id: nextId++,
        name,
        email,
        age: age || null,
        createdAt: new Date().toISOString()
    };
    
    users.push(newUser);
    
    res.status(201).json({
        message: 'User created successfully',
        user: newUser
    });
});

// PUT /users/:id - Update user completely
app.put('/users/:id', validateUser, (req, res) => {
    const id = parseInt(req.params.id);
    const userIndex = users.findIndex(u => u.id === id);
    
    if (userIndex === -1) {
        return res.status(404).json({ error: 'User not found' });
    }
    
    const { name, email, age } = req.body;
    
    users[userIndex] = {
        ...users[userIndex],
        name,
        email,
        age: age || null,
        updatedAt: new Date().toISOString()
    };
    
    res.json({
        message: 'User updated successfully',
        user: users[userIndex]
    });
});

// PATCH /users/:id - Update user partially
app.patch('/users/:id', (req, res) => {
    const id = parseInt(req.params.id);
    const userIndex = users.findIndex(u => u.id === id);
    
    if (userIndex === -1) {
        return res.status(404).json({ error: 'User not found' });
    }
    
    const updates = req.body;
    users[userIndex] = {
        ...users[userIndex],
        ...updates,
        updatedAt: new Date().toISOString()
    };
    
    res.json({
        message: 'User updated successfully',
        user: users[userIndex]
    });
});

// DELETE /users/:id - Delete user
app.delete('/users/:id', (req, res) => {
    const id = parseInt(req.params.id);
    const userIndex = users.findIndex(u => u.id === id);
    
    if (userIndex === -1) {
        return res.status(404).json({ error: 'User not found' });
    }
    
    const deletedUser = users.splice(userIndex, 1)[0];
    
    res.json({
        message: 'User deleted successfully',
        user: deletedUser
    });
});

app.listen(3000, () => {
    console.log('CRUD API running on port 3000');
});
```

---

## 🔗 Related Topics
- [[04-HTTP-Server]] - Native Node.js HTTP server
- [[06-Express-Router]] - Advanced routing with Express Router
- [[07-Serving-Static-Files]] - Serving static content
- [[20-Middlewares]] - Custom middleware patterns

---

*Back to [[00-Main-Index]]*
