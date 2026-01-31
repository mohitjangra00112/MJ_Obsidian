# Express Sessions

**Navigation**: [[16-Joi-Validation]] | [[00-Main-Index]] | Next: [[18-MVC-Architecture]]

---

## 🔐 Express Sessions Management

**Express Sessions** provide a way to **store user data** across multiple requests. Sessions are essential for maintaining user state, implementing authentication, and creating personalized user experiences.

---

## 📦 Session Setup and Configuration

### Basic Session Setup
```bash
npm install express-session connect-redis redis
```

```javascript
// app.js
const express = require('express');
const session = require('express-session');
const RedisStore = require('connect-redis')(session);
const redis = require('redis');

const app = express();

// Redis client setup
const redisClient = redis.createClient({
    host: process.env.REDIS_HOST || 'localhost',
    port: process.env.REDIS_PORT || 6379,
    password: process.env.REDIS_PASSWORD
});

// Session configuration
app.use(session({
    store: new RedisStore({ client: redisClient }),
    secret: process.env.SESSION_SECRET || 'your-secret-key',
    resave: false,                          // Don't save session if unmodified
    saveUninitialized: false,               // Don't create session until something stored
    rolling: true,                          // Reset expiration on activity
    cookie: {
        secure: process.env.NODE_ENV === 'production',  // HTTPS only in production
        httpOnly: true,                     // Prevent XSS attacks
        maxAge: 24 * 60 * 60 * 1000,       // 24 hours
        sameSite: 'strict'                  // CSRF protection
    },
    name: 'sessionId',                      // Change default session name
    genid: () => {
        return require('crypto').randomBytes(16).toString('hex');
    }
}));

// Session usage examples
app.get('/login', (req, res) => {
    // Check if user is already logged in
    if (req.session.userId) {
        return res.redirect('/dashboard');
    }
    
    res.render('login');
});

app.post('/login', async (req, res) => {
    const { email, password } = req.body;
    
    try {
        // Validate user credentials
        const user = await authenticateUser(email, password);
        
        if (user) {
            // Store user data in session
            req.session.userId = user.id;
            req.session.email = user.email;
            req.session.role = user.role;
            req.session.loginTime = new Date();
            
            res.json({ success: true, redirect: '/dashboard' });
        } else {
            res.status(401).json({ error: 'Invalid credentials' });
        }
    } catch (error) {
        res.status(500).json({ error: 'Login failed' });
    }
});

app.post('/logout', (req, res) => {
    req.session.destroy((err) => {
        if (err) {
            return res.status(500).json({ error: 'Logout failed' });
        }
        
        res.clearCookie('sessionId');
        res.json({ success: true, redirect: '/login' });
    });
});

app.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

---

## 🛡️ Session Security and Best Practices

### Secure Session Configuration
```javascript
// config/session.js
const session = require('express-session');
const MongoStore = require('connect-mongo');
const RedisStore = require('connect-redis')(session);
const redis = require('redis');

class SessionManager {
    static createRedisStore() {
        const redisClient = redis.createClient({
            url: process.env.REDIS_URL,
            password: process.env.REDIS_PASSWORD,
            socket: {
                tls: process.env.NODE_ENV === 'production',
                rejectUnauthorized: false
            }
        });
        
        redisClient.on('error', (err) => {
            console.error('Redis Client Error:', err);
        });
        
        redisClient.on('connect', () => {
            console.log('Connected to Redis');
        });
        
        return new RedisStore({ 
            client: redisClient,
            prefix: 'sess:',
            ttl: 86400 // 24 hours in seconds
        });
    }
    
    static createMongoStore() {
        return MongoStore.create({
            mongoUrl: process.env.MONGODB_URI,
            touchAfter: 24 * 3600, // Lazy session update
            crypto: {
                secret: process.env.SESSION_SECRET
            },
            collectionName: 'sessions'
        });
    }
    
