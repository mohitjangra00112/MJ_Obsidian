# Rate Limiting

**Navigation**: [[12-JWT-Tokens]] | [[00-Main-Index]] | Next: [[14-Environment-Variables]]

---

## 🛡️ Rate Limiting in Node.js

Rate limiting **controls the rate** of requests a client can make to prevent abuse, DDoS attacks, and ensure fair resource usage.

---

## 📦 Express Rate Limit

### Basic Installation and Setup
```bash
npm install express-rate-limit
```

### Simple Rate Limiting
```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');

const app = express();

// Basic rate limiting
const limiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100, // Limit each IP to 100 requests per windowMs
    message: 'Too many requests from this IP, please try again later.',
    standardHeaders: true, // Return rate limit info in the `RateLimit-*` headers
    legacyHeaders: false, // Disable the `X-RateLimit-*` headers
});

// Apply rate limiting to all requests
app.use(limiter);

app.get('/', (req, res) => {
    res.json({ 
        message: 'Hello World!',
        remaining: req.rateLimit.remaining,
        resetTime: new Date(req.rateLimit.resetTime)
    });
});

app.listen(3000);
```

### Advanced Rate Limiting Configuration
```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const redis = require('redis');

const app = express();

// Redis client for distributed rate limiting
const redisClient = redis.createClient({
    host: process.env.REDIS_HOST || 'localhost',
    port: process.env.REDIS_PORT || 6379
});

// Advanced rate limiter
const advancedLimiter = rateLimit({
    store: new RedisStore({
        sendCommand: (...args) => redisClient.call(...args),
    }),
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100,
    message: {
        error: 'Too many requests',
        retryAfter: 15 * 60 // seconds
    },
    standardHeaders: true,
    legacyHeaders: false,
    
    // Custom key generator
    keyGenerator: (req) => {
        // Rate limit by IP + User ID if authenticated
        const ip = req.ip;
        const userId = req.user?.id || 'anonymous';
        return `${ip}:${userId}`;
    },
    
    // Custom handler
    handler: (req, res) => {
        res.status(429).json({
            error: 'Rate limit exceeded',
            message: 'Too many requests, please slow down',
            retryAfter: Math.ceil(req.rateLimit.resetTime / 1000),
            limit: req.rateLimit.limit,
            current: req.rateLimit.current,
            remaining: req.rateLimit.remaining
        });
    },
    
    // Skip certain requests
    skip: (req) => {
        // Skip rate limiting for admin users
        return req.user?.role === 'admin';
    }
});

app.use(advancedLimiter);

app.listen(3000);
```

---

## 🎯 Specific Endpoint Rate Limiting

### Different Limits for Different Endpoints
```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');

const app = express();

// Strict rate limiting for login
const loginLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5, // 5 login attempts per window
    message: {
        error: 'Too many login attempts',
        message: 'Please try again in 15 minutes'
    },
    skipSuccessfulRequests: true, // Don't count successful requests
});

// Moderate rate limiting for API
const apiLimiter = rateLimit({
    windowMs: 1 * 60 * 1000, // 1 minute
    max: 20, // 20 requests per minute
    message: 'API rate limit exceeded'
});

// Lenient rate limiting for static content
const staticLimiter = rateLimit({
    windowMs: 1 * 60 * 1000, // 1 minute
    max: 100, // 100 requests per minute
});

// Apply different limits to different routes
app.use('/auth/login', loginLimiter);
app.use('/api/', apiLimiter);
app.use('/static/', staticLimiter);

// Authentication routes
app.post('/auth/login', (req, res) => {
    // Login logic here
    res.json({ message: 'Login attempt' });
});

// API routes
app.get('/api/users', (req, res) => {
    res.json({ users: [] });
});

// Static content
app.get('/static/image.jpg', (req, res) => {
    res.send('Image content');
});

app.listen(3000);
```

