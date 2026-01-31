# Cookies & Sessions

**Navigation**: [[22-Error-Handling]] | [[00-Main-Index]] | Next: [[24-File-Uploads]]

---

## 🍪 Cookies & Sessions in Node.js

**Cookies** and **Sessions** are fundamental for maintaining state in web applications. They enable user authentication, preferences storage, shopping carts, and personalized experiences.

---

## 🔧 Understanding Cookies

### What are Cookies?
```javascript
// Cookies are small pieces of data stored in the user's browser
// They are sent with every HTTP request to the same domain

// Basic cookie structure:
// Name=Value; Expires=Date; Path=/; Domain=example.com; Secure; HttpOnly; SameSite=Strict
```

### Cookie Implementation
```javascript
// Basic cookie operations
const express = require('express');
const cookieParser = require('cookie-parser');

const app = express();

// Parse cookies from incoming requests
app.use(cookieParser());

// Setting cookies
app.get('/set-cookie', (req, res) => {
    // Simple cookie
    res.cookie('username', 'john_doe');
    
    // Cookie with options
    res.cookie('preferences', JSON.stringify({
        theme: 'dark',
        language: 'en'
    }), {
        maxAge: 30 * 24 * 60 * 60 * 1000, // 30 days
        httpOnly: true, // Not accessible via JavaScript
        secure: process.env.NODE_ENV === 'production', // HTTPS only in production
        sameSite: 'strict' // CSRF protection
    });
    
    // Signed cookie (tamper-proof)
    res.cookie('userId', '12345', {
        signed: true,
        maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
    });
    
    res.json({ message: 'Cookies set successfully' });
});

// Reading cookies
app.get('/get-cookies', (req, res) => {
    const cookies = req.cookies;
    const signedCookies = req.signedCookies;
    
    res.json({
        cookies,
        signedCookies,
        username: cookies.username,
        preferences: cookies.preferences ? JSON.parse(cookies.preferences) : null
    });
});

// Clearing cookies
app.get('/clear-cookies', (req, res) => {
    res.clearCookie('username');
    res.clearCookie('preferences');
    res.clearCookie('userId');
    
    res.json({ message: 'Cookies cleared' });
});
```

---

## 🗄️ Cookie Security Best Practices

### Secure Cookie Configuration
```javascript
// utils/cookieConfig.js
const getCookieConfig = (type = 'default') => {
    const baseConfig = {
        httpOnly: true,
        secure: process.env.NODE_ENV === 'production',
        sameSite: 'strict'
    };
    
    const configs = {
        // Authentication cookies
        auth: {
            ...baseConfig,
            maxAge: 7 * 24 * 60 * 60 * 1000, // 7 days
            signed: true
        },
        
        // Session cookies
        session: {
            ...baseConfig,
            maxAge: 24 * 60 * 60 * 1000, // 24 hours
            signed: true
        },
        
        // Remember me cookies
        rememberMe: {
            ...baseConfig,
            maxAge: 30 * 24 * 60 * 60 * 1000, // 30 days
            signed: true
        },
        
        // Preference cookies
        preferences: {
            httpOnly: false, // Accessible via JavaScript
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'lax',
            maxAge: 365 * 24 * 60 * 60 * 1000 // 1 year
        },
        
        // CSRF token cookies
        csrf: {
            httpOnly: false, // Need to read in frontend
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'strict',
            maxAge: 60 * 60 * 1000 // 1 hour
        }
    };
    
    return configs[type] || baseConfig;
};

// Cookie encryption/decryption
const crypto = require('crypto');

class CookieEncryption {
    constructor(secretKey) {
        this.algorithm = 'aes-256-gcm';
        this.secretKey = crypto.scryptSync(secretKey, 'salt', 32);
    }
    
    encrypt(text) {
        const iv = crypto.randomBytes(16);
        const cipher = crypto.createCipher(this.algorithm, this.secretKey);
        cipher.setAAD(Buffer.from('cookie-data', 'utf8'));
        
        let encrypted = cipher.update(text, 'utf8', 'hex');
        encrypted += cipher.final('hex');
        
        const authTag = cipher.getAuthTag();
        
        return {
            iv: iv.toString('hex'),
            authTag: authTag.toString('hex'),
            encrypted
        };
    }
    
    decrypt(encryptedData) {
        try {
            const { iv, authTag, encrypted } = encryptedData;
            const decipher = crypto.createDecipher(this.algorithm, this.secretKey);
            
            decipher.setAAD(Buffer.from('cookie-data', 'utf8'));
            decipher.setAuthTag(Buffer.from(authTag, 'hex'));
            
            let decrypted = decipher.update(encrypted, 'hex', 'utf8');
            decrypted += decipher.final('utf8');
            
            return decrypted;
        } catch (error) {
            throw new Error('Failed to decrypt cookie data');
        }
    }
}

const cookieEncryption = new CookieEncryption(process.env.COOKIE_SECRET || 'default-secret');

module.exports = {
    getCookieConfig,
    cookieEncryption
};
```