    static getSessionConfig(environment = 'development') {
        const isProduction = environment === 'production';
        
        const baseConfig = {
            secret: process.env.SESSION_SECRET,
            resave: false,
            saveUninitialized: false,
            rolling: true,
            cookie: {
                secure: isProduction,
                httpOnly: true,
                sameSite: isProduction ? 'none' : 'lax',
                maxAge: parseInt(process.env.SESSION_MAX_AGE) || 24 * 60 * 60 * 1000
            },
            name: process.env.SESSION_NAME || 'sessionId'
        };
        
        // Choose store based on environment
        if (process.env.REDIS_URL) {
            baseConfig.store = SessionManager.createRedisStore();
        } else if (process.env.MONGODB_URI) {
            baseConfig.store = SessionManager.createMongoStore();
        }
        
        // Production-specific settings
        if (isProduction) {
            baseConfig.proxy = true; // Trust reverse proxy
            baseConfig.cookie.domain = process.env.COOKIE_DOMAIN;
        }
        
        return baseConfig;
    }
    
    static middleware() {
        return session(SessionManager.getSessionConfig(process.env.NODE_ENV));
    }
}

module.exports = SessionManager;
```

### Session Security Middleware
```javascript
// middleware/sessionSecurity.js
class SessionSecurity {
    // Session fixation protection
    static regenerateSessionId() {
        return (req, res, next) => {
            if (req.session && req.session.userId && !req.session.regenerated) {
                req.session.regenerate((err) => {
                    if (err) {
                        return next(err);
                    }
                    req.session.regenerated = true;
                    next();
                });
            } else {
                next();
            }
        };
    }
    
    // Session timeout check
    static checkSessionTimeout(timeoutMinutes = 30) {
        return (req, res, next) => {
            if (req.session.lastActivity) {
                const now = new Date();
                const lastActivity = new Date(req.session.lastActivity);
                const timeDiff = (now - lastActivity) / (1000 * 60); // minutes
                
                if (timeDiff > timeoutMinutes) {
                    req.session.destroy((err) => {
                        if (err) console.error('Session destroy error:', err);
                    });
                    return res.status(401).json({ 
                        error: 'Session expired',
                        code: 'SESSION_TIMEOUT'
                    });
                }
            }
            
            req.session.lastActivity = new Date();
            next();
        };
    }
    
    // Concurrent session limiting
    static limitConcurrentSessions(maxSessions = 3) {
        return async (req, res, next) => {
            if (req.session.userId) {
                const userId = req.session.userId;
                const sessionStore = req.sessionStore;
                
                // Get all sessions for this user
                sessionStore.all((err, sessions) => {
                    if (err) return next(err);
                    
                    const userSessions = Object.values(sessions || {})
                        .filter(session => session.userId === userId)
                        .sort((a, b) => new Date(b.lastActivity) - new Date(a.lastActivity));
                    
                    // Remove excess sessions
                    if (userSessions.length > maxSessions) {
                        const sessionsToRemove = userSessions.slice(maxSessions);
                        sessionsToRemove.forEach(session => {
                            sessionStore.destroy(session.id, (err) => {
                                if (err) console.error('Session destroy error:', err);
                            });
                        });
                    }
                    
                    next();
                });
            } else {
                next();
            }
        };
    }
    
    // IP address validation
    static validateIpAddress() {
        return (req, res, next) => {
            if (req.session.ipAddress) {
                const currentIp = req.ip || req.connection.remoteAddress;
                if (req.session.ipAddress !== currentIp) {
                    req.session.destroy();
                    return res.status(401).json({
                        error: 'IP address mismatch',
                        code: 'IP_MISMATCH'
                    });
                }
            } else if (req.session.userId) {
                req.session.ipAddress = req.ip || req.connection.remoteAddress;
            }
            
            next();
        };
    }
    
    // User agent validation
    static validateUserAgent() {
        return (req, res, next) => {
            if (req.session.userAgent) {
                if (req.session.userAgent !== req.get('User-Agent')) {
                    req.session.destroy();
                    return res.status(401).json({
                        error: 'User agent mismatch',
                        code: 'USER_AGENT_MISMATCH'
                    });
                }
            } else if (req.session.userId) {
                req.session.userAgent = req.get('User-Agent');
            }
            
            next();
        };
    }
}

module.exports = SessionSecurity;
```

---

## 🔧 Session-Based Authentication System

### Authentication Service
```javascript
// services/authService.js
const bcrypt = require('bcrypt');
const User = require('../models/User');

