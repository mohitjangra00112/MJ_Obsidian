# JWT Tokens

**Navigation**: [[11-Authentication-and-Authorization]] | [[00-Main-Index]] | Next: [[13-Rate-Limiting]]

---

## 🎯 JSON Web Tokens (JWT)

JWT is a **compact, URL-safe** token format for securely transmitting information between parties. It's stateless and perfect for scalable applications.

---

## 🏗️ JWT Structure

### Token Anatomy
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

Header.Payload.Signature
```

### JWT Components
```javascript
// Header
{
  "alg": "HS256",    // Algorithm
  "typ": "JWT"       // Token type
}

// Payload
{
  "sub": "1234567890",           // Subject (user ID)
  "name": "John Doe",            // Custom claims
  "iat": 1516239022,             // Issued at
  "exp": 1516242622,             // Expiration
  "iss": "myapp.com",            // Issuer
  "aud": "myapp-users"           // Audience
}

// Signature
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

---

## 📦 Installation and Basic Setup

### Installing JWT Package
```bash
npm install jsonwebtoken
npm install --save-dev @types/jsonwebtoken  # For TypeScript
```

### Basic JWT Implementation
```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

const app = express();
app.use(express.json());

const JWT_SECRET = process.env.JWT_SECRET || 'your-secret-key';

// Mock users database
const users = [
    {
        id: 1,
        username: 'admin',
        email: 'admin@example.com',
        password: '$2b$10$hash...', // bcrypt hash
        role: 'admin'
    },
    {
        id: 2,
        username: 'user',
        email: 'user@example.com',
        password: '$2b$10$hash...',
        role: 'user'
    }
];

// JWT utilities
const jwtUtils = {
    // Generate access token
    generateAccessToken: (user) => {
        const payload = {
            id: user.id,
            username: user.username,
            email: user.email,
            role: user.role
        };
        
        return jwt.sign(payload, JWT_SECRET, {
            expiresIn: '15m',              // 15 minutes
            issuer: 'myapp.com',
            audience: 'myapp-users'
        });
    },
    
    // Verify token
    verifyToken: (token) => {
        try {
            return jwt.verify(token, JWT_SECRET, {
                issuer: 'myapp.com',
                audience: 'myapp-users'
            });
        } catch (error) {
            return null;
        }
    },
    
    // Decode token without verification (for debugging)
    decodeToken: (token) => {
        return jwt.decode(token, { complete: true });
    }
};

// Authentication middleware
const authenticateToken = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN
    
    if (!token) {
        return res.status(401).json({ error: 'Access token required' });
    }
    
    const decoded = jwtUtils.verifyToken(token);
    if (!decoded) {
        return res.status(403).json({ error: 'Invalid or expired token' });
    }
    
    req.user = decoded;
    next();
};

// Login endpoint
app.post('/login', async (req, res) => {
    try {
        const { username, password } = req.body;
        
        // Find user
        const user = users.find(u => u.username === username);
        if (!user) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        // Verify password
        const isValidPassword = await bcrypt.compare(password, user.password);
        if (!isValidPassword) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        // Generate token
        const accessToken = jwtUtils.generateAccessToken(user);
        
        res.json({
            message: 'Login successful',
            accessToken: accessToken,
            user: {
                id: user.id,
                username: user.username,
                email: user.email,
                role: user.role
            }
        });
        
    } catch (error) {
        res.status(500).json({ error: 'Login failed' });
    }
});

// Protected route
app.get('/profile', authenticateToken, (req, res) => {
    res.json({
        message: 'Profile data',
        user: req.user
    });
});

app.listen(3000);
```

---

## 🔄 Refresh Token Pattern