---

## 📝 Session Management

### Express Session Configuration
```javascript
// config/session.js
const session = require('express-session');
const MongoStore = require('connect-mongo');
const RedisStore = require('connect-redis')(session);
const Redis = require('ioredis');

// Redis client for session storage
const redisClient = new Redis(process.env.REDIS_URL || 'redis://localhost:6379');

// Session configuration
const getSessionConfig = () => {
    const config = {
        name: 'sessionId', // Don't use default 'connect.sid'
        secret: process.env.SESSION_SECRET || 'your-secret-key',
        resave: false,
        saveUninitialized: false,
        rolling: true, // Reset expiration on activity
        cookie: {
            secure: process.env.NODE_ENV === 'production',
            httpOnly: true,
            maxAge: 24 * 60 * 60 * 1000, // 24 hours
            sameSite: 'strict'
        }
    };
    
    // Choose session store based on environment
    if (process.env.REDIS_URL) {
        // Redis store for production
        config.store = new RedisStore({ 
            client: redisClient,
            prefix: 'sess:',
            ttl: 24 * 60 * 60 // 24 hours in seconds
        });
    } else if (process.env.MONGODB_URI) {
        // MongoDB store
        config.store = MongoStore.create({
            mongoUrl: process.env.MONGODB_URI,
            collectionName: 'sessions',
            ttl: 24 * 60 * 60 // 24 hours in seconds
        });
    }
    // Default memory store for development (not for production)
    
    return config;
};

module.exports = {
    getSessionConfig,
    redisClient
};
```

