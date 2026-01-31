# Environment Variables (.env)

**Navigation**: [[13-Rate-Limiting]] | [[00-Main-Index]] | Next: [[15-Query-Parameters]]

---

## 🌍 Environment Variables in Node.js

Environment variables provide a way to **configure applications** without hardcoding sensitive information like API keys, database URLs, and secrets.

---

## 📦 Using dotenv Package

### Installation and Setup
```bash
npm install dotenv
```

### Basic .env File
```bash
# .env file
NODE_ENV=development
PORT=3000
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
DB_USER=postgres
DB_PASSWORD=secret123
JWT_SECRET=your-super-secret-jwt-key
API_KEY=your-api-key-here
REDIS_URL=redis://localhost:6379
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your-email@gmail.com
EMAIL_PASS=your-app-password
```

### Loading Environment Variables
```javascript
// Load environment variables at the top of your app
require('dotenv').config();

const express = require('express');
const app = express();

// Access environment variables
const PORT = process.env.PORT || 3000;
const NODE_ENV = process.env.NODE_ENV || 'development';
const DB_URL = process.env.DATABASE_URL;

console.log('Environment:', NODE_ENV);
console.log('Port:', PORT);

app.get('/', (req, res) => {
    res.json({
        environment: NODE_ENV,
        port: PORT,
        // Don't expose sensitive data
        hasDbUrl: !!DB_URL
    });
});

app.listen(PORT, () => {
    console.log(`Server running on port ${PORT} in ${NODE_ENV} mode`);
});
```

---

## 🔧 Environment Configuration Patterns

### Configuration Object Pattern
```javascript
// config/index.js
require('dotenv').config();

const config = {
    // Server configuration
    server: {
        port: parseInt(process.env.PORT) || 3000,
        host: process.env.HOST || 'localhost',
        env: process.env.NODE_ENV || 'development'
    },
    
    // Database configuration
    database: {
        host: process.env.DB_HOST || 'localhost',
        port: parseInt(process.env.DB_PORT) || 5432,
        name: process.env.DB_NAME || 'myapp',
        user: process.env.DB_USER || 'postgres',
        password: process.env.DB_PASSWORD,
        url: process.env.DATABASE_URL,
        ssl: process.env.DB_SSL === 'true',
        pool: {
            min: parseInt(process.env.DB_POOL_MIN) || 2,
            max: parseInt(process.env.DB_POOL_MAX) || 10
        }
    },
    
    // Authentication configuration
    auth: {
        jwtSecret: process.env.JWT_SECRET,
        jwtExpiration: process.env.JWT_EXPIRATION || '1h',
        jwtRefreshSecret: process.env.JWT_REFRESH_SECRET,
        jwtRefreshExpiration: process.env.JWT_REFRESH_EXPIRATION || '7d',
        bcryptRounds: parseInt(process.env.BCRYPT_ROUNDS) || 10
    },
    
    // External services
    services: {
        redis: {
            url: process.env.REDIS_URL || 'redis://localhost:6379',
            password: process.env.REDIS_PASSWORD
        },
        email: {
            host: process.env.EMAIL_HOST,
            port: parseInt(process.env.EMAIL_PORT) || 587,
            secure: process.env.EMAIL_SECURE === 'true',
            user: process.env.EMAIL_USER,
            password: process.env.EMAIL_PASSWORD
        },
        aws: {
            region: process.env.AWS_REGION || 'us-east-1',
            accessKeyId: process.env.AWS_ACCESS_KEY_ID,
            secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
            s3Bucket: process.env.AWS_S3_BUCKET
        }
    },
    
    // Feature flags
    features: {
        enableRegistration: process.env.ENABLE_REGISTRATION === 'true',
        enableEmailVerification: process.env.ENABLE_EMAIL_VERIFICATION === 'true',
        enableLogging: process.env.ENABLE_LOGGING !== 'false',
        maintenanceMode: process.env.MAINTENANCE_MODE === 'true'
    },
    
    // Rate limiting
    rateLimit: {
        windowMs: parseInt(process.env.RATE_LIMIT_WINDOW) || 15 * 60 * 1000,
        maxRequests: parseInt(process.env.RATE_LIMIT_MAX) || 100,
        enabled: process.env.RATE_LIMIT_ENABLED !== 'false'
    }
};

// Validation
function validateConfig() {
    const required = [
        'JWT_SECRET',
        'DB_PASSWORD'
    ];
    
    const missing = required.filter(key => !process.env[key]);
    
    if (missing.length > 0) {
        throw new Error(`Missing required environment variables: ${missing.join(', ')}`);
    }
}

// Only validate in production
if (config.server.env === 'production') {
    validateConfig();
}

module.exports = config;
```

