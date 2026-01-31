# Authentication and Authorization

**Navigation**: [[10-Express-Sessions]] | [[00-Main-Index]] | Next: [[12-JWT-Tokens]]

---

## 🔐 Authentication vs Authorization

**Authentication** = Who you are (Login)  
**Authorization** = What you can do (Permissions)

---

## 🎯 Basic Authentication Strategies

### 1. Session-Based Authentication
```javascript
const express = require('express');
const session = require('express-session');
const bcrypt = require('bcrypt');

const app = express();

app.use(express.json());
app.use(session({
    secret: 'auth-secret',
    resave: false,
    saveUninitialized: false,
    cookie: { maxAge: 24 * 60 * 60 * 1000 } // 24 hours
}));

// Mock user database
const users = [
    {
        id: 1,
        username: 'admin',
        email: 'admin@example.com',
        password: '$2b$10$hash...', // Pre-hashed password
        role: 'admin',
        permissions: ['read', 'write', 'delete', 'admin']
    },
    {
        id: 2,
        username: 'user',
        email: 'user@example.com', 
        password: '$2b$10$hash...',
        role: 'user',
        permissions: ['read', 'write']
    }
];

// Authentication middleware
const authenticate = (req, res, next) => {
    if (req.session && req.session.userId) {
        const user = users.find(u => u.id === req.session.userId);
        if (user) {
            req.user = user;
            return next();
        }
    }
    res.status(401).json({ error: 'Authentication required' });
};

// Authorization middleware
const authorize = (roles = []) => {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ error: 'Authentication required' });
        }
        
        if (roles.length && !roles.includes(req.user.role)) {
            return res.status(403).json({ error: 'Access forbidden' });
        }
        
        next();
    };
};

// Permission-based authorization
const requirePermission = (permission) => {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ error: 'Authentication required' });
        }
        
        if (!req.user.permissions.includes(permission)) {
            return res.status(403).json({ error: `Permission '${permission}' required` });
        }
        
        next();
    };
};

// Routes
app.post('/login', async (req, res) => {
    const { username, password } = req.body;
    
    const user = users.find(u => u.username === username);
    if (!user || !await bcrypt.compare(password, user.password)) {
        return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    req.session.userId = user.id;
    res.json({ 
        message: 'Login successful',
        user: { id: user.id, username: user.username, role: user.role }
    });
});

// Public route
app.get('/public', (req, res) => {
    res.json({ message: 'Public data' });
});

// Protected route - any authenticated user
app.get('/protected', authenticate, (req, res) => {
    res.json({ 
        message: 'Protected data',
        user: req.user.username 
    });
});

// Admin only route
app.get('/admin', authenticate, authorize(['admin']), (req, res) => {
    res.json({ message: 'Admin data' });
});

// Permission-based route
app.delete('/data/:id', authenticate, requirePermission('delete'), (req, res) => {
    res.json({ message: `Data ${req.params.id} deleted` });
});

app.listen(3000);
```

---

## 🔑 JWT-Based Authentication

