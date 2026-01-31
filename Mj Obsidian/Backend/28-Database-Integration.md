# Database Integration

**Navigation**: [[27-Performance-Monitoring]] | [[00-Main-Index]] | Next: [[29-Testing-Strategies]]

---

## 🗄️ Database Integration for Node.js Backend

**Database integration** is fundamental to backend development. This comprehensive guide covers PostgreSQL, MongoDB, database design patterns, connection management, and optimization strategies.

---

## 🐘 PostgreSQL Integration

### PostgreSQL Connection Setup
```javascript
// config/database.js
const { Pool } = require('pg');
const fs = require('fs');

class PostgreSQLConnection {
    constructor() {
        this.pools = new Map();
        this.defaultConfig = {
            host: process.env.DB_HOST || 'localhost',
            port: process.env.DB_PORT || 5432,
            database: process.env.DB_NAME || 'myapp',
            user: process.env.DB_USER || 'postgres',
            password: process.env.DB_PASSWORD || 'password',
            
            // Connection pool settings
            max: 20, // Maximum connections in pool
            min: 2,  // Minimum connections in pool
            idleTimeoutMillis: 30000, // 30 seconds
            connectionTimeoutMillis: 2000, // 2 seconds
            
            // SSL configuration
            ssl: process.env.NODE_ENV === 'production' ? {
                rejectUnauthorized: false,
                ca: process.env.DB_SSL_CA,
                cert: process.env.DB_SSL_CERT,
                key: process.env.DB_SSL_KEY
            } : false,
            
            // Performance settings
            statement_timeout: 30000, // 30 seconds
            query_timeout: 30000,
            application_name: 'nodejs_app',
            
            // Error handling
            keepAlive: true,
            keepAliveInitialDelayMillis: 10000
        };
        
        this.createPool('default', this.defaultConfig);
    }
    
    // Create connection pool
    createPool(name, config = {}) {
        const poolConfig = { ...this.defaultConfig, ...config };
        const pool = new Pool(poolConfig);
        
        // Pool event handlers
        pool.on('connect', (client) => {
            console.log(`PostgreSQL client connected to pool: ${name}`);
            
            // Set default settings for new connections
            client.query(`
                SET timezone = 'UTC';
                SET search_path = public;
                SET statement_timeout = '30s';
            `).catch(err => {
                console.error('Error setting client defaults:', err);
            });
        });
        
        pool.on('error', (err, client) => {
            console.error(`PostgreSQL pool error in ${name}:`, err);
        });
        
        pool.on('remove', () => {
            console.log(`PostgreSQL client removed from pool: ${name}`);
        });
        
        this.pools.set(name, pool);
        return pool;
    }
    
    // Get pool by name
    getPool(name = 'default') {
        const pool = this.pools.get(name);
        if (!pool) {
            throw new Error(`Pool '${name}' not found`);
        }
        return pool;
    }
    
    // Execute query with automatic connection handling
    async query(text, params = [], poolName = 'default') {
        const pool = this.getPool(poolName);
        const client = await pool.connect();
        
        try {
            const start = Date.now();
            const result = await client.query(text, params);
            const duration = Date.now() - start;
            
            // Log slow queries
            if (duration > 1000) {
                console.warn(`Slow query detected (${duration}ms):`, {
                    query: text,
                    params,
                    duration
                });
            }
            
            return result;
        } finally {
            client.release();
        }
    }
    
    // Transaction wrapper
    async transaction(callback, poolName = 'default') {
        const pool = this.getPool(poolName);
        const client = await pool.connect();
        
        try {
            await client.query('BEGIN');
            
            const result = await callback(client);
            
            await client.query('COMMIT');
            return result;
            
        } catch (error) {
            await client.query('ROLLBACK');
            throw error;
        } finally {
            client.release();
        }
    }
    
    // Batch insert with COPY
    async batchInsert(tableName, columns, data, poolName = 'default') {
        const pool = this.getPool(poolName);
        const client = await pool.connect();
        
        try {
            const columnNames = columns.join(', ');
            const copyQuery = `COPY ${tableName} (${columnNames}) FROM STDIN WITH CSV`;
            
            const stream = client.query(copyFrom(copyQuery));
            
            for (const row of data) {
                const csvRow = columns.map(col => {
                    const value = row[col];
                    if (value === null || value === undefined) return '';
                    if (typeof value === 'string' && value.includes(',')) {
                        return `"${value.replace(/"/g, '""')}"`;
                    }
                    return value.toString();
                }).join(',');
                
                stream.write(`${csvRow}\n`);
            }
            
            await stream.end();
            return data.length;
            
        } finally {
            client.release();
        }
    }
    
    // Health check
    async healthCheck(poolName = 'default') {
        try {
            const result = await this.query('SELECT 1 as health', [], poolName);
            return {
                status: 'healthy',
                connected: true,
                result: result.rows[0]
            };
        } catch (error) {
            return {
                status: 'unhealthy',
                connected: false,
                error: error.message
            };
        }
    }
    
    // Get pool statistics
    getPoolStats(poolName = 'default') {
        const pool = this.getPool(poolName);
        
        return {
            totalCount: pool.totalCount,
            idleCount: pool.idleCount,
            waitingCount: pool.waitingCount,
            maxConnections: pool.options.max,
            minConnections: pool.options.min
        };
    }
    
    // Close all pools
    async closeAll() {
        const closePromises = [];
        
        for (const [name, pool] of this.pools) {
            console.log(`Closing PostgreSQL pool: ${name}`);
            closePromises.push(pool.end());
        }
        
        await Promise.all(closePromises);
        this.pools.clear();
    }
}

