# Middlewares

**Navigation**: [[20-Pagination]] | [[00-Main-Index]] | Next: [[22-Error-Handling]]

---

## 🔧 Express.js Middlewares

**Middleware** functions are the heart of Express.js applications. They have access to the **request object** (`req`), **response object** (`res`), and the **next middleware function** in the application's request-response cycle.

---

## 📋 Understanding Middleware

### What is Middleware?
```javascript
// Basic middleware structure
function middleware(req, res, next) {
    // Do something with request/response
    console.log('Time:', Date.now());
    
    // Call next() to pass control to the next middleware
    next();
}

// Middleware with error handling
function middlewareWithError(req, res, next) {
    try {
        // Some operation that might fail
        if (someCondition) {
            throw new Error('Something went wrong');
        }
        next();
    } catch (error) {
        next(error); // Pass error to error handling middleware
    }
}
```

### Middleware Types
```javascript
// 1. Application-level middleware
app.use(middleware);

// 2. Router-level middleware
router.use(middleware);

// 3. Error-handling middleware
app.use((err, req, res, next) => {
    // Handle error
});

// 4. Built-in middleware
app.use(express.json());
app.use(express.static('public'));

// 5. Third-party middleware
app.use(cors());
app.use(helmet());
```

---

## 🛠️ Built-in Middlewares

### Essential Express Middlewares
```javascript
// app.js
const express = require('express');
const path = require('path');
const app = express();

// Parse JSON bodies
app.use(express.json({ limit: '10mb' }));

// Parse URL-encoded bodies
app.use(express.urlencoded({ 
    extended: true,
    limit: '10mb'
}));

// Serve static files
app.use(express.static(path.join(__dirname, 'public'), {
    maxAge: '1d', // Cache static files for 1 day
    etag: true,
    lastModified: true
}));

// Serve uploaded files
app.use('/uploads', express.static(path.join(__dirname, 'uploads'), {
    maxAge: '30d'
}));

// Raw body parser for webhooks
app.use('/webhooks', express.raw({ type: 'application/json' }));

// Text body parser
app.use('/api/text', express.text());
```

---

## 🔐 Security Middlewares

### Authentication Middleware
```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');
const User = require('../models/User');

// JWT Authentication
const authenticateToken = async (req, res, next) => {
    try {
        const authHeader = req.headers['authorization'];
        const token = authHeader && authHeader.split(' ')[1]; // Bearer TOKEN
        
        if (!token) {
            return res.status(401).json({ 
                success: false, 
                message: 'Access token required' 
            });
        }
        
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        const user = await User.findById(decoded.userId).select('-password');
        
        if (!user) {
            return res.status(401).json({ 
                success: false, 
                message: 'Invalid token' 
            });
        }
        
        req.user = user;
        next();
    } catch (error) {
        if (error.name === 'TokenExpiredError') {
            return res.status(401).json({ 
                success: false, 
                message: 'Token expired' 
            });
        }
        
        return res.status(403).json({ 
            success: false, 
            message: 'Invalid token' 
        });
    }
};

// Optional Authentication (user might or might not be logged in)
const optionalAuth = async (req, res, next) => {
    try {
        const authHeader = req.headers['authorization'];
        const token = authHeader && authHeader.split(' ')[1];
        
        if (token) {
            const decoded = jwt.verify(token, process.env.JWT_SECRET);
            const user = await User.findById(decoded.userId).select('-password');
            req.user = user;
        }
    } catch (error) {
        // Ignore authentication errors for optional auth
        req.user = null;
    }
    
    next();
};

// Session-based Authentication
const authenticateSession = (req, res, next) => {
    if (req.session && req.session.userId) {
        User.findById(req.session.userId)
            .select('-password')
            .then(user => {
                if (user) {
                    req.user = user;
                    next();
                } else {
                    req.session.destroy();
                    res.status(401).json({ 
                        success: false, 
                        message: 'Invalid session' 
                    });
                }
            })
            .catch(next);
    } else {
        res.status(401).json({ 
            success: false, 
            message: 'Authentication required' 
        });
    }
};

module.exports = {
    authenticateToken,
    optionalAuth,
    authenticateSession
};
```