### Complete JWT Authentication System
```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

const app = express();
app.use(express.json());

const JWT_SECRET = process.env.JWT_SECRET || 'your-jwt-secret';
const JWT_REFRESH_SECRET = process.env.JWT_REFRESH_SECRET || 'your-refresh-secret';

// Mock users database
const users = [
    {
        id: 1,
        username: 'admin',
        email: 'admin@example.com',
        password: '$2b$10$hash...',
        role: 'admin',
        refreshTokens: []
    }
];

// JWT utilities
const jwtUtils = {
    generateTokens: (user) => {
        const payload = {
            id: user.id,
            username: user.username,
            role: user.role
        };
        
        const accessToken = jwt.sign(payload, JWT_SECRET, { expiresIn: '15m' });
        const refreshToken = jwt.sign({ id: user.id }, JWT_REFRESH_SECRET, { expiresIn: '7d' });
        
        return { accessToken, refreshToken };
    },
    
    verifyToken: (token, secret = JWT_SECRET) => {
        try {
            return jwt.verify(token, secret);
        } catch (error) {
            return null;
        }
    }
};

// JWT Authentication middleware
const jwtAuth = (req, res, next) => {
    const authHeader = req.headers.authorization;
    const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN
    
    if (!token) {
        return res.status(401).json({ error: 'Access token required' });
    }
    
    const decoded = jwtUtils.verifyToken(token);
    if (!decoded) {
        return res.status(401).json({ error: 'Invalid or expired token' });
    }
    
    req.user = decoded;
    next();
};

// Role-based authorization
const requireRole = (roles) => {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ error: 'Authentication required' });
        }
        
        if (!roles.includes(req.user.role)) {
            return res.status(403).json({ error: 'Insufficient permissions' });
        }
        
        next();
    };
};

// Login endpoint
app.post('/auth/login', async (req, res) => {
    try {
        const { username, password } = req.body;
        
        const user = users.find(u => u.username === username);
        if (!user || !await bcrypt.compare(password, user.password)) {
            return res.status(401).json({ error: 'Invalid credentials' });
        }
        
        const { accessToken, refreshToken } = jwtUtils.generateTokens(user);
        
        // Store refresh token
        user.refreshTokens.push(refreshToken);
        
        res.json({
            message: 'Login successful',
            accessToken,
            refreshToken,
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
    
    const decoded = jwtUtils.verifyToken(refreshToken, JWT_REFRESH_SECRET);
    if (!decoded) {
        return res.status(401).json({ error: 'Invalid refresh token' });
    }
    
    const user = users.find(u => u.id === decoded.id);
    if (!user || !user.refreshTokens.includes(refreshToken)) {
        return res.status(401).json({ error: 'Invalid refresh token' });
    }
    
    const { accessToken, refreshToken: newRefreshToken } = jwtUtils.generateTokens(user);
    
    // Replace old refresh token
    user.refreshTokens = user.refreshTokens.filter(token => token !== refreshToken);
    user.refreshTokens.push(newRefreshToken);
    
    res.json({ accessToken, refreshToken: newRefreshToken });
});

// Logout endpoint
app.post('/auth/logout', jwtAuth, (req, res) => {
    const { refreshToken } = req.body;
    
    const user = users.find(u => u.id === req.user.id);
    if (user && refreshToken) {
        user.refreshTokens = user.refreshTokens.filter(token => token !== refreshToken);
    }
    
    res.json({ message: 'Logout successful' });
});

// Protected routes
app.get('/profile', jwtAuth, (req, res) => {
    res.json({
        message: 'Profile data',
        user: req.user
    });
});

app.get('/admin', jwtAuth, requireRole(['admin']), (req, res) => {
    res.json({ message: 'Admin area' });
});

app.listen(3000);
```

---

## 🛡️ Advanced Authorization Patterns