// Copy helper for batch inserts
const { Readable } = require('stream');
const copyFrom = (text) => {
    return new Readable({
        read() {}
    });
};

module.exports = new PostgreSQLConnection();
```

### PostgreSQL Models and Repositories
```javascript
// models/UserModel.js
const db = require('../config/database');
const bcrypt = require('bcrypt');

class UserModel {
    constructor() {
        this.tableName = 'users';
        this.fields = [
            'id', 'email', 'password_hash', 'first_name', 'last_name',
            'avatar_url', 'is_verified', 'is_active', 'role',
            'created_at', 'updated_at', 'last_login_at'
        ];
    }
    
    // Create user
    async create(userData) {
        const {
            email,
            password,
            firstName,
            lastName,
            role = 'user'
        } = userData;
        
        const passwordHash = await bcrypt.hash(password, 12);
        
        const query = `
            INSERT INTO ${this.tableName} (
                email, password_hash, first_name, last_name, role, created_at, updated_at
            ) VALUES (
                $1, $2, $3, $4, $5, NOW(), NOW()
            )
            RETURNING id, email, first_name, last_name, role, created_at
        `;
        
        const result = await db.query(query, [
            email.toLowerCase(),
            passwordHash,
            firstName,
            lastName,
            role
        ]);
        
        return result.rows[0];
    }
    
    // Find user by ID
    async findById(id) {
        const query = `
            SELECT ${this.fields.join(', ')}
            FROM ${this.tableName}
            WHERE id = $1 AND is_active = true
        `;
        
        const result = await db.query(query, [id]);
        return result.rows[0] || null;
    }
    
    // Find user by email
    async findByEmail(email) {
        const query = `
            SELECT ${this.fields.join(', ')}
            FROM ${this.tableName}
            WHERE email = $1 AND is_active = true
        `;
        
        const result = await db.query(query, [email.toLowerCase()]);
        return result.rows[0] || null;
    }
    
    // Update user
    async update(id, updateData) {
        const fields = [];
        const values = [];
        let paramIndex = 1;
        
        const allowedFields = {
            firstName: 'first_name',
            lastName: 'last_name',
            avatarUrl: 'avatar_url',
            isVerified: 'is_verified',
            lastLoginAt: 'last_login_at'
        };
        
        for (const [key, value] of Object.entries(updateData)) {
            if (allowedFields[key] && value !== undefined) {
                fields.push(`${allowedFields[key]} = $${paramIndex}`);
                values.push(value);
                paramIndex++;
            }
        }
        
        if (fields.length === 0) {
            throw new Error('No valid fields to update');
        }
        
        fields.push(`updated_at = NOW()`);
        values.push(id);
        
        const query = `
            UPDATE ${this.tableName}
            SET ${fields.join(', ')}
            WHERE id = $${paramIndex}
            RETURNING id, email, first_name, last_name, role, updated_at
        `;
        
        const result = await db.query(query, values);
        return result.rows[0] || null;
    }
    
    // Soft delete user
    async delete(id) {
        const query = `
            UPDATE ${this.tableName}
            SET is_active = false, updated_at = NOW()
            WHERE id = $1
            RETURNING id
        `;
        
        const result = await db.query(query, [id]);
        return result.rows[0] ? true : false;
    }
    
    // Verify password
    async verifyPassword(email, password) {
        const user = await this.findByEmail(email);
        if (!user) return null;
        
        const isValid = await bcrypt.compare(password, user.password_hash);
        if (!isValid) return null;
        
        // Update last login
        await this.update(user.id, { lastLoginAt: new Date() });
        
        // Remove password hash from returned user
        delete user.password_hash;
        return user;
    }
    
    // Search users with pagination
    async search(options = {}) {
        const {
            search = '',
            role = null,
            isVerified = null,
            page = 1,
            limit = 20,
            sortBy = 'created_at',
            sortOrder = 'DESC'
        } = options;
        
        let whereConditions = ['is_active = true'];
        const values = [];
        let paramIndex = 1;
        
        // Search filter
        if (search) {
            whereConditions.push(`(
                email ILIKE $${paramIndex} OR 
                first_name ILIKE $${paramIndex} OR 
                last_name ILIKE $${paramIndex}
            )`);
            values.push(`%${search}%`);
            paramIndex++;
        }
        
        // Role filter
        if (role) {
            whereConditions.push(`role = $${paramIndex}`);
            values.push(role);
            paramIndex++;
        }
        
        // Verification filter
        if (isVerified !== null) {
            whereConditions.push(`is_verified = $${paramIndex}`);
            values.push(isVerified);
            paramIndex++;
        }
        
        // Count total
        const countQuery = `
            SELECT COUNT(*) as total
            FROM ${this.tableName}
            WHERE ${whereConditions.join(' AND ')}
        `;
        
        const countResult = await db.query(countQuery, values);
        const total = parseInt(countResult.rows[0].total);
        
        // Get data with pagination
        const offset = (page - 1) * limit;
        const selectFields = this.fields.filter(f => f !== 'password_hash');
        
        const dataQuery = `
            SELECT ${selectFields.join(', ')}
            FROM ${this.tableName}
            WHERE ${whereConditions.join(' AND ')}
            ORDER BY ${sortBy} ${sortOrder}
            LIMIT $${paramIndex} OFFSET $${paramIndex + 1}
        `;
        
        values.push(limit, offset);
        const dataResult = await db.query(dataQuery, values);
        
        return {
            data: dataResult.rows,
            pagination: {
                page,
                limit,
                total,
                totalPages: Math.ceil(total / limit),
                hasNext: page < Math.ceil(total / limit),
                hasPrev: page > 1
            }
        };
    }
    
