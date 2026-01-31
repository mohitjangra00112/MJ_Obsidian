# Error Handling

**Navigation**: [[21-Middlewares]] | [[00-Main-Index]] | Next: [[23-Cookies-Sessions]]

---

## 🚨 Error Handling in Node.js Backend

**Error handling** is critical for building robust applications. Proper error handling improves user experience, aids debugging, and prevents application crashes.

---

## 🔧 Types of Errors

### Synchronous Errors
```javascript
// Try-catch for synchronous operations
function divide(a, b) {
    try {
        if (b === 0) {
            throw new Error('Division by zero is not allowed');
        }
        return a / b;
    } catch (error) {
        console.error('Calculation error:', error.message);
        throw error; // Re-throw to let caller handle
    }
}

// Usage
try {
    const result = divide(10, 0);
    console.log(result);
} catch (error) {
    console.log('Handled error:', error.message);
}
```

### Asynchronous Errors
```javascript
// Promise-based error handling
async function fetchUserData(userId) {
    try {
        const response = await fetch(`/api/users/${userId}`);
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        const userData = await response.json();
        return userData;
    } catch (error) {
        // Handle network errors, parsing errors, etc.
        console.error('Failed to fetch user data:', error.message);
        throw error;
    }
}

// Callback-based error handling
const fs = require('fs');

function readConfigFile(callback) {
    fs.readFile('config.json', 'utf8', (err, data) => {
        if (err) {
            // Handle file reading errors
            if (err.code === 'ENOENT') {
                return callback(new Error('Configuration file not found'));
            }
            return callback(new Error(`Failed to read config: ${err.message}`));
        }
        
        try {
            const config = JSON.parse(data);
            callback(null, config);
        } catch (parseError) {
            callback(new Error('Invalid JSON in configuration file'));
        }
    });
}
```

---

## 🎯 Custom Error Classes

### Base Error Classes
```javascript
// errors/AppError.js
class AppError extends Error {
    constructor(message, statusCode = 500, errorCode = null, isOperational = true) {
        super(message);
        
        this.name = this.constructor.name;
        this.statusCode = statusCode;
        this.errorCode = errorCode;
        this.isOperational = isOperational;
        this.timestamp = new Date().toISOString();
        
        // Capture stack trace
        Error.captureStackTrace(this, this.constructor);
    }
    
    // Convert error to JSON for API responses
    toJSON() {
        return {
            name: this.name,
            message: this.message,
            statusCode: this.statusCode,
            errorCode: this.errorCode,
            timestamp: this.timestamp,
            ...(process.env.NODE_ENV === 'development' && { stack: this.stack })
        };
    }
}

// Specific error types
class ValidationError extends AppError {
    constructor(message, errors = []) {
        super(message, 400, 'VALIDATION_ERROR');
        this.errors = errors;
    }
    
    toJSON() {
        return {
            ...super.toJSON(),
            errors: this.errors
        };
    }
}

class AuthenticationError extends AppError {
    constructor(message = 'Authentication required') {
        super(message, 401, 'AUTHENTICATION_ERROR');
    }
}

class AuthorizationError extends AppError {
    constructor(message = 'Insufficient permissions') {
        super(message, 403, 'AUTHORIZATION_ERROR');
    }
}

class NotFoundError extends AppError {
    constructor(resource = 'Resource') {
        super(`${resource} not found`, 404, 'NOT_FOUND');
        this.resource = resource;
    }
}

class ConflictError extends AppError {
    constructor(message = 'Resource already exists') {
        super(message, 409, 'CONFLICT_ERROR');
    }
}

class RateLimitError extends AppError {
    constructor(message = 'Too many requests') {
        super(message, 429, 'RATE_LIMIT_ERROR');
    }
}

class DatabaseError extends AppError {
    constructor(message, originalError = null) {
        super(message, 500, 'DATABASE_ERROR');
        this.originalError = originalError;
    }
}

class ExternalServiceError extends AppError {
    constructor(service, message, statusCode = 502) {
        super(`${service} service error: ${message}`, statusCode, 'EXTERNAL_SERVICE_ERROR');
        this.service = service;
    }
}

module.exports = {
    AppError,
    ValidationError,
    AuthenticationError,
    AuthorizationError,
    NotFoundError,
    ConflictError,
    RateLimitError,
    DatabaseError,
    ExternalServiceError
};
```

