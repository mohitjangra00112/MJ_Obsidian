# Security Best Practices

**Navigation**: [[24-File-Uploads]] | [[00-Main-Index]] | Next: [[26-Caching-Strategies]]

---

## 🛡️ Security Best Practices for Node.js Backend

**Security** is paramount in backend development. This comprehensive guide covers authentication, authorization, data protection, and defense against common vulnerabilities.

---

## 🔐 Authentication Security

### Secure Password Handling
```javascript
// utils/passwordSecurity.js
const bcrypt = require('bcrypt');
const crypto = require('crypto');
const rateLimit = require('express-rate-limit');

class PasswordSecurity {
    constructor() {
        this.saltRounds = 12;
        this.minPasswordLength = 8;
        this.maxPasswordLength = 128;
    }
    
    // Password strength validation
    validatePasswordStrength(password) {
        const errors = [];
        
        if (password.length < this.minPasswordLength) {
            errors.push(`Password must be at least ${this.minPasswordLength} characters long`);
        }
        
        if (password.length > this.maxPasswordLength) {
            errors.push(`Password must not exceed ${this.maxPasswordLength} characters`);
        }
        
        if (!/[a-z]/.test(password)) {
            errors.push('Password must contain at least one lowercase letter');
        }
        
        if (!/[A-Z]/.test(password)) {
            errors.push('Password must contain at least one uppercase letter');
        }
        
        if (!/\d/.test(password)) {
            errors.push('Password must contain at least one number');
        }
        
        if (!/[!@#$%^&*()_+\-=\[\]{};':"\\|,.<>\/?]/.test(password)) {
            errors.push('Password must contain at least one special character');
        }
        
        // Check for common passwords
        if (this.isCommonPassword(password)) {
            errors.push('Password is too common. Please choose a more unique password');
        }
        
        return {
            isValid: errors.length === 0,
            errors,
            strength: this.calculatePasswordStrength(password)
        };
    }
    
    calculatePasswordStrength(password) {
        let score = 0;
        
        // Length bonus
        if (password.length >= 8) score += 1;
        if (password.length >= 12) score += 1;
        if (password.length >= 16) score += 1;
        
        // Character variety
        if (/[a-z]/.test(password)) score += 1;
        if (/[A-Z]/.test(password)) score += 1;
        if (/\d/.test(password)) score += 1;
        if (/[^a-zA-Z\d]/.test(password)) score += 2;
        
        // Patterns (negative points)
        if (/(.)\1{2,}/.test(password)) score -= 1; // Repeated characters
        if (/123|abc|qwe/i.test(password)) score -= 1; // Sequential patterns
        
        if (score <= 2) return 'weak';
        if (score <= 4) return 'medium';
        if (score <= 6) return 'strong';
        return 'very_strong';
    }
    
    isCommonPassword(password) {
        const commonPasswords = [
            'password', '123456', '123456789', 'qwerty', 'abc123',
            'password123', 'admin', 'letmein', 'welcome', 'monkey'
        ];
        
        return commonPasswords.includes(password.toLowerCase());
    }
    
    async hashPassword(password) {
        try {
            const validation = this.validatePasswordStrength(password);
            if (!validation.isValid) {
                throw new Error(validation.errors.join(', '));
            }
            
            const hash = await bcrypt.hash(password, this.saltRounds);
            return hash;
        } catch (error) {
            throw new Error(`Password hashing failed: ${error.message}`);
        }
    }
    
    async verifyPassword(password, hash) {
        try {
            return await bcrypt.compare(password, hash);
        } catch (error) {
            throw new Error(`Password verification failed: ${error.message}`);
        }
    }
    
    generateSecureToken(length = 32) {
        return crypto.randomBytes(length).toString('hex');
    }
    
    // Password reset token with expiration
    generatePasswordResetToken() {
        const token = this.generateSecureToken();
        const expires = new Date(Date.now() + 60 * 60 * 1000); // 1 hour
        
        return { token, expires };
    }
    
    // Account verification token
    generateVerificationToken() {
        const token = this.generateSecureToken();
        const expires = new Date(Date.now() + 24 * 60 * 60 * 1000); // 24 hours
        
        return { token, expires };
    }
}

module.exports = new PasswordSecurity();
```

