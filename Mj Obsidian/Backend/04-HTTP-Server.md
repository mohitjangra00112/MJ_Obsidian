# HTTP Server

**Navigation**: [[03-Event-Loop-and-Threading]] | [[00-Main-Index]] | Next: [[05-Express-Framework]]

---

## 🌐 Creating HTTP Server in Node.js

Node.js provides a built-in `http` module to create HTTP servers without any external dependencies.

---

## 🚀 Basic HTTP Server

### Simple Server
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello, World!');
});

const PORT = 3000;
server.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
});
```

### Server with Request Information
```javascript
const http = require('http');
const url = require('url');

const server = http.createServer((req, res) => {
    const parsedUrl = url.parse(req.url, true);
    
    // Set response headers
    res.writeHead(200, { 
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*'
    });
    
    // Response data
    const responseData = {
        method: req.method,
        url: req.url,
        pathname: parsedUrl.pathname,
        query: parsedUrl.query,
        headers: req.headers,
        timestamp: new Date().toISOString()
    };
    
    res.end(JSON.stringify(responseData, null, 2));
});

server.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

---

## 🛤️ Routing

### Basic Routing
```javascript
const http = require('http');
const url = require('url');

const server = http.createServer((req, res) => {
    const parsedUrl = url.parse(req.url, true);
    const path = parsedUrl.pathname;
    const method = req.method;
    
    // Set default headers
    res.setHeader('Content-Type', 'application/json');
    
    // Home route
    if (path === '/' && method === 'GET') {
        res.statusCode = 200;
        res.end(JSON.stringify({ message: 'Welcome to Home Page' }));
    }
    // Users route
    else if (path === '/users' && method === 'GET') {
        res.statusCode = 200;
        res.end(JSON.stringify({ 
            users: [
                { id: 1, name: 'John' },
                { id: 2, name: 'Jane' }
            ]
        }));
    }
    // About route
    else if (path === '/about' && method === 'GET') {
        res.statusCode = 200;
        res.end(JSON.stringify({ message: 'About Us Page' }));
    }
    // 404 Not Found
    else {
        res.statusCode = 404;
        res.end(JSON.stringify({ error: 'Route not found' }));
    }
});

server.listen(3000, () => {
    console.log('Server with routing running on port 3000');
});
```

### Advanced Routing with Parameters
```javascript
const http = require('http');
const url = require('url');

const server = http.createServer((req, res) => {
    const parsedUrl = url.parse(req.url, true);
    const path = parsedUrl.pathname;
    const method = req.method;
    const pathParts = path.split('/').filter(part => part !== '');
    
    res.setHeader('Content-Type', 'application/json');
    
    // GET /users/:id
    if (pathParts[0] === 'users' && pathParts[1] && method === 'GET') {
        const userId = pathParts[1];
        res.statusCode = 200;
        res.end(JSON.stringify({ 
            user: { id: userId, name: `User ${userId}` }
        }));
    }
    // GET /posts/:postId/comments/:commentId
    else if (pathParts[0] === 'posts' && pathParts[2] === 'comments' && method === 'GET') {
        const postId = pathParts[1];
        const commentId = pathParts[3];
        res.statusCode = 200;
        res.end(JSON.stringify({
            post: postId,
            comment: commentId,
            data: `Comment ${commentId} from Post ${postId}`
        }));
    }
    else {
        res.statusCode = 404;
        res.end(JSON.stringify({ error: 'Route not found' }));
    }
});

server.listen(3000);
```

---

## 📨 Handling Different HTTP Methods