### Progressive Rate Limiting
```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');

const app = express();

// Progressive rate limiting - gets stricter with each violation
const progressiveLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100,
    
    // Custom key generator to track violations
    keyGenerator: (req) => {
        return req.ip;
    },
    
    // Skip handler to implement progressive logic
    skip: (req) => {
        const violations = getViolationCount(req.ip);
        
        if (violations === 0) {
            return false; // Apply normal rate limit
        } else if (violations === 1) {
            // Second violation: reduce limit by 50%
            req.rateLimit.limit = 50;
        } else if (violations >= 2) {
            // Multiple violations: very strict limit
            req.rateLimit.limit = 10;
        }
        
        return false;
    }
});

// Track violations (use Redis in production)
const violations = new Map();

function getViolationCount(ip) {
    return violations.get(ip) || 0;
}

function incrementViolation(ip) {
    const count = getViolationCount(ip) + 1;
    violations.set(ip, count);
    
    // Reset violations after 24 hours
    setTimeout(() => {
        violations.delete(ip);
    }, 24 * 60 * 60 * 1000);
}

app.use(progressiveLimiter);

app.listen(3000);
```

---

## 🔐 Authentication-Based Rate Limiting

### Different Limits for Different User Types
```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');

const app = express();
app.use(express.json());

// Mock authentication middleware
const authenticate = (req, res, next) => {
    const token = req.headers.authorization;
    
    // Mock user data based on token
    if (token === 'Bearer admin-token') {
        req.user = { id: 1, role: 'admin', tier: 'premium' };
    } else if (token === 'Bearer user-token') {
        req.user = { id: 2, role: 'user', tier: 'free' };
    } else {
        req.user = null; // Anonymous user
    }
    
    next();
};

// Tiered rate limiting
const tieredLimiter = rateLimit({
    windowMs: 1 * 60 * 1000, // 1 minute
    
    // Dynamic max based on user tier
    max: (req) => {
        if (!req.user) return 10;           // Anonymous: 10/min
        if (req.user.role === 'admin') return 1000;   // Admin: 1000/min
        if (req.user.tier === 'premium') return 100;  // Premium: 100/min
        return 30;                                     // Free users: 30/min
    },
    
    keyGenerator: (req) => {
        // Use user ID if authenticated, otherwise IP
        return req.user ? `user:${req.user.id}` : `ip:${req.ip}`;
    },
    
    message: (req) => ({
        error: 'Rate limit exceeded',
        tier: req.user?.tier || 'anonymous',
        upgradeMessage: req.user?.tier === 'free' ? 
            'Upgrade to premium for higher limits' : null
    })
});

app.use(authenticate);
app.use(tieredLimiter);

app.get('/api/data', (req, res) => {
    res.json({
        message: 'API data',
        user: req.user,
        rateLimit: {
            limit: req.rateLimit.limit,
            remaining: req.rateLimit.remaining,
            resetTime: req.rateLimit.resetTime
        }
    });
});

app.listen(3000);
```

---

## 🚦 Custom Rate Limiting Implementation