### Domain-Specific Errors
```javascript
// errors/UserErrors.js
const { AppError, ValidationError, ConflictError, NotFoundError } = require('./AppError');

class UserNotFoundError extends NotFoundError {
    constructor(identifier) {
        super('User');
        this.message = `User with identifier '${identifier}' not found`;
        this.identifier = identifier;
    }
}

class EmailAlreadyExistsError extends ConflictError {
    constructor(email) {
        super(`Email '${email}' is already registered`);
        this.email = email;
    }
}

class InvalidCredentialsError extends AppError {
    constructor() {
        super('Invalid email or password', 401, 'INVALID_CREDENTIALS');
    }
}

class AccountNotActivatedError extends AppError {
    constructor() {
        super('Account not activated. Please check your email.', 403, 'ACCOUNT_NOT_ACTIVATED');
    }
}

class PasswordResetTokenInvalidError extends AppError {
    constructor() {
        super('Password reset token is invalid or expired', 400, 'INVALID_RESET_TOKEN');
    }
}

// errors/ProductErrors.js
class ProductNotFoundError extends NotFoundError {
    constructor(productId) {
        super('Product');
        this.message = `Product with ID '${productId}' not found`;
        this.productId = productId;
    }
}

class InsufficientStockError extends AppError {
    constructor(productName, available, requested) {
        super(`Insufficient stock for ${productName}. Available: ${available}, Requested: ${requested}`, 400, 'INSUFFICIENT_STOCK');
        this.productName = productName;
        this.available = available;
        this.requested = requested;
    }
}

class ProductOutOfStockError extends AppError {
    constructor(productName) {
        super(`Product '${productName}' is out of stock`, 400, 'OUT_OF_STOCK');
        this.productName = productName;
    }
}

module.exports = {
    UserNotFoundError,
    EmailAlreadyExistsError,
    InvalidCredentialsError,
    AccountNotActivatedError,
    PasswordResetTokenInvalidError,
    ProductNotFoundError,
    InsufficientStockError,
    ProductOutOfStockError
};
```

---

## 🛡️ Error Handling Middleware

