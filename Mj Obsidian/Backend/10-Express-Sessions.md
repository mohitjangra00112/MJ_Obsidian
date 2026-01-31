# Express Sessions

**Navigation**: [[09-Cookie-Parser]] | [[00-Main-Index]] | Next: [[11-Authentication-and-Authorization]]

---

## 🗂️ Express Sessions

Express Sessions provide a way to **store user data** across multiple requests. Sessions use cookies to maintain state but store the actual data server-side for better security.

---

## 📦 Installation and Basic Setup

### Installing express-session
```bash
npm install express-session
```

### Basic Session Configuration
```javascript
const express = require('express');
const session = require('express-session');

const app = express();

// Basic session configuration
app.use(session({
    secret: 'your-secret-key', // Change this in production
    resave: false,             // Don't save session if unmodified
    saveUninitialized: false,  // Don't create session until something stored
    cookie: {
        secure: false,         // Set to true with HTTPS
        maxAge: 24 * 60 * 60 * 1000 // 24 hours
    }
}));

app.get('/', (req, res) => {
    if (!req.session.views) {
        req.session.views = 0;
    }
    req.session.views++;
    
    res.json({
        message: 'Session demo',
        views: req.session.views,
        sessionId: req.sessionID
    });
});

app.listen(3000, () => {
    console.log('Session server running on port 3000');
});
```

---

## 🔧 Advanced Session Configuration

### Production-Ready Session Setup
```javascript
const express = require('express');
const session = require('express-session');
const MongoStore = require('connect-mongo'); // For MongoDB session store

const app = express();

// Production session configuration
app.use(session({
    secret: process.env.SESSION_SECRET || 'fallback-secret',
    name: 'sessionId', // Change default session name
    resave: false,
    saveUninitialized: false,
    rolling: true, // Reset expiration on activity
    
    // Cookie settings
    cookie: {
        secure: process.env.NODE_ENV === 'production', // HTTPS only in production
        httpOnly: true,    // Prevent XSS
        maxAge: 30 * 60 * 1000, // 30 minutes
        sameSite: 'strict' // CSRF protection
    },
    
    // Session store (use MongoDB, Redis, etc. in production)
    store: MongoStore.create({
        mongoUrl: process.env.MONGODB_URI || 'mongodb://localhost:27017/sessions',
        touchAfter: 24 * 3600 // Lazy session update
    })
}));

app.listen(3000);
```

### Memory Store vs Persistent Store
```javascript
const express = require('express');
const session = require('express-session');
const RedisStore = require('connect-redis')(session);
const redis = require('redis');

const app = express();

// Redis client
const redisClient = redis.createClient({
    host: process.env.REDIS_HOST || 'localhost',
    port: process.env.REDIS_PORT || 6379,
    password: process.env.REDIS_PASSWORD
});

// Session with Redis store
app.use(session({
    store: new RedisStore({ client: redisClient }),
    secret: process.env.SESSION_SECRET,
    resave: false,
    saveUninitialized: false,
    cookie: {
        secure: process.env.NODE_ENV === 'production',
        httpOnly: true,
        maxAge: 60 * 60 * 1000 // 1 hour
    }
}));

app.listen(3000);
```

---

## 🔐 Session-Based Authentication