### Session Middleware and Utilities
```javascript
// middleware/sessionMiddleware.js
const session = require('express-session');
const { getSessionConfig } = require('../config/session');

// Session middleware
const sessionMiddleware = session(getSessionConfig());

// Session helpers
const sessionHelpers = {
    // Initialize user session
    initUserSession: (req, user) => {
        req.session.userId = user.id;
        req.session.userEmail = user.email;
        req.session.userRole = user.role;
        req.session.loginTime = new Date().toISOString();
        req.session.isAuthenticated = true;
    },
    
    // Destroy user session
    destroyUserSession: (req) => {
        return new Promise((resolve, reject) => {
            req.session.destroy((err) => {
                if (err) {
                    reject(err);
                } else {
                    resolve();
                }
            });
        });
    },
    
    // Regenerate session ID (prevent session fixation)
    regenerateSession: (req) => {
        return new Promise((resolve, reject) => {
            const sessionData = req.session;
            
            req.session.regenerate((err) => {
                if (err) {
                    reject(err);
                } else {
                    // Restore session data
                    Object.assign(req.session, sessionData);
                    resolve();
                }
            });
        });
    },
    
    // Check if user is authenticated
    isAuthenticated: (req) => {
        return req.session && req.session.isAuthenticated === true;
    },
    
    // Get user data from session
    getSessionUser: (req) => {
        if (!sessionHelpers.isAuthenticated(req)) {
            return null;
        }
        
        return {
            id: req.session.userId,
            email: req.session.userEmail,
            role: req.session.userRole,
            loginTime: req.session.loginTime
        };
    },
    
    // Update session data
    updateSession: (req, data) => {
        Object.assign(req.session, data);
    },
    
    // Flash messages
    setFlashMessage: (req, type, message) => {
        if (!req.session.flash) {
            req.session.flash = {};
        }
        if (!req.session.flash[type]) {
            req.session.flash[type] = [];
        }
        req.session.flash[type].push(message);
    },
    
    getFlashMessages: (req) => {
        const flash = req.session.flash || {};
        req.session.flash = {}; // Clear after reading
        return flash;
    }
};

// Authentication middleware using sessions
const requireAuth = (req, res, next) => {
    if (!sessionHelpers.isAuthenticated(req)) {
        if (req.xhr || req.headers.accept?.includes('application/json')) {
            return res.status(401).json({
                success: false,
                message: 'Authentication required'
            });
        } else {
            sessionHelpers.setFlashMessage(req, 'error', 'Please log in to access this page');
            return res.redirect('/login');
        }
    }
    
    // Add user data to request
    req.user = sessionHelpers.getSessionUser(req);
    next();
};

// Role-based authorization
const requireRole = (roles) => {
    return (req, res, next) => {
        if (!sessionHelpers.isAuthenticated(req)) {
            return res.status(401).json({
                success: false,
                message: 'Authentication required'
            });
        }
        
        const userRole = req.session.userRole;
        const allowedRoles = Array.isArray(roles) ? roles : [roles];
        
        if (!allowedRoles.includes(userRole)) {
            return res.status(403).json({
                success: false,
                message: 'Insufficient permissions'
            });
        }
        
        next();
    };
};

module.exports = {
    sessionMiddleware,
    sessionHelpers,
    requireAuth,
    requireRole
};
```

---

## 🔐 Authentication with Sessions

### Session-Based Authentication Service
```javascript
// services/AuthService.js
const bcrypt = require('bcrypt');
const User = require('../models/User');
const { sessionHelpers } = require('../middleware/sessionMiddleware');

class AuthService {
    async login(email, password, req, rememberMe = false) {
        try {
            // Find user
            const user = await User.findOne({ email }).select('+password');
            if (!user) {
                throw new Error('Invalid email or password');
            }
            
            // Check password
            const isPasswordValid = await bcrypt.compare(password, user.password);
            if (!isPasswordValid) {
                throw new Error('Invalid email or password');
            }
            
            // Check if account is active
            if (!user.isActive) {
                throw new Error('Account is not activated');
            }
            
            // Regenerate session to prevent session fixation
            await sessionHelpers.regenerateSession(req);
            
            // Initialize session
            sessionHelpers.initUserSession(req, user);
            
            // Set remember me cookie if requested
            if (rememberMe) {
                req.session.cookie.maxAge = 30 * 24 * 60 * 60 * 1000; // 30 days
            }
            
            // Update last login
            await User.findByIdAndUpdate(user.id, {
                lastLogin: new Date(),
                $inc: { loginCount: 1 }
            });
            
            // Return user data (without password)
            const userData = user.toObject();
            delete userData.password;
            
            return userData;
            
        } catch (error) {
            throw error;
        }
    }
    
    async logout(req) {
        try {
            // Clear session
            await sessionHelpers.destroyUserSession(req);
            
            return { message: 'Logged out successfully' };
            
        } catch (error) {
            throw new Error('Failed to logout');
        }
    }
    
    async register(userData, req) {
        try {
            // Check if user already exists
            const existingUser = await User.findOne({ email: userData.email });
            if (existingUser) {
                throw new Error('Email already registered');
            }
            
            // Hash password
            const saltRounds = 12;
            const hashedPassword = await bcrypt.hash(userData.password, saltRounds);
            
            // Create user
            const user = new User({
                ...userData,
                password: hashedPassword,
                isActive: false, // Require email verification
                verificationToken: this.generateVerificationToken()
            });
            
            await user.save();
            
            // Send verification email
            await this.sendVerificationEmail(user);
            
            return {
                message: 'Registration successful. Please check your email to verify your account.',
                userId: user.id
            };
            
        } catch (error) {
            throw error;
        }
    }
    
    async changePassword(userId, currentPassword, newPassword) {
        try {
            const user = await User.findById(userId).select('+password');
            if (!user) {
                throw new Error('User not found');
            }
            
            // Verify current password
            const isCurrentPasswordValid = await bcrypt.compare(currentPassword, user.password);
            if (!isCurrentPasswordValid) {
                throw new Error('Current password is incorrect');
            }
            
            // Hash new password
            const saltRounds = 12;
            const hashedNewPassword = await bcrypt.hash(newPassword, saltRounds);
            
            // Update password
            await User.findByIdAndUpdate(userId, {
                password: hashedNewPassword,
                passwordChangedAt: new Date()
            });
            
            return { message: 'Password changed successfully' };
            
        } catch (error) {
            throw error;
        }
    }
    
    generateVerificationToken() {
        const crypto = require('crypto');
        return crypto.randomBytes(32).toString('hex');
    }
    
    async sendVerificationEmail(user) {
        // Implement email sending logic
        console.log(`Verification email sent to ${user.email}`);
        // In real application, use nodemailer or similar service
    }
}

module.exports = new AuthService();
```