### Global Error Handler
```javascript
// middleware/errorHandler.js
const { AppError } = require('../errors/AppError');

// Main error handling middleware
const errorHandler = (err, req, res, next) => {
    // Log error
    logError(err, req);
    
    // Handle different error types
    let error = err;
    
    // Convert known errors to AppError
    if (!(error instanceof AppError)) {
        error = convertToAppError(err);
    }
    
    // Send error response
    sendErrorResponse(error, req, res);
};

// Error logging function
const logError = (error, req) => {
    const errorInfo = {
        timestamp: new Date().toISOString(),
        method: req.method,
        url: req.originalUrl,
        ip: req.ip,
        userAgent: req.get('User-Agent'),
        userId: req.user?.id || 'anonymous',
        requestId: req.id,
        error: {
            name: error.name,
            message: error.message,
            stack: error.stack,
            statusCode: error.statusCode
        }
    };
    
    // Log to console (in development) or external service (in production)
    if (process.env.NODE_ENV === 'development') {
        console.error('Error occurred:', JSON.stringify(errorInfo, null, 2));
    } else {
        // In production, log to external service (e.g., Winston, Sentry)
        require('../utils/logger').error('Request error', errorInfo);
    }
    
    // Log critical errors to file
    if (error.statusCode >= 500) {
        const fs = require('fs');
        const path = require('path');
        const logFile = path.join(__dirname, '../logs/errors.log');
        
        fs.appendFile(logFile, JSON.stringify(errorInfo) + '\n', (writeErr) => {
            if (writeErr) console.error('Failed to write error log:', writeErr);
        });
    }
};

// Convert various error types to AppError
const convertToAppError = (err) => {
    // MongoDB/Mongoose errors
    if (err.name === 'ValidationError') {
        const errors = Object.values(err.errors).map(e => ({
            field: e.path,
            message: e.message,
            value: e.value
        }));
        
        return new (require('../errors/AppError').ValidationError)(
            'Validation failed',
            errors
        );
    }
    
    if (err.name === 'MongoError' || err.name === 'MongoServerError') {
        if (err.code === 11000) {
            // Duplicate key error
            const field = Object.keys(err.keyPattern)[0];
            return new (require('../errors/AppError').ConflictError)(
                `${field} already exists`
            );
        }
        
        return new (require('../errors/AppError').DatabaseError)(
            'Database operation failed',
            err
        );
    }
    
    // PostgreSQL errors
    if (err.code && err.code.startsWith('23')) {
        if (err.code === '23505') {
            // Unique violation
            return new (require('../errors/AppError').ConflictError)(
                'Resource already exists'
            );
        }
        
        if (err.code === '23503') {
            // Foreign key violation
            return new (require('../errors/AppError').ValidationError)(
                'Referenced resource does not exist'
            );
        }
        
        return new (require('../errors/AppError').DatabaseError)(
            'Database constraint violation',
            err
        );
    }
    
    // JWT errors
    if (err.name === 'JsonWebTokenError') {
        return new (require('../errors/AppError').AuthenticationError)(
            'Invalid token'
        );
    }
    
    if (err.name === 'TokenExpiredError') {
        return new (require('../errors/AppError').AuthenticationError)(
            'Token expired'
        );
    }
    
    // Multer errors
    if (err.code === 'LIMIT_FILE_SIZE') {
        return new (require('../errors/AppError').ValidationError)(
            'File size too large'
        );
    }
    
    if (err.code === 'LIMIT_FILE_COUNT') {
        return new (require('../errors/AppError').ValidationError)(
            'Too many files uploaded'
        );
    }
    
    // Rate limiting errors
    if (err.message && err.message.includes('Too many requests')) {
        return new (require('../errors/AppError').RateLimitError)();
    }
    
    // Default to internal server error
    return new AppError(
        process.env.NODE_ENV === 'production' 
            ? 'Internal server error' 
            : err.message,
        500,
        'INTERNAL_ERROR'
    );
};

// Send appropriate error response
const sendErrorResponse = (error, req, res) => {
    // Determine if request expects JSON
    const wantsJSON = req.xhr || 
                     req.headers.accept?.includes('application/json') ||
                     req.path.startsWith('/api');
    
    if (wantsJSON) {
        // Send JSON error response
        res.status(error.statusCode).json({
            success: false,
            error: error.toJSON(),
            requestId: req.id,
            timestamp: new Date().toISOString()
        });
    } else {
        // Render error page for web requests
        res.status(error.statusCode).render('errors/error', {
            title: `Error ${error.statusCode}`,
            error: {
                statusCode: error.statusCode,
                message: error.message,
                showDetails: process.env.NODE_ENV === 'development'
            }
        });
    }
};

// 404 handler
const notFoundHandler = (req, res, next) => {
    const error = new (require('../errors/AppError').NotFoundError)('Route');
    error.message = `Route ${req.method} ${req.path} not found`;
    next(error);
};

// Async error wrapper
const asyncErrorHandler = (fn) => {
    return (req, res, next) => {
        Promise.resolve(fn(req, res, next)).catch(next);
    };
};

module.exports = {
    errorHandler,
    notFoundHandler,
    asyncErrorHandler
};
```

---

## 🔄 Service Layer Error Handling