### Multi-Factor Authentication (MFA)
```javascript
// services/MFAService.js
const speakeasy = require('speakeasy');
const QRCode = require('qrcode');
const crypto = require('crypto');

class MFAService {
    // Generate TOTP secret for user
    generateTOTPSecret(userEmail, appName = 'Your App') {
        const secret = speakeasy.generateSecret({
            name: userEmail,
            issuer: appName,
            length: 20
        });
        
        return {
            secret: secret.base32,
            otpauthUrl: secret.otpauth_url,
            qrCodeUrl: null // Will be generated separately
        };
    }
    
    // Generate QR code for TOTP setup
    async generateQRCode(otpauthUrl) {
        try {
            const qrCodeDataUrl = await QRCode.toDataURL(otpauthUrl);
            return qrCodeDataUrl;
        } catch (error) {
            throw new Error(`QR code generation failed: ${error.message}`);
        }
    }
    
    // Verify TOTP token
    verifyTOTP(token, secret) {
        return speakeasy.totp.verify({
            secret,
            encoding: 'base32',
            token,
            window: 1 // Allow 1 time step tolerance
        });
    }
    
    // Generate backup codes
    generateBackupCodes(count = 10) {
        const codes = [];
        for (let i = 0; i < count; i++) {
            const code = crypto.randomBytes(4).toString('hex').toUpperCase();
            codes.push(code);
        }
        return codes;
    }
    
    // Hash backup codes for storage
    async hashBackupCodes(codes) {
        const bcrypt = require('bcrypt');
        const hashedCodes = [];
        
        for (const code of codes) {
            const hash = await bcrypt.hash(code, 10);
            hashedCodes.push({
                hash,
                used: false,
                createdAt: new Date()
            });
        }
        
        return hashedCodes;
    }
    
    // Verify backup code
    async verifyBackupCode(inputCode, storedCodes) {
        const bcrypt = require('bcrypt');
        
        for (const storedCode of storedCodes) {
            if (!storedCode.used) {
                const isValid = await bcrypt.compare(inputCode, storedCode.hash);
                if (isValid) {
                    storedCode.used = true;
                    storedCode.usedAt = new Date();
                    return true;
                }
            }
        }
        
        return false;
    }
    
    // Generate SMS verification code
    generateSMSCode() {
        return Math.floor(100000 + Math.random() * 900000).toString(); // 6-digit code
    }
    
    // Send SMS code (integration with SMS service)
    async sendSMSCode(phoneNumber, code) {
        // Example using Twilio
        try {
            const twilio = require('twilio');
            const client = twilio(process.env.TWILIO_SID, process.env.TWILIO_AUTH_TOKEN);
            
            await client.messages.create({
                body: `Your verification code is: ${code}`,
                from: process.env.TWILIO_PHONE_NUMBER,
                to: phoneNumber
            });
            
            return { success: true };
        } catch (error) {
            throw new Error(`SMS sending failed: ${error.message}`);
        }
    }
}

module.exports = new MFAService();
```

---

## 🔒 JWT Security

