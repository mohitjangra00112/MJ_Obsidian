# Headers and Cookies

**Navigation**: [[07-Serving-Static-Files]] | [[00-Main-Index]] | Next: [[09-Cookie-Parser]]

---

## 🔧 HTTP Headers in Express

HTTP headers provide **metadata** about requests and responses. They're essential for security, caching, content type specification, and more.

---

## 📨 Request Headers

### Reading Request Headers
```javascript
const express = require('express');
const app = express();

app.get('/headers', (req, res) => {
    // Get all headers
    const allHeaders = req.headers;
    
    // Get specific headers
    const userAgent = req.get('User-Agent');
    const contentType = req.get('Content-Type');
    const authorization = req.get('Authorization');
    const accept = req.get('Accept');
    
    // Headers are case-insensitive
    const host = req.headers.host;
    const referer = req.headers.referer;
    
    res.json({
        allHeaders,
        specific: {
            userAgent,
            contentType,
            authorization,
            accept,
            host,
            referer
        }
    });
});

app.listen(3000);
```

### Custom Request Headers
```javascript
const express = require('express');
const app = express();

app.use(express.json());

// Middleware to check custom headers
const checkApiKey = (req, res, next) => {
    const apiKey = req.get('X-API-Key');
    
    if (!apiKey) {
        return res.status(401).json({ error: 'API key required' });
    }
    
    if (apiKey !== 'your-secret-api-key') {
        return res.status(401).json({ error: 'Invalid API key' });
    }
    
    next();
};

// Protected route with custom header
app.get('/protected', checkApiKey, (req, res) => {
    res.json({ 
        message: 'Access granted',
        apiKey: req.get('X-API-Key'),
        timestamp: new Date().toISOString()
    });
});

app.listen(3000);
```

---

## 📤 Response Headers

### Setting Response Headers
```javascript
const express = require('express');
const app = express();

app.get('/response-headers', (req, res) => {
    // Set individual headers
    res.set('X-API-Version', '1.0.0');
    res.set('X-Response-Time', '100ms');
    
    // Set multiple headers at once
    res.set({
        'Cache-Control': 'no-cache',
        'Pragma': 'no-cache',
        'Expires': '0'
    });
    
    // Alternative method
    res.header('X-Custom-Header', 'custom-value');
    
    // Set content type
    res.type('application/json');
    // or res.set('Content-Type', 'application/json');
    
    res.json({ 
        message: 'Headers set successfully',
        timestamp: new Date().toISOString()
    });
});

// Headers with specific content types
app.get('/xml', (req, res) => {
    res.set('Content-Type', 'application/xml');
    res.send(`<?xml version="1.0"?>
        <response>
            <message>XML Response</message>
        </response>`);
});

app.get('/csv', (req, res) => {
    res.set({
        'Content-Type': 'text/csv',
        'Content-Disposition': 'attachment; filename="data.csv"'
    });
    res.send('Name,Age,Email\nJohn,30,john@example.com\nJane,25,jane@example.com');
});

app.listen(3000);
```

---

## 🔒 Security Headers

### Essential Security Headers
```javascript
const express = require('express');
const helmet = require('helmet'); // npm install helmet
const app = express();

// Use Helmet for basic security headers
app.use(helmet());

// Or set security headers manually
const securityHeaders = (req, res, next) => {
    // Prevent XSS attacks
    res.set('X-XSS-Protection', '1; mode=block');
    
    // Prevent MIME type sniffing
    res.set('X-Content-Type-Options', 'nosniff');
    
    // Prevent clickjacking
    res.set('X-Frame-Options', 'DENY');
    
    // Strict Transport Security (HTTPS only)
    res.set('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
    
    // Content Security Policy
    res.set('Content-Security-Policy', 
        "default-src 'self'; " +
        "script-src 'self' 'unsafe-inline'; " +
        "style-src 'self' 'unsafe-inline'; " +
        "img-src 'self' data: https:;"
    );
    
    // Referrer Policy
    res.set('Referrer-Policy', 'strict-origin-when-cross-origin');
    
    // Feature Policy (deprecated, use Permissions-Policy)
    res.set('Permissions-Policy', 
        'geolocation=(), microphone=(), camera=()'
    );
    
    next();
};

app.use(securityHeaders);

app.get('/', (req, res) => {
    res.json({ message: 'Secure response with security headers' });
});

app.listen(3000);
```