### Access Token + Refresh Token Implementation
```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const crypto = require('crypto');

const app = express();
app.use(express.json());

const JWT_SECRET = process.env.JWT_SECRET || 'access-token-secret';
const REFRESH_SECRET = process.env.REFRESH_SECRET || 'refresh-token-secret';

// Store refresh tokens (use Redis in production)
const refreshTokens = new Set();

// Enhanced JWT utilities
const jwtUtils = {
    generateAccessToken: (user) => {
        return jwt.sign(
            {
                id: user.id,
                username: user.username,
                role: user.role
            },
            JWT_SECRET,
            { 
                expiresIn: '15m',
                issuer: 'myapp',
                audience: 'myapp-users'
            }
        );
    },
    
    generateRefreshToken: (user) => {
        const refreshToken = jwt.sign(
            { id: user.id },
            REFRESH_SECRET,
            { 
                expiresIn: '7d',
                issuer: 'myapp',
                audience: 'myapp-refresh'
            }
        );
        
        // Store refresh token
        refreshTokens.add(refreshToken);
        return refreshToken;
    },
    
    generateTokenPair: (user) => {
        return {
            accessToken: jwtUtils.generateAccessToken(user),
            refreshToken: jwtUtils.generateRefreshToken(user)
        };
    },
    
    verifyAccessToken: (token) => {
        try {
            return jwt.verify(token, JWT_SECRET, {
                issuer: 'myapp',
                audience: 'myapp-users'
            });
        } catch (error) {
            return null;
        }
    },
    
    verifyRefreshToken: (token) => {
        try {
            if (!refreshTokens.has(token)) {
                throw new Error('Token not found');
            }
            
            return jwt.verify(token, REFRESH_SECRET, {
                issuer: 'myapp',
                audience: 'myapp-refresh'
            });
        } catch (error) {
            return null;
        }
    },
    
    revokeRefreshToken: (token) => {
        refreshTokens.delete(token);
    },
    
    revokeAllUserTokens: (userId) => {
        // In production, you'd query your token store
        for (const token of refreshTokens) {
            try {
                const decoded = jwt.decode(token);
                if (decoded && decoded.id === userId) {
                    refreshTokens.delete(token);
                }
            } catch (error) {
                // Ignore invalid tokens
            }
        }
    }
};

// Authentication middleware
const authenticateToken = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    
    if (!token) {
        return res.status(401).json({ error: 'Access token required' });
    }
    
    const decoded = jwtUtils.verifyAccessToken(token);
    if (!decoded) {
        return res.status(403).json({ 
            error: 'Invalid or expired token',
            code: 'TOKEN_EXPIRED'
        });
    }
    
    req.user = decoded;
    next();
};

// Login with token pair
app.post('/auth/login', async (req, res) => {
    try {
        const { username, password } = req.body;
        
        // Validate credentials (simplified)
        const user = users.find(u => u.username === username);
        if (!user || !await bcrypt.compare(password, user.password)) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        // Generate token pair
        const tokens = jwtUtils.generateTokenPair(user);
        
        res.json({
            message: 'Login successful',
            ...tokens,
            user: {
                id: user.id,
                username: user.username,
                role: user.role
            }
        });
        
    } catch (error) {
        res.status(500).json({ error: 'Login failed' });
    }
});

// Refresh token endpoint
app.post('/auth/refresh', (req, res) => {
    const { refreshToken } = req.body;
    
    if (!refreshToken) {
        return res.status(401).json({ error: 'Refresh token required' });
    }
    
    const decoded = jwtUtils.verifyRefreshToken(refreshToken);
    if (!decoded) {
        return res.status(403).json({ error: 'Invalid or expired refresh token' });
    }
    
    // Find user
    const user = users.find(u => u.id === decoded.id);
    if (!user) {
        return res.status(403).json({ error: 'User not found' });
    }
    
    // Revoke old refresh token and generate new pair
    jwtUtils.revokeRefreshToken(refreshToken);
    const tokens = jwtUtils.generateTokenPair(user);
    
    res.json({
        message: 'Tokens refreshed',
        ...tokens
    });
});

// Logout endpoint
app.post('/auth/logout', (req, res) => {
    const { refreshToken } = req.body;
    
    if (refreshToken) {
        jwtUtils.revokeRefreshToken(refreshToken);
    }
    
    res.json({ message: 'Logout successful' });
});

// Logout from all devices
app.post('/auth/logout-all', authenticateToken, (req, res) => {
    jwtUtils.revokeAllUserTokens(req.user.id);
    res.json({ message: 'Logged out from all devices' });
});

// Token info endpoint
app.get('/auth/token-info', authenticateToken, (req, res) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader.split(' ')[1];
    const decoded = jwtUtils.decodeToken(token);
    
    res.json({
        user: req.user,
        tokenInfo: {
            issuedAt: new Date(decoded.payload.iat * 1000),
            expiresAt: new Date(decoded.payload.exp * 1000),
            issuer: decoded.payload.iss,
            audience: decoded.payload.aud
        }
    });
});

app.listen(3000);
```