### Complete Authentication System
```javascript
const express = require('express');
const session = require('express-session');
const bcrypt = require('bcrypt');

const app = express();

app.use(express.json());
app.use(session({
    secret: 'auth-secret-key',
    resave: false,
    saveUninitialized: false,
    cookie: {
        secure: false, // Set to true with HTTPS
        httpOnly: true,
        maxAge: 24 * 60 * 60 * 1000 // 24 hours
    }
}));

// Mock user database
const users = [
    {
        id: 1,
        username: 'admin',
        email: 'admin@example.com',
        password: '$2b$10$hash...', // bcrypt hash of 'password123'
        role: 'admin'
    },
    {
        id: 2,
        username: 'user',
        email: 'user@example.com',
        password: '$2b$10$hash...', // bcrypt hash of 'userpass'
        role: 'user'
    }
];

// Authentication middleware
const requireAuth = (req, res, next) => {
    if (req.session && req.session.userId) {
        return next();
    } else {
        return res.status(401).json({ error: 'Authentication required' });
    }
};

const requireAdmin = (req, res, next) => {
    if (req.session && req.session.user && req.session.user.role === 'admin') {
        return next();
    } else {
        return res.status(403).json({ error: 'Admin access required' });
    }
};

// Registration endpoint
app.post('/register', async (req, res) => {
    try {
        const { username, email, password } = req.body;
        
        // Check if user exists
        const existingUser = users.find(u => u.email === email || u.username === username);
        if (existingUser) {
            return res.status(409).json({ error: 'User already exists' });
        }
        
        // Hash password
        const hashedPassword = await bcrypt.hash(password, 10);
        
        // Create new user
        const newUser = {
            id: users.length + 1,
            username,
            email,
            password: hashedPassword,
            role: 'user'
        };
        
        users.push(newUser);
        
        // Create session
        req.session.userId = newUser.id;
        req.session.user = {
            id: newUser.id,
            username: newUser.username,
            email: newUser.email,
            role: newUser.role
        };
        
        res.status(201).json({
            message: 'Registration successful',
            user: req.session.user
        });
        
    } catch (error) {
        res.status(500).json({ error: 'Registration failed' });
    }
});

// Login endpoint
app.post('/login', async (req, res) => {
    try {
        const { username, password } = req.body;
        
        // Find user
        const user = users.find(u => u.username === username || u.email === username);
        if (!user) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        // Verify password
        const isValidPassword = await bcrypt.compare(password, user.password);
        if (!isValidPassword) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        // Create session
        req.session.userId = user.id;
        req.session.user = {
            id: user.id,
            username: user.username,
            email: user.email,
            role: user.role
        };
        
        res.json({
            message: 'Login successful',
            user: req.session.user
        });
        
    } catch (error) {
        res.status(500).json({ error: 'Login failed' });
    }
});

// Logout endpoint
app.post('/logout', (req, res) => {
    req.session.destroy((err) => {
        if (err) {
            return res.status(500).json({ error: 'Logout failed' });
        }
        
        res.clearCookie('connect.sid'); // Default session cookie name
        res.json({ message: 'Logout successful' });
    });
});

// Profile endpoint
app.get('/profile', requireAuth, (req, res) => {
    res.json({
        message: 'Profile data',
        user: req.session.user,
        sessionInfo: {
            sessionId: req.sessionID,
            cookie: req.session.cookie
        }
    });
});

// Admin only endpoint
app.get('/admin', requireAuth, requireAdmin, (req, res) => {
    res.json({
        message: 'Admin panel access granted',
        user: req.session.user,
        adminData: {
            totalUsers: users.length,
            activeUsers: 'Session count would go here'
        }
    });
});

app.listen(3000);
```

---

## 🛍️ Shopping Cart with Sessions