    // Get user statistics
    async getStats() {
        const query = `
            SELECT 
                COUNT(*) as total_users,
                COUNT(*) FILTER (WHERE is_verified = true) as verified_users,
                COUNT(*) FILTER (WHERE is_active = true) as active_users,
                COUNT(*) FILTER (WHERE created_at >= NOW() - INTERVAL '30 days') as new_users_last_30_days,
                COUNT(*) FILTER (WHERE last_login_at >= NOW() - INTERVAL '7 days') as active_last_7_days
            FROM ${this.tableName}
        `;
        
        const result = await db.query(query);
        return result.rows[0];
    }
}

module.exports = new UserModel();
```

---

## 🍃 MongoDB Integration

### MongoDB Connection Setup
```javascript
// config/mongodb.js
const { MongoClient, ServerApiVersion } = require('mongodb');
const mongoose = require('mongoose');

class MongoDBConnection {
    constructor() {
        this.client = null;
        this.isConnected = false;
        this.databases = new Map();
        
        this.config = {
            uri: process.env.MONGODB_URI || 'mongodb://localhost:27017/myapp',
            options: {
                serverApi: {
                    version: ServerApiVersion.v1,
                    strict: true,
                    deprecationErrors: true,
                },
                maxPoolSize: 10,
                minPoolSize: 2,
                maxIdleTimeMS: 30000,
                serverSelectionTimeoutMS: 5000,
                socketTimeoutMS: 45000,
                bufferMaxEntries: 0,
                retryWrites: true,
                writeConcern: {
                    w: 'majority'
                },
                readPreference: 'primaryPreferred'
            }
        };
    }
    
    // Connect to MongoDB
    async connect() {
        try {
            this.client = new MongoClient(this.config.uri, this.config.options);
            await this.client.connect();
            
            // Test connection
            await this.client.db('admin').command({ ping: 1 });
            
            this.isConnected = true;
            console.log('MongoDB connected successfully');
            
            // Setup event listeners
            this.setupEventListeners();
            
            return this.client;
        } catch (error) {
            console.error('MongoDB connection error:', error);
            throw error;
        }
    }
    
    // Connect with Mongoose
    async connectMongoose() {
        try {
            mongoose.set('strictQuery', false);
            
            await mongoose.connect(this.config.uri, {
                ...this.config.options,
                useNewUrlParser: true,
                useUnifiedTopology: true
            });
            
            console.log('Mongoose connected successfully');
            
            // Mongoose event listeners
            mongoose.connection.on('connected', () => {
                console.log('Mongoose connected to MongoDB');
            });
            
            mongoose.connection.on('error', (err) => {
                console.error('Mongoose connection error:', err);
            });
            
            mongoose.connection.on('disconnected', () => {
                console.log('Mongoose disconnected from MongoDB');
            });
            
            // Graceful shutdown
            process.on('SIGINT', async () => {
                await mongoose.connection.close();
                console.log('Mongoose connection closed through app termination');
                process.exit(0);
            });
            
            return mongoose.connection;
        } catch (error) {
            console.error('Mongoose connection error:', error);
            throw error;
        }
    }
    
    // Setup event listeners
    setupEventListeners() {
        this.client.on('serverOpening', () => {
            console.log('MongoDB server connection opening');
        });
        
        this.client.on('serverClosed', () => {
            console.log('MongoDB server connection closed');
        });
        
        this.client.on('error', (error) => {
            console.error('MongoDB client error:', error);
        });
        
        this.client.on('timeout', () => {
            console.warn('MongoDB operation timeout');
        });
    }
    
    // Get database
    getDatabase(dbName = null) {
        if (!this.isConnected) {
            throw new Error('MongoDB not connected');
        }
        
        const databaseName = dbName || this.extractDbNameFromUri();
        
        if (!this.databases.has(databaseName)) {
            const db = this.client.db(databaseName);
            this.databases.set(databaseName, db);
        }
        
        return this.databases.get(databaseName);
    }
    
    // Extract database name from URI
    extractDbNameFromUri() {
        const match = this.config.uri.match(/\/([^/?]+)(\?|$)/);
        return match ? match[1] : 'test';
    }
    
    // Health check
    async healthCheck() {
        try {
            if (!this.isConnected) {
                return { status: 'disconnected', connected: false };
            }
            
            const admin = this.client.db('admin');
            const result = await admin.command({ ping: 1 });
            
            return {
                status: 'healthy',
                connected: true,
                ping: result.ok === 1
            };
        } catch (error) {
            return {
                status: 'unhealthy',
                connected: false,
                error: error.message
            };
        }
    }
    