---

## 🛡️ JWT Security Best Practices

### Secure JWT Implementation
```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const crypto = require('crypto');
const rateLimit = require('express-rate-limit');

const app = express();
app.use(express.json());

// Security configurations
const JWT_CONFIG = {
    accessTokenSecret: process.env.JWT_ACCESS_SECRET,
    refreshTokenSecret: process.env.JWT_REFRESH_SECRET,
    accessTokenExpiry: '15m',
    refreshTokenExpiry: '7d',
    issuer: 'myapp.com',
    audience: 'myapp-users'
};

// Rate limiting for auth endpoints
const authLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5, // 5 attempts per window
    message: 'Too many authentication attempts',
    standardHeaders: true,
    legacyHeaders: false
});

// Blacklisted tokens (use Redis in production)
const tokenBlacklist = new Set();

// Enhanced security utilities
const securityUtils = {
    // Generate cryptographically secure JTI
    generateJTI: () => {
        return crypto.randomBytes(16).toString('hex');
    },
    
    // Hash sensitive data
    hashData: (data) => {
        return crypto.createHash('sha256').update(data).digest('hex');
    },
    
    // Check if token is blacklisted
    isTokenBlacklisted: (jti) => {
        return tokenBlacklist.has(jti);
    },
    
    // Blacklist token
    blacklistToken: (jti, expiry) => {
        tokenBlacklist.add(jti);
        
        // Auto-remove from blacklist after expiry
        setTimeout(() => {
            tokenBlacklist.delete(jti);
        }, expiry * 1000);
    }
};

// Secure JWT utilities
const secureJwtUtils = {
    generateAccessToken: (user, additionalClaims = {}) => {
        const jti = securityUtils.generateJTI();
        const now = Math.floor(Date.now() / 1000);
        
        const payload = {
            // Standard claims
            jti: jti,                          // JWT ID
            sub: user.id.toString(),           // Subject
            iat: now,                          // Issued at
            exp: now + (15 * 60),              // Expires in 15 minutes
            iss: JWT_CONFIG.issuer,            // Issuer
            aud: JWT_CONFIG.audience,          // Audience
            
            // Custom claims
            username: user.username,
            role: user.role,
            sessionId: securityUtils.generateJTI(),
            ...additionalClaims
        };
        
        return jwt.sign(payload, JWT_CONFIG.accessTokenSecret, {
            algorithm: 'HS256'
        });
    },
    
    verifyToken: (token, tokenType = 'access') => {
        try {
            const secret = tokenType === 'access' 
                ? JWT_CONFIG.accessTokenSecret 
                : JWT_CONFIG.refreshTokenSecret;
            
            const decoded = jwt.verify(token, secret, {
                issuer: JWT_CONFIG.issuer,
                audience: JWT_CONFIG.audience
            });
            
            // Check if token is blacklisted
            if (securityUtils.isTokenBlacklisted(decoded.jti)) {
                throw new Error('Token has been revoked');
            }
            
            return decoded;
            
        } catch (error) {
            return null;
        }
    },
    
    revokeToken: (token) => {
        try {
            const decoded = jwt.decode(token);
            if (decoded && decoded.jti) {
                const timeToExpiry = decoded.exp - Math.floor(Date.now() / 1000);
                if (timeToExpiry > 0) {
                    securityUtils.blacklistToken(decoded.jti, timeToExpiry);
                }
            }
        } catch (error) {
            // Token is invalid, nothing to revoke
        }
    }
};

// Enhanced authentication middleware
const authenticateSecureToken = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    
    if (!token) {
        return res.status(401).json({ 
            error: 'Access token required',
            code: 'NO_TOKEN'
        });
    }
    
    const decoded = secureJwtUtils.verifyToken(token);
    if (!decoded) {
        return res.status(403).json({ 
            error: 'Invalid or expired token',
            code: 'INVALID_TOKEN'
        });
    }
    
    // Additional security checks
    if (decoded.exp < Math.floor(Date.now() / 1000)) {
        return res.status(403).json({ 
            error: 'Token expired',
            code: 'TOKEN_EXPIRED'
        });
    }
    
    req.user = decoded;
    req.tokenJTI = decoded.jti;
    next();
};

// Secure login with additional claims
app.post('/auth/secure-login', authLimiter, async (req, res) => {
    try {
        const { username, password, deviceInfo } = req.body;
        
        // Validate credentials
        const user = users.find(u => u.username === username);
        if (!user || !await bcrypt.compare(password, user.password)) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        // Additional claims for security
        const additionalClaims = {
            deviceFingerprint: securityUtils.hashData(deviceInfo || req.get('User-Agent')),
            loginTime: new Date().toISOString(),
            ipAddress: securityUtils.hashData(req.ip)
        };
        
        const accessToken = secureJwtUtils.generateAccessToken(user, additionalClaims);
        
        res.json({
            message: 'Secure login successful',
            accessToken: accessToken,
            user: {
                id: user.id,
                username: user.username,
                role: user.role
            }
        });
        
    } catch (error) {
        res.status(500).json({ error: 'Login failed' });
    }
});

// Secure logout with token revocation
app.post('/auth/secure-logout', authenticateSecureToken, (req, res) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader.split(' ')[1];
    
    // Revoke the current token
    secureJwtUtils.revokeToken(token);
    
    res.json({ 
        message: 'Secure logout successful',
        tokenRevoked: true
    });
});

// Token validation endpoint
app.get('/auth/validate', authenticateSecureToken, (req, res) => {
    res.json({
        valid: true,
        user: {
            id: req.user.sub,
            username: req.user.username,
            role: req.user.role
        },
        tokenInfo: {
            jti: req.user.jti,
            issuedAt: new Date(req.user.iat * 1000),
            expiresAt: new Date(req.user.exp * 1000),
            sessionId: req.user.sessionId
        }
    });
});

app.listen(3000);
```