### Session-Based Shopping Cart
```javascript
const express = require('express');
const session = require('express-session');

const app = express();

app.use(express.json());
app.use(session({
    secret: 'cart-secret',
    resave: false,
    saveUninitialized: true, // Create session for cart even if user not logged in
    cookie: {
        maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days for cart persistence
    }
}));

// Mock products
const products = [
    { id: 1, name: 'Laptop', price: 999.99, stock: 10 },
    { id: 2, name: 'Mouse', price: 29.99, stock: 50 },
    { id: 3, name: 'Keyboard', price: 79.99, stock: 25 }
];

// Initialize cart in session
const initializeCart = (req, res, next) => {
    if (!req.session.cart) {
        req.session.cart = [];
    }
    next();
};

app.use(initializeCart);

// Cart utilities
const cartUtils = {
    findItem: (cart, productId) => {
        return cart.find(item => item.productId === productId);
    },
    
    calculateTotal: (cart) => {
        return cart.reduce((total, item) => total + (item.price * item.quantity), 0);
    },
    
    getCartSummary: (cart) => {
        return {
            items: cart,
            totalItems: cart.reduce((sum, item) => sum + item.quantity, 0),
            totalPrice: cartUtils.calculateTotal(cart)
        };
    }
};

// Add item to cart
app.post('/cart/add', (req, res) => {
    const { productId, quantity = 1 } = req.body;
    
    // Find product
    const product = products.find(p => p.id === productId);
    if (!product) {
        return res.status(404).json({ error: 'Product not found' });
    }
    
    // Check stock
    if (product.stock < quantity) {
        return res.status(400).json({ error: 'Insufficient stock' });
    }
    
    // Check if item already in cart
    const existingItem = cartUtils.findItem(req.session.cart, productId);
    
    if (existingItem) {
        existingItem.quantity += quantity;
    } else {
        req.session.cart.push({
            productId: product.id,
            name: product.name,
            price: product.price,
            quantity: quantity
        });
    }
    
    res.json({
        message: 'Item added to cart',
        cart: cartUtils.getCartSummary(req.session.cart)
    });
});

// Update cart item
app.put('/cart/update/:productId', (req, res) => {
    const productId = parseInt(req.params.productId);
    const { quantity } = req.body;
    
    if (quantity <= 0) {
        return res.status(400).json({ error: 'Quantity must be positive' });
    }
    
    const item = cartUtils.findItem(req.session.cart, productId);
    if (!item) {
        return res.status(404).json({ error: 'Item not found in cart' });
    }
    
    item.quantity = quantity;
    
    res.json({
        message: 'Cart updated',
        cart: cartUtils.getCartSummary(req.session.cart)
    });
});

// Remove item from cart
app.delete('/cart/remove/:productId', (req, res) => {
    const productId = parseInt(req.params.productId);
    
    req.session.cart = req.session.cart.filter(item => item.productId !== productId);
    
    res.json({
        message: 'Item removed from cart',
        cart: cartUtils.getCartSummary(req.session.cart)
    });
});

// Get cart
app.get('/cart', (req, res) => {
    res.json({
        cart: cartUtils.getCartSummary(req.session.cart)
    });
});

// Clear cart
app.delete('/cart', (req, res) => {
    req.session.cart = [];
    
    res.json({
        message: 'Cart cleared',
        cart: cartUtils.getCartSummary(req.session.cart)
    });
});

// Checkout
app.post('/checkout', (req, res) => {
    if (req.session.cart.length === 0) {
        return res.status(400).json({ error: 'Cart is empty' });
    }
    
    const cartSummary = cartUtils.getCartSummary(req.session.cart);
    
    // Process order (simplified)
    const order = {
        id: Date.now(),
        items: req.session.cart,
        total: cartSummary.totalPrice,
        timestamp: new Date().toISOString()
    };
    
    // Clear cart after checkout
    req.session.cart = [];
    
    res.json({
        message: 'Order placed successfully',
        order: order
    });
});

app.listen(3000);
```

---

## 📊 Session Management and Analytics