### Secure JWT Implementation
```javascript
// utils/jwtSecurity.js
const jwt = require('jsonwebtoken');
const crypto = require('crypto');
const Redis = require('ioredis');

const redis = new Redis(process.env.REDIS_URL);

class JWTSecurity {
    constructor() {
        this.accessTokenSecret = process.env.JWT_ACCESS_SECRET;
        this.refreshTokenSecret = process.env.JWT_REFRESH_SECRET;
        this.accessTokenExpiry = '15m';
        this.refreshTokenExpiry = '7d';
    }
    
    // Generate token pair
    generateTokenPair(payload) {
        const jti = crypto.randomUUID(); // Unique token ID
        
        const accessToken = jwt.sign(
            { 
                ...payload, 
                type: 'access',
                jti,
                iat: Math.floor(Date.now() / 1000)
            },
            this.accessTokenSecret,
            { 
                expiresIn: this.accessTokenExpiry,
                issuer: process.env.JWT_ISSUER || 'your-app',
                audience: process.env.JWT_AUDIENCE || 'your-app-users'
            }
        );
        
        const refreshToken = jwt.sign(
            { 
                userId: payload.userId,
                type: 'refresh',
                jti,
                iat: Math.floor(Date.now() / 1000)
            },
            this.refreshTokenSecret,
            { 
                expiresIn: this.refreshTokenExpiry,
                issuer: process.env.JWT_ISSUER || 'your-app',
                audience: process.env.JWT_AUDIENCE || 'your-app-users'
            }
        );
        
        // Store refresh token in Redis with expiration
        const refreshExpiry = 7 * 24 * 60 * 60; // 7 days in seconds
        redis.setex(`refresh_token:${jti}`, refreshExpiry, refreshToken);
        
        return {
            accessToken,
            refreshToken,
            tokenType: 'Bearer',
            expiresIn: this.parseExpiry(this.accessTokenExpiry)
        };
    }
    
    // Verify access token
    verifyAccessToken(token) {
        try {
            const decoded = jwt.verify(token, this.accessTokenSecret);
            
            if (decoded.type !== 'access') {
                throw new Error('Invalid token type');
            }
            
            return decoded;
        } catch (error) {
            if (error.name === 'TokenExpiredError') {
                throw new Error('Access token expired');
            } else if (error.name === 'JsonWebTokenError') {
                throw new Error('Invalid access token');
            }
            throw error;
        }
    }
    
    // Verify refresh token
    async verifyRefreshToken(token) {
        try {
            const decoded = jwt.verify(token, this.refreshTokenSecret);
            
            if (decoded.type !== 'refresh') {
                throw new Error('Invalid token type');
            }
            
            // Check if token exists in Redis
            const storedToken = await redis.get(`refresh_token:${decoded.jti}`);
            if (!storedToken || storedToken !== token) {
                throw new Error('Refresh token revoked or invalid');
            }
            
            return decoded;
        } catch (error) {
            if (error.name === 'TokenExpiredError') {
                throw new Error('Refresh token expired');
            } else if (error.name === 'JsonWebTokenError') {
                throw new Error('Invalid refresh token');
            }
            throw error;
        }
    }
    
    // Refresh access token
    async refreshAccessToken(refreshToken) {
        try {
            const decoded = await this.verifyRefreshToken(refreshToken);
            
            // Generate new access token
            const newAccessToken = jwt.sign(
                {
                    userId: decoded.userId,
                    type: 'access',
                    jti: decoded.jti,
                    iat: Math.floor(Date.now() / 1000)
                },
                this.accessTokenSecret,
                { 
                    expiresIn: this.accessTokenExpiry,
                    issuer: process.env.JWT_ISSUER || 'your-app',
                    audience: process.env.JWT_AUDIENCE || 'your-app-users'
                }
            );
            
            return {
                accessToken: newAccessToken,
                tokenType: 'Bearer',
                expiresIn: this.parseExpiry(this.accessTokenExpiry)
            };
            
        } catch (error) {
            throw error;
        }
    }
    
    // Revoke refresh token
    async revokeRefreshToken(jti) {
        try {
            await redis.del(`refresh_token:${jti}`);
            return { success: true };
        } catch (error) {
            throw new Error(`Token revocation failed: ${error.message}`);
        }
    }
    
    // Revoke all user tokens
    async revokeAllUserTokens(userId) {
        try {
            const pattern = `refresh_token:*`;
            const keys = await redis.keys(pattern);
            
            for (const key of keys) {
                const token = await redis.get(key);
                if (token) {
                    try {
                        const decoded = jwt.verify(token, this.refreshTokenSecret);
                        if (decoded.userId === userId) {
                            await redis.del(key);
                        }
                    } catch (error) {
                        // Token might be invalid, delete anyway
                        await redis.del(key);
                    }
                }
            }
            
            return { success: true };
        } catch (error) {
            throw new Error(`Token revocation failed: ${error.message}`);
        }
    }
    
    // Parse expiry string to seconds
    parseExpiry(expiry) {
        const units = {
            's': 1,
            'm': 60,
            'h': 3600,
            'd': 86400
        };
        
        const match = expiry.match(/^(\d+)([smhd])$/);
        if (!match) return 900; // Default 15 minutes
        
        const [, amount, unit] = match;
        return parseInt(amount) * units[unit];
    }
    
    // Generate secure API key
    generateAPIKey() {
        return crypto.randomBytes(32).toString('hex');
    }
}

module.exports = new JWTSecurity();
```

---

## 🛡️ Input Validation & Sanitization