---

## 🔧 JWT with Different Algorithms

### RS256 (RSA) Implementation
```javascript
const fs = require('fs');
const jwt = require('jsonwebtoken');

// Load RSA keys
const privateKey = fs.readFileSync('private.pem', 'utf8');
const publicKey = fs.readFileSync('public.pem', 'utf8');

const rsaJwtUtils = {
    generateToken: (payload) => {
        return jwt.sign(payload, privateKey, {
            algorithm: 'RS256',
            expiresIn: '1h',
            issuer: 'myapp.com',
            audience: 'myapp-users'
        });
    },
    
    verifyToken: (token) => {
        try {
            return jwt.verify(token, publicKey, {
                algorithms: ['RS256'],
                issuer: 'myapp.com',
                audience: 'myapp-users'
            });
        } catch (error) {
            return null;
        }
    }
};

// Generate RSA key pair (run once)
// openssl genrsa -out private.pem 2048
// openssl rsa -in private.pem -pubout -out public.pem
```

### Multiple Algorithm Support
```javascript
const jwt = require('jsonwebtoken');

const multiAlgoJwtUtils = {
    algorithms: {
        HS256: {
            sign: (payload, secret) => jwt.sign(payload, secret, { algorithm: 'HS256' }),
            verify: (token, secret) => jwt.verify(token, secret, { algorithms: ['HS256'] })
        },
        RS256: {
            sign: (payload, privateKey) => jwt.sign(payload, privateKey, { algorithm: 'RS256' }),
            verify: (token, publicKey) => jwt.verify(token, publicKey, { algorithms: ['RS256'] })
        }
    },
    
    generateToken: (payload, algorithm = 'HS256', secret) => {
        const algoImpl = multiAlgoJwtUtils.algorithms[algorithm];
        if (!algoImpl) {
            throw new Error(`Unsupported algorithm: ${algorithm}`);
        }
        
        return algoImpl.sign(payload, secret);
    },
    
    verifyToken: (token, algorithm = 'HS256', secret) => {
        try {
            const algoImpl = multiAlgoJwtUtils.algorithms[algorithm];
            if (!algoImpl) {
                throw new Error(`Unsupported algorithm: ${algorithm}`);
            }
            
            return algoImpl.verify(token, secret);
        } catch (error) {
            return null;
        }
    }
};
```

---

## 📊 JWT Analytics and Monitoring