---

## 🎮 Authentication Controllers

### Session-Based Auth Controller
```javascript
// controllers/AuthController.js
const AuthService = require('../services/AuthService');
const { sessionHelpers } = require('../middleware/sessionMiddleware');
const { asyncErrorHandler } = require('../middleware/errorHandler');

class AuthController {
    // Login
    static login = asyncErrorHandler(async (req, res) => {
        const { email, password, rememberMe } = req.body;
        
        const user = await AuthService.login(email, password, req, rememberMe);
        
        sessionHelpers.setFlashMessage(req, 'success', 'Welcome back!');
        
        // Determine redirect URL
        const redirectUrl = req.session.returnUrl || '/dashboard';
        delete req.session.returnUrl;
        
        if (req.xhr || req.headers.accept?.includes('application/json')) {
            res.json({
                success: true,
                message: 'Login successful',
                user,
                redirectUrl
            });
        } else {
            res.redirect(redirectUrl);
        }
    });
    
    // Register
    static register = asyncErrorHandler(async (req, res) => {
        const userData = req.body;
        
        const result = await AuthService.register(userData, req);
        
        sessionHelpers.setFlashMessage(req, 'success', result.message);
        
        if (req.xhr || req.headers.accept?.includes('application/json')) {
            res.status(201).json({
                success: true,
                message: result.message
            });
        } else {
            res.redirect('/login');
        }
    });
    
    // Logout
    static logout = asyncErrorHandler(async (req, res) => {
        await AuthService.logout(req);
        
        if (req.xhr || req.headers.accept?.includes('application/json')) {
            res.json({
                success: true,
                message: 'Logged out successfully'
            });
        } else {
            res.redirect('/');
        }
    });
    
    // Get current user
    static getCurrentUser = asyncErrorHandler(async (req, res) => {
        const user = sessionHelpers.getSessionUser(req);
        
        if (!user) {
            return res.status(401).json({
                success: false,
                message: 'Not authenticated'
            });
        }
        
        res.json({
            success: true,
            user
        });
    });
    
    // Change password
    static changePassword = asyncErrorHandler(async (req, res) => {
        const { currentPassword, newPassword } = req.body;
        const userId = req.session.userId;
        
        const result = await AuthService.changePassword(userId, currentPassword, newPassword);
        
        sessionHelpers.setFlashMessage(req, 'success', result.message);
        
        res.json({
            success: true,
            message: result.message
        });
    });
    
    // Render login page
    static renderLogin = (req, res) => {
        if (sessionHelpers.isAuthenticated(req)) {
            return res.redirect('/dashboard');
        }
        
        const flash = sessionHelpers.getFlashMessages(req);
        
        res.render('auth/login', {
            title: 'Login',
            flash
        });
    };
    
    // Render register page
    static renderRegister = (req, res) => {
        if (sessionHelpers.isAuthenticated(req)) {
            return res.redirect('/dashboard');
        }
        
        const flash = sessionHelpers.getFlashMessages(req);
        
        res.render('auth/register', {
            title: 'Register',
            flash
        });
    };
}

module.exports = AuthController;
```