    // Get connection statistics
    async getStats() {
        try {
            const admin = this.client.db('admin');
            const serverStatus = await admin.command({ serverStatus: 1 });
            
            return {
                connections: serverStatus.connections,
                network: serverStatus.network,
                memory: serverStatus.mem,
                uptime: serverStatus.uptime,
                version: serverStatus.version
            };
        } catch (error) {
            console.error('Error getting MongoDB stats:', error);
            return null;
        }
    }
    
    // Close connection
    async close() {
        try {
            if (this.client) {
                await this.client.close();
                this.isConnected = false;
                this.databases.clear();
                console.log('MongoDB connection closed');
            }
            
            if (mongoose.connection.readyState === 1) {
                await mongoose.connection.close();
                console.log('Mongoose connection closed');
            }
        } catch (error) {
            console.error('Error closing MongoDB connection:', error);
        }
    }
}

module.exports = new MongoDBConnection();
```

### Mongoose Models and Schemas
```javascript
// models/User.js (Mongoose)
const mongoose = require('mongoose');
const bcrypt = require('bcrypt');
const validator = require('validator');

const userSchema = new mongoose.Schema({
    email: {
        type: String,
        required: [true, 'Email is required'],
        unique: true,
        lowercase: true,
        validate: [validator.isEmail, 'Please provide a valid email'],
        index: true
    },
    
    password: {
        type: String,
        required: [true, 'Password is required'],
        minlength: [8, 'Password must be at least 8 characters long'],
        select: false // Don't include password in queries by default
    },
    
    profile: {
        firstName: {
            type: String,
            required: [true, 'First name is required'],
            trim: true,
            maxlength: [50, 'First name cannot exceed 50 characters']
        },
        lastName: {
            type: String,
            required: [true, 'Last name is required'],
            trim: true,
            maxlength: [50, 'Last name cannot exceed 50 characters']
        },
        avatar: {
            url: String,
            publicId: String
        },
        bio: {
            type: String,
            maxlength: [500, 'Bio cannot exceed 500 characters']
        },
        dateOfBirth: Date,
        phoneNumber: {
            type: String,
            validate: {
                validator: function(v) {
                    return !v || validator.isMobilePhone(v);
                },
                message: 'Please provide a valid phone number'
            }
        }
    },
    
    role: {
        type: String,
        enum: ['user', 'admin', 'moderator'],
        default: 'user'
    },
    
    status: {
        type: String,
        enum: ['pending', 'active', 'suspended', 'deleted'],
        default: 'pending'
    },
    
    verification: {
        isEmailVerified: {
            type: Boolean,
            default: false
        },
        emailVerificationToken: String,
        emailVerificationExpires: Date,
        isPhoneVerified: {
            type: Boolean,
            default: false
        }
    },
    
    security: {
        passwordResetToken: String,
        passwordResetExpires: Date,
        passwordChangedAt: Date,
        loginAttempts: {
            type: Number,
            default: 0
        },
        lockUntil: Date,
        twoFactorAuth: {
            enabled: {
                type: Boolean,
                default: false
            },
            secret: String,
            backupCodes: [String]
        }
    },
    
    preferences: {
        language: {
            type: String,
            default: 'en'
        },
        timezone: {
            type: String,
            default: 'UTC'
        },
        notifications: {
            email: {
                type: Boolean,
                default: true
            },
            push: {
                type: Boolean,
                default: true
            },
            sms: {
                type: Boolean,
                default: false
            }
        }
    },
    
    metadata: {
        lastLoginAt: Date,
        lastLoginIP: String,
        loginHistory: [{
            timestamp: Date,
            ip: String,
            userAgent: String,
            location: {
                country: String,
                city: String
            }
        }],
        registrationIP: String,
        registrationUserAgent: String
    }
    
}, {
    timestamps: true,
    toJSON: { 
        virtuals: true,
        transform: function(doc, ret) {
            delete ret.password;
            delete ret.security.passwordResetToken;
            delete ret.security.twoFactorAuth.secret;
            delete ret.verification.emailVerificationToken;
            return ret;
        }
    },
    toObject: { virtuals: true }
});

// Indexes
userSchema.index({ email: 1 });
userSchema.index({ 'profile.firstName': 1, 'profile.lastName': 1 });
userSchema.index({ role: 1 });
userSchema.index({ status: 1 });
userSchema.index({ createdAt: -1 });
userSchema.index({ 'metadata.lastLoginAt': -1 });

// Virtual for full name
userSchema.virtual('profile.fullName').get(function() {
    return `${this.profile.firstName} ${this.profile.lastName}`;
});

// Virtual for account lock status
userSchema.virtual('security.isLocked').get(function() {
    return !!(this.security.lockUntil && this.security.lockUntil > Date.now());
});

// Pre-save middleware
userSchema.pre('save', async function(next) {
    // Only hash password if it's modified
    if (!this.isModified('password')) return next();
    
    try {
        // Hash password
        this.password = await bcrypt.hash(this.password, 12);
        
        // Set password changed timestamp
        if (!this.isNew) {
            this.security.passwordChangedAt = new Date();
        }
        
        next();
    } catch (error) {
        next(error);
    }
});

// Instance methods
userSchema.methods.comparePassword = async function(candidatePassword) {
    try {
        return await bcrypt.compare(candidatePassword, this.password);
    } catch (error) {
        throw new Error('Password comparison failed');
    }
};