### Role-Based Access Control (RBAC)
```javascript
const express = require('express');
const app = express();

app.use(express.json());

// RBAC Configuration
const permissions = {
    'user:read': 'Read user data',
    'user:write': 'Create/update user data',
    'user:delete': 'Delete user data',
    'post:read': 'Read posts',
    'post:write': 'Create/update posts',
    'post:delete': 'Delete posts',
    'admin:manage': 'Manage system'
};

const roles = {
    'guest': [],
    'user': ['user:read', 'post:read'],
    'moderator': ['user:read', 'post:read', 'post:write', 'post:delete'],
    'admin': Object.keys(permissions)
};

// Mock users with roles
const users = [
    {
        id: 1,
        username: 'admin',
        role: 'admin',
        customPermissions: [] // Additional permissions
    },
    {
        id: 2,
        username: 'moderator',
        role: 'moderator',
        customPermissions: ['user:write'] // Extra permission
    },
    {
        id: 3,
        username: 'user',
        role: 'user',
        customPermissions: []
    }
];

// RBAC utilities
const rbacUtils = {
    getUserPermissions: (user) => {
        const rolePermissions = roles[user.role] || [];
        return [...rolePermissions, ...user.customPermissions];
    },
    
    hasPermission: (user, permission) => {
        const userPermissions = rbacUtils.getUserPermissions(user);
        return userPermissions.includes(permission);
    },
    
    hasAnyPermission: (user, permissionList) => {
        return permissionList.some(permission => 
            rbacUtils.hasPermission(user, permission)
        );
    },
    
    hasAllPermissions: (user, permissionList) => {
        return permissionList.every(permission => 
            rbacUtils.hasPermission(user, permission)
        );
    }
};

// RBAC middleware
const requirePermissions = (requiredPermissions, requireAll = true) => {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ error: 'Authentication required' });
        }
        
        const hasAccess = requireAll 
            ? rbacUtils.hasAllPermissions(req.user, requiredPermissions)
            : rbacUtils.hasAnyPermission(req.user, requiredPermissions);
        
        if (!hasAccess) {
            return res.status(403).json({ 
                error: 'Insufficient permissions',
                required: requiredPermissions,
                userPermissions: rbacUtils.getUserPermissions(req.user)
            });
        }
        
        next();
    };
};

// Mock authentication middleware
const authenticate = (req, res, next) => {
    const userId = req.headers['x-user-id'];
    const user = users.find(u => u.id === parseInt(userId));
    
    if (!user) {
        return res.status(401).json({ error: 'Authentication required' });
    }
    
    req.user = user;
    next();
};

// Routes with RBAC
app.get('/users', authenticate, requirePermissions(['user:read']), (req, res) => {
    res.json({ message: 'User list', users: users.map(u => ({ id: u.id, username: u.username })) });
});

app.post('/users', authenticate, requirePermissions(['user:write']), (req, res) => {
    res.json({ message: 'User created' });
});

app.delete('/users/:id', authenticate, requirePermissions(['user:delete']), (req, res) => {
    res.json({ message: `User ${req.params.id} deleted` });
});

// Multiple permissions (require all)
app.get('/admin/users', authenticate, requirePermissions(['user:read', 'admin:manage'], true), (req, res) => {
    res.json({ message: 'Admin user management' });
});

// Multiple permissions (require any)
app.get('/content', authenticate, requirePermissions(['post:read', 'admin:manage'], false), (req, res) => {
    res.json({ message: 'Content data' });
});

// Check user permissions endpoint
app.get('/permissions', authenticate, (req, res) => {
    res.json({
        user: req.user.username,
        role: req.user.role,
        permissions: rbacUtils.getUserPermissions(req.user)
    });
});

app.listen(3000);
```

---

## 🔒 Multi-Factor Authentication (MFA)