### Using the Configuration
```javascript
// app.js
const config = require('./config');
const express = require('express');

const app = express();

// Use configuration throughout the app
app.use(express.json({ limit: config.server.maxPayloadSize }));

// Database connection
const dbConfig = config.database;
console.log(`Connecting to database: ${dbConfig.host}:${dbConfig.port}/${dbConfig.name}`);

// Feature flags
if (config.features.maintenanceMode) {
    app.use((req, res) => {
        res.status(503).json({ message: 'System under maintenance' });
    });
}

app.listen(config.server.port, () => {
    console.log(`Server running on ${config.server.host}:${config.server.port}`);
});
```

---

## 🔒 Environment-Specific Configurations

### Multiple Environment Files
```bash
# Directory structure
.env                 # Default/development
.env.local          # Local overrides (git ignored)
.env.development    # Development specific
.env.staging        # Staging environment
.env.production     # Production environment
.env.test          # Test environment
```

### Environment-Specific Loading
```javascript
// config/env.js
const path = require('path');

function loadEnvironment() {
    const env = process.env.NODE_ENV || 'development';
    
    // Load in order of precedence
    const envFiles = [
        `.env.${env}.local`,
        `.env.local`,
        `.env.${env}`,
        '.env'
    ];
    
    envFiles.forEach(file => {
        const envPath = path.resolve(process.cwd(), file);
        require('dotenv').config({ path: envPath });
    });
}

loadEnvironment();

const environmentConfig = {
    development: {
        database: {
            host: 'localhost',
            port: 5432,
            ssl: false,
            logging: true
        },
        logging: {
            level: 'debug',
            prettyPrint: true
        },
        cors: {
            origin: ['http://localhost:3000', 'http://localhost:3001']
        }
    },
    
    staging: {
        database: {
            ssl: true,
            logging: false
        },
        logging: {
            level: 'info',
            prettyPrint: false
        },
        cors: {
            origin: ['https://staging.myapp.com']
        }
    },
    
    production: {
        database: {
            ssl: true,
            logging: false,
            pool: {
                min: 5,
                max: 20
            }
        },
        logging: {
            level: 'warn',
            prettyPrint: false
        },
        cors: {
            origin: ['https://myapp.com']
        }
    },
    
    test: {
        database: {
            host: 'localhost',
            name: 'myapp_test',
            logging: false
        },
        logging: {
            level: 'error'
        }
    }
};

const env = process.env.NODE_ENV || 'development';
module.exports = environmentConfig[env];
```

---

## 🛡️ Security Best Practices

### Sensitive Data Handling
```javascript
// utils/secrets.js
const crypto = require('crypto');

class SecretsManager {
    constructor() {
        this.secrets = new Map();
        this.loadSecrets();
    }
    
    loadSecrets() {
        // Load and validate secrets
        const requiredSecrets = [
            'JWT_SECRET',
            'DB_PASSWORD',
            'ENCRYPTION_KEY'
        ];
        
        requiredSecrets.forEach(key => {
            const value = process.env[key];
            if (!value) {
                throw new Error(`Missing required secret: ${key}`);
            }
            
            // Validate secret strength
            if (key.includes('SECRET') && value.length < 32) {
                throw new Error(`Secret ${key} is too weak (minimum 32 characters)`);
            }
            
            this.secrets.set(key, value);
        });
    }
    
    get(key) {
        return this.secrets.get(key);
    }
    
    // Encrypt sensitive data
    encrypt(text) {
        const key = this.get('ENCRYPTION_KEY');
        const iv = crypto.randomBytes(16);
        const cipher = crypto.createCipher('aes-256-cbc', key);
        
        let encrypted = cipher.update(text, 'utf8', 'hex');
        encrypted += cipher.final('hex');
        
        return iv.toString('hex') + ':' + encrypted;
    }
    
    // Decrypt sensitive data
    decrypt(encryptedText) {
        const key = this.get('ENCRYPTION_KEY');
        const [ivHex, encrypted] = encryptedText.split(':');
        const iv = Buffer.from(ivHex, 'hex');
        const decipher = crypto.createDecipher('aes-256-cbc', key);
        
        let decrypted = decipher.update(encrypted, 'hex', 'utf8');
        decrypted += decipher.final('utf8');
        
        return decrypted;
    }
}

module.exports = new SecretsManager();
```