### Comprehensive Input Security
```javascript
// middleware/inputSecurity.js
const validator = require('validator');
const DOMPurify = require('isomorphic-dompurify');
const rateLimit = require('express-rate-limit');

class InputSecurity {
    // Sanitize user input
    sanitizeInput(input, options = {}) {
        const {
            allowHTML = false,
            maxLength = 1000,
            trim = true,
            removeNullBytes = true
        } = options;
        
        if (typeof input !== 'string') {
            return input;
        }
        
        let sanitized = input;
        
        // Remove null bytes
        if (removeNullBytes) {
            sanitized = sanitized.replace(/\0/g, '');
        }
        
        // Trim whitespace
        if (trim) {
            sanitized = sanitized.trim();
        }
        
        // Truncate if too long
        if (sanitized.length > maxLength) {
            sanitized = sanitized.substring(0, maxLength);
        }
        
        // HTML sanitization
        if (!allowHTML) {
            sanitized = validator.escape(sanitized);
        } else {
            sanitized = DOMPurify.sanitize(sanitized, {
                ALLOWED_TAGS: ['b', 'i', 'u', 'strong', 'em', 'p', 'br'],
                ALLOWED_ATTR: []
            });
        }
        
        return sanitized;
    }
    
    // Validate email
    validateEmail(email) {
        if (!email || typeof email !== 'string') {
            return { isValid: false, error: 'Email is required' };
        }
        
        const sanitizedEmail = this.sanitizeInput(email, { maxLength: 255 });
        
        if (!validator.isEmail(sanitizedEmail)) {
            return { isValid: false, error: 'Invalid email format' };
        }
        
        // Additional checks
        if (sanitizedEmail.length > 255) {
            return { isValid: false, error: 'Email too long' };
        }
        
        // Check for disposable email domains
        if (this.isDisposableEmail(sanitizedEmail)) {
            return { isValid: false, error: 'Disposable email addresses not allowed' };
        }
        
        return { isValid: true, email: sanitizedEmail.toLowerCase() };
    }
    
    // Validate URL
    validateURL(url, options = {}) {
        const { 
            allowedProtocols = ['http', 'https'],
            allowedDomains = null,
            maxLength = 2000
        } = options;
        
        if (!url || typeof url !== 'string') {
            return { isValid: false, error: 'URL is required' };
        }
        
        const sanitizedUrl = this.sanitizeInput(url, { maxLength });
        
        if (!validator.isURL(sanitizedUrl, {
            protocols: allowedProtocols,
            require_protocol: true
        })) {
            return { isValid: false, error: 'Invalid URL format' };
        }
        
        // Check allowed domains
        if (allowedDomains) {
            try {
                const urlObj = new URL(sanitizedUrl);
                if (!allowedDomains.includes(urlObj.hostname)) {
                    return { isValid: false, error: 'Domain not allowed' };
                }
            } catch (error) {
                return { isValid: false, error: 'Invalid URL' };
            }
        }
        
        return { isValid: true, url: sanitizedUrl };
    }
    
    // Validate phone number
    validatePhoneNumber(phone, countryCode = 'US') {
        if (!phone || typeof phone !== 'string') {
            return { isValid: false, error: 'Phone number is required' };
        }
        
        const sanitizedPhone = phone.replace(/[^\d+\-\s()]/g, '');
        
        if (!validator.isMobilePhone(sanitizedPhone, countryCode)) {
            return { isValid: false, error: 'Invalid phone number format' };
        }
        
        return { isValid: true, phone: sanitizedPhone };
    }
    
    // Validate file uploads
    validateFileUpload(file, options = {}) {
        const {
            allowedTypes = ['image/jpeg', 'image/png'],
            maxSize = 5 * 1024 * 1024, // 5MB
            allowedExtensions = ['.jpg', '.jpeg', '.png']
        } = options;
        
        const errors = [];
        
        // Check file type
        if (!allowedTypes.includes(file.mimetype)) {
            errors.push(`File type ${file.mimetype} not allowed`);
        }
        
        // Check file size
        if (file.size > maxSize) {
            errors.push(`File size ${file.size} exceeds maximum ${maxSize} bytes`);
        }
        
        // Check file extension
        const ext = require('path').extname(file.originalname).toLowerCase();
        if (!allowedExtensions.includes(ext)) {
            errors.push(`File extension ${ext} not allowed`);
        }
        
        // Check for malicious file names
        if (file.originalname.includes('..') || 
            file.originalname.includes('/') || 
            file.originalname.includes('\\')) {
            errors.push('Invalid file name');
        }
        
        return {
            isValid: errors.length === 0,
            errors
        };
    }
    
    // Check for SQL injection patterns
    detectSQLInjection(input) {
        if (typeof input !== 'string') return false;
        
        const sqlPatterns = [
            /(\b(SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|UNION)\b)/i,
            /(SCRIPT|JAVASCRIPT|VBSCRIPT)/i,
            /['";\-\-]/,
            /(\b(OR|AND)\b.*[=<>])/i
        ];
        
        return sqlPatterns.some(pattern => pattern.test(input));
    }
    
    // Check for XSS patterns
    detectXSS(input) {
        if (typeof input !== 'string') return false;
        
        const xssPatterns = [
            /<script[^>]*>.*?<\/script>/gi,
            /javascript:/gi,
            /on\w+\s*=/gi,
            /<iframe[^>]*>.*?<\/iframe>/gi,
            /<object[^>]*>.*?<\/object>/gi,
            /<embed[^>]*>/gi
        ];
        
        return xssPatterns.some(pattern => pattern.test(input));
    }
    
    // Check for disposable email domains
    isDisposableEmail(email) {
        const disposableDomains = [
            '10minutemail.com', 'tempmail.org', 'guerrillamail.com',
            'throwaway.email', 'temp-mail.org', 'mailinator.com'
        ];
        
        const domain = email.split('@')[1];
        return disposableDomains.includes(domain.toLowerCase());
    }
    
    // Rate limiting for input validation
    createValidationRateLimit() {
        return rateLimit({
            windowMs: 15 * 60 * 1000, // 15 minutes
            max: 100, // 100 validation requests per window
            message: 'Too many validation requests',
            standardHeaders: true,
            legacyHeaders: false
        });
    }
}

module.exports = new InputSecurity();
```