class AuthService {
    static async login(req, email, password) {
        try {
            // Find user by email
            const user = await User.findOne({ email: email.toLowerCase() });
            if (!user) {
                throw new Error('Invalid credentials');
            }
            
            // Check if account is locked
            if (user.lockUntil && user.lockUntil > Date.now()) {
                throw new Error('Account temporarily locked');
            }
            
            // Verify password
            const isValidPassword = await bcrypt.compare(password, user.password);
            if (!isValidPassword) {
                // Increment failed login attempts
                await AuthService.handleFailedLogin(user);
                throw new Error('Invalid credentials');
            }
            
            // Reset failed login attempts on successful login
            if (user.loginAttempts > 0) {
                await User.findByIdAndUpdate(user._id, {
                    $unset: { loginAttempts: 1, lockUntil: 1 }
                });
            }
            
            // Create session
            await AuthService.createSession(req, user);
            
            // Update last login
            await User.findByIdAndUpdate(user._id, {
                lastLogin: new Date(),
                lastIpAddress: req.ip
            });
            
            return {
                user: {
                    id: user._id,
                    email: user.email,
                    role: user.role,
                    firstName: user.firstName,
                    lastName: user.lastName
                },
                sessionId: req.sessionID
            };
            
        } catch (error) {
            throw error;
        }
    }
    
    static async createSession(req, user) {
        return new Promise((resolve, reject) => {
            req.session.regenerate((err) => {
                if (err) return reject(err);
                
                // Store user data in session
                req.session.userId = user._id.toString();
                req.session.email = user.email;
                req.session.role = user.role;
                req.session.loginTime = new Date();
                req.session.lastActivity = new Date();
                req.session.ipAddress = req.ip;
                req.session.userAgent = req.get('User-Agent');
                
                // Set session flags
                req.session.isAuthenticated = true;
                req.session.regenerated = true;
                
                req.session.save((err) => {
                    if (err) return reject(err);
                    resolve(req.session);
                });
            });
        });
    }
    
    static async handleFailedLogin(user) {
        const maxAttempts = 5;
        const lockDuration = 30 * 60 * 1000; // 30 minutes
        
        const update = { $inc: { loginAttempts: 1 } };
        
        // Lock account if max attempts reached
        if (user.loginAttempts >= maxAttempts - 1) {
            update.$set = { lockUntil: Date.now() + lockDuration };
        }
        
        await User.findByIdAndUpdate(user._id, update);
    }
    
    static async logout(req) {
        return new Promise((resolve, reject) => {
            const userId = req.session.userId;
            
            req.session.destroy((err) => {
                if (err) return reject(err);
                resolve(userId);
            });
        });
    }
    
    static async getCurrentUser(req) {
        if (!req.session.userId) {
            return null;
        }
        
        try {
            const user = await User.findById(req.session.userId).select('-password');
            return user;
        } catch (error) {
            return null;
        }
    }
    
    static isAuthenticated(req) {
        return !!(req.session && req.session.userId && req.session.isAuthenticated);
    }
    
    static hasRole(req, requiredRole) {
        if (!AuthService.isAuthenticated(req)) {
            return false;
        }
        
        const userRole = req.session.role;
        const roleHierarchy = {
            'user': 1,
            'moderator': 2,
            'admin': 3,
            'superadmin': 4
        };
        
        return roleHierarchy[userRole] >= roleHierarchy[requiredRole];
    }
}

module.exports = AuthService;
```

### Authentication Middleware
```javascript
// middleware/auth.js
const AuthService = require('../services/authService');

class AuthMiddleware {
    // Require authentication
    static requireAuth(req, res, next) {
        if (!AuthService.isAuthenticated(req)) {
            return res.status(401).json({
                error: 'Authentication required',
                code: 'AUTH_REQUIRED'
            });
        }
        next();
    }
    
    // Require specific role
    static requireRole(role) {
        return (req, res, next) => {
            if (!AuthService.hasRole(req, role)) {
                return res.status(403).json({
                    error: 'Insufficient permissions',
                    code: 'INSUFFICIENT_PERMISSIONS',
                    requiredRole: role,
                    userRole: req.session.role
                });
            }
            next();
        };
    }
    
    // Optional authentication (adds user if available)
    static optionalAuth(req, res, next) {
        if (AuthService.isAuthenticated(req)) {
            AuthService.getCurrentUser(req)
                .then(user => {
                    req.user = user;
                    next();
                })
                .catch(() => next());
        } else {
            next();
        }
    }
    