---

## 🛒 Shopping Cart with Sessions

### Session-Based Shopping Cart
```javascript
// services/CartService.js
const Product = require('../models/Product');

class CartService {
    static initializeCart(req) {
        if (!req.session.cart) {
            req.session.cart = {
                items: [],
                total: 0,
                itemCount: 0
            };
        }
        return req.session.cart;
    }
    
    static async addToCart(req, productId, quantity = 1) {
        try {
            const cart = this.initializeCart(req);
            
            // Get product details
            const product = await Product.findById(productId);
            if (!product) {
                throw new Error('Product not found');
            }
            
            // Check stock availability
            if (product.stock < quantity) {
                throw new Error('Insufficient stock');
            }
            
            // Check if item already exists in cart
            const existingItemIndex = cart.items.findIndex(
                item => item.productId.toString() === productId
            );
            
            if (existingItemIndex > -1) {
                // Update quantity
                const newQuantity = cart.items[existingItemIndex].quantity + quantity;
                
                if (newQuantity > product.stock) {
                    throw new Error('Insufficient stock');
                }
                
                cart.items[existingItemIndex].quantity = newQuantity;
                cart.items[existingItemIndex].subtotal = newQuantity * product.price;
            } else {
                // Add new item
                cart.items.push({
                    productId: product._id,
                    name: product.name,
                    price: product.price,
                    quantity,
                    subtotal: quantity * product.price,
                    image: product.images?.[0]?.url || null
                });
            }
            
            // Recalculate totals
            this.recalculateCart(cart);
            
            return {
                message: 'Item added to cart successfully',
                cart
            };
            
        } catch (error) {
            throw error;
        }
    }
    
    static updateCartItem(req, productId, quantity) {
        const cart = this.initializeCart(req);
        
        const itemIndex = cart.items.findIndex(
            item => item.productId.toString() === productId
        );
        
        if (itemIndex === -1) {
            throw new Error('Item not found in cart');
        }
        
        if (quantity <= 0) {
            // Remove item
            cart.items.splice(itemIndex, 1);
        } else {
            // Update quantity
            cart.items[itemIndex].quantity = quantity;
            cart.items[itemIndex].subtotal = quantity * cart.items[itemIndex].price;
        }
        
        this.recalculateCart(cart);
        
        return {
            message: 'Cart updated successfully',
            cart
        };
    }
    
    static removeFromCart(req, productId) {
        const cart = this.initializeCart(req);
        
        cart.items = cart.items.filter(
            item => item.productId.toString() !== productId
        );
        
        this.recalculateCart(cart);
        
        return {
            message: 'Item removed from cart',
            cart
        };
    }
    
    static clearCart(req) {
        req.session.cart = {
            items: [],
            total: 0,
            itemCount: 0
        };
        
        return {
            message: 'Cart cleared successfully',
            cart: req.session.cart
        };
    }
    
    static getCart(req) {
        return this.initializeCart(req);
    }
    
    static recalculateCart(cart) {
        cart.total = cart.items.reduce((sum, item) => sum + item.subtotal, 0);
        cart.itemCount = cart.items.reduce((sum, item) => sum + item.quantity, 0);
    }
    
    static async validateCart(req) {
        const cart = this.initializeCart(req);
        const invalidItems = [];
        
        for (const item of cart.items) {
            const product = await Product.findById(item.productId);
            
            if (!product) {
                invalidItems.push({
                    ...item,
                    error: 'Product no longer available'
                });
            } else if (product.stock < item.quantity) {
                invalidItems.push({
                    ...item,
                    error: `Only ${product.stock} items available`,
                    availableStock: product.stock
                });
            } else if (product.price !== item.price) {
                // Update price
                item.price = product.price;
                item.subtotal = item.quantity * product.price;
            }
        }
        
        // Remove invalid items
        cart.items = cart.items.filter(item => 
            !invalidItems.some(invalid => 
                invalid.productId.toString() === item.productId.toString()
            )
        );
        
        this.recalculateCart(cart);
        
        return {
            cart,
            invalidItems,
            isValid: invalidItems.length === 0
        };
    }
}

module.exports = CartService;
```