### Service with Proper Error Handling
```javascript
// services/UserService.js
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');
const User = require('../models/User');
const {
    UserNotFoundError,
    EmailAlreadyExistsError,
    InvalidCredentialsError,
    AccountNotActivatedError,
    ValidationError,
    DatabaseError
} = require('../errors/UserErrors');

class UserService {
    async createUser(userData) {
        try {
            // Validate input
            const { error, value } = this.validateUserData(userData);
            if (error) {
                throw new ValidationError('User validation failed', error.details);
            }
            
            // Check if email already exists
            const existingUser = await User.findOne({ email: value.email });
            if (existingUser) {
                throw new EmailAlreadyExistsError(value.email);
            }
            
            // Hash password
            const saltRounds = 12;
            const hashedPassword = await bcrypt.hash(value.password, saltRounds);
            
            // Create user
            const user = new User({
                ...value,
                password: hashedPassword,
                isActive: false, // Require email verification
                createdAt: new Date()
            });
            
            await user.save();
            
            // Remove password from response
            const userResponse = user.toObject();
            delete userResponse.password;
            
            return userResponse;
            
        } catch (error) {
            if (error instanceof ValidationError || 
                error instanceof EmailAlreadyExistsError) {
                throw error;
            }
            
            // Handle unexpected database errors
            if (error.name === 'MongoError' || error.code) {
                throw new DatabaseError('Failed to create user', error);
            }
            
            // Re-throw other known errors
            throw error;
        }
    }
    
    async authenticateUser(email, password) {
        try {
            // Find user by email
            const user = await User.findOne({ email }).select('+password');
            if (!user) {
                throw new InvalidCredentialsError();
            }
            
            // Check if account is activated
            if (!user.isActive) {
                throw new AccountNotActivatedError();
            }
            
            // Verify password
            const isPasswordValid = await bcrypt.compare(password, user.password);
            if (!isPasswordValid) {
                // Log failed login attempt
                await this.logFailedLogin(user.id, 'invalid_password');
                throw new InvalidCredentialsError();
            }
            
            // Generate JWT token
            const token = jwt.sign(
                { userId: user.id, email: user.email },
                process.env.JWT_SECRET,
                { expiresIn: '7d' }
            );
            
            // Update last login
            await User.findByIdAndUpdate(user.id, {
                lastLogin: new Date(),
                $inc: { loginCount: 1 }
            });
            
            // Return user data (without password) and token
            const userResponse = user.toObject();
            delete userResponse.password;
            
            return {
                user: userResponse,
                token
            };
            
        } catch (error) {
            if (error instanceof InvalidCredentialsError || 
                error instanceof AccountNotActivatedError) {
                throw error;
            }
            
            throw new DatabaseError('Authentication failed', error);
        }
    }
    
    async getUserById(userId) {
        try {
            const user = await User.findById(userId).select('-password');
            if (!user) {
                throw new UserNotFoundError(userId);
            }
            
            return user;
            
        } catch (error) {
            if (error instanceof UserNotFoundError) {
                throw error;
            }
            
            // Handle invalid ObjectId
            if (error.name === 'CastError') {
                throw new UserNotFoundError(userId);
            }
            
            throw new DatabaseError('Failed to fetch user', error);
        }
    }
    
    async updateUser(userId, updateData) {
        try {
            // Validate update data
            const { error, value } = this.validateUpdateData(updateData);
            if (error) {
                throw new ValidationError('Update validation failed', error.details);
            }
            
            // Check if user exists
            const existingUser = await User.findById(userId);
            if (!existingUser) {
                throw new UserNotFoundError(userId);
            }
            
            // Check for email conflicts (if email is being updated)
            if (value.email && value.email !== existingUser.email) {
                const emailExists = await User.findOne({ 
                    email: value.email,
                    _id: { $ne: userId }
                });
                
                if (emailExists) {
                    throw new EmailAlreadyExistsError(value.email);
                }
            }
            
            // Hash password if provided
            if (value.password) {
                const saltRounds = 12;
                value.password = await bcrypt.hash(value.password, saltRounds);
            }
            
            // Update user
            const updatedUser = await User.findByIdAndUpdate(
                userId,
                { ...value, updatedAt: new Date() },
                { new: true, runValidators: true }
            ).select('-password');
            
            return updatedUser;
            
        } catch (error) {
            if (error instanceof ValidationError || 
                error instanceof UserNotFoundError ||
                error instanceof EmailAlreadyExistsError) {
                throw error;
            }
            
            throw new DatabaseError('Failed to update user', error);
        }
    }
    
    async deleteUser(userId) {
        try {
            const user = await User.findByIdAndDelete(userId);
            if (!user) {
                throw new UserNotFoundError(userId);
            }
            
            // Perform cleanup operations
            await this.cleanupUserData(userId);
            
            return { message: 'User deleted successfully' };
            
        } catch (error) {
            if (error instanceof UserNotFoundError) {
                throw error;
            }
            
            throw new DatabaseError('Failed to delete user', error);
        }
    }
    
    // Private helper methods
    validateUserData(data) {
        const Joi = require('joi');
        
        const schema = Joi.object({
            name: Joi.string().min(2).max(50).required(),
            email: Joi.string().email().required(),
            password: Joi.string().min(8).required(),
            phone: Joi.string().pattern(/^[0-9+\-\s()]+$/).optional(),
            dateOfBirth: Joi.date().max('now').optional()
        });
        
        return schema.validate(data, { abortEarly: false });
    }
    
    validateUpdateData(data) {
        const Joi = require('joi');
        
        const schema = Joi.object({
            name: Joi.string().min(2).max(50),
            email: Joi.string().email(),
            password: Joi.string().min(8),
            phone: Joi.string().pattern(/^[0-9+\-\s()]+$/),
            dateOfBirth: Joi.date().max('now')
        }).min(1); // At least one field must be provided
        
        return schema.validate(data, { abortEarly: false });
    }
    
    async logFailedLogin(userId, reason) {
        // Log failed login attempt for security monitoring
        console.warn(`Failed login attempt for user ${userId}: ${reason}`);
        // Could also store in database or send to monitoring service
    }
    
    async cleanupUserData(userId) {
        // Cleanup related data when user is deleted
        // e.g., delete user posts, comments, sessions, etc.
        await Promise.all([
            // Post.deleteMany({ userId }),
            // Comment.deleteMany({ userId }),
            // Session.deleteMany({ userId })
        ]);
    }
}

module.exports = new UserService();
```