### Authorization Middleware
```javascript
// middleware/authorization.js

// Role-based authorization
const requireRole = (roles) => {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ 
                success: false, 
                message: 'Authentication required' 
            });
        }
        
        const userRoles = Array.isArray(req.user.roles) ? req.user.roles : [req.user.role];
        const requiredRoles = Array.isArray(roles) ? roles : [roles];
        
        const hasRequiredRole = requiredRoles.some(role => userRoles.includes(role));
        
        if (!hasRequiredRole) {
            return res.status(403).json({ 
                success: false, 
                message: 'Insufficient permissions' 
            });
        }
        
        next();
    };
};

// Permission-based authorization
const requirePermission = (permission) => {
    return (req, res, next) => {
        if (!req.user) {
            return res.status(401).json({ 
                success: false, 
                message: 'Authentication required' 
            });
        }
        
        const userPermissions = req.user.permissions || [];
        
        if (!userPermissions.includes(permission)) {
            return res.status(403).json({ 
                success: false, 
                message: `Permission '${permission}' required` 
            });
        }
        
        next();
    };
};

// Resource ownership check
const requireOwnership = (getResourceId) => {
    return async (req, res, next) => {
        try {
            if (!req.user) {
                return res.status(401).json({ 
                    success: false, 
                    message: 'Authentication required' 
                });
            }
            
            const resourceId = typeof getResourceId === 'function' 
                ? getResourceId(req) 
                : req.params.id;
            
            // Check if user owns the resource or is admin
            if (req.user.role === 'admin' || req.user.id.toString() === resourceId) {
                return next();
            }
            
            // For other resources, implement specific ownership logic
            // Example: Check if user owns a post
            const Post = require('../models/Post');
            const post = await Post.findById(resourceId);
            
            if (!post) {
                return res.status(404).json({ 
                    success: false, 
                    message: 'Resource not found' 
                });
            }
            
            if (post.userId.toString() !== req.user.id.toString()) {
                return res.status(403).json({ 
                    success: false, 
                    message: 'Access denied' 
                });
            }
            
            req.resource = post;
            next();
            
        } catch (error) {
            next(error);
        }
    };
};

module.exports = {
    requireRole,
    requirePermission,
    requireOwnership
};
```

---

## 📊 Logging Middleware

### Custom Logging Middleware
```javascript
// middleware/logging.js
const fs = require('fs');
const path = require('path');

// Ensure logs directory exists
const logsDir = path.join(__dirname, '../logs');
if (!fs.existsSync(logsDir)) {
    fs.mkdirSync(logsDir, { recursive: true });
}

// Custom logger
const logger = (options = {}) => {
    const {
        logFile = 'access.log',
        skipSuccessfulGET = false,
        includeBody = false,
        excludeRoutes = ['/health', '/favicon.ico']
    } = options;
    
    const logPath = path.join(logsDir, logFile);
    
    return (req, res, next) => {
        const startTime = Date.now();
        const originalSend = res.send;
        
        // Skip certain routes
        if (excludeRoutes.includes(req.path)) {
            return next();
        }
        
        // Capture response data
        res.send = function(body) {
            const endTime = Date.now();
            const duration = endTime - startTime;
            
            // Skip successful GET requests if configured
            if (skipSuccessfulGET && req.method === 'GET' && res.statusCode < 400) {
                return originalSend.call(this, body);
            }
            
            const logData = {
                timestamp: new Date().toISOString(),
                method: req.method,
                url: req.originalUrl,
                ip: req.ip || req.connection.remoteAddress,
                userAgent: req.get('User-Agent'),
                statusCode: res.statusCode,
                duration: `${duration}ms`,
                userId: req.user?.id || 'anonymous',
                requestId: req.id
            };
            
            // Include request body for non-GET requests
            if (includeBody && req.method !== 'GET' && req.body) {
                logData.requestBody = JSON.stringify(req.body);
            }
            
            // Include response body for errors
            if (res.statusCode >= 400) {
                try {
                    logData.responseBody = JSON.stringify(JSON.parse(body));
                } catch (e) {
                    logData.responseBody = body?.substring(0, 500);
                }
            }
            
            // Write to log file
            const logLine = JSON.stringify(logData) + '\n';
            fs.appendFile(logPath, logLine, (err) => {
                if (err) console.error('Failed to write to log:', err);
            });
            
            // Call original send
            return originalSend.call(this, body);
        };
        
        next();
    };
};

// Request ID middleware
const requestId = (req, res, next) => {
    req.id = require('crypto').randomUUID();
    res.setHeader('X-Request-ID', req.id);
    next();
};

// Performance monitoring
const performanceMonitor = (req, res, next) => {
    const startTime = process.hrtime.bigint();
    
    res.on('finish', () => {
        const endTime = process.hrtime.bigint();
        const duration = Number(endTime - startTime) / 1000000; // Convert to milliseconds
        
        // Log slow requests (> 1000ms)
        if (duration > 1000) {
            console.warn(`Slow request detected: ${req.method} ${req.originalUrl} - ${duration.toFixed(2)}ms`);
        }
        
        // Add performance header
        res.setHeader('X-Response-Time', `${duration.toFixed(2)}ms`);
    });
    
    next();
};

module.exports = {
    logger,
    requestId,
    performanceMonitor
};
```

