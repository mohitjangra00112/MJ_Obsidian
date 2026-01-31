# Cookie Parser

**Navigation**: [[08-Headers-and-Cookies]] | [[00-Main-Index]] | Next: [[10-Express-Sessions]]

---

## 🍪 Cookie Parser Middleware

Cookie Parser is an Express middleware that **parses cookies** attached to the client request object. It makes working with cookies much easier and provides additional security features.

---

## 📦 Installation and Setup

### Installing Cookie Parser
```bash
npm install cookie-parser
```

### Basic Setup
```javascript
const express = require('express');
const cookieParser = require('cookie-parser');

const app = express();

// Basic cookie parser
app.use(cookieParser());

// Cookie parser with secret for signed cookies
app.use(cookieParser('your-secret-key'));

app.listen(3000, () => {
    console.log('Server running with cookie parser');
});
```

---

## 🎯 Basic Cookie Operations

### Reading Cookies
```javascript
const express = require('express');
const cookieParser = require('cookie-parser');

const app = express();
app.use(cookieParser('my-secret-key'));

app.get('/cookies', (req, res) => {
    // All unsigned cookies
    console.log('Cookies:', req.cookies);
    
    // All signed cookies
    console.log('Signed Cookies:', req.signedCookies);
    
    // Specific cookie
    const username = req.cookies.username;
    const userId = req.signedCookies.userId;
    
    res.json({
        allCookies: req.cookies,
        signedCookies: req.signedCookies,
        specific: {
            username: username,
            userId: userId
        }
    });
});

app.listen(3000);
```

### Setting Cookies
```javascript
const express = require('express');
const cookieParser = require('cookie-parser');

const app = express();
app.use(cookieParser('secret-key-for-signing'));

app.get('/set-cookie', (req, res) => {
    // Simple cookie
    res.cookie('simple', 'value');
    
    // Cookie with options
    res.cookie('user', 'john', {
        maxAge: 900000, // 15 minutes
        httpOnly: true,
        secure: false // Set to true in production with HTTPS
    });
    
    // Signed cookie
    res.cookie('session', 'abc123', {
        signed: true,
        maxAge: 3600000 // 1 hour
    });
    
    res.json({ message: 'Cookies set successfully' });
});

app.listen(3000);
```

---

## 🔐 Signed Cookies

### Creating and Verifying Signed Cookies
```javascript
const express = require('express');
const cookieParser = require('cookie-parser');
const crypto = require('crypto');

const app = express();

// Secret key for signing (use environment variable in production)
const SECRET_KEY = process.env.COOKIE_SECRET || 'your-very-secure-secret-key';
app.use(cookieParser(SECRET_KEY));

// Set signed cookie
app.post('/login', (req, res) => {
    const userId = 12345;
    const userRole = 'admin';
    
    // Set signed cookies
    res.cookie('userId', userId, {
        signed: true,
        httpOnly: true,
        maxAge: 24 * 60 * 60 * 1000, // 24 hours
        sameSite: 'strict'
    });
    
    res.cookie('userRole', userRole, {
        signed: true,
        httpOnly: true,
        maxAge: 24 * 60 * 60 * 1000
    });
    
    res.json({ message: 'Login successful, signed cookies set' });
});

// Read signed cookies
app.get('/profile', (req, res) => {
    const userId = req.signedCookies.userId;
    const userRole = req.signedCookies.userRole;
    
    // Check if cookies were tampered with
    if (userId === false || userRole === false) {
        return res.status(401).json({ error: 'Invalid or tampered cookies' });
    }
    
    if (!userId || !userRole) {
        return res.status(401).json({ error: 'Authentication required' });
    }
    
    res.json({
        message: 'Profile access granted',
        user: {
            id: userId,
            role: userRole
        }
    });
});

app.listen(3000);
```

---

## 🛠️ Advanced Cookie Parser Usage

### Multiple Secrets and Rotation
```javascript
const express = require('express');
const cookieParser = require('cookie-parser');

const app = express();

// Cookie parser with multiple secrets for key rotation
const secrets = [
    'current-secret-key',    // Current key
    'previous-secret-key-1', // Previous key for compatibility
    'previous-secret-key-2'  // Older key for compatibility
];

app.use(cookieParser(secrets));

// Cookie validation with multiple secrets
app.get('/validate-cookie', (req, res) => {
    const signedValue = req.signedCookies.important_data;
    
    if (signedValue === false) {
        return res.status(401).json({ error: 'Cookie signature invalid' });
    }
    
    if (!signedValue) {
        return res.status(401).json({ error: 'No signed cookie found' });
    }
    
    res.json({ 
        message: 'Cookie validated successfully',
        value: signedValue 
    });
});

app.listen(3000);
```