---

## 🔐 Data Encryption

### Encryption Service
```javascript
// services/EncryptionService.js
const crypto = require('crypto');
const bcrypt = require('bcrypt');

class EncryptionService {
    constructor() {
        this.algorithm = 'aes-256-gcm';
        this.keyLength = 32;
        this.ivLength = 16;
        this.tagLength = 16;
        this.secretKey = process.env.ENCRYPTION_KEY || this.generateKey();
    }
    
    // Generate encryption key
    generateKey() {
        return crypto.randomBytes(this.keyLength);
    }
    
    // Encrypt data
    encrypt(text) {
        try {
            const iv = crypto.randomBytes(this.ivLength);
            const cipher = crypto.createCipher(this.algorithm, this.secretKey);
            cipher.setAAD(Buffer.from('additional-data', 'utf8'));
            
            let encrypted = cipher.update(text, 'utf8', 'hex');
            encrypted += cipher.final('hex');
            
            const authTag = cipher.getAuthTag();
            
            return {
                iv: iv.toString('hex'),
                authTag: authTag.toString('hex'),
                encrypted
            };
        } catch (error) {
            throw new Error(`Encryption failed: ${error.message}`);
        }
    }
    
    // Decrypt data
    decrypt(encryptedData) {
        try {
            const { iv, authTag, encrypted } = encryptedData;
            const decipher = crypto.createDecipher(this.algorithm, this.secretKey);
            
            decipher.setAAD(Buffer.from('additional-data', 'utf8'));
            decipher.setAuthTag(Buffer.from(authTag, 'hex'));
            
            let decrypted = decipher.update(encrypted, 'hex', 'utf8');
            decrypted += decipher.final('utf8');
            
            return decrypted;
        } catch (error) {
            throw new Error(`Decryption failed: ${error.message}`);
        }
    }
    
    // Hash sensitive data (one-way)
    async hashData(data, saltRounds = 12) {
        try {
            return await bcrypt.hash(data, saltRounds);
        } catch (error) {
            throw new Error(`Hashing failed: ${error.message}`);
        }
    }
    
    // Verify hashed data
    async verifyHash(data, hash) {
        try {
            return await bcrypt.compare(data, hash);
        } catch (error) {
            throw new Error(`Hash verification failed: ${error.message}`);
        }
    }
    
    // Generate secure random token
    generateSecureToken(length = 32) {
        return crypto.randomBytes(length).toString('hex');
    }
    
    // Create HMAC signature
    createHMAC(data, secret = this.secretKey) {
        return crypto.createHmac('sha256', secret)
            .update(data)
            .digest('hex');
    }
    
    // Verify HMAC signature
    verifyHMAC(data, signature, secret = this.secretKey) {
        const expectedSignature = this.createHMAC(data, secret);
        return crypto.timingSafeEqual(
            Buffer.from(signature, 'hex'),
            Buffer.from(expectedSignature, 'hex')
        );
    }
    
    // Encrypt file
    encryptFile(inputPath, outputPath) {
        return new Promise((resolve, reject) => {
            const iv = crypto.randomBytes(this.ivLength);
            const cipher = crypto.createCipher(this.algorithm, this.secretKey);
            
            const input = require('fs').createReadStream(inputPath);
            const output = require('fs').createWriteStream(outputPath);
            
            // Write IV to the beginning of the file
            output.write(iv);
            
            input.pipe(cipher).pipe(output);
            
            output.on('finish', () => {
                resolve({ success: true, iv: iv.toString('hex') });
            });
            
            output.on('error', reject);
            input.on('error', reject);
        });
    }
    
    // Decrypt file
    decryptFile(inputPath, outputPath, iv) {
        return new Promise((resolve, reject) => {
            const decipher = crypto.createDecipher(this.algorithm, this.secretKey);
            
            const input = require('fs').createReadStream(inputPath, { start: this.ivLength });
            const output = require('fs').createWriteStream(outputPath);
            
            input.pipe(decipher).pipe(output);
            
            output.on('finish', () => {
                resolve({ success: true });
            });
            
            output.on('error', reject);
            input.on('error', reject);
        });
    }
}

module.exports = new EncryptionService();
```

---

## 🚫 CORS Security