---

## 🎮 Controller Error Handling

### Controller with Error Handling
```javascript
// controllers/UserController.js
const UserService = require('../services/UserService');
const { asyncErrorHandler } = require('../middleware/errorHandler');

class UserController {
    // Create user
    static createUser = asyncErrorHandler(async (req, res) => {
        const userData = req.body;
        const user = await UserService.createUser(userData);
        
        res.success(user, 'User created successfully', 201);
    });
    
    // Login user
    static loginUser = asyncErrorHandler(async (req, res) => {
        const { email, password } = req.body;
        const authResult = await UserService.authenticateUser(email, password);
        
        // Set token in HTTP-only cookie
        res.cookie('token', authResult.token, {
            httpOnly: true,
            secure: process.env.NODE_ENV === 'production',
            sameSite: 'strict',
            maxAge: 7 * 24 * 60 * 60 * 1000 // 7 days
        });
        
        res.success(authResult.user, 'Login successful');
    });
    
    // Get user profile
    static getUserProfile = asyncErrorHandler(async (req, res) => {
        const userId = req.user.id;
        const user = await UserService.getUserById(userId);
        
        res.success(user, 'User profile retrieved successfully');
    });
    
    // Get user by ID (admin only)
    static getUserById = asyncErrorHandler(async (req, res) => {
        const { id } = req.params;
        const user = await UserService.getUserById(id);
        
        res.success(user, 'User retrieved successfully');
    });
    
    // Update user
    static updateUser = asyncErrorHandler(async (req, res) => {
        const { id } = req.params;
        const updateData = req.body;
        
        // Check if user can update this profile
        if (req.user.id !== id && req.user.role !== 'admin') {
            throw new (require('../errors/AppError').AuthorizationError)();
        }
        
        const updatedUser = await UserService.updateUser(id, updateData);
        
        res.success(updatedUser, 'User updated successfully');
    });
    
    // Delete user
    static deleteUser = asyncErrorHandler(async (req, res) => {
        const { id } = req.params;
        
        // Check if user can delete this profile
        if (req.user.id !== id && req.user.role !== 'admin') {
            throw new (require('../errors/AppError').AuthorizationError)();
        }
        
        const result = await UserService.deleteUser(id);
        
        res.success(result, 'User deleted successfully');
    });
    
    // List users with pagination (admin only)
    static listUsers = asyncErrorHandler(async (req, res) => {
        const {
            page = 1,
            limit = 10,
            search,
            sortBy = 'createdAt',
            sortOrder = 'desc'
        } = req.query;
        
        const result = await UserService.listUsers({
            page: parseInt(page),
            limit: parseInt(limit),
            search,
            sortBy,
            sortOrder
        });
        
        res.paginated(result.users, result.pagination, 'Users retrieved successfully');
    });
}

module.exports = UserController;
```