### Complete CRUD Operations
```javascript
const http = require('http');
const url = require('url');

// In-memory data store
let users = [
    { id: 1, name: 'John Doe', email: 'john@example.com' },
    { id: 2, name: 'Jane Doe', email: 'jane@example.com' }
];

let nextId = 3;

const server = http.createServer((req, res) => {
    const parsedUrl = url.parse(req.url, true);
    const path = parsedUrl.pathname;
    const method = req.method;
    const query = parsedUrl.query;
    
    // Set CORS headers
    res.setHeader('Access-Control-Allow-Origin', '*');
    res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type');
    res.setHeader('Content-Type', 'application/json');
    
    // Handle preflight requests
    if (method === 'OPTIONS') {
        res.statusCode = 200;
        res.end();
        return;
    }
    
    // GET /users - Get all users
    if (path === '/users' && method === 'GET') {
        res.statusCode = 200;
        res.end(JSON.stringify({ users, count: users.length }));
    }
    // GET /users/:id - Get user by ID
    else if (path.match(/^\/users\/\d+$/) && method === 'GET') {
        const id = parseInt(path.split('/')[2]);
        const user = users.find(u => u.id === id);
        
        if (user) {
            res.statusCode = 200;
            res.end(JSON.stringify({ user }));
        } else {
            res.statusCode = 404;
            res.end(JSON.stringify({ error: 'User not found' }));
        }
    }
    // POST /users - Create new user
    else if (path === '/users' && method === 'POST') {
        let body = '';
        
        req.on('data', chunk => {
            body += chunk.toString();
        });
        
        req.on('end', () => {
            try {
                const userData = JSON.parse(body);
                const newUser = {
                    id: nextId++,
                    name: userData.name,
                    email: userData.email
                };
                
                users.push(newUser);
                
                res.statusCode = 201;
                res.end(JSON.stringify({ user: newUser }));
            } catch (error) {
                res.statusCode = 400;
                res.end(JSON.stringify({ error: 'Invalid JSON' }));
            }
        });
    }
    // PUT /users/:id - Update user
    else if (path.match(/^\/users\/\d+$/) && method === 'PUT') {
        const id = parseInt(path.split('/')[2]);
        let body = '';
        
        req.on('data', chunk => {
            body += chunk.toString();
        });
        
        req.on('end', () => {
            try {
                const userData = JSON.parse(body);
                const userIndex = users.findIndex(u => u.id === id);
                
                if (userIndex !== -1) {
                    users[userIndex] = { ...users[userIndex], ...userData };
                    res.statusCode = 200;
                    res.end(JSON.stringify({ user: users[userIndex] }));
                } else {
                    res.statusCode = 404;
                    res.end(JSON.stringify({ error: 'User not found' }));
                }
            } catch (error) {
                res.statusCode = 400;
                res.end(JSON.stringify({ error: 'Invalid JSON' }));
            }
        });
    }
    // DELETE /users/:id - Delete user
    else if (path.match(/^\/users\/\d+$/) && method === 'DELETE') {
        const id = parseInt(path.split('/')[2]);
        const userIndex = users.findIndex(u => u.id === id);
        
        if (userIndex !== -1) {
            const deletedUser = users.splice(userIndex, 1)[0];
            res.statusCode = 200;
            res.end(JSON.stringify({ message: 'User deleted', user: deletedUser }));
        } else {
            res.statusCode = 404;
            res.end(JSON.stringify({ error: 'User not found' }));
        }
    }
    // 404 Not Found
    else {
        res.statusCode = 404;
        res.end(JSON.stringify({ error: 'Route not found' }));
    }
});

server.listen(3000, () => {
    console.log('CRUD API server running on port 3000');
});
```

---

## 📄 Serving Static Files

### Basic Static File Server
```javascript
const http = require('http');
const fs = require('fs');
const path = require('path');

const server = http.createServer((req, res) => {
    let filePath = path.join(__dirname, 'public', req.url === '/' ? 'index.html' : req.url);
    
    // Get file extension
    const extname = path.extname(filePath);
    
    // Set content type based on extension
    let contentType = 'text/html';
    switch (extname) {
        case '.js':
            contentType = 'text/javascript';
            break;
        case '.css':
            contentType = 'text/css';
            break;
        case '.json':
            contentType = 'application/json';
            break;
        case '.png':
            contentType = 'image/png';
            break;
        case '.jpg':
            contentType = 'image/jpg';
            break;
        case '.gif':
            contentType = 'image/gif';
            break;
    }
    
    // Read and serve file
    fs.readFile(filePath, (err, content) => {
        if (err) {
            if (err.code === 'ENOENT') {
                // File not found
                res.writeHead(404, { 'Content-Type': 'text/html' });
                res.end('<h1>404 - File Not Found</h1>');
            } else {
                // Server error
                res.writeHead(500, { 'Content-Type': 'text/html' });
                res.end('<h1>500 - Internal Server Error</h1>');
            }
        } else {
            // Success
            res.writeHead(200, { 'Content-Type': contentType });
            res.end(content);
        }
    });
});

server.listen(3000, () => {
    console.log('Static file server running on port 3000');
});
```