### Custom Cookie Parser Configuration
```javascript
const express = require('express');
const cookieParser = require('cookie-parser');

const app = express();

// Custom cookie parser options
app.use(cookieParser('secret', {
    decode: function(val) {
        // Custom decode function
        try {
            return decodeURIComponent(val);
        } catch (err) {
            return val;
        }
    }
}));

// Custom cookie utilities
const cookieUtils = {
    // Set encrypted cookie
    setSecureCookie: (res, name, value, options = {}) => {
        const encrypted = Buffer.from(JSON.stringify(value)).toString('base64');
        res.cookie(name, encrypted, {
            signed: true,
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'strict',
            ...options
        });
    },
    
    // Get encrypted cookie
    getSecureCookie: (req, name) => {
        const encrypted = req.signedCookies[name];
        if (!encrypted || encrypted === false) return null;
        
        try {
            return JSON.parse(Buffer.from(encrypted, 'base64').toString());
        } catch (err) {
            return null;
        }
    },
    
    // Clear cookie
    clearCookie: (res, name, options = {}) => {
        res.clearCookie(name, {
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'strict',
            ...options
        });
    }
};

// Using custom cookie utilities
app.post('/set-user-data', (req, res) => {
    const userData = {
        id: 123,
        name: 'John Doe',
        preferences: {
            theme: 'dark',
            language: 'en'
        }
    };
    
    cookieUtils.setSecureCookie(res, 'user_data', userData, {
        maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
    });
    
    res.json({ message: 'User data stored in secure cookie' });
});

app.get('/get-user-data', (req, res) => {
    const userData = cookieUtils.getSecureCookie(req, 'user_data');
    
    if (!userData) {
        return res.status(401).json({ error: 'No user data found' });
    }
    
    res.json({ userData });
});

app.listen(3000);
```

---

## 🔄 Session-like Behavior with Cookies

### Implementing Simple Sessions
```javascript
const express = require('express');
const cookieParser = require('cookie-parser');
const crypto = require('crypto');

const app = express();
app.use(cookieParser('session-secret-key'));

// In-memory session store (use Redis in production)
const sessions = new Map();

// Session utilities
const sessionUtils = {
    generateSessionId: () => {
        return crypto.randomBytes(32).toString('hex');
    },
    
    createSession: (res, data) => {
        const sessionId = sessionUtils.generateSessionId();
        
        // Store session data
        sessions.set(sessionId, {
            ...data,
            createdAt: new Date(),
            lastAccessed: new Date()
        });
        
        // Set session cookie
        res.cookie('sessionId', sessionId, {
            signed: true,
            httpOnly: true,
            maxAge: 30 * 60 * 1000, // 30 minutes
            sameSite: 'strict'
        });
        
        return sessionId;
    },
    
    getSession: (req) => {
        const sessionId = req.signedCookies.sessionId;
        
        if (!sessionId || sessionId === false) {
            return null;
        }
        
        const session = sessions.get(sessionId);
        if (!session) {
            return null;
        }
        
        // Update last accessed time
        session.lastAccessed = new Date();
        sessions.set(sessionId, session);
        
        return { sessionId, ...session };
    },
    
    destroySession: (req, res) => {
        const sessionId = req.signedCookies.sessionId;
        
        if (sessionId && sessionId !== false) {
            sessions.delete(sessionId);
        }
        
        res.clearCookie('sessionId', {
            signed: true,
            httpOnly: true,
            sameSite: 'strict'
        });
    }
};

// Login endpoint
app.post('/login', (req, res) => {
    // Validate credentials (simplified)
    const { username, password } = req.body;
    
    if (username === 'admin' && password === 'password') {
        const sessionId = sessionUtils.createSession(res, {
            userId: 1,
            username: 'admin',
            role: 'administrator'
        });
        
        res.json({ 
            message: 'Login successful',
            sessionId: sessionId
        });
    } else {
        res.status(401).json({ error: 'Invalid credentials' });
    }
});

// Protected route
app.get('/dashboard', (req, res) => {
    const session = sessionUtils.getSession(req);
    
    if (!session) {
        return res.status(401).json({ error: 'Session required' });
    }
    
    res.json({
        message: 'Dashboard access granted',
        user: {
            id: session.userId,
            username: session.username,
            role: session.role
        },
        sessionInfo: {
            createdAt: session.createdAt,
            lastAccessed: session.lastAccessed
        }
    });
});

// Logout endpoint
app.post('/logout', (req, res) => {
    sessionUtils.destroySession(req, res);
    res.json({ message: 'Logout successful' });
});

// Session cleanup (remove expired sessions)
setInterval(() => {
    const now = new Date();
    const expireTime = 30 * 60 * 1000; // 30 minutes
    
    for (const [sessionId, session] of sessions.entries()) {
        if (now - session.lastAccessed > expireTime) {
            sessions.delete(sessionId);
            console.log(`Expired session removed: ${sessionId}`);
        }
    }
}, 5 * 60 * 1000); // Check every 5 minutes

app.listen(3000);
```