### Cart Controller
```javascript
// controllers/CartController.js
const CartService = require('../services/CartService');
const { asyncErrorHandler } = require('../middleware/errorHandler');

class CartController {
    // Add item to cart
    static addToCart = asyncErrorHandler(async (req, res) => {
        const { productId, quantity = 1 } = req.body;
        
        const result = await CartService.addToCart(req, productId, parseInt(quantity));
        
        res.json({
            success: true,
            message: result.message,
            cart: result.cart
        });
    });
    
    // Update cart item
    static updateCartItem = asyncErrorHandler(async (req, res) => {
        const { productId } = req.params;
        const { quantity } = req.body;
        
        const result = CartService.updateCartItem(req, productId, parseInt(quantity));
        
        res.json({
            success: true,
            message: result.message,
            cart: result.cart
        });
    });
    
    // Remove item from cart
    static removeFromCart = asyncErrorHandler(async (req, res) => {
        const { productId } = req.params;
        
        const result = CartService.removeFromCart(req, productId);
        
        res.json({
            success: true,
            message: result.message,
            cart: result.cart
        });
    });
    
    // Get cart
    static getCart = asyncErrorHandler(async (req, res) => {
        const cart = CartService.getCart(req);
        
        res.json({
            success: true,
            cart
        });
    });
    
    // Clear cart
    static clearCart = asyncErrorHandler(async (req, res) => {
        const result = CartService.clearCart(req);
        
        res.json({
            success: true,
            message: result.message,
            cart: result.cart
        });
    });
    
    // Render cart page
    static renderCart = asyncErrorHandler(async (req, res) => {
        const validationResult = await CartService.validateCart(req);
        
        res.render('cart/index', {
            title: 'Shopping Cart',
            cart: validationResult.cart,
            invalidItems: validationResult.invalidItems,
            isValid: validationResult.isValid
        });
    });
}

module.exports = CartController;
```

---

## 🎨 Session Templates