### Session Monitoring and Management
```javascript
const express = require('express');
const session = require('express-session');

const app = express();

// Session store for tracking active sessions
const activeSessions = new Map();

app.use(session({
    secret: 'monitoring-secret',
    resave: false,
    saveUninitialized: false,
    cookie: {
        maxAge: 60 * 60 * 1000 // 1 hour
    }
}));

// Session tracking middleware
const sessionTracker = (req, res, next) => {
    const sessionId = req.sessionID;
    
    // Track session activity
    if (req.session) {
        activeSessions.set(sessionId, {
            sessionId: sessionId,
            userId: req.session.userId || null,
            lastActivity: new Date(),
            userAgent: req.get('User-Agent'),
            ip: req.ip,
            pages: activeSessions.get(sessionId)?.pages || 0
        });
        
        // Increment page views
        const sessionData = activeSessions.get(sessionId);
        sessionData.pages++;
        activeSessions.set(sessionId, sessionData);
    }
    
    next();
};

app.use(sessionTracker);

// Session analytics endpoints
app.get('/admin/sessions', (req, res) => {
    const sessions = Array.from(activeSessions.values());
    
    res.json({
        activeSessions: sessions.length,
        sessions: sessions.map(session => ({
            sessionId: session.sessionId,
            userId: session.userId,
            lastActivity: session.lastActivity,
            pages: session.pages,
            duration: new Date() - new Date(session.lastActivity)
        }))
    });
});

// Clean up expired sessions
setInterval(() => {
    const now = new Date();
    const expireTime = 60 * 60 * 1000; // 1 hour
    
    for (const [sessionId, sessionData] of activeSessions.entries()) {
        if (now - sessionData.lastActivity > expireTime) {
            activeSessions.delete(sessionId);
            console.log(`Cleaned up expired session: ${sessionId}`);
        }
    }
}, 10 * 60 * 1000); // Clean up every 10 minutes

// Session info endpoint
app.get('/session-info', (req, res) => {
    const sessionData = activeSessions.get(req.sessionID);
    
    res.json({
        sessionId: req.sessionID,
        sessionData: sessionData,
        cookie: req.session.cookie,
        isAuthenticated: !!req.session.userId
    });
});

app.listen(3000);
```

---

## 🔒 Session Security Best Practices

### Secure Session Implementation
```javascript
const express = require('express');
const session = require('express-session');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

const app = express();

// Security middleware
app.use(helmet());

// Rate limiting
const loginLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5, // 5 attempts per window
    message: 'Too many login attempts, please try again later',
    standardHeaders: true,
    legacyHeaders: false,
});

// Secure session configuration
app.use(session({
    secret: process.env.SESSION_SECRET,
    name: 'sessionId', // Don't use default name
    resave: false,
    saveUninitialized: false,
    rolling: true, // Reset expiration on activity
    
    cookie: {
        secure: process.env.NODE_ENV === 'production',
        httpOnly: true,
        maxAge: 30 * 60 * 1000, // 30 minutes
        sameSite: 'strict'
    }
}));

// Session security middleware
const sessionSecurity = (req, res, next) => {
    // Regenerate session ID on login
    if (req.path === '/login' && req.method === 'POST') {
        req.session.regenerate((err) => {
            if (err) {
                return next(err);
            }
            next();
        });
    } else {
        next();
    }
};

app.use(sessionSecurity);

// Secure login with rate limiting
app.post('/login', loginLimiter, async (req, res) => {
    // Login logic with session regeneration
    // ... authentication code ...
    
    // After successful authentication
    req.session.userId = user.id;
    req.session.loginTime = new Date();
    
    res.json({ message: 'Login successful' });
});

// Session validation middleware
const validateSession = (req, res, next) => {
    if (req.session && req.session.userId) {
        // Check session age
        const sessionAge = new Date() - new Date(req.session.loginTime);
        const maxSessionAge = 8 * 60 * 60 * 1000; // 8 hours
        
        if (sessionAge > maxSessionAge) {
            req.session.destroy();
            return res.status(401).json({ error: 'Session expired' });
        }
        
        next();
    } else {
        res.status(401).json({ error: 'Authentication required' });
    }
};

app.listen(3000);
```

---

## 🔗 Related Topics
- [[09-Cookie-Parser]] - Cookie handling basics
- [[11-Authentication-and-Authorization]] - Authentication strategies
- [[12-JWT-Tokens]] - Alternative to sessions
- [[14-Environment-Variables]] - Securing session secrets

---

*Back to [[00-Main-Index]]*