### Manual Rate Limiting with Redis
```javascript
const express = require('express');
const redis = require('redis');

const app = express();
const redisClient = redis.createClient();

// Custom rate limiter class
class CustomRateLimiter {
    constructor(options = {}) {
        this.window = options.window || 60; // seconds
        this.maxRequests = options.max || 100;
        this.keyPrefix = options.keyPrefix || 'rate_limit:';
    }
    
    async isAllowed(identifier) {
        const key = `${this.keyPrefix}${identifier}`;
        const current = await redisClient.get(key);
        
        if (!current) {
            // First request in window
            await redisClient.setex(key, this.window, 1);
            return {
                allowed: true,
                count: 1,
                remaining: this.maxRequests - 1,
                resetTime: Date.now() + (this.window * 1000)
            };
        }
        
        const count = parseInt(current);
        
        if (count >= this.maxRequests) {
            const ttl = await redisClient.ttl(key);
            return {
                allowed: false,
                count: count,
                remaining: 0,
                resetTime: Date.now() + (ttl * 1000)
            };
        }
        
        // Increment counter
        await redisClient.incr(key);
        const ttl = await redisClient.ttl(key);
        
        return {
            allowed: true,
            count: count + 1,
            remaining: this.maxRequests - (count + 1),
            resetTime: Date.now() + (ttl * 1000)
        };
    }
    
    middleware() {
        return async (req, res, next) => {
            const identifier = req.ip;
            
            try {
                const result = await this.isAllowed(identifier);
                
                // Add rate limit headers
                res.set({
                    'X-RateLimit-Limit': this.maxRequests,
                    'X-RateLimit-Remaining': result.remaining,
                    'X-RateLimit-Reset': Math.ceil(result.resetTime / 1000)
                });
                
                if (!result.allowed) {
                    return res.status(429).json({
                        error: 'Rate limit exceeded',
                        retryAfter: Math.ceil((result.resetTime - Date.now()) / 1000)
                    });
                }
                
                req.rateLimit = result;
                next();
                
            } catch (error) {
                console.error('Rate limiting error:', error);
                next(); // Continue on error
            }
        };
    }
}

// Usage
const apiLimiter = new CustomRateLimiter({
    window: 60,    // 1 minute
    max: 100,      // 100 requests
    keyPrefix: 'api:'
});

app.use('/api', apiLimiter.middleware());

app.listen(3000);
```

### Sliding Window Rate Limiter
```javascript
class SlidingWindowRateLimiter {
    constructor(options = {}) {
        this.window = options.window || 60; // seconds
        this.maxRequests = options.max || 100;
        this.keyPrefix = options.keyPrefix || 'sliding:';
    }
    
    async isAllowed(identifier) {
        const key = `${this.keyPrefix}${identifier}`;
        const now = Date.now();
        const windowStart = now - (this.window * 1000);
        
        // Remove old entries
        await redisClient.zremrangebyscore(key, 0, windowStart);
        
        // Count current requests in window
        const currentCount = await redisClient.zcard(key);
        
        if (currentCount >= this.maxRequests) {
            return {
                allowed: false,
                count: currentCount,
                remaining: 0,
                resetTime: await this.getOldestTimestamp(key) + (this.window * 1000)
            };
        }
        
        // Add current request
        await redisClient.zadd(key, now, `${now}-${Math.random()}`);
        await redisClient.expire(key, this.window);
        
        return {
            allowed: true,
            count: currentCount + 1,
            remaining: this.maxRequests - (currentCount + 1),
            resetTime: now + (this.window * 1000)
        };
    }
    
    async getOldestTimestamp(key) {
        const oldest = await redisClient.zrange(key, 0, 0, 'WITHSCORES');
        return oldest.length > 0 ? parseInt(oldest[1]) : Date.now();
    }
}
```

---

## 🌐 Global Rate Limiting Strategies

### IP-Based vs User-Based Limiting
```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');

const app = express();

// IP-based limiting for public endpoints
const ipLimiter = rateLimit({
    windowMs: 1 * 60 * 1000,
    max: 20,
    keyGenerator: (req) => req.ip,
    message: 'Too many requests from this IP'
});

// User-based limiting for authenticated endpoints
const userLimiter = rateLimit({
    windowMs: 1 * 60 * 1000,
    max: 100,
    keyGenerator: (req) => {
        return req.user ? `user:${req.user.id}` : `ip:${req.ip}`;
    },
    skip: (req) => !req.user, // Skip if not authenticated
    message: 'Too many requests for this user'
});

// Combined limiting strategy
const combinedLimiter = rateLimit({
    windowMs: 1 * 60 * 1000,
    max: (req) => {
        // Higher limits for authenticated users
        return req.user ? 200 : 50;
    },
    keyGenerator: (req) => {
        // Rate limit by user if authenticated, otherwise by IP
        return req.user ? `user:${req.user.id}` : `ip:${req.ip}`;
    }
});

// Apply different strategies
app.use('/public', ipLimiter);
app.use('/api/auth', combinedLimiter);
app.use('/api/user', authenticate, userLimiter);

app.listen(3000);
```

---