### Secure CORS Configuration
```javascript
// middleware/corsConfig.js
const cors = require('cors');

const createCORSConfig = () => {
    const isDevelopment = process.env.NODE_ENV === 'development';
    
    // Development origins
    const developmentOrigins = [
        'http://localhost:3000',
        'http://localhost:3001',
        'http://127.0.0.1:3000',
        'http://localhost:8080'
    ];
    
    // Production origins
    const productionOrigins = process.env.ALLOWED_ORIGINS
        ? process.env.ALLOWED_ORIGINS.split(',')
        : ['https://yourdomain.com'];
    
    const allowedOrigins = isDevelopment 
        ? [...developmentOrigins, ...productionOrigins]
        : productionOrigins;
    
    return cors({
        origin: (origin, callback) => {
            // Allow requests with no origin (mobile apps, etc.)
            if (!origin) return callback(null, true);
            
            if (allowedOrigins.includes(origin)) {
                callback(null, true);
            } else {
                console.warn(`CORS blocked origin: ${origin}`);
                callback(new Error('Not allowed by CORS'));
            }
        },
        credentials: true,
        methods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS', 'PATCH'],
        allowedHeaders: [
            'Origin',
            'X-Requested-With',
            'Content-Type',
            'Accept',
            'Authorization',
            'X-API-Key',
            'X-Request-ID'
        ],
        exposedHeaders: [
            'X-Request-ID',
            'X-Response-Time',
            'X-RateLimit-Limit',
            'X-RateLimit-Remaining'
        ],
        maxAge: 86400, // 24 hours
        preflightContinue: false,
        optionsSuccessStatus: 204
    });
};

// API-specific CORS
const apiCorsConfig = cors({
    origin: process.env.API_ALLOWED_ORIGINS?.split(',') || false,
    credentials: false,
    methods: ['GET', 'POST', 'PUT', 'DELETE'],
    allowedHeaders: ['Content-Type', 'Authorization', 'X-API-Key']
});

module.exports = {
    createCORSConfig,
    apiCorsConfig
};
```

---

## 🛡️ Security Headers

### Helmet Security Configuration
```javascript
// middleware/securityHeaders.js
const helmet = require('helmet');

const securityHeaders = helmet({
    // Content Security Policy
    contentSecurityPolicy: {
        directives: {
            defaultSrc: ["'self'"],
            scriptSrc: [
                "'self'",
                "'unsafe-inline'", // Only for development
                "https://cdnjs.cloudflare.com",
                "https://cdn.jsdelivr.net"
            ],
            styleSrc: [
                "'self'",
                "'unsafe-inline'",
                "https://fonts.googleapis.com",
                "https://cdnjs.cloudflare.com"
            ],
            fontSrc: [
                "'self'",
                "https://fonts.gstatic.com"
            ],
            imgSrc: [
                "'self'",
                "data:",
                "https:",
                "blob:"
            ],
            connectSrc: [
                "'self'",
                process.env.API_URL || "https://api.yourdomain.com"
            ],
            mediaSrc: ["'self'"],
            objectSrc: ["'none'"],
            frameSrc: ["'none'"],
            baseUri: ["'self'"],
            formAction: ["'self'"],
            upgradeInsecureRequests: process.env.NODE_ENV === 'production' ? [] : null
        },
        reportOnly: process.env.NODE_ENV === 'development'
    },
    
    // HTTP Strict Transport Security
    hsts: {
        maxAge: 31536000, // 1 year
        includeSubDomains: true,
        preload: true
    },
    
    // X-Frame-Options
    frameguard: {
        action: 'deny'
    },
    
    // X-Content-Type-Options
    noSniff: true,
    
    // X-XSS-Protection
    xssFilter: true,
    
    // Referrer Policy
    referrerPolicy: {
        policy: 'strict-origin-when-cross-origin'
    },
    
    // Hide X-Powered-By header
    hidePoweredBy: true,
    
    // DNS Prefetch Control
    dnsPrefetchControl: {
        allow: false
    },
    
    // IE No Open
    ieNoOpen: true,
    
    // Permissions Policy
    permissionsPolicy: {
        camera: [],
        microphone: [],
        geolocation: [],
        notifications: [],
        push: [],
        sync: [],
        magnetometer: [],
        gyroscope: [],
        accelerometer: []
    }
});

// Additional security headers
const additionalHeaders = (req, res, next) => {
    // Remove server header
    res.removeHeader('X-Powered-By');
    res.removeHeader('Server');
    
    // Set security headers
    res.setHeader('X-Request-ID', req.id || 'unknown');
    res.setHeader('X-Response-Time', res.getHeader('X-Response-Time') || '0ms');
    
    // Prevent caching of sensitive pages
    if (req.path.includes('/admin') || req.path.includes('/api/auth')) {
        res.setHeader('Cache-Control', 'no-store, no-cache, must-revalidate, proxy-revalidate');
        res.setHeader('Pragma', 'no-cache');
        res.setHeader('Expires', '0');
    }
    
    next();
};

module.exports = {
    securityHeaders,
    additionalHeaders
};
```