---

## 📋 Validation Error Handling

### Input Validation with Error Handling
```javascript
// middleware/validationErrorHandler.js
const Joi = require('joi');
const { ValidationError } = require('../errors/AppError');

// Enhanced validation middleware
const validateRequest = (schema, property = 'body') => {
    return (req, res, next) => {
        try {
            const { error, value } = schema.validate(req[property], {
                abortEarly: false,
                stripUnknown: true,
                allowUnknown: false
            });
            
            if (error) {
                const validationErrors = error.details.map(detail => ({
                    field: detail.path.join('.'),
                    message: detail.message,
                    type: detail.type,
                    value: detail.context?.value
                }));
                
                throw new ValidationError('Validation failed', validationErrors);
            }
            
            // Replace request data with validated and sanitized data
            req[property] = value;
            next();
            
        } catch (err) {
            next(err);
        }
    };
};

// Validation schemas with detailed error messages
const userSchemas = {
    register: Joi.object({
        name: Joi.string()
            .min(2)
            .max(50)
            .required()
            .messages({
                'string.empty': 'Name is required',
                'string.min': 'Name must be at least 2 characters long',
                'string.max': 'Name cannot exceed 50 characters',
                'any.required': 'Name is required'
            }),
        
        email: Joi.string()
            .email({ tlds: { allow: false } })
            .required()
            .messages({
                'string.empty': 'Email is required',
                'string.email': 'Please provide a valid email address',
                'any.required': 'Email is required'
            }),
        
        password: Joi.string()
            .min(8)
            .max(128)
            .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]/)
            .required()
            .messages({
                'string.empty': 'Password is required',
                'string.min': 'Password must be at least 8 characters long',
                'string.max': 'Password cannot exceed 128 characters',
                'string.pattern.base': 'Password must contain at least one uppercase letter, one lowercase letter, one number, and one special character',
                'any.required': 'Password is required'
            }),
        
        confirmPassword: Joi.string()
            .valid(Joi.ref('password'))
            .required()
            .messages({
                'any.only': 'Passwords do not match',
                'any.required': 'Password confirmation is required'
            }),
        
        phone: Joi.string()
            .pattern(/^[\+]?[1-9][\d]{0,15}$/)
            .optional()
            .messages({
                'string.pattern.base': 'Please provide a valid phone number'
            }),
        
        dateOfBirth: Joi.date()
            .max('now')
            .iso()
            .optional()
            .messages({
                'date.max': 'Date of birth cannot be in the future',
                'date.format': 'Please provide a valid date in ISO format'
            }),
        
        termsAccepted: Joi.boolean()
            .valid(true)
            .required()
            .messages({
                'any.only': 'You must accept the terms and conditions',
                'any.required': 'You must accept the terms and conditions'
            })
    }),
    
    login: Joi.object({
        email: Joi.string()
            .email()
            .required()
            .messages({
                'string.empty': 'Email is required',
                'string.email': 'Please provide a valid email address',
                'any.required': 'Email is required'
            }),
        
        password: Joi.string()
            .required()
            .messages({
                'string.empty': 'Password is required',
                'any.required': 'Password is required'
            }),
        
        rememberMe: Joi.boolean().default(false)
    }),
    
    updateProfile: Joi.object({
        name: Joi.string().min(2).max(50),
        phone: Joi.string().pattern(/^[\+]?[1-9][\d]{0,15}$/),
        dateOfBirth: Joi.date().max('now').iso()
    }).min(1).messages({
        'object.min': 'At least one field must be provided for update'
    })
};

// File validation
const fileValidation = {
    image: Joi.object({
        fieldname: Joi.string().required(),
        originalname: Joi.string().required(),
        mimetype: Joi.string().valid('image/jpeg', 'image/png', 'image/webp').required(),
        size: Joi.number().max(5 * 1024 * 1024).required() // 5MB max
    }).messages({
        'string.valid': 'Only JPEG, PNG, and WebP images are allowed',
        'number.max': 'Image size cannot exceed 5MB'
    })
};

module.exports = {
    validateRequest,
    userSchemas,
    fileValidation
};
```