### TOTP (Time-based One-Time Password) Implementation
```javascript
const express = require('express');
const speakeasy = require('speakeasy');
const qrcode = require('qrcode');

const app = express();
app.use(express.json());

// Mock user database with MFA
const users = [
    {
        id: 1,
        username: 'admin',
        password: 'hashedpassword',
        mfaEnabled: false,
        mfaSecret: null,
        mfaBackupCodes: []
    }
];

// MFA utilities
const mfaUtils = {
    generateSecret: (username) => {
        return speakeasy.generateSecret({
            name: `MyApp (${username})`,
            issuer: 'MyApp'
        });
    },
    
    generateBackupCodes: () => {
        const codes = [];
        for (let i = 0; i < 10; i++) {
            codes.push(Math.random().toString(36).substr(2, 8).toUpperCase());
        }
        return codes;
    },
    
    verifyToken: (secret, token) => {
        return speakeasy.totp.verify({
            secret: secret,
            encoding: 'base32',
            token: token,
            window: 2 // Allow 2 time windows
        });
    }
};

// Step 1: Enable MFA - Generate QR code
app.post('/mfa/setup', authenticate, async (req, res) => {
    try {
        const user = users.find(u => u.id === req.user.id);
        
        if (user.mfaEnabled) {
            return res.status(400).json({ error: 'MFA already enabled' });
        }
        
        const secret = mfaUtils.generateSecret(user.username);
        
        // Generate QR code
        const qrCodeUrl = await qrcode.toDataURL(secret.otpauth_url);
        
        // Store secret temporarily (not yet enabled)
        user.mfaTempSecret = secret.base32;
        
        res.json({
            message: 'Scan QR code with authenticator app',
            qrCode: qrCodeUrl,
            manualEntryKey: secret.base32
        });
        
    } catch (error) {
        res.status(500).json({ error: 'MFA setup failed' });
    }
});

// Step 2: Verify and activate MFA
app.post('/mfa/verify-setup', authenticate, (req, res) => {
    const { token } = req.body;
    const user = users.find(u => u.id === req.user.id);
    
    if (!user.mfaTempSecret) {
        return res.status(400).json({ error: 'MFA setup not initiated' });
    }
    
    const isValid = mfaUtils.verifyToken(user.mfaTempSecret, token);
    
    if (!isValid) {
        return res.status(401).json({ error: 'Invalid verification code' });
    }
    
    // Activate MFA
    user.mfaSecret = user.mfaTempSecret;
    user.mfaEnabled = true;
    user.mfaBackupCodes = mfaUtils.generateBackupCodes();
    delete user.mfaTempSecret;
    
    res.json({
        message: 'MFA enabled successfully',
        backupCodes: user.mfaBackupCodes
    });
});

// MFA verification middleware
const requireMFA = (req, res, next) => {
    if (!req.user.mfaEnabled) {
        return next(); // Skip if MFA not enabled
    }
    
    if (req.session && req.session.mfaVerified) {
        return next(); // Already verified in this session
    }
    
    res.status(401).json({ 
        error: 'MFA verification required',
        mfaRequired: true 
    });
};

// MFA verification endpoint
app.post('/mfa/verify', authenticate, (req, res) => {
    const { token, backupCode } = req.body;
    const user = users.find(u => u.id === req.user.id);
    
    if (!user.mfaEnabled) {
        return res.status(400).json({ error: 'MFA not enabled' });
    }
    
    let isValid = false;
    
    if (token) {
        isValid = mfaUtils.verifyToken(user.mfaSecret, token);
    } else if (backupCode) {
        const codeIndex = user.mfaBackupCodes.indexOf(backupCode.toUpperCase());
        if (codeIndex !== -1) {
            user.mfaBackupCodes.splice(codeIndex, 1); // Remove used backup code
            isValid = true;
        }
    }
    
    if (!isValid) {
        return res.status(401).json({ error: 'Invalid verification code' });
    }
    
    // Mark as verified in session
    if (req.session) {
        req.session.mfaVerified = true;
    }
    
    res.json({ message: 'MFA verification successful' });
});

// Protected route requiring MFA
app.get('/sensitive-data', authenticate, requireMFA, (req, res) => {
    res.json({ message: 'Sensitive data accessed with MFA' });
});

app.listen(3000);
```

---

## 🌐 OAuth 2.0 Integration

### Google OAuth Implementation
```javascript
const express = require('express');
const passport = require('passport');
const GoogleStrategy = require('passport-google-oauth20').Strategy;

const app = express();

app.use(express.json());
app.use(passport.initialize());

// Configure Google OAuth strategy
passport.use(new GoogleStrategy({
    clientID: process.env.GOOGLE_CLIENT_ID,
    clientSecret: process.env.GOOGLE_CLIENT_SECRET,
    callbackURL: "/auth/google/callback"
}, async (accessToken, refreshToken, profile, done) => {
    try {
        // Check if user exists in database
        let user = users.find(u => u.googleId === profile.id);
        
        if (!user) {
            // Create new user
            user = {
                id: users.length + 1,
                googleId: profile.id,
                username: profile.displayName,
                email: profile.emails[0].value,
                avatar: profile.photos[0].value,
                role: 'user',
                provider: 'google'
            };
            users.push(user);
        }
        
        return done(null, user);
    } catch (error) {
        return done(error, null);
    }
}));

// OAuth routes
app.get('/auth/google', 
    passport.authenticate('google', { scope: ['profile', 'email'] })
);

app.get('/auth/google/callback',
    passport.authenticate('google', { session: false }),
    (req, res) => {
        // Generate JWT token for the user
        const token = jwtUtils.generateTokens(req.user);
        
        // Redirect to frontend with token
        res.redirect(`${process.env.FRONTEND_URL}/auth/success?token=${token.accessToken}`);
    }
);

app.listen(3000);
```

---

## 🔗 Related Topics
- [[10-Express-Sessions]] - Session-based authentication
- [[12-JWT-Tokens]] - JWT implementation details  
- [[13-Rate-Limiting]] - Protecting authentication endpoints
- [[14-Environment-Variables]] - Securing auth secrets

---

*Back to [[00-Main-Index]]*