---

## 🔄 CORS Middleware

### Custom CORS Implementation
```javascript
// middleware/cors.js

const corsMiddleware = (options = {}) => {
    const {
        origins = ['http://localhost:3000'],
        methods = ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'],
        allowedHeaders = ['Content-Type', 'Authorization', 'X-Requested-With'],
        credentials = true,
        maxAge = 86400 // 24 hours
    } = options;
    
    return (req, res, next) => {
        const origin = req.headers.origin;
        
        // Check if origin is allowed
        if (origins.includes('*') || origins.includes(origin)) {
            res.setHeader('Access-Control-Allow-Origin', origin || '*');
        }
        
        // Set other CORS headers
        res.setHeader('Access-Control-Allow-Methods', methods.join(', '));
        res.setHeader('Access-Control-Allow-Headers', allowedHeaders.join(', '));
        res.setHeader('Access-Control-Allow-Credentials', credentials.toString());
        res.setHeader('Access-Control-Max-Age', maxAge.toString());
        
        // Handle preflight requests
        if (req.method === 'OPTIONS') {
            res.status(204).end();
            return;
        }
        
        next();
    };
};

// Environment-based CORS
const environmentCors = () => {
    const isDevelopment = process.env.NODE_ENV === 'development';
    
    return corsMiddleware({
        origins: isDevelopment 
            ? ['http://localhost:3000', 'http://localhost:3001', 'http://127.0.0.1:3000']
            : process.env.ALLOWED_ORIGINS?.split(',') || ['https://yourdomain.com'],
        credentials: true
    });
};

module.exports = {
    corsMiddleware,
    environmentCors
};
```

---

## 📝 Validation Middleware

### Request Validation
```javascript
// middleware/validation.js
const Joi = require('joi');

// Generic validation middleware
const validate = (schema, property = 'body') => {
    return (req, res, next) => {
        const dataToValidate = req[property];
        const { error, value } = schema.validate(dataToValidate, { 
            abortEarly: false,
            stripUnknown: true
        });
        
        if (error) {
            const errors = error.details.map(detail => ({
                field: detail.path.join('.'),
                message: detail.message,
                value: detail.context?.value
            }));
            
            return res.status(400).json({
                success: false,
                message: 'Validation failed',
                errors
            });
        }
        
        // Replace request data with validated and sanitized data
        req[property] = value;
        next();
    };
};

// Validation schemas
const schemas = {
    // User registration
    registerUser: Joi.object({
        name: Joi.string().min(2).max(50).required(),
        email: Joi.string().email().required(),
        password: Joi.string().min(8)
            .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/)
            .required()
            .messages({
                'string.pattern.base': 'Password must contain at least one uppercase letter, one lowercase letter, one number, and one special character'
            }),
        confirmPassword: Joi.string().valid(Joi.ref('password')).required()
            .messages({
                'any.only': 'Passwords do not match'
            }),
        termsAccepted: Joi.boolean().valid(true).required()
    }),
    
    // User login
    loginUser: Joi.object({
        email: Joi.string().email().required(),
        password: Joi.string().required(),
        rememberMe: Joi.boolean().default(false)
    }),
    
    // Product creation
    createProduct: Joi.object({
        name: Joi.string().min(2).max(100).required(),
        description: Joi.string().min(10).max(1000).required(),
        price: Joi.number().positive().precision(2).required(),
        categoryId: Joi.string().uuid().required(),
        tags: Joi.array().items(Joi.string().max(30)).max(10),
        featured: Joi.boolean().default(false),
        status: Joi.string().valid('draft', 'published').default('draft')
    }),
    
    // Query parameters
    paginationQuery: Joi.object({
        page: Joi.number().integer().min(1).default(1),
        limit: Joi.number().integer().min(1).max(100).default(10),
        sortBy: Joi.string().valid('name', 'price', 'createdAt').default('createdAt'),
        sortOrder: Joi.string().valid('asc', 'desc').default('desc'),
        search: Joi.string().max(100),
        category: Joi.string().uuid()
    })
};

// Pre-built validation middlewares
const validateRegistration = validate(schemas.registerUser);
const validateLogin = validate(schemas.loginUser);
const validateProduct = validate(schemas.createProduct);
const validatePaginationQuery = validate(schemas.paginationQuery, 'query');

// ID parameter validation
const validateObjectId = (paramName = 'id') => {
    return (req, res, next) => {
        const id = req.params[paramName];
        
        // For MongoDB ObjectId
        if (!/^[0-9a-fA-F]{24}$/.test(id)) {
            return res.status(400).json({
                success: false,
                message: 'Invalid ID format'
            });
        }
        
        next();
    };
};

// UUID validation
const validateUUID = (paramName = 'id') => {
    return (req, res, next) => {
        const id = req.params[paramName];
        const uuidRegex = /^[0-9a-f]{8}-[0-9a-f]{4}-[1-5][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i;
        
        if (!uuidRegex.test(id)) {
            return res.status(400).json({
                success: false,
                message: 'Invalid UUID format'
            });
        }
        
        next();
    };
};

module.exports = {
    validate,
    schemas,
    validateRegistration,
    validateLogin,
    validateProduct,
    validatePaginationQuery,
    validateObjectId,
    validateUUID
};
```