---

## 📊 Error Monitoring and Logging

### Advanced Error Logging
```javascript
// utils/logger.js
const winston = require('winston');
const path = require('path');

// Create logs directory if it doesn't exist
const logsDir = path.join(__dirname, '../logs');
require('fs').mkdirSync(logsDir, { recursive: true });

// Custom log format
const logFormat = winston.format.combine(
    winston.format.timestamp({ format: 'YYYY-MM-DD HH:mm:ss' }),
    winston.format.errors({ stack: true }),
    winston.format.json(),
    winston.format.prettyPrint()
);

// Create logger instance
const logger = winston.createLogger({
    level: process.env.LOG_LEVEL || 'info',
    format: logFormat,
    defaultMeta: { 
        service: 'backend-api',
        environment: process.env.NODE_ENV 
    },
    transports: [
        // Error logs
        new winston.transports.File({
            filename: path.join(logsDir, 'error.log'),
            level: 'error',
            maxsize: 10 * 1024 * 1024, // 10MB
            maxFiles: 5
        }),
        
        // Combined logs
        new winston.transports.File({
            filename: path.join(logsDir, 'combined.log'),
            maxsize: 10 * 1024 * 1024,
            maxFiles: 10
        }),
        
        // Console output for development
        new winston.transports.Console({
            format: winston.format.combine(
                winston.format.colorize(),
                winston.format.simple()
            )
        })
    ]
});

// Error tracking service integration (e.g., Sentry)
const Sentry = require('@sentry/node');

if (process.env.SENTRY_DSN) {
    Sentry.init({
        dsn: process.env.SENTRY_DSN,
        environment: process.env.NODE_ENV,
        tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0
    });
}

// Enhanced error logging function
const logError = (error, context = {}) => {
    const errorLog = {
        message: error.message,
        stack: error.stack,
        name: error.name,
        statusCode: error.statusCode,
        errorCode: error.errorCode,
        isOperational: error.isOperational,
        timestamp: new Date().toISOString(),
        ...context
    };
    
    // Log to Winston
    logger.error('Application error', errorLog);
    
    // Send to Sentry for critical errors
    if (error.statusCode >= 500 && process.env.SENTRY_DSN) {
        Sentry.captureException(error, {
            tags: {
                statusCode: error.statusCode,
                errorCode: error.errorCode
            },
            extra: context
        });
    }
    
    // Send alerts for critical errors
    if (error.statusCode >= 500) {
        sendCriticalErrorAlert(errorLog);
    }
};

// Alert system for critical errors
const sendCriticalErrorAlert = async (errorLog) => {
    // Implement your alerting mechanism here
    // Examples: Slack, Discord, email, SMS, etc.
    
    if (process.env.SLACK_WEBHOOK_URL) {
        try {
            const fetch = require('node-fetch');
            
            await fetch(process.env.SLACK_WEBHOOK_URL, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    text: `🚨 Critical Error in ${process.env.NODE_ENV}`,
                    attachments: [{
                        color: 'danger',
                        fields: [
                            { title: 'Error', value: errorLog.message, short: false },
                            { title: 'Status Code', value: errorLog.statusCode, short: true },
                            { title: 'Timestamp', value: errorLog.timestamp, short: true }
                        ]
                    }]
                })
            });
        } catch (alertError) {
            logger.error('Failed to send critical error alert', { error: alertError.message });
        }
    }
};

module.exports = {
    logger,
    logError
};
```

---

## 🎨 Error Pages Templates