### Environment Variable Validation
```javascript
// utils/validation.js
const joi = require('joi');

const envSchema = joi.object({
    NODE_ENV: joi.string()
        .valid('development', 'staging', 'production', 'test')
        .default('development'),
    
    PORT: joi.number()
        .port()
        .default(3000),
    
    DB_HOST: joi.string()
        .hostname()
        .required(),
    
    DB_PORT: joi.number()
        .port()
        .default(5432),
    
    DB_NAME: joi.string()
        .alphanum()
        .min(3)
        .required(),
    
    DB_USER: joi.string()
        .alphanum()
        .required(),
    
    DB_PASSWORD: joi.string()
        .min(8)
        .required(),
    
    JWT_SECRET: joi.string()
        .min(32)
        .required(),
    
    JWT_EXPIRATION: joi.string()
        .pattern(/^\d+[smhdw]$/)
        .default('1h'),
    
    API_KEY: joi.string()
        .alphanum()
        .min(16)
        .required(),
    
    EMAIL_HOST: joi.string()
        .hostname()
        .when('ENABLE_EMAIL', {
            is: 'true',
            then: joi.required()
        }),
    
    REDIS_URL: joi.string()
        .uri({ scheme: ['redis', 'rediss'] })
        .default('redis://localhost:6379'),
    
    ENABLE_REGISTRATION: joi.boolean()
        .default(true),
    
    RATE_LIMIT_MAX: joi.number()
        .positive()
        .default(100)
});

function validateEnvironment() {
    const { error, value } = envSchema.validate(process.env, {
        allowUnknown: true,
        stripUnknown: false
    });
    
    if (error) {
        throw new Error(`Environment validation error: ${error.message}`);
    }
    
    return value;
}

module.exports = { validateEnvironment };
```

---

## 🔧 Development Tools and Utilities

### Environment Variable Management
```javascript
// scripts/env-check.js
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');

function checkEnvironmentFiles() {
    const envFiles = ['.env', '.env.local', '.env.production'];
    const missingFiles = [];
    const existingFiles = [];
    
    envFiles.forEach(file => {
        const filePath = path.join(process.cwd(), file);
        if (fs.existsSync(filePath)) {
            existingFiles.push(file);
        } else {
            missingFiles.push(file);
        }
    });
    
    console.log('Environment Files Status:');
    console.log('Existing:', existingFiles.join(', ') || 'None');
    console.log('Missing:', missingFiles.join(', ') || 'None');
    
    return { existingFiles, missingFiles };
}

function generateEnvTemplate() {
    const template = `# Environment Configuration
# Copy this file to .env and fill in your values

# Server Configuration
NODE_ENV=development
PORT=3000
HOST=localhost

# Database Configuration
DB_HOST=localhost
DB_PORT=5432
DB_NAME=myapp
DB_USER=postgres
DB_PASSWORD=

# Authentication
JWT_SECRET=
JWT_EXPIRATION=1h
JWT_REFRESH_SECRET=
JWT_REFRESH_EXPIRATION=7d

# External Services
REDIS_URL=redis://localhost:6379
EMAIL_HOST=
EMAIL_PORT=587
EMAIL_USER=
EMAIL_PASSWORD=

# API Keys
API_KEY=
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_S3_BUCKET=

# Feature Flags
ENABLE_REGISTRATION=true
ENABLE_EMAIL_VERIFICATION=false
MAINTENANCE_MODE=false

# Rate Limiting
RATE_LIMIT_WINDOW=900000
RATE_LIMIT_MAX=100
`;
    
    fs.writeFileSync('.env.template', template);
    console.log('Environment template generated: .env.template');
}

// CLI usage
if (require.main === module) {
    const command = process.argv[2];
    
    switch (command) {
        case 'check':
            checkEnvironmentFiles();
            break;
        case 'template':
            generateEnvTemplate();
            break;
        default:
            console.log('Usage: node env-check.js [check|template]');
    }
}

module.exports = { checkEnvironmentFiles, generateEnvTemplate };
```