---

## 🛡️ Security Best Practices

### Secure Cookie Configuration
```javascript
const express = require('express');
const cookieParser = require('cookie-parser');

const app = express();
app.use(cookieParser(process.env.COOKIE_SECRET));

// Security middleware for cookies
const secureCookieSettings = {
    httpOnly: true, // Prevent XSS attacks
    secure: process.env.NODE_ENV === 'production', // HTTPS only in production
    sameSite: 'strict', // CSRF protection
    maxAge: 60 * 60 * 1000 // 1 hour
};

// Secure authentication cookie
app.post('/secure-login', (req, res) => {
    // Authentication logic here...
    
    res.cookie('auth_token', 'secure-token', {
        ...secureCookieSettings,
        signed: true
    });
    
    res.json({ message: 'Secure login successful' });
});

// Cookie validation middleware
const validateAuthCookie = (req, res, next) => {
    const authToken = req.signedCookies.auth_token;
    
    // Check if cookie exists and is not tampered
    if (!authToken || authToken === false) {
        return res.status(401).json({ error: 'Invalid or missing authentication' });
    }
    
    // Validate token (implement your validation logic)
    if (authToken !== 'secure-token') {
        return res.status(401).json({ error: 'Invalid authentication token' });
    }
    
    req.isAuthenticated = true;
    next();
};

// Protected route
app.get('/secure-data', validateAuthCookie, (req, res) => {
    res.json({ 
        message: 'Secure data accessed successfully',
        timestamp: new Date().toISOString()
    });
});

app.listen(3000);
```

### Cookie GDPR Compliance
```javascript
const express = require('express');
const cookieParser = require('cookie-parser');

const app = express();
app.use(cookieParser());

// GDPR consent management
const gdprMiddleware = (req, res, next) => {
    const consent = req.cookies.gdpr_consent;
    
    if (!consent) {
        // Set only essential cookies before consent
        res.locals.canSetAnalyticsCookies = false;
        res.locals.canSetMarketingCookies = false;
    } else {
        const consentData = JSON.parse(consent);
        res.locals.canSetAnalyticsCookies = consentData.analytics;
        res.locals.canSetMarketingCookies = consentData.marketing;
    }
    
    next();
};

app.use(gdprMiddleware);

// GDPR consent endpoint
app.post('/gdpr-consent', (req, res) => {
    const { analytics, marketing } = req.body;
    
    const consentData = {
        analytics: analytics || false,
        marketing: marketing || false,
        timestamp: new Date().toISOString()
    };
    
    res.cookie('gdpr_consent', JSON.stringify(consentData), {
        maxAge: 365 * 24 * 60 * 60 * 1000, // 1 year
        httpOnly: true,
        sameSite: 'strict'
    });
    
    res.json({ message: 'Consent preferences saved' });
});

// Conditional cookie setting
app.get('/track-visit', (req, res) => {
    // Always set essential cookies
    res.cookie('last_visit', new Date().toISOString(), {
        maxAge: 24 * 60 * 60 * 1000, // 24 hours
        httpOnly: true
    });
    
    // Set analytics cookies only with consent
    if (res.locals.canSetAnalyticsCookies) {
        res.cookie('analytics_id', 'unique-id', {
            maxAge: 30 * 24 * 60 * 60 * 1000, // 30 days
            httpOnly: true
        });
    }
    
    res.json({ 
        message: 'Visit tracked',
        analyticsEnabled: res.locals.canSetAnalyticsCookies
    });
});

app.listen(3000);
```

---

## 🔗 Related Topics
- [[08-Headers-and-Cookies]] - Basic cookie concepts
- [[10-Express-Sessions]] - Advanced session management
- [[11-Authentication-and-Authorization]] - Using cookies for auth
- [[14-Environment-Variables]] - Securing cookie secrets

---

*Back to [[00-Main-Index]]*