    // Load current user into request
    static loadUser(req, res, next) {
        if (req.session.userId) {
            AuthService.getCurrentUser(req)
                .then(user => {
                    req.user = user;
                    next();
                })
                .catch(err => {
                    console.error('Error loading user:', err);
                    req.session.destroy();
                    res.status(401).json({ error: 'Invalid session' });
                });
        } else {
            next();
        }
    }
    
    // Check if user owns resource
    static requireOwnership(getResourceUserId) {
        return async (req, res, next) => {
            try {
                const resourceUserId = await getResourceUserId(req);
                const currentUserId = req.session.userId;
                
                if (resourceUserId !== currentUserId && !AuthService.hasRole(req, 'admin')) {
                    return res.status(403).json({
                        error: 'Access denied',
                        code: 'ACCESS_DENIED'
                    });
                }
                
                next();
            } catch (error) {
                res.status(500).json({ error: 'Authorization check failed' });
            }
        };
    }
}

module.exports = AuthMiddleware;
```

---

## 🔧 Session Data Management

### Session Data Helper
```javascript
// utils/sessionData.js
class SessionData {
    constructor(req) {
        this.session = req.session;
    }
    
    // User-related data
    setUser(user) {
        this.session.userId = user.id;
        this.session.email = user.email;
        this.session.role = user.role;
        this.session.firstName = user.firstName;
        this.session.lastName = user.lastName;
    }
    
    getUser() {
        if (!this.session.userId) return null;
        
        return {
            id: this.session.userId,
            email: this.session.email,
            role: this.session.role,
            firstName: this.session.firstName,
            lastName: this.session.lastName
        };
    }
    
    // Flash messages
    setFlash(type, message) {
        if (!this.session.flash) {
            this.session.flash = {};
        }
        this.session.flash[type] = message;
    }
    
    getFlash(type) {
        if (!this.session.flash || !this.session.flash[type]) {
            return null;
        }
        
        const message = this.session.flash[type];
        delete this.session.flash[type];
        
        // Clean up empty flash object
        if (Object.keys(this.session.flash).length === 0) {
            delete this.session.flash;
        }
        
        return message;
    }
    
    getAllFlash() {
        const flash = this.session.flash || {};
        delete this.session.flash;
        return flash;
    }
    
    // Shopping cart
    addToCart(productId, quantity = 1) {
        if (!this.session.cart) {
            this.session.cart = {};
        }
        
        if (this.session.cart[productId]) {
            this.session.cart[productId] += quantity;
        } else {
            this.session.cart[productId] = quantity;
        }
    }
    
    removeFromCart(productId) {
        if (this.session.cart) {
            delete this.session.cart[productId];
        }
    }
    
    updateCartQuantity(productId, quantity) {
        if (this.session.cart && quantity > 0) {
            this.session.cart[productId] = quantity;
        } else {
            this.removeFromCart(productId);
        }
    }
    
    getCart() {
        return this.session.cart || {};
    }
    
    getCartCount() {
        const cart = this.getCart();
        return Object.values(cart).reduce((sum, quantity) => sum + quantity, 0);
    }
    
    clearCart() {
        delete this.session.cart;
    }
    
    // Preferences
    setPreference(key, value) {
        if (!this.session.preferences) {
            this.session.preferences = {};
        }
        this.session.preferences[key] = value;
    }
    
    getPreference(key, defaultValue = null) {
        return this.session.preferences?.[key] || defaultValue;
    }
    
    // Navigation history
    addToHistory(url) {
        if (!this.session.history) {
            this.session.history = [];
        }
        
        // Remove duplicate and add to front
        this.session.history = this.session.history.filter(item => item !== url);
        this.session.history.unshift(url);
        
        // Keep only last 10 pages
        this.session.history = this.session.history.slice(0, 10);
    }
    
    getHistory() {
        return this.session.history || [];
    }
    
    getPreviousPage() {
        const history = this.getHistory();
        return history.length > 1 ? history[1] : null;
    }
    
    // Form data persistence
    setFormData(formId, data) {
        if (!this.session.formData) {
            this.session.formData = {};
        }
        this.session.formData[formId] = data;
    }
    
    getFormData(formId) {
        return this.session.formData?.[formId] || null;
    }
    