---

## 🍪 Working with Cookies

### Setting Cookies
```javascript
const express = require('express');
const app = express();

app.get('/set-cookies', (req, res) => {
    // Basic cookie
    res.cookie('username', 'john_doe');
    
    // Cookie with options
    res.cookie('session_id', 'abc123', {
        maxAge: 24 * 60 * 60 * 1000, // 24 hours in milliseconds
        httpOnly: true, // Not accessible via JavaScript
        secure: false, // Set to true in production with HTTPS
        sameSite: 'strict' // CSRF protection
    });
    
    // Signed cookie (requires cookie-parser with secret)
    res.cookie('user_id', '12345', { 
        signed: true,
        maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
    });
    
    // Cookie with domain and path
    res.cookie('preferences', 'dark-theme', {
        domain: 'localhost',
        path: '/user',
        maxAge: 30 * 24 * 60 * 60 * 1000 // 30 days
    });
    
    res.json({ message: 'Cookies set successfully' });
});

// Reading cookies (requires cookie-parser middleware)
app.get('/get-cookies', (req, res) => {
    res.json({
        cookies: req.cookies,
        signedCookies: req.signedCookies
    });
});

// Clearing cookies
app.get('/clear-cookies', (req, res) => {
    // Clear specific cookie
    res.clearCookie('username');
    
    // Clear with options (must match original options)
    res.clearCookie('session_id', {
        httpOnly: true,
        secure: false,
        sameSite: 'strict'
    });
    
    res.json({ message: 'Cookies cleared' });
});

app.listen(3000);
```

### Advanced Cookie Management
```javascript
const express = require('express');
const cookieParser = require('cookie-parser');
const app = express();

// Use cookie-parser with secret for signed cookies
app.use(cookieParser('your-secret-key'));

// Cookie utility functions
const cookieUtils = {
    setAuthCookie: (res, token) => {
        res.cookie('auth_token', token, {
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'strict',
            maxAge: 24 * 60 * 60 * 1000 // 24 hours
        });
    },
    
    setUserPreferences: (res, preferences) => {
        res.cookie('user_prefs', JSON.stringify(preferences), {
            maxAge: 30 * 24 * 60 * 60 * 1000, // 30 days
            sameSite: 'lax'
        });
    },
    
    clearAuthCookie: (res) => {
        res.clearCookie('auth_token', {
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'strict'
        });
    }
};

// Login endpoint
app.post('/login', (req, res) => {
    // Validate credentials (simplified)
    const token = 'jwt-token-here';
    
    // Set authentication cookie
    cookieUtils.setAuthCookie(res, token);
    
    // Set user preferences
    cookieUtils.setUserPreferences(res, {
        theme: 'dark',
        language: 'en',
        notifications: true
    });
    
    res.json({ message: 'Login successful' });
});

// Logout endpoint
app.post('/logout', (req, res) => {
    cookieUtils.clearAuthCookie(res);
    res.clearCookie('user_prefs');
    
    res.json({ message: 'Logout successful' });
});

// Protected route using cookies
app.get('/profile', (req, res) => {
    const authToken = req.cookies.auth_token;
    
    if (!authToken) {
        return res.status(401).json({ error: 'Authentication required' });
    }
    
    // Validate token (simplified)
    if (authToken !== 'jwt-token-here') {
        return res.status(401).json({ error: 'Invalid token' });
    }
    
    const userPrefs = req.cookies.user_prefs ? 
        JSON.parse(req.cookies.user_prefs) : {};
    
    res.json({
        message: 'Profile data',
        preferences: userPrefs,
        authenticated: true
    });
});

app.listen(3000);
```

---

## 🚀 CORS Headers