userSchema.methods.createPasswordResetToken = function() {
    const crypto = require('crypto');
    const resetToken = crypto.randomBytes(32).toString('hex');
    
    this.security.passwordResetToken = crypto
        .createHash('sha256')
        .update(resetToken)
        .digest('hex');
    
    this.security.passwordResetExpires = Date.now() + 10 * 60 * 1000; // 10 minutes
    
    return resetToken;
};

userSchema.methods.createEmailVerificationToken = function() {
    const crypto = require('crypto');
    const verificationToken = crypto.randomBytes(32).toString('hex');
    
    this.verification.emailVerificationToken = crypto
        .createHash('sha256')
        .update(verificationToken)
        .digest('hex');
    
    this.verification.emailVerificationExpires = Date.now() + 24 * 60 * 60 * 1000; // 24 hours
    
    return verificationToken;
};

userSchema.methods.incrementLoginAttempts = function() {
    // If we have a previous lock that has expired, restart at 1
    if (this.security.lockUntil && this.security.lockUntil < Date.now()) {
        return this.updateOne({
            $unset: { 'security.lockUntil': 1 },
            $set: { 'security.loginAttempts': 1 }
        });
    }
    
    const updates = { $inc: { 'security.loginAttempts': 1 } };
    
    // If we have reached max attempts and it's not locked yet, lock the account
    if (this.security.loginAttempts + 1 >= 5 && !this.security.isLocked) {
        updates.$set = { 'security.lockUntil': Date.now() + 2 * 60 * 60 * 1000 }; // 2 hours
    }
    
    return this.updateOne(updates);
};

userSchema.methods.resetLoginAttempts = function() {
    return this.updateOne({
        $unset: {
            'security.loginAttempts': 1,
            'security.lockUntil': 1
        }
    });
};

userSchema.methods.updateLoginInfo = function(ip, userAgent, location = {}) {
    const loginEntry = {
        timestamp: new Date(),
        ip,
        userAgent,
        location
    };
    
    // Keep only last 10 login entries
    if (this.metadata.loginHistory.length >= 10) {
        this.metadata.loginHistory = this.metadata.loginHistory.slice(-9);
    }
    
    this.metadata.loginHistory.push(loginEntry);
    this.metadata.lastLoginAt = new Date();
    this.metadata.lastLoginIP = ip;
    
    return this.save();
};

// Static methods
userSchema.statics.findByEmail = function(email) {
    return this.findOne({ email: email.toLowerCase() });
};

userSchema.statics.findActiveUsers = function() {
    return this.find({ status: 'active' });
};

userSchema.statics.findByRole = function(role) {
    return this.find({ role });
};

userSchema.statics.getStatistics = async function() {
    const stats = await this.aggregate([
        {
            $group: {
                _id: null,
                totalUsers: { $sum: 1 },
                activeUsers: {
                    $sum: { $cond: [{ $eq: ['$status', 'active'] }, 1, 0] }
                },
                verifiedUsers: {
                    $sum: { $cond: ['$verification.isEmailVerified', 1, 0] }
                },
                newUsersLast30Days: {
                    $sum: {
                        $cond: [
                            { $gte: ['$createdAt', new Date(Date.now() - 30 * 24 * 60 * 60 * 1000)] },
                            1,
                            0
                        ]
                    }
                }
            }
        }
    ]);
    
    return stats[0] || {
        totalUsers: 0,
        activeUsers: 0,
        verifiedUsers: 0,
        newUsersLast30Days: 0
    };
};

module.exports = mongoose.model('User', userSchema);
```

---

## 🔄 Database Patterns

### Repository Pattern
```javascript
// repositories/BaseRepository.js
class BaseRepository {
    constructor(model) {
        this.model = model;
    }
    
    // Create new record
    async create(data) {
        try {
            const record = new this.model(data);
            await record.save();
            return record;
        } catch (error) {
            throw this.handleError(error);
        }
    }
    
    // Find by ID
    async findById(id, select = null) {
        try {
            let query = this.model.findById(id);
            
            if (select) {
                query = query.select(select);
            }
            
            const record = await query.exec();
            return record;
        } catch (error) {
            throw this.handleError(error);
        }
    }
    
    // Find one by criteria
    async findOne(criteria, select = null) {
        try {
            let query = this.model.findOne(criteria);
            
            if (select) {
                query = query.select(select);
            }
            
            const record = await query.exec();
            return record;
        } catch (error) {
            throw this.handleError(error);
        }
    }
    
    // Find multiple with pagination
    async find(criteria = {}, options = {}) {
        try {
            const {
                select = null,
                sort = { createdAt: -1 },
                page = 1,
                limit = 20,
                populate = null
            } = options;
            
            const skip = (page - 1) * limit;
            
            let query = this.model.find(criteria);
            
            if (select) {
                query = query.select(select);
            }
            
            if (populate) {
                if (Array.isArray(populate)) {
                    populate.forEach(pop => query = query.populate(pop));
                } else {
                    query = query.populate(populate);
                }
            }
            
            query = query.sort(sort).skip(skip).limit(limit);
            
            const [data, total] = await Promise.all([
                query.exec(),
                this.model.countDocuments(criteria)
            ]);
            
            return {
                data,
                pagination: {
                    page,
                    limit,
                    total,
                    totalPages: Math.ceil(total / limit),
                    hasNext: page < Math.ceil(total / limit),
                    hasPrev: page > 1
                }
            };
        } catch (error) {
            throw this.handleError(error);
        }
    }
    