---

## 🚦 Rate Limiting Middleware

### Custom Rate Limiter
```javascript
// middleware/rateLimiter.js
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');
const Redis = require('ioredis');

// Redis client for distributed rate limiting
const redis = new Redis(process.env.REDIS_URL || 'redis://localhost:6379');

// Create different rate limiters for different endpoints
const createRateLimit = (options = {}) => {
    const {
        windowMs = 15 * 60 * 1000, // 15 minutes
        max = 100, // limit each IP to 100 requests per windowMs
        message = 'Too many requests from this IP, please try again later.',
        skipSuccessfulRequests = false,
        skipFailedRequests = false,
        standardHeaders = true,
        legacyHeaders = false,
        store = process.env.REDIS_URL ? new RedisStore({
            sendCommand: (...args) => redis.call(...args),
        }) : undefined
    } = options;
    
    return rateLimit({
        windowMs,
        max,
        message: {
            success: false,
            message,
            retryAfter: Math.ceil(windowMs / 1000)
        },
        skipSuccessfulRequests,
        skipFailedRequests,
        standardHeaders,
        legacyHeaders,
        store,
        keyGenerator: (req) => {
            // Use user ID if authenticated, otherwise IP
            return req.user?.id || req.ip;
        },
        onLimitReached: (req, res, options) => {
            console.warn(`Rate limit exceeded for ${req.ip}: ${req.method} ${req.originalUrl}`);
        }
    });
};

// Specific rate limiters
const generalLimiter = createRateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 100 // 100 requests per 15 minutes
});

const authLimiter = createRateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 5, // 5 login attempts per 15 minutes
    message: 'Too many authentication attempts, please try again later.',
    skipSuccessfulRequests: true
});

const apiLimiter = createRateLimit({
    windowMs: 60 * 1000, // 1 minute
    max: 60, // 60 API calls per minute
    message: 'API rate limit exceeded. Please slow down your requests.'
});

const uploadLimiter = createRateLimit({
    windowMs: 60 * 60 * 1000, // 1 hour
    max: 10, // 10 file uploads per hour
    message: 'Upload limit exceeded. Please try again later.'
});

// Progressive rate limiting
const progressiveLimiter = (req, res, next) => {
    const key = `progressive_${req.ip}`;
    
    redis.get(key).then(attempts => {
        attempts = parseInt(attempts) || 0;
        
        let maxRequests;
        let windowMs;
        
        if (attempts < 100) {
            maxRequests = 100;
            windowMs = 15 * 60 * 1000; // 15 minutes
        } else if (attempts < 200) {
            maxRequests = 50;
            windowMs = 30 * 60 * 1000; // 30 minutes
        } else {
            maxRequests = 10;
            windowMs = 60 * 60 * 1000; // 1 hour
        }
        
        const limiter = createRateLimit({ max: maxRequests, windowMs });
        limiter(req, res, next);
        
        // Increment attempts
        redis.incr(key);
        redis.expire(key, windowMs / 1000);
    }).catch(next);
};

module.exports = {
    createRateLimit,
    generalLimiter,
    authLimiter,
    apiLimiter,
    uploadLimiter,
    progressiveLimiter
};
```