    clearFormData(formId) {
        if (this.session.formData) {
            delete this.session.formData[formId];
        }
    }
    
    // Session metadata
    getSessionInfo() {
        return {
            id: this.session.id,
            loginTime: this.session.loginTime,
            lastActivity: this.session.lastActivity,
            ipAddress: this.session.ipAddress,
            userAgent: this.session.userAgent,
            isAuthenticated: this.session.isAuthenticated || false
        };
    }
}

// Middleware to add session helper to request
function sessionDataMiddleware(req, res, next) {
    req.sessionData = new SessionData(req);
    next();
}

module.exports = { SessionData, sessionDataMiddleware };
```

### Shopping Cart Example
```javascript
// routes/cart.js
const express = require('express');
const router = express.Router();
const Product = require('../models/Product');

// Add item to cart
router.post('/add', async (req, res) => {
    try {
        const { productId, quantity = 1 } = req.body;
        
        // Validate product exists
        const product = await Product.findById(productId);
        if (!product) {
            return res.status(404).json({ error: 'Product not found' });
        }
        
        // Check stock availability
        if (product.stock < quantity) {
            return res.status(400).json({ 
                error: 'Insufficient stock',
                available: product.stock 
            });
        }
        
        // Add to session cart
        req.sessionData.addToCart(productId, parseInt(quantity));
        
        res.json({
            success: true,
            cartCount: req.sessionData.getCartCount(),
            message: 'Item added to cart'
        });
        
    } catch (error) {
        res.status(500).json({ error: 'Failed to add item to cart' });
    }
});

// Get cart contents
router.get('/', async (req, res) => {
    try {
        const cart = req.sessionData.getCart();
        const productIds = Object.keys(cart);
        
        if (productIds.length === 0) {
            return res.json({ cart: [], total: 0, count: 0 });
        }
        
        // Get product details
        const products = await Product.find({ _id: { $in: productIds } });
        
        const cartItems = products.map(product => ({
            product: {
                id: product._id,
                name: product.name,
                price: product.price,
                image: product.image
            },
            quantity: cart[product._id.toString()],
            subtotal: product.price * cart[product._id.toString()]
        }));
        
        const total = cartItems.reduce((sum, item) => sum + item.subtotal, 0);
        
        res.json({
            cart: cartItems,
            total,
            count: req.sessionData.getCartCount()
        });
        
    } catch (error) {
        res.status(500).json({ error: 'Failed to get cart' });
    }
});

// Update cart item quantity
router.put('/update', (req, res) => {
    const { productId, quantity } = req.body;
    
    if (quantity <= 0) {
        req.sessionData.removeFromCart(productId);
    } else {
        req.sessionData.updateCartQuantity(productId, parseInt(quantity));
    }
    
    res.json({
        success: true,
        cartCount: req.sessionData.getCartCount()
    });
});

// Remove item from cart
router.delete('/remove/:productId', (req, res) => {
    const { productId } = req.params;
    
    req.sessionData.removeFromCart(productId);
    
    res.json({
        success: true,
        cartCount: req.sessionData.getCartCount()
    });
});

// Clear entire cart
router.delete('/clear', (req, res) => {
    req.sessionData.clearCart();
    
    res.json({
        success: true,
        cartCount: 0
    });
});

module.exports = router;
```

---

## 🔧 Session Monitoring and Analytics

### Session Analytics
```javascript
// services/sessionAnalytics.js
const redis = require('redis');

class SessionAnalytics {
    constructor() {
        this.redis = redis.createClient(process.env.REDIS_URL);
    }
    
    // Track session creation
    async trackSessionStart(sessionId, userId, metadata) {
        const sessionKey = `session:${sessionId}`;
        const userKey = `user:${userId}:sessions`;
        
        const sessionData = {
            sessionId,
            userId,
            startTime: Date.now(),
            ipAddress: metadata.ipAddress,
            userAgent: metadata.userAgent,
            country: metadata.country || 'unknown'
        };
        
        await Promise.all([
            this.redis.hSet(sessionKey, sessionData),
            this.redis.sAdd(userKey, sessionId),
            this.redis.expire(sessionKey, 86400), // 24 hours
            this.redis.expire(userKey, 86400 * 7), // 7 days
            this.incrementDailyActiveUsers(userId),
            this.trackSessionByCountry(metadata.country)
        ]);
    }
    