    // Update by ID
    async updateById(id, updateData, options = {}) {
        try {
            const { returnNew = true, runValidators = true } = options;
            
            const record = await this.model.findByIdAndUpdate(
                id,
                updateData,
                {
                    new: returnNew,
                    runValidators,
                    ...options
                }
            );
            
            return record;
        } catch (error) {
            throw this.handleError(error);
        }
    }
    
    // Update one by criteria
    async updateOne(criteria, updateData, options = {}) {
        try {
            const { returnNew = true, runValidators = true } = options;
            
            const record = await this.model.findOneAndUpdate(
                criteria,
                updateData,
                {
                    new: returnNew,
                    runValidators,
                    ...options
                }
            );
            
            return record;
        } catch (error) {
            throw this.handleError(error);
        }
    }
    
    // Delete by ID
    async deleteById(id, soft = true) {
        try {
            if (soft && this.model.schema.paths.deletedAt) {
                return await this.updateById(id, { deletedAt: new Date() });
            } else {
                return await this.model.findByIdAndDelete(id);
            }
        } catch (error) {
            throw this.handleError(error);
        }
    }
    
    // Delete one by criteria
    async deleteOne(criteria, soft = true) {
        try {
            if (soft && this.model.schema.paths.deletedAt) {
                return await this.updateOne(criteria, { deletedAt: new Date() });
            } else {
                return await this.model.findOneAndDelete(criteria);
            }
        } catch (error) {
            throw this.handleError(error);
        }
    }
    
    // Bulk operations
    async bulkCreate(dataArray) {
        try {
            const records = await this.model.insertMany(dataArray);
            return records;
        } catch (error) {
            throw this.handleError(error);
        }
    }
    
    async bulkUpdate(updates) {
        try {
            const bulkOps = updates.map(update => ({
                updateOne: {
                    filter: update.filter,
                    update: update.data,
                    upsert: update.upsert || false
                }
            }));
            
            const result = await this.model.bulkWrite(bulkOps);
            return result;
        } catch (error) {
            throw this.handleError(error);
        }
    }
    
    // Aggregation
    async aggregate(pipeline) {
        try {
            const result = await this.model.aggregate(pipeline);
            return result;
        } catch (error) {
            throw this.handleError(error);
        }
    }
    
    // Count documents
    async count(criteria = {}) {
        try {
            const count = await this.model.countDocuments(criteria);
            return count;
        } catch (error) {
            throw this.handleError(error);
        }
    }
    
    // Check if exists
    async exists(criteria) {
        try {
            const exists = await this.model.exists(criteria);
            return !!exists;
        } catch (error) {
            throw this.handleError(error);
        }
    }
    
    // Handle database errors
    handleError(error) {
        if (error.name === 'ValidationError') {
            const errors = Object.values(error.errors).map(err => err.message);
            return new Error(`Validation Error: ${errors.join(', ')}`);
        }
        
        if (error.code === 11000) {
            const field = Object.keys(error.keyPattern)[0];
            return new Error(`Duplicate value for field: ${field}`);
        }
        
        if (error.name === 'CastError') {
            return new Error(`Invalid ${error.path}: ${error.value}`);
        }
        
        return error;
    }
}

module.exports = BaseRepository;
```

### User Repository Implementation
```javascript
// repositories/UserRepository.js
const BaseRepository = require('./BaseRepository');
const User = require('../models/User');

class UserRepository extends BaseRepository {
    constructor() {
        super(User);
    }
    
    // Find user by email
    async findByEmail(email) {
        return await this.findOne({ email: email.toLowerCase() });
    }
    
    // Find user with password (for authentication)
    async findByEmailWithPassword(email) {
        try {
            const user = await this.model
                .findOne({ email: email.toLowerCase() })
                .select('+password');
            return user;
        } catch (error) {
            throw this.handleError(error);
        }
    }
    
    // Search users with advanced filters
    async searchUsers(filters = {}, options = {}) {
        const {
            search,
            role,
            status,
            isVerified,
            dateFrom,
            dateTo
        } = filters;
        
        let criteria = {};
        
        // Text search
        if (search) {
            criteria.$or = [
                { email: { $regex: search, $options: 'i' } },
                { 'profile.firstName': { $regex: search, $options: 'i' } },
                { 'profile.lastName': { $regex: search, $options: 'i' } }
            ];
        }
        
        // Role filter
        if (role) {
            criteria.role = role;
        }
        
        // Status filter
        if (status) {
            criteria.status = status;
        }
        
        // Verification filter
        if (isVerified !== undefined) {
            criteria['verification.isEmailVerified'] = isVerified;
        }
        
        // Date range filter
        if (dateFrom || dateTo) {
            criteria.createdAt = {};
            if (dateFrom) criteria.createdAt.$gte = new Date(dateFrom);
            if (dateTo) criteria.createdAt.$lte = new Date(dateTo);
        }
        
        return await this.find(criteria, options);
    }
    
    // Get active users with recent login
    async getActiveUsers(days = 30) {
        const cutoffDate = new Date(Date.now() - days * 24 * 60 * 60 * 1000);
        
        return await this.find({
            status: 'active',
            'metadata.lastLoginAt': { $gte: cutoffDate }
        });
    }
    