---

## 📁 File Upload Middleware

### Multer Configuration
```javascript
// middleware/upload.js
const multer = require('multer');
const path = require('path');
const fs = require('fs');
const sharp = require('sharp'); // For image processing

// Ensure upload directories exist
const ensureUploadDir = (dir) => {
    if (!fs.existsSync(dir)) {
        fs.mkdirSync(dir, { recursive: true });
    }
};

// Storage configuration
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        let uploadPath = 'uploads/';
        
        // Organize by file type
        if (file.mimetype.startsWith('image/')) {
            uploadPath += 'images/';
        } else if (file.mimetype.startsWith('video/')) {
            uploadPath += 'videos/';
        } else if (file.mimetype === 'application/pdf') {
            uploadPath += 'documents/';
        } else {
            uploadPath += 'others/';
        }
        
        // Organize by date
        const date = new Date();
        uploadPath += `${date.getFullYear()}/${(date.getMonth() + 1).toString().padStart(2, '0')}/`;
        
        ensureUploadDir(uploadPath);
        cb(null, uploadPath);
    },
    filename: (req, file, cb) => {
        // Generate unique filename
        const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
        const extension = path.extname(file.originalname);
        const basename = path.basename(file.originalname, extension);
        
        // Sanitize filename
        const sanitizedBasename = basename.replace(/[^a-zA-Z0-9]/g, '_');
        const filename = `${sanitizedBasename}_${uniqueSuffix}${extension}`;
        
        cb(null, filename);
    }
});

// File filter
const fileFilter = (allowedTypes = []) => {
    return (req, file, cb) => {
        if (allowedTypes.length === 0) {
            return cb(null, true); // Allow all files
        }
        
        const isAllowed = allowedTypes.some(type => {
            if (type.startsWith('.')) {
                // Extension check
                return path.extname(file.originalname).toLowerCase() === type.toLowerCase();
            } else {
                // MIME type check
                return file.mimetype.startsWith(type);
            }
        });
        
        if (isAllowed) {
            cb(null, true);
        } else {
            cb(new Error(`File type not allowed. Allowed types: ${allowedTypes.join(', ')}`), false);
        }
    };
};

// Upload configurations
const uploadConfig = {
    // Single image upload
    singleImage: multer({
        storage,
        fileFilter: fileFilter(['image/']),
        limits: {
            fileSize: 5 * 1024 * 1024, // 5MB
            files: 1
        }
    }).single('image'),
    
    // Multiple images upload
    multipleImages: multer({
        storage,
        fileFilter: fileFilter(['image/']),
        limits: {
            fileSize: 5 * 1024 * 1024, // 5MB per file
            files: 10 // Maximum 10 files
        }
    }).array('images', 10),
    
    // Profile picture upload
    profilePicture: multer({
        storage,
        fileFilter: fileFilter(['image/jpeg', 'image/png', 'image/webp']),
        limits: {
            fileSize: 2 * 1024 * 1024 // 2MB
        }
    }).single('profilePicture'),
    
    // Document upload
    documents: multer({
        storage,
        fileFilter: fileFilter(['.pdf', '.doc', '.docx', '.txt']),
        limits: {
            fileSize: 10 * 1024 * 1024 // 10MB
        }
    }).array('documents', 5),
    
    // General file upload
    general: multer({
        storage,
        limits: {
            fileSize: 50 * 1024 * 1024 // 50MB
        }
    })
};

// Image processing middleware
const processImage = (options = {}) => {
    const {
        resize = { width: 800, height: 600 },
        quality = 80,
        format = 'jpeg',
        generateThumbnail = true,
        thumbnailSize = { width: 200, height: 200 }
    } = options;
    
    return async (req, res, next) => {
        if (!req.file || !req.file.mimetype.startsWith('image/')) {
            return next();
        }
        
        try {
            const inputPath = req.file.path;
            const outputDir = path.dirname(inputPath);
            const basename = path.basename(inputPath, path.extname(inputPath));
            
            // Process main image
            const processedPath = path.join(outputDir, `${basename}_processed.${format}`);
            await sharp(inputPath)
                .resize(resize.width, resize.height, {
                    fit: 'inside',
                    withoutEnlargement: true
                })
                .jpeg({ quality })
                .toFile(processedPath);
            
            // Generate thumbnail
            if (generateThumbnail) {
                const thumbnailPath = path.join(outputDir, `${basename}_thumb.${format}`);
                await sharp(inputPath)
                    .resize(thumbnailSize.width, thumbnailSize.height, {
                        fit: 'cover'
                    })
                    .jpeg({ quality: 70 })
                    .toFile(thumbnailPath);
                
                req.file.thumbnailPath = thumbnailPath;
            }
            
            // Replace original with processed image
            fs.unlinkSync(inputPath); // Delete original
            req.file.path = processedPath;
            req.file.filename = `${basename}_processed.${format}`;
            
            next();
        } catch (error) {
            next(error);
        }
    };
};

// File cleanup middleware (for failed requests)
const cleanupFiles = (req, res, next) => {
    const originalSend = res.send;
    
    res.send = function(data) {
        // If request failed and files were uploaded, clean them up
        if (res.statusCode >= 400) {
            if (req.file) {
                fs.unlink(req.file.path, () => {});
                if (req.file.thumbnailPath) {
                    fs.unlink(req.file.thumbnailPath, () => {});
                }
            }
            
            if (req.files) {
                req.files.forEach(file => {
                    fs.unlink(file.path, () => {});
                    if (file.thumbnailPath) {
                        fs.unlink(file.thumbnailPath, () => {});
                    }
                });
            }
        }
        
        return originalSend.call(this, data);
    };
    
    next();
};

module.exports = {
    uploadConfig,
    processImage,
    cleanupFiles,
    ensureUploadDir
};
```