---

## 🔒 Request and Response Headers

### Working with Headers
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
    // Reading request headers
    console.log('User-Agent:', req.headers['user-agent']);
    console.log('Accept:', req.headers.accept);
    console.log('Content-Type:', req.headers['content-type']);
    
    // Setting response headers
    res.setHeader('Content-Type', 'application/json');
    res.setHeader('X-API-Version', '1.0.0');
    res.setHeader('Cache-Control', 'no-cache');
    
    // Multiple values for same header
    res.setHeader('Set-Cookie', [
        'session=abc123; HttpOnly',
        'theme=dark; Max-Age=86400'
    ]);
    
    // Security headers
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('X-Frame-Options', 'DENY');
    res.setHeader('X-XSS-Protection', '1; mode=block');
    
    const responseData = {
        requestHeaders: req.headers,
        timestamp: new Date().toISOString()
    };
    
    res.statusCode = 200;
    res.end(JSON.stringify(responseData, null, 2));
});

server.listen(3000);
```

---

## 🎯 Advanced Server Features

### Server with Error Handling
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
    try {
        // Simulate potential error
        if (req.url === '/error') {
            throw new Error('Simulated server error');
        }
        
        res.statusCode = 200;
        res.setHeader('Content-Type', 'application/json');
        res.end(JSON.stringify({ message: 'Success' }));
        
    } catch (error) {
        console.error('Server error:', error.message);
        
        res.statusCode = 500;
        res.setHeader('Content-Type', 'application/json');
        res.end(JSON.stringify({ 
            error: 'Internal Server Error',
            message: error.message 
        }));
    }
});

// Handle server errors
server.on('error', (err) => {
    console.error('Server error:', err);
});

// Handle client errors
server.on('clientError', (err, socket) => {
    console.error('Client error:', err);
    socket.end('HTTP/1.1 400 Bad Request\r\n\r\n');
});

server.listen(3000, () => {
    console.log('Server with error handling running on port 3000');
});
```

### Server with Request Timeout
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
    // Set request timeout (5 seconds)
    req.setTimeout(5000, () => {
        res.statusCode = 408;
        res.end('Request Timeout');
    });
    
    // Simulate long-running operation
    if (req.url === '/slow') {
        setTimeout(() => {
            res.statusCode = 200;
            res.end('Slow response');
        }, 3000);
    } else {
        res.statusCode = 200;
        res.end('Fast response');
    }
});

server.timeout = 10000; // 10 seconds server timeout

server.listen(3000);
```

---

## 📊 Server Monitoring

### Request Logging Middleware
```javascript
const http = require('http');

function logger(req, res, next) {
    const start = Date.now();
    const timestamp = new Date().toISOString();
    
    console.log(`[${timestamp}] ${req.method} ${req.url} - Start`);
    
    // Override res.end to log completion
    const originalEnd = res.end;
    res.end = function(...args) {
        const duration = Date.now() - start;
        console.log(`[${new Date().toISOString()}] ${req.method} ${req.url} - ${res.statusCode} - ${duration}ms`);
        originalEnd.apply(this, args);
    };
    
    if (next) next();
}

const server = http.createServer((req, res) => {
    // Apply logging
    logger(req, res);
    
    // Your route logic here
    res.statusCode = 200;
    res.setHeader('Content-Type', 'application/json');
    res.end(JSON.stringify({ message: 'Hello World' }));
});

server.listen(3000);
```

---

## 🔗 Related Topics
- [[05-Express-Framework]] - Simplified HTTP server with Express
- [[06-Express-Router]] - Advanced routing patterns
- [[07-Serving-Static-Files]] - Static file serving with Express
- [[08-Headers-and-Cookies]] - Working with headers and cookies

---

*Back to [[00-Main-Index]]*