### Cross-Origin Resource Sharing
```javascript
const express = require('express');
const cors = require('cors'); // npm install cors
const app = express();

// Basic CORS
app.use(cors());

// Custom CORS configuration
const corsOptions = {
    origin: ['http://localhost:3000', 'https://mydomain.com'],
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization', 'X-API-Key'],
    credentials: true, // Allow cookies
    maxAge: 86400 // Preflight cache time
};

app.use(cors(corsOptions));

// Manual CORS headers
const customCors = (req, res, next) => {
    res.header('Access-Control-Allow-Origin', '*');
    res.header('Access-Control-Allow-Methods', 'GET,PUT,POST,DELETE,OPTIONS');
    res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    
    // Handle preflight requests
    if (req.method === 'OPTIONS') {
        res.sendStatus(200);
    } else {
        next();
    }
};

// API routes
app.get('/api/data', (req, res) => {
    res.json({
        message: 'CORS-enabled response',
        origin: req.get('Origin'),
        timestamp: new Date().toISOString()
    });
});

app.listen(3000);
```

---

## 📊 Caching Headers

### Cache Control Headers
```javascript
const express = require('express');
const app = express();

// Cache control middleware
const cacheControl = (duration) => {
    return (req, res, next) => {
        res.set('Cache-Control', `public, max-age=${duration}`);
        next();
    };
};

// Different caching strategies
app.get('/static-data', cacheControl(3600), (req, res) => {
    // Cache for 1 hour
    res.json({
        data: 'This data rarely changes',
        timestamp: new Date().toISOString()
    });
});

app.get('/dynamic-data', (req, res) => {
    // No cache for dynamic data
    res.set({
        'Cache-Control': 'no-cache, no-store, must-revalidate',
        'Pragma': 'no-cache',
        'Expires': '0'
    });
    
    res.json({
        data: 'This data changes frequently',
        timestamp: new Date().toISOString()
    });
});

// ETag support
app.get('/etag-demo', (req, res) => {
    const data = { message: 'ETag demo', id: 1 };
    const etag = `"${Buffer.from(JSON.stringify(data)).toString('base64')}"`;
    
    res.set('ETag', etag);
    
    // Check if client has current version
    if (req.get('If-None-Match') === etag) {
        return res.status(304).end(); // Not Modified
    }
    
    res.json(data);
});

app.listen(3000);
```

---

## 🔧 Custom Header Middleware

### Header Management Middleware
```javascript
const express = require('express');
const app = express();

// Request timing middleware
const requestTimer = (req, res, next) => {
    const start = Date.now();
    
    // Override res.end to calculate response time
    const originalEnd = res.end;
    res.end = function(...args) {
        const duration = Date.now() - start;
        res.set('X-Response-Time', `${duration}ms`);
        originalEnd.apply(this, args);
    };
    
    next();
};

// Request ID middleware
const requestId = (req, res, next) => {
    const id = Math.random().toString(36).substr(2, 9);
    req.requestId = id;
    res.set('X-Request-ID', id);
    next();
};

// Rate limit headers
const rateLimitHeaders = (req, res, next) => {
    const limit = 100;
    const remaining = Math.floor(Math.random() * limit);
    const reset = Date.now() + (15 * 60 * 1000); // 15 minutes
    
    res.set({
        'X-RateLimit-Limit': limit,
        'X-RateLimit-Remaining': remaining,
        'X-RateLimit-Reset': reset
    });
    
    next();
};

// Apply middleware
app.use(requestTimer);
app.use(requestId);
app.use(rateLimitHeaders);

app.get('/api/test', (req, res) => {
    res.json({
        message: 'Test endpoint',
        requestId: req.requestId,
        timestamp: new Date().toISOString()
    });
});

app.listen(3000);
```

---

## 🔗 Related Topics
- [[09-Cookie-Parser]] - Advanced cookie handling
- [[10-Express-Sessions]] - Session management with cookies
- [[11-Authentication-and-Authorization]] - Using headers for auth
- [[13-Rate-Limiting]] - Rate limiting with headers

---

*Back to [[00-Main-Index]]*