    // Update user preferences
    async updatePreferences(userId, preferences) {
        return await this.updateById(userId, {
            $set: { preferences }
        });
    }
    
    // Add login history entry
    async addLoginHistory(userId, loginData) {
        const { ip, userAgent, location } = loginData;
        
        return await this.updateById(userId, {
            $push: {
                'metadata.loginHistory': {
                    $each: [{
                        timestamp: new Date(),
                        ip,
                        userAgent,
                        location
                    }],
                    $slice: -10 // Keep only last 10 entries
                }
            },
            $set: {
                'metadata.lastLoginAt': new Date(),
                'metadata.lastLoginIP': ip
            }
        });
    }
    
    // Get user statistics by role
    async getStatsByRole() {
        return await this.aggregate([
            {
                $group: {
                    _id: '$role',
                    count: { $sum: 1 },
                    activeCount: {
                        $sum: { $cond: [{ $eq: ['$status', 'active'] }, 1, 0] }
                    },
                    verifiedCount: {
                        $sum: { $cond: ['$verification.isEmailVerified', 1, 0] }
                    }
                }
            },
            {
                $sort: { count: -1 }
            }
        ]);
    }
    
    // Get registration trends
    async getRegistrationTrends(days = 30) {
        const startDate = new Date(Date.now() - days * 24 * 60 * 60 * 1000);
        
        return await this.aggregate([
            {
                $match: {
                    createdAt: { $gte: startDate }
                }
            },
            {
                $group: {
                    _id: {
                        year: { $year: '$createdAt' },
                        month: { $month: '$createdAt' },
                        day: { $dayOfMonth: '$createdAt' }
                    },
                    count: { $sum: 1 }
                }
            },
            {
                $sort: {
                    '_id.year': 1,
                    '_id.month': 1,
                    '_id.day': 1
                }
            },
            {
                $project: {
                    date: {
                        $dateFromParts: {
                            year: '$_id.year',
                            month: '$_id.month',
                            day: '$_id.day'
                        }
                    },
                    count: 1,
                    _id: 0
                }
            }
        ]);
    }
    
    // Clean up inactive users
    async cleanupInactiveUsers(days = 365) {
        const cutoffDate = new Date(Date.now() - days * 24 * 60 * 60 * 1000);
        
        return await this.model.updateMany(
            {
                status: 'pending',
                createdAt: { $lt: cutoffDate }
            },
            {
                $set: { status: 'deleted' }
            }
        );
    }
}

module.exports = new UserRepository();
```

---

## 🔧 Query Optimization

### Query Builder Service
```javascript
// services/QueryBuilderService.js
class QueryBuilderService {
    constructor(model) {
        this.model = model;
        this.query = model.find();
        this.aggregationPipeline = [];
        this.isAggregation = false;
    }
    
    // Static method to create new instance
    static for(model) {
        return new QueryBuilderService(model);
    }
    
    // Add where conditions
    where(field, operator = '=', value = null) {
        if (this.isAggregation) {
            throw new Error('Cannot use where() in aggregation mode');
        }
        
        if (arguments.length === 2) {
            // If only field and value provided
            value = operator;
            operator = '=';
        }
        
        let condition = {};
        
        switch (operator) {
            case '=':
            case '==':
                condition[field] = value;
                break;
            case '!=':
            case '<>':
                condition[field] = { $ne: value };
                break;
            case '>':
                condition[field] = { $gt: value };
                break;
            case '>=':
                condition[field] = { $gte: value };
                break;
            case '<':
                condition[field] = { $lt: value };
                break;
            case '<=':
                condition[field] = { $lte: value };
                break;
            case 'in':
                condition[field] = { $in: Array.isArray(value) ? value : [value] };
                break;
            case 'not in':
                condition[field] = { $nin: Array.isArray(value) ? value : [value] };
                break;
            case 'like':
                condition[field] = { $regex: value, $options: 'i' };
                break;
            case 'exists':
                condition[field] = { $exists: value };
                break;
            default:
                throw new Error(`Unsupported operator: ${operator}`);
        }
        
        this.query = this.query.find(condition);
        return this;
    }
    
    // OR conditions
    orWhere(conditions) {
        if (this.isAggregation) {
            throw new Error('Cannot use orWhere() in aggregation mode');
        }
        
        const orConditions = conditions.map(condition => {
            const [field, operator, value] = condition;
            let conditionObj = {};
            
            switch (operator) {
                case '=':
                    conditionObj[field] = value;
                    break;
                case '!=':
                    conditionObj[field] = { $ne: value };
                    break;
                case 'like':
                    conditionObj[field] = { $regex: value, $options: 'i' };
                    break;
                default:
                    conditionObj[field] = { [`$${operator}`]: value };
            }
            
            return conditionObj;
        });
        
        this.query = this.query.or(orConditions);
        return this;
    }
    
    // Select specific fields
    select(fields) {
        if (this.isAggregation) {
            const selectObj = {};
            
            if (typeof fields === 'string') {
                fields.split(' ').forEach(field => {
                    selectObj[field.trim()] = 1;
                });
            } else if (Array.isArray(fields)) {
                fields.forEach(field => {
                    selectObj[field] = 1;
                });
            } else if (typeof fields === 'object') {
                Object.assign(selectObj, fields);
            }
            
            this.aggregationPipeline.push({ $project: selectObj });
        } else {
            this.query = this.query.select(fields);
        }
        
        return this;
    }
    