### Login Form Template
```html
<!-- views/auth/login.ejs -->
<% layout('layouts/main') -%>

<div class="container mt-5">
    <div class="row justify-content-center">
        <div class="col-md-6 col-lg-4">
            <div class="card shadow">
                <div class="card-body p-4">
                    <h2 class="card-title text-center mb-4">Login</h2>
                    
                    <!-- Flash Messages -->
                    <% if (flash.error) { %>
                        <% flash.error.forEach(message => { %>
                            <div class="alert alert-danger alert-dismissible fade show">
                                <%= message %>
                                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                            </div>
                        <% }) %>
                    <% } %>
                    
                    <% if (flash.success) { %>
                        <% flash.success.forEach(message => { %>
                            <div class="alert alert-success alert-dismissible fade show">
                                <%= message %>
                                <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
                            </div>
                        <% }) %>
                    <% } %>
                    
                    <form id="loginForm" method="POST" action="/auth/login">
                        <!-- Email -->
                        <div class="mb-3">
                            <label for="email" class="form-label">Email Address</label>
                            <input type="email" class="form-control" id="email" name="email" required>
                            <div class="invalid-feedback"></div>
                        </div>
                        
                        <!-- Password -->
                        <div class="mb-3">
                            <label for="password" class="form-label">Password</label>
                            <div class="input-group">
                                <input type="password" class="form-control" id="password" name="password" required>
                                <button class="btn btn-outline-secondary" type="button" onclick="togglePassword()">
                                    <i class="fas fa-eye" id="toggleIcon"></i>
                                </button>
                            </div>
                            <div class="invalid-feedback"></div>
                        </div>
                        
                        <!-- Remember Me -->
                        <div class="mb-3 form-check">
                            <input type="checkbox" class="form-check-input" id="rememberMe" name="rememberMe">
                            <label class="form-check-label" for="rememberMe">
                                Remember me for 30 days
                            </label>
                        </div>
                        
                        <!-- Submit Button -->
                        <button type="submit" class="btn btn-primary w-100 mb-3">
                            <span id="submitText">Login</span>
                            <span id="submitSpinner" class="spinner-border spinner-border-sm ms-2 d-none"></span>
                        </button>
                        
                        <!-- Links -->
                        <div class="text-center">
                            <p class="mb-1">
                                <a href="/auth/forgot-password" class="text-decoration-none">
                                    Forgot your password?
                                </a>
                            </p>
                            <p class="mb-0">
                                Don't have an account? 
                                <a href="/auth/register" class="text-decoration-none">Sign up</a>
                            </p>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>

<script>
function togglePassword() {
    const passwordInput = document.getElementById('password');
    const toggleIcon = document.getElementById('toggleIcon');
    
    if (passwordInput.type === 'password') {
        passwordInput.type = 'text';
        toggleIcon.classList.remove('fa-eye');
        toggleIcon.classList.add('fa-eye-slash');
    } else {
        passwordInput.type = 'password';
        toggleIcon.classList.remove('fa-eye-slash');
        toggleIcon.classList.add('fa-eye');
    }
}

// AJAX form submission
document.getElementById('loginForm').addEventListener('submit', async (e) => {
    e.preventDefault();
    
    const submitBtn = e.target.querySelector('button[type="submit"]');
    const submitText = document.getElementById('submitText');
    const submitSpinner = document.getElementById('submitSpinner');
    
    // Show loading state
    submitBtn.disabled = true;
    submitText.textContent = 'Logging in...';
    submitSpinner.classList.remove('d-none');
    
    try {
        const formData = new FormData(e.target);
        const response = await fetch('/auth/login', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'X-Requested-With': 'XMLHttpRequest'
            },
            body: JSON.stringify(Object.fromEntries(formData))
        });
        
        const result = await response.json();
        
        if (result.success) {
            window.location.href = result.redirectUrl || '/dashboard';
        } else {
            // Show error message
            showAlert('error', result.message || 'Login failed');
        }
    } catch (error) {
        showAlert('error', 'An error occurred. Please try again.');
    } finally {
        // Reset loading state
        submitBtn.disabled = false;
        submitText.textContent = 'Login';
        submitSpinner.classList.add('d-none');
    }
});

function showAlert(type, message) {
    const alertDiv = document.createElement('div');
    alertDiv.className = `alert alert-${type === 'error' ? 'danger' : 'success'} alert-dismissible fade show`;
    alertDiv.innerHTML = `
        ${message}
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    `;
    
    const form = document.getElementById('loginForm');
    form.insertBefore(alertDiv, form.firstChild);
}
</script>
```

---

## 🔧 Application Setup

### Complete Session Setup
```javascript
// app.js
const express = require('express');
const { sessionMiddleware } = require('./middleware/sessionMiddleware');
const { getCookieConfig } = require('./utils/cookieConfig');

const app = express();

// Cookie parser with secret
app.use(require('cookie-parser')(process.env.COOKIE_SECRET));

// Session middleware
app.use(sessionMiddleware);

// Make session helpers available in templates
app.use((req, res, next) => {
    res.locals.session = req.session;
    res.locals.user = req.session ? {
        id: req.session.userId,
        email: req.session.userEmail,
        role: req.session.userRole,
        isAuthenticated: req.session.isAuthenticated || false
    } : null;
    next();
});

// Routes
app.use('/auth', require('./routes/authRoutes'));
app.use('/cart', require('./routes/cartRoutes'));

module.exports = app;
```

---

## 🔗 Related Topics
- [[12-Authentication]] - Authentication systems
- [[13-JWT-Tokens]] - Token-based authentication  
- [[21-Middlewares]] - Session middleware
- [[25-Security-Best-Practices]] - Security considerations

---

*Back to [[00-Main-Index]]*