## 📊 Rate Limiting Analytics

### Monitoring and Analytics
```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');

const app = express();

// Rate limiting analytics
const rateLimitAnalytics = {
    violations: new Map(),
    requests: new Map(),
    
    logViolation: (identifier, endpoint) => {
        const key = `${identifier}:${endpoint}`;
        const current = rateLimitAnalytics.violations.get(key) || 0;
        rateLimitAnalytics.violations.set(key, current + 1);
    },
    
    logRequest: (identifier, endpoint) => {
        const key = `${identifier}:${endpoint}`;
        const current = rateLimitAnalytics.requests.get(key) || 0;
        rateLimitAnalytics.requests.set(key, current + 1);
    },
    
    getStats: () => {
        const totalViolations = Array.from(rateLimitAnalytics.violations.values())
            .reduce((sum, count) => sum + count, 0);
        const totalRequests = Array.from(rateLimitAnalytics.requests.values())
            .reduce((sum, count) => sum + count, 0);
        
        return {
            totalRequests,
            totalViolations,
            violationRate: totalRequests > 0 ? (totalViolations / totalRequests) * 100 : 0,
            topViolators: Array.from(rateLimitAnalytics.violations.entries())
                .sort(([,a], [,b]) => b - a)
                .slice(0, 10)
        };
    }
};

// Analytics middleware
const analyticsLimiter = rateLimit({
    windowMs: 1 * 60 * 1000,
    max: 100,
    
    handler: (req, res) => {
        const identifier = req.ip;
        const endpoint = req.path;
        
        // Log violation
        rateLimitAnalytics.logViolation(identifier, endpoint);
        
        res.status(429).json({
            error: 'Rate limit exceeded',
            endpoint: endpoint,
            identifier: identifier.substr(0, 8) + '...' // Partial IP for privacy
        });
    },
    
    onLimitReached: (req, res, options) => {
        console.log(`Rate limit reached for ${req.ip} on ${req.path}`);
    }
});

// Request logging middleware
app.use((req, res, next) => {
    rateLimitAnalytics.logRequest(req.ip, req.path);
    next();
});

app.use(analyticsLimiter);

// Analytics endpoint
app.get('/admin/rate-limit-stats', (req, res) => {
    res.json(rateLimitAnalytics.getStats());
});

app.listen(3000);
```

---

## 🛠️ Rate Limiting Best Practices

### Production-Ready Implementation
```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');
const helmet = require('helmet');

const app = express();

// Security headers
app.use(helmet());

// Production rate limiting configuration
const productionLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100,
    
    // Use Redis in production
    store: process.env.NODE_ENV === 'production' ? new RedisStore({
        sendCommand: (...args) => redisClient.call(...args),
    }) : undefined,
    
    // Detailed error responses
    handler: (req, res) => {
        const retryAfter = Math.ceil(req.rateLimit.resetTime / 1000);
        
        res.status(429).json({
            error: {
                code: 'RATE_LIMIT_EXCEEDED',
                message: 'Too many requests',
                details: {
                    limit: req.rateLimit.limit,
                    current: req.rateLimit.current,
                    remaining: req.rateLimit.remaining,
                    retryAfter: retryAfter
                }
            }
        });
    },
    
    // Custom headers
    standardHeaders: true,
    legacyHeaders: false,
    
    // Skip successful requests for login endpoints
    skipSuccessfulRequests: true,
    
    // Fail open on store errors
    skipFailedRequests: false
});

app.use(productionLimiter);

// Health check endpoint (excluded from rate limiting)
app.get('/health', (req, res) => {
    res.json({ status: 'OK', timestamp: new Date() });
});

app.listen(3000);
```

---

## 🔗 Related Topics
- [[11-Authentication-and-Authorization]] - Protecting auth endpoints
- [[12-JWT-Tokens]] - Rate limiting token endpoints
- [[21-Error-Handling]] - Handling rate limit errors
- [[32-Development-vs-Production]] - Environment-specific limits

---

*Back to [[00-Main-Index]]*