    // Join/populate related data
    populate(path, select = null) {
        if (this.isAggregation) {
            // Handle population in aggregation
            if (typeof path === 'string') {
                const lookupStage = {
                    $lookup: {
                        from: this.getCollectionName(path),
                        localField: path,
                        foreignField: '_id',
                        as: path
                    }
                };
                
                this.aggregationPipeline.push(lookupStage);
                
                if (select) {
                    this.aggregationPipeline.push({
                        $project: {
                            [path]: { $map: {
                                input: `$${path}`,
                                as: 'item',
                                in: this.createSelectProjection(select, 'item')
                            }}
                        }
                    });
                }
            }
        } else {
            if (select) {
                this.query = this.query.populate(path, select);
            } else {
                this.query = this.query.populate(path);
            }
        }
        
        return this;
    }
    
    // Sort results
    sort(sortObj) {
        if (this.isAggregation) {
            this.aggregationPipeline.push({ $sort: sortObj });
        } else {
            this.query = this.query.sort(sortObj);
        }
        
        return this;
    }
    
    // Limit results
    limit(count) {
        if (this.isAggregation) {
            this.aggregationPipeline.push({ $limit: count });
        } else {
            this.query = this.query.limit(count);
        }
        
        return this;
    }
    
    // Skip results
    skip(count) {
        if (this.isAggregation) {
            this.aggregationPipeline.push({ $skip: count });
        } else {
            this.query = this.query.skip(count);
        }
        
        return this;
    }
    
    // Pagination
    paginate(page = 1, limit = 20) {
        const skip = (page - 1) * limit;
        
        this.skip(skip);
        this.limit(limit);
        
        return this;
    }
    
    // Count results
    async count() {
        if (this.isAggregation) {
            const pipeline = [...this.aggregationPipeline, { $count: 'total' }];
            const result = await this.model.aggregate(pipeline);
            return result[0]?.total || 0;
        } else {
            return await this.query.clone().countDocuments();
        }
    }
    
    // Switch to aggregation mode
    aggregate() {
        this.isAggregation = true;
        return this;
    }
    
    // Add aggregation stage
    addStage(stage) {
        if (!this.isAggregation) {
            throw new Error('Must be in aggregation mode to add stages');
        }
        
        this.aggregationPipeline.push(stage);
        return this;
    }
    
    // Group by
    groupBy(groupFields, aggregations = {}) {
        if (!this.isAggregation) {
            this.aggregate();
        }
        
        let groupObj = {};
        
        if (typeof groupFields === 'string') {
            groupObj._id = `$${groupFields}`;
        } else if (Array.isArray(groupFields)) {
            groupObj._id = {};
            groupFields.forEach(field => {
                groupObj._id[field] = `$${field}`;
            });
        } else if (typeof groupFields === 'object') {
            groupObj._id = groupFields;
        }
        
        Object.assign(groupObj, aggregations);
        
        this.aggregationPipeline.push({ $group: groupObj });
        return this;
    }
    
    // Execute query
    async exec() {
        if (this.isAggregation) {
            return await this.model.aggregate(this.aggregationPipeline);
        } else {
            return await this.query.exec();
        }
    }
    
    // Execute with pagination info
    async execWithPagination(page = 1, limit = 20) {
        const skip = (page - 1) * limit;
        
        if (this.isAggregation) {
            // Clone pipeline for count
            const countPipeline = [...this.aggregationPipeline, { $count: 'total' }];
            
            // Add pagination to main pipeline
            this.aggregationPipeline.push({ $skip: skip }, { $limit: limit });
            
            const [data, countResult] = await Promise.all([
                this.model.aggregate(this.aggregationPipeline),
                this.model.aggregate(countPipeline)
            ]);
            
            const total = countResult[0]?.total || 0;
            
            return {
                data,
                pagination: {
                    page,
                    limit,
                    total,
                    totalPages: Math.ceil(total / limit),
                    hasNext: page < Math.ceil(total / limit),
                    hasPrev: page > 1
                }
            };
        } else {
            const [data, total] = await Promise.all([
                this.query.clone().skip(skip).limit(limit).exec(),
                this.query.clone().countDocuments()
            ]);
            
            return {
                data,
                pagination: {
                    page,
                    limit,
                    total,
                    totalPages: Math.ceil(total / limit),
                    hasNext: page < Math.ceil(total / limit),
                    hasPrev: page > 1
                }
            };
        }
    }
    
    // Helper methods
    getCollectionName(field) {
        // This would need to be implemented based on your schema relationships
        // For now, return a simple pluralized version
        return `${field}s`;
    }
    
    createSelectProjection(select, prefix) {
        if (typeof select === 'string') {
            const fields = {};
            select.split(' ').forEach(field => {
                fields[field.trim()] = `${prefix}.${field.trim()}`;
            });
            return fields;
        }
        return select;
    }
}

module.exports = QueryBuilderService;
```

---

## 🔗 Related Topics
- [[20-Database-Performance]] - Database optimization
- [[26-Caching-Strategies]] - Database caching
- [[27-Performance-Monitoring]] - Database monitoring
- [[29-Testing-Strategies]] - Database testing

---

*Back to [[00-Main-Index]]*