### Token Usage Analytics
```javascript
const express = require('express');
const jwt = require('jsonwebtoken');

const app = express();

// Token analytics store
const tokenAnalytics = {
    issued: new Map(),      // tokenId -> issuance info
    accesses: new Map(),    // tokenId -> access history
    revoked: new Set()      // revoked token IDs
};

// Analytics utilities
const analyticsUtils = {
    logTokenIssuance: (tokenId, userId, metadata = {}) => {
        tokenAnalytics.issued.set(tokenId, {
            userId,
            issuedAt: new Date(),
            metadata,
            accessCount: 0
        });
    },
    
    logTokenAccess: (tokenId, endpoint, ip) => {
        if (!tokenAnalytics.accesses.has(tokenId)) {
            tokenAnalytics.accesses.set(tokenId, []);
        }
        
        tokenAnalytics.accesses.get(tokenId).push({
            endpoint,
            ip,
            timestamp: new Date()
        });
        
        // Increment access count
        const issuanceInfo = tokenAnalytics.issued.get(tokenId);
        if (issuanceInfo) {
            issuanceInfo.accessCount++;
        }
    },
    
    getTokenStats: (tokenId) => {
        return {
            issuance: tokenAnalytics.issued.get(tokenId),
            accesses: tokenAnalytics.accesses.get(tokenId) || [],
            isRevoked: tokenAnalytics.revoked.has(tokenId)
        };
    },
    
    getUserTokens: (userId) => {
        const userTokens = [];
        for (const [tokenId, info] of tokenAnalytics.issued.entries()) {
            if (info.userId === userId) {
                userTokens.push({
                    tokenId,
                    ...info,
                    accesses: tokenAnalytics.accesses.get(tokenId) || []
                });
            }
        }
        return userTokens;
    }
};

// Analytics middleware
const tokenAnalyticsMiddleware = (req, res, next) => {
    const authHeader = req.headers['authorization'];
    const token = authHeader && authHeader.split(' ')[1];
    
    if (token) {
        try {
            const decoded = jwt.decode(token);
            if (decoded && decoded.jti) {
                analyticsUtils.logTokenAccess(decoded.jti, req.path, req.ip);
            }
        } catch (error) {
            // Ignore decode errors for analytics
        }
    }
    
    next();
};

app.use(tokenAnalyticsMiddleware);

// Enhanced token generation with analytics
const generateAnalyticsToken = (user, metadata = {}) => {
    const jti = crypto.randomBytes(16).toString('hex');
    
    const payload = {
        jti,
        sub: user.id.toString(),
        username: user.username,
        role: user.role,
        iat: Math.floor(Date.now() / 1000),
        exp: Math.floor(Date.now() / 1000) + (15 * 60)
    };
    
    const token = jwt.sign(payload, JWT_SECRET);
    
    // Log token issuance
    analyticsUtils.logTokenIssuance(jti, user.id, metadata);
    
    return token;
};

// Analytics endpoints
app.get('/admin/token-analytics', authenticateToken, (req, res) => {
    const stats = {
        totalIssued: tokenAnalytics.issued.size,
        totalRevoked: tokenAnalytics.revoked.size,
        activeTokens: tokenAnalytics.issued.size - tokenAnalytics.revoked.size,
        totalAccesses: Array.from(tokenAnalytics.accesses.values())
            .reduce((total, accesses) => total + accesses.length, 0)
    };
    
    res.json({ stats });
});

app.get('/admin/user-tokens/:userId', authenticateToken, (req, res) => {
    const userId = parseInt(req.params.userId);
    const userTokens = analyticsUtils.getUserTokens(userId);
    
    res.json({ userTokens });
});

app.get('/user/my-tokens', authenticateToken, (req, res) => {
    const userTokens = analyticsUtils.getUserTokens(req.user.sub);
    const currentTokenStats = analyticsUtils.getTokenStats(req.user.jti);
    
    res.json({ 
        currentToken: currentTokenStats,
        allTokens: userTokens 
    });
});

app.listen(3000);
```

---

## 🔗 Related Topics
- [[11-Authentication-and-Authorization]] - Auth strategies overview
- [[10-Express-Sessions]] - Session-based alternative
- [[13-Rate-Limiting]] - Protecting token endpoints
- [[14-Environment-Variables]] - Securing JWT secrets

---

*Back to [[00-Main-Index]]*