---

## 🔍 Security Monitoring

### Security Monitoring Service
```javascript
// services/SecurityMonitoringService.js
const EventEmitter = require('events');

class SecurityMonitoringService extends EventEmitter {
    constructor() {
        super();
        this.suspiciousActivities = new Map();
        this.blockedIPs = new Set();
        this.alertThresholds = {
            failedLogins: 5,
            suspiciousRequests: 10,
            timeWindow: 15 * 60 * 1000 // 15 minutes
        };
    }
    
    // Log security event
    logSecurityEvent(event) {
        const timestamp = new Date().toISOString();
        const logEntry = {
            timestamp,
            ...event
        };
        
        console.log(`[SECURITY] ${timestamp}:`, logEntry);
        
        // Store in database or external logging service
        this.storeSecurityEvent(logEntry);
        
        // Check for suspicious patterns
        this.analyzeSuspiciousActivity(event);
        
        // Emit event for real-time monitoring
        this.emit('securityEvent', logEntry);
    }
    
    // Analyze suspicious activity patterns
    analyzeSuspiciousActivity(event) {
        const { ip, type, userId } = event;
        const key = `${ip}_${type}`;
        const now = Date.now();
        
        if (!this.suspiciousActivities.has(key)) {
            this.suspiciousActivities.set(key, []);
        }
        
        const activities = this.suspiciousActivities.get(key);
        
        // Clean old entries
        const cleanedActivities = activities.filter(
            activity => now - activity.timestamp < this.alertThresholds.timeWindow
        );
        
        // Add current activity
        cleanedActivities.push({ timestamp: now, userId });
        this.suspiciousActivities.set(key, cleanedActivities);
        
        // Check thresholds
        if (type === 'failed_login' && cleanedActivities.length >= this.alertThresholds.failedLogins) {
            this.handleSuspiciousActivity('multiple_failed_logins', { ip, attempts: cleanedActivities.length });
        }
        
        if (cleanedActivities.length >= this.alertThresholds.suspiciousRequests) {
            this.handleSuspiciousActivity('suspicious_requests', { ip, type, count: cleanedActivities.length });
        }
    }
    
    // Handle suspicious activity
    handleSuspiciousActivity(alertType, details) {
        const alert = {
            type: alertType,
            timestamp: new Date().toISOString(),
            ...details
        };
        
        console.warn(`[SECURITY ALERT] ${alertType}:`, alert);
        
        // Temporary IP blocking
        if (alertType === 'multiple_failed_logins') {
            this.blockIP(details.ip, 60 * 60 * 1000); // Block for 1 hour
        }
        
        // Send alerts
        this.sendSecurityAlert(alert);
        
        this.emit('securityAlert', alert);
    }
    
    // Block IP address
    blockIP(ip, duration = 60 * 60 * 1000) {
        this.blockedIPs.add(ip);
        
        setTimeout(() => {
            this.blockedIPs.delete(ip);
            console.log(`[SECURITY] IP ${ip} unblocked`);
        }, duration);
        
        console.log(`[SECURITY] IP ${ip} blocked for ${duration / 1000 / 60} minutes`);
    }
    
    // Check if IP is blocked
    isIPBlocked(ip) {
        return this.blockedIPs.has(ip);
    }
    
    // Send security alerts
    async sendSecurityAlert(alert) {
        try {
            // Send to Slack, email, or other notification services
            if (process.env.SLACK_WEBHOOK_URL) {
                await this.sendSlackAlert(alert);
            }
            
            if (process.env.SECURITY_EMAIL) {
                await this.sendEmailAlert(alert);
            }
        } catch (error) {
            console.error('Failed to send security alert:', error);
        }
    }
    
    // Send Slack alert
    async sendSlackAlert(alert) {
        const fetch = require('node-fetch');
        
        const payload = {
            text: `🚨 Security Alert: ${alert.type}`,
            attachments: [{
                color: 'danger',
                fields: Object.entries(alert).map(([key, value]) => ({
                    title: key,
                    value: typeof value === 'object' ? JSON.stringify(value) : value,
                    short: true
                }))
            }]
        };
        
        await fetch(process.env.SLACK_WEBHOOK_URL, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(payload)
        });
    }
    
    // Send email alert
    async sendEmailAlert(alert) {
        const nodemailer = require('nodemailer');
        
        const transporter = nodemailer.createTransporter({
            // Configure your email provider
        });
        
        await transporter.sendMail({
            from: process.env.ALERT_FROM_EMAIL,
            to: process.env.SECURITY_EMAIL,
            subject: `Security Alert: ${alert.type}`,
            html: `
                <h2>Security Alert</h2>
                <p><strong>Type:</strong> ${alert.type}</p>
                <p><strong>Time:</strong> ${alert.timestamp}</p>
                <pre>${JSON.stringify(alert, null, 2)}</pre>
            `
        });
    }
    
    // Store security event (implement based on your storage solution)
    async storeSecurityEvent(event) {
        // Store in MongoDB, PostgreSQL, or other database
        try {
            const SecurityEvent = require('../models/SecurityEvent');
            await SecurityEvent.create(event);
        } catch (error) {
            console.error('Failed to store security event:', error);
        }
    }
    
    // Get security statistics
    async getSecurityStats(timeRange = 24 * 60 * 60 * 1000) {
        const since = new Date(Date.now() - timeRange);
        
        try {
            const SecurityEvent = require('../models/SecurityEvent');
            
            const stats = await SecurityEvent.aggregate([
                { $match: { timestamp: { $gte: since } } },
                {
                    $group: {
                        _id: '$type',
                        count: { $sum: 1 },
                        uniqueIPs: { $addToSet: '$ip' }
                    }
                },
                {
                    $project: {
                        type: '$_id',
                        count: 1,
                        uniqueIPCount: { $size: '$uniqueIPs' }
                    }
                }
            ]);
            
            return stats;
        } catch (error) {
            console.error('Failed to get security stats:', error);
            return [];
        }
    }
}

module.exports = new SecurityMonitoringService();
```