---

## 📱 API Response Middleware

### Response Formatting
```javascript
// middleware/responseFormatter.js

// Standard API response format
const formatResponse = (req, res, next) => {
    // Success response helper
    res.success = (data, message = 'Success', statusCode = 200) => {
        const response = {
            success: true,
            message,
            data,
            timestamp: new Date().toISOString()
        };
        
        // Add pagination if available
        if (res.locals.pagination) {
            response.pagination = res.locals.pagination;
        }
        
        // Add request ID if available
        if (req.id) {
            response.requestId = req.id;
        }
        
        return res.status(statusCode).json(response);
    };
    
    // Error response helper
    res.error = (message = 'An error occurred', statusCode = 500, errors = null) => {
        const response = {
            success: false,
            message,
            timestamp: new Date().toISOString()
        };
        
        if (errors) {
            response.errors = errors;
        }
        
        if (req.id) {
            response.requestId = req.id;
        }
        
        return res.status(statusCode).json(response);
    };
    
    // Paginated response helper
    res.paginated = (data, pagination, message = 'Success') => {
        return res.status(200).json({
            success: true,
            message,
            data,
            pagination,
            timestamp: new Date().toISOString(),
            requestId: req.id
        });
    };
    
    next();
};

// Cache control middleware
const cacheControl = (options = {}) => {
    const {
        maxAge = 300, // 5 minutes default
        mustRevalidate = false,
        noCache = false,
        noStore = false,
        isPublic = true
    } = options;
    
    return (req, res, next) => {
        if (noCache) {
            res.setHeader('Cache-Control', 'no-cache, no-store, must-revalidate');
            res.setHeader('Pragma', 'no-cache');
            res.setHeader('Expires', '0');
        } else {
            const directives = [];
            
            if (isPublic) {
                directives.push('public');
            } else {
                directives.push('private');
            }
            
            directives.push(`max-age=${maxAge}`);
            
            if (mustRevalidate) {
                directives.push('must-revalidate');
            }
            
            if (noStore) {
                directives.push('no-store');
            }
            
            res.setHeader('Cache-Control', directives.join(', '));
        }
        
        next();
    };
};

// Compression middleware
const compression = require('compression');

const compressionMiddleware = compression({
    level: 6, // Compression level (1-9)
    threshold: 1024, // Only compress if response is larger than 1KB
    filter: (req, res) => {
        // Don't compress if the request has a 'x-no-compression' header
        if (req.headers['x-no-compression']) {
            return false;
        }
        
        // Fallback to standard filter function
        return compression.filter(req, res);
    }
});

module.exports = {
    formatResponse,
    cacheControl,
    compressionMiddleware
};
```