### Error Page Template
```html
<!-- views/errors/error.ejs -->
<% layout('layouts/main') -%>

<div class="container mt-5">
    <div class="row justify-content-center">
        <div class="col-md-8 col-lg-6">
            <div class="text-center">
                <!-- Error Icon -->
                <div class="mb-4">
                    <% if (error.statusCode === 404) { %>
                        <i class="fas fa-search fa-5x text-muted"></i>
                    <% } else if (error.statusCode === 403) { %>
                        <i class="fas fa-lock fa-5x text-warning"></i>
                    <% } else if (error.statusCode === 500) { %>
                        <i class="fas fa-exclamation-triangle fa-5x text-danger"></i>
                    <% } else { %>
                        <i class="fas fa-exclamation-circle fa-5x text-muted"></i>
                    <% } %>
                </div>
                
                <!-- Error Code -->
                <h1 class="display-1 fw-bold text-muted"><%= error.statusCode %></h1>
                
                <!-- Error Message -->
                <h2 class="mb-3">
                    <% if (error.statusCode === 404) { %>
                        Page Not Found
                    <% } else if (error.statusCode === 403) { %>
                        Access Forbidden
                    <% } else if (error.statusCode === 500) { %>
                        Internal Server Error
                    <% } else { %>
                        Oops! Something went wrong
                    <% } %>
                </h2>
                
                <p class="lead text-muted mb-4"><%= error.message %></p>
                
                <!-- Action Buttons -->
                <div class="d-flex flex-column flex-sm-row gap-3 justify-content-center">
                    <a href="/" class="btn btn-primary">
                        <i class="fas fa-home me-2"></i>Go Home
                    </a>
                    <button onclick="window.history.back()" class="btn btn-outline-secondary">
                        <i class="fas fa-arrow-left me-2"></i>Go Back
                    </button>
                    <% if (error.statusCode === 404) { %>
                        <a href="/contact" class="btn btn-outline-info">
                            <i class="fas fa-envelope me-2"></i>Report Issue
                        </a>
                    <% } %>
                </div>
                
                <!-- Development Error Details -->
                <% if (error.showDetails) { %>
                    <div class="mt-5">
                        <details class="text-start">
                            <summary class="btn btn-outline-danger">
                                <i class="fas fa-bug me-2"></i>Show Technical Details
                            </summary>
                            <div class="mt-3">
                                <pre class="bg-light p-3 rounded text-danger small"><%= error.stack || error.message %></pre>
                            </div>
                        </details>
                    </div>
                <% } %>
            </div>
        </div>
    </div>
</div>

<style>
.display-1 {
    font-size: 6rem;
    opacity: 0.3;
}

details summary {
    cursor: pointer;
    list-style: none;
}

details summary::-webkit-details-marker {
    display: none;
}

pre {
    white-space: pre-wrap;
    word-wrap: break-word;
    max-height: 300px;
    overflow-y: auto;
}
</style>
```

---

## 🔧 Application Setup with Error Handling

### Complete Error Handling Setup
```javascript
// app.js
const express = require('express');
const { errorHandler, notFoundHandler } = require('./middleware/errorHandler');

const app = express();

// ... other middleware

// Routes
app.use('/api/users', require('./routes/userRoutes'));
app.use('/api/products', require('./routes/productRoutes'));

// Health check endpoint
app.get('/health', (req, res) => {
    res.json({ 
        status: 'healthy', 
        timestamp: new Date().toISOString(),
        uptime: process.uptime()
    });
});

// 404 handler (must be after all routes)
app.use(notFoundHandler);

// Global error handler (must be last)
app.use(errorHandler);

// Graceful error handling for unhandled promises and exceptions
process.on('unhandledRejection', (reason, promise) => {
    console.error('Unhandled Rejection at:', promise, 'reason:', reason);
    // Application specific logging, throwing an error, or other logic here
    
    // Gracefully close server
    process.exit(1);
});

process.on('uncaughtException', (error) => {
    console.error('Uncaught Exception:', error);
    
    // Gracefully close server
    process.exit(1);
});

module.exports = app;
```

---

## 🔗 Related Topics
- [[21-Middlewares]] - Error handling middleware
- [[13-JWT-Tokens]] - Authentication errors
- [[16-Joi-Validation]] - Validation errors
- [[24-File-Uploads]] - File upload errors

---

*Back to [[00-Main-Index]]*