---

## 🔧 Security Middleware Integration

### Complete Security Setup
```javascript
// app.js - Security configuration
const express = require('express');
const { securityHeaders, additionalHeaders } = require('./middleware/securityHeaders');
const { createCORSConfig } = require('./middleware/corsConfig');
const SecurityMonitoringService = require('./services/SecurityMonitoringService');
const InputSecurity = require('./middleware/inputSecurity');

const app = express();

// Security headers (first)
app.use(securityHeaders);
app.use(additionalHeaders);

// CORS configuration
app.use(createCORSConfig());

// Security monitoring middleware
app.use((req, res, next) => {
    // Check if IP is blocked
    if (SecurityMonitoringService.isIPBlocked(req.ip)) {
        SecurityMonitoringService.logSecurityEvent({
            type: 'blocked_ip_attempt',
            ip: req.ip,
            url: req.originalUrl,
            userAgent: req.get('User-Agent')
        });
        
        return res.status(429).json({
            success: false,
            message: 'Access temporarily blocked due to suspicious activity'
        });
    }
    
    // Log all requests for monitoring
    SecurityMonitoringService.logSecurityEvent({
        type: 'request',
        ip: req.ip,
        method: req.method,
        url: req.originalUrl,
        userAgent: req.get('User-Agent'),
        userId: req.user?.id
    });
    
    next();
});

// Input validation rate limiting
app.use('/api', InputSecurity.createValidationRateLimit());

// Body parsing with security limits
app.use(express.json({ 
    limit: '10mb',
    verify: (req, res, buf) => {
        // Detect potential JSON bombs
        if (buf.length > 10 * 1024 * 1024) { // 10MB
            throw new Error('Request entity too large');
        }
    }
}));

app.use(express.urlencoded({ 
    extended: true, 
    limit: '10mb' 
}));

// Global input sanitization
app.use((req, res, next) => {
    // Sanitize query parameters
    for (const key in req.query) {
        if (typeof req.query[key] === 'string') {
            req.query[key] = InputSecurity.sanitizeInput(req.query[key]);
        }
    }
    
    // Sanitize body parameters
    if (req.body && typeof req.body === 'object') {
        req.body = sanitizeObject(req.body);
    }
    
    next();
});

function sanitizeObject(obj) {
    const sanitized = {};
    for (const key in obj) {
        if (typeof obj[key] === 'string') {
            sanitized[key] = InputSecurity.sanitizeInput(obj[key]);
        } else if (typeof obj[key] === 'object' && obj[key] !== null) {
            sanitized[key] = sanitizeObject(obj[key]);
        } else {
            sanitized[key] = obj[key];
        }
    }
    return sanitized;
}

module.exports = app;
```

---

## 🔗 Related Topics
- [[24-File-Uploads]] - File upload security
- [[23-Cookies-Sessions]] - Session security
- [[13-JWT-Tokens]] - Token security
- [[17-Rate-Limiting]] - Rate limiting strategies

---

*Back to [[00-Main-Index]]*