---

## 🔧 Application Setup with Middlewares

### Complete App Configuration
```javascript
// app.js
const express = require('express');
const helmet = require('helmet');
const compression = require('compression');

// Import custom middlewares
const { logger, requestId, performanceMonitor } = require('./middleware/logging');
const { environmentCors } = require('./middleware/cors');
const { formatResponse, cacheControl, compressionMiddleware } = require('./middleware/responseFormatter');
const { generalLimiter, authLimiter } = require('./middleware/rateLimiter');
const { authenticateToken, optionalAuth } = require('./middleware/auth');
const { validatePaginationQuery } = require('./middleware/validation');

const app = express();

// Security middleware (should be first)
app.use(helmet({
    contentSecurityPolicy: {
        directives: {
            defaultSrc: ["'self'"],
            scriptSrc: ["'self'", "'unsafe-inline'", "cdnjs.cloudflare.com"],
            styleSrc: ["'self'", "'unsafe-inline'", "fonts.googleapis.com"],
            fontSrc: ["'self'", "fonts.gstatic.com"],
            imgSrc: ["'self'", "data:", "https:"],
            connectSrc: ["'self'"]
        }
    }
}));

// CORS (before other middlewares)
app.use(environmentCors());

// Compression
app.use(compressionMiddleware);

// Request tracking
app.use(requestId);
app.use(performanceMonitor);

// Rate limiting
app.use('/api/auth', authLimiter);
app.use('/api', generalLimiter);

// Body parsing
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// Static files with caching
app.use('/static', express.static('public', {
    maxAge: '1d',
    etag: true
}));

// Response formatting
app.use(formatResponse);

// Logging (after request ID and before routes)
app.use(logger({
    logFile: 'access.log',
    skipSuccessfulGET: process.env.NODE_ENV === 'production'
}));

// Optional authentication for all routes
app.use(optionalAuth);

// Routes with specific middlewares
app.use('/api/products', 
    validatePaginationQuery,
    cacheControl({ maxAge: 300, isPublic: true }),
    require('./routes/productRoutes')
);

app.use('/api/auth', require('./routes/authRoutes'));

app.use('/api/admin', 
    authenticateToken,
    require('./middleware/authorization').requireRole('admin'),
    require('./routes/adminRoutes')
);

// Health check (no authentication required)
app.get('/health', (req, res) => {
    res.success({ status: 'healthy', timestamp: new Date().toISOString() });
});

// 404 handler
app.use('*', (req, res) => {
    res.error('Route not found', 404);
});

// Global error handler (must be last)
app.use((err, req, res, next) => {
    console.error('Error:', err);
    
    // Multer errors
    if (err instanceof multer.MulterError) {
        if (err.code === 'LIMIT_FILE_SIZE') {
            return res.error('File too large', 400);
        }
        if (err.code === 'LIMIT_FILE_COUNT') {
            return res.error('Too many files', 400);
        }
        return res.error(err.message, 400);
    }
    
    // Validation errors
    if (err.name === 'ValidationError') {
        return res.error('Validation failed', 400, err.details);
    }
    
    // JWT errors
    if (err.name === 'JsonWebTokenError') {
        return res.error('Invalid token', 401);
    }
    
    // Database errors
    if (err.code === '23505') { // PostgreSQL unique violation
        return res.error('Resource already exists', 409);
    }
    
    // Default error
    const statusCode = err.statusCode || err.status || 500;
    const message = process.env.NODE_ENV === 'production' 
        ? 'Internal server error' 
        : err.message;
    
    res.error(message, statusCode);
});

module.exports = app;
```

---

## 🔗 Related Topics
- [[20-Pagination]] - Pagination middleware
- [[22-Error-Handling]] - Error handling middleware
- [[13-JWT-Tokens]] - Authentication middleware
- [[17-Rate-Limiting]] - Rate limiting middleware

---

*Back to [[00-Main-Index]]*