    // Track session activity
    async trackActivity(sessionId, action, data = {}) {
        const activityKey = `session:${sessionId}:activity`;
        
        const activity = {
            action,
            timestamp: Date.now(),
            ...data
        };
        
        await this.redis.lPush(activityKey, JSON.stringify(activity));
        await this.redis.lTrim(activityKey, 0, 99); // Keep last 100 activities
        await this.redis.expire(activityKey, 86400);
    }
    
    // Track session end
    async trackSessionEnd(sessionId, userId) {
        const sessionKey = `session:${sessionId}`;
        const session = await this.redis.hGetAll(sessionKey);
        
        if (session.startTime) {
            const duration = Date.now() - parseInt(session.startTime);
            
            await Promise.all([
                this.redis.hSet(sessionKey, 'endTime', Date.now()),
                this.redis.hSet(sessionKey, 'duration', duration),
                this.trackSessionDuration(duration),
                this.updateUserSessionStats(userId, duration)
            ]);
        }
    }
    
    // Get active sessions count
    async getActiveSessionsCount() {
        const keys = await this.redis.keys('session:*');
        return keys.filter(key => !key.includes(':activity')).length;
    }
    
    // Get user's active sessions
    async getUserActiveSessions(userId) {
        const userKey = `user:${userId}:sessions`;
        const sessionIds = await this.redis.sMembers(userKey);
        
        const sessions = await Promise.all(
            sessionIds.map(async (sessionId) => {
                const sessionData = await this.redis.hGetAll(`session:${sessionId}`);
                return sessionData.sessionId ? sessionData : null;
            })
        );
        
        return sessions.filter(Boolean);
    }
    
    // Daily active users
    async incrementDailyActiveUsers(userId) {
        const today = new Date().toISOString().split('T')[0];
        const key = `dau:${today}`;
        
        await this.redis.sAdd(key, userId);
        await this.redis.expire(key, 86400 * 7); // Keep for 7 days
    }
    
    async getDailyActiveUsers(date) {
        const key = `dau:${date}`;
        return await this.redis.sCard(key);
    }
    
    // Session duration tracking
    async trackSessionDuration(duration) {
        const bucket = this.getDurationBucket(duration);
        const today = new Date().toISOString().split('T')[0];
        const key = `session:duration:${today}`;
        
        await this.redis.hIncrBy(key, bucket, 1);
        await this.redis.expire(key, 86400 * 30); // Keep for 30 days
    }
    
    getDurationBucket(duration) {
        const minutes = duration / (1000 * 60);
        
        if (minutes < 1) return '0-1min';
        if (minutes < 5) return '1-5min';
        if (minutes < 15) return '5-15min';
        if (minutes < 30) return '15-30min';
        if (minutes < 60) return '30-60min';
        return '60+min';
    }
    
    // Geographic tracking
    async trackSessionByCountry(country) {
        if (!country) return;
        
        const today = new Date().toISOString().split('T')[0];
        const key = `sessions:country:${today}`;
        
        await this.redis.hIncrBy(key, country, 1);
        await this.redis.expire(key, 86400 * 30);
    }
    
    // Get session statistics
    async getSessionStats(days = 7) {
        const stats = {};
        const today = new Date();
        
        for (let i = 0; i < days; i++) {
            const date = new Date(today - i * 86400 * 1000).toISOString().split('T')[0];
            const dauKey = `dau:${date}`;
            const durationKey = `session:duration:${date}`;
            const countryKey = `sessions:country:${date}`;
            
            const [dau, durations, countries] = await Promise.all([
                this.redis.sCard(dauKey),
                this.redis.hGetAll(durationKey),
                this.redis.hGetAll(countryKey)
            ]);
            
            stats[date] = {
                activeUsers: dau,
                sessionDurations: durations,
                countries
            };
        }
        
        return stats;
    }
}

module.exports = SessionAnalytics;
```

---

## 🔗 Related Topics
- [[10-Cookie-Parser]] - Cookie handling for sessions
- [[11-Authentication-and-Authorization]] - User authentication
- [[12-JWT-Tokens]] - Alternative to sessions
- [[14-Environment-Variables]] - Session configuration

---

*Back to [[00-Main-Index]]*