### Dynamic Environment Loading
```javascript
// utils/env-loader.js
const fs = require('fs');
const path = require('path');

class EnvironmentLoader {
    constructor() {
        this.loaded = false;
        this.variables = new Map();
    }
    
    load(options = {}) {
        if (this.loaded && !options.reload) {
            return this.variables;
        }
        
        const env = options.environment || process.env.NODE_ENV || 'development';
        const basePath = options.basePath || process.cwd();
        
        // Define environment file priority
        const envFiles = [
            `.env.${env}.local`,
            '.env.local',
            `.env.${env}`,
            '.env'
        ];
        
        // Load each file in order
        envFiles.forEach(file => {
            const filePath = path.join(basePath, file);
            if (fs.existsSync(filePath)) {
                this.loadFile(filePath);
            }
        });
        
        this.loaded = true;
        return this.variables;
    }
    
    loadFile(filePath) {
        const content = fs.readFileSync(filePath, 'utf8');
        const lines = content.split('\n');
        
        lines.forEach(line => {
            line = line.trim();
            
            // Skip comments and empty lines
            if (!line || line.startsWith('#')) return;
            
            const [key, ...valueParts] = line.split('=');
            const value = valueParts.join('=').trim();
            
            if (key && value !== undefined) {
                // Remove quotes if present
                const cleanValue = value.replace(/^["']|["']$/g, '');
                
                // Only set if not already in process.env
                if (!process.env[key]) {
                    process.env[key] = cleanValue;
                }
                
                this.variables.set(key, cleanValue);
            }
        });
    }
    
    get(key, defaultValue = undefined) {
        return process.env[key] || this.variables.get(key) || defaultValue;
    }
    
    getRequired(key) {
        const value = this.get(key);
        if (value === undefined || value === '') {
            throw new Error(`Required environment variable missing: ${key}`);
        }
        return value;
    }
    
    getInt(key, defaultValue = undefined) {
        const value = this.get(key, defaultValue);
        return value !== undefined ? parseInt(value, 10) : undefined;
    }
    
    getBool(key, defaultValue = false) {
        const value = this.get(key);
        if (value === undefined) return defaultValue;
        return value.toLowerCase() === 'true' || value === '1';
    }
    
    getArray(key, separator = ',', defaultValue = []) {
        const value = this.get(key);
        if (!value) return defaultValue;
        return value.split(separator).map(item => item.trim());
    }
}

module.exports = new EnvironmentLoader();
```

---

## 📋 Environment Variable Checklist

### Production Deployment Checklist
```javascript
// scripts/production-check.js
const requiredProductionVars = [
    'NODE_ENV',
    'PORT',
    'DATABASE_URL',
    'JWT_SECRET',
    'JWT_REFRESH_SECRET',
    'ENCRYPTION_KEY',
    'REDIS_URL'
];

const securityChecks = [
    {
        name: 'JWT_SECRET length',
        check: () => process.env.JWT_SECRET && process.env.JWT_SECRET.length >= 32,
        message: 'JWT_SECRET must be at least 32 characters'
    },
    {
        name: 'NODE_ENV is production',
        check: () => process.env.NODE_ENV === 'production',
        message: 'NODE_ENV must be set to production'
    },
    {
        name: 'Database SSL enabled',
        check: () => process.env.DB_SSL === 'true' || process.env.DATABASE_URL?.includes('sslmode'),
        message: 'Database SSL should be enabled in production'
    },
    {
        name: 'No default passwords',
        check: () => {
            const defaultPasswords = ['password', '123456', 'admin', 'secret'];
            return !defaultPasswords.some(pwd => 
                Object.values(process.env).some(val => val?.toLowerCase().includes(pwd))
            );
        },
        message: 'No default passwords should be used'
    }
];

function runProductionCheck() {
    console.log('🔍 Production Environment Check\n');
    
    let allPassed = true;
    
    // Check required variables
    console.log('Required Variables:');
    requiredProductionVars.forEach(varName => {
        const exists = !!process.env[varName];
        console.log(`  ${exists ? '✅' : '❌'} ${varName}`);
        if (!exists) allPassed = false;
    });
    
    console.log('\nSecurity Checks:');
    securityChecks.forEach(check => {
        const passed = check.check();
        console.log(`  ${passed ? '✅' : '❌'} ${check.name}`);
        if (!passed) {
            console.log(`      ${check.message}`);
            allPassed = false;
        }
    });
    
    console.log(`\n${allPassed ? '✅ All checks passed' : '❌ Some checks failed'}`);
    
    if (!allPassed) {
        process.exit(1);
    }
}

if (require.main === module) {
    runProductionCheck();
}

module.exports = { runProductionCheck };
```

---

## 🔗 Related Topics
- [[11-Authentication-and-Authorization]] - Securing auth secrets
- [[12-JWT-Tokens]] - JWT secret management
- [[28-PostgreSQL-Integration]] - Database connection strings
- [[32-Development-vs-Production]] - Environment-specific configs

---

*Back to [[00-Main-Index]]*
