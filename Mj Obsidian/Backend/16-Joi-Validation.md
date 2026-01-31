# Joi Validation

**Navigation**: [[15-Query-Parameters]] | [[00-Main-Index]] | Next: [[17-Express-Sessions]]

---

## 🛡️ Joi Validation in Node.js

**Joi** is a powerful **schema validation library** for JavaScript that allows you to validate data with a simple, intuitive, and readable API. It's essential for validating user input, API requests, and configuration data.

---

## 📦 Installation and Basic Setup

### Installation
```bash
npm install joi
```

### Basic Joi Usage
```javascript
const Joi = require('joi');

// Simple schema validation
const schema = Joi.object({
    username: Joi.string().min(3).max(30).required(),
    email: Joi.string().email().required(),
    age: Joi.number().integer().min(18).max(100),
    password: Joi.string().min(8).pattern(new RegExp('^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#\$%\^&\*])')).required()
});

// Validate data
const userData = {
    username: 'john_doe',
    email: 'john@example.com',
    age: 25,
    password: 'MyPass123!'
};

const { error, value } = schema.validate(userData);

if (error) {
    console.log('Validation error:', error.details);
} else {
    console.log('Valid data:', value);
}
```

---

## 🔧 Joi Schema Types and Validations

### String Validations
```javascript
const stringSchema = Joi.object({
    // Basic string validations
    username: Joi.string()
        .min(3)                              // Minimum length
        .max(20)                             // Maximum length
        .alphanum()                          // Only alphanumeric characters
        .required(),                         // Required field
    
    email: Joi.string()
        .email()                             // Email format
        .lowercase()                         // Convert to lowercase
        .required(),
    
    password: Joi.string()
        .min(8)
        .pattern(new RegExp('^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#\$%\^&\*])'))
        .required()
        .messages({
            'string.pattern.base': 'Password must contain at least one lowercase letter, one uppercase letter, one number, and one special character'
        }),
    
    // Custom patterns
    phoneNumber: Joi.string()
        .pattern(/^\+?[1-9]\d{1,14}$/)       // International phone format
        .messages({
            'string.pattern.base': 'Phone number must be in international format'
        }),
    
    // Enumerated values
    role: Joi.string()
        .valid('admin', 'user', 'moderator')
        .default('user'),
    
    // Case insensitive
    country: Joi.string()
        .valid('US', 'UK', 'CA', 'AU')
        .insensitive(),
    
    // Custom validation
    slug: Joi.string()
        .pattern(/^[a-z0-9-]+$/)
        .custom((value, helpers) => {
            if (value.startsWith('-') || value.endsWith('-')) {
                return helpers.error('any.invalid');
            }
            return value;
        })
        .messages({
            'any.invalid': 'Slug cannot start or end with a hyphen'
        })
});
```

### Number Validations
```javascript
const numberSchema = Joi.object({
    // Integer validations
    age: Joi.number()
        .integer()                           // Must be integer
        .min(18)                            // Minimum value
        .max(100)                           // Maximum value
        .required(),
    
    // Float validations
    price: Joi.number()
        .precision(2)                       // Maximum 2 decimal places
        .positive()                         // Must be positive
        .required(),
    
    // Specific values
    rating: Joi.number()
        .valid(1, 2, 3, 4, 5)              // Only specific values
        .required(),
    
    // Multiple of
    quantity: Joi.number()
        .integer()
        .multiple(5)                        // Must be multiple of 5
        .min(5),
    
    // Port number
    port: Joi.number()
        .port()                             // Valid port number (1-65535)
        .default(3000),
    
    // Custom validation
    discount: Joi.number()
        .min(0)
        .max(100)
        .custom((value, helpers) => {
            if (value > 50 && !helpers.state.ancestors[0].isPremium) {
                return helpers.error('any.invalid');
            }
            return value;
        })
        .messages({
            'any.invalid': 'Discount cannot exceed 50% for non-premium users'
        })
});
```

### Date Validations
```javascript
const dateSchema = Joi.object({
    // Basic date
    birthDate: Joi.date()
        .max('now')                         // Cannot be in future
        .required(),
    
    // Date with minimum age
    adultBirthDate: Joi.date()
        .max(new Date(Date.now() - 18 * 365 * 24 * 60 * 60 * 1000))
        .required(),
    
    // ISO date string
    createdAt: Joi.date()
        .iso()                              // ISO format
        .default(Date.now),
    
    // Date range
    eventDate: Joi.date()
        .min('now')                         // Must be in future
        .max(new Date(Date.now() + 365 * 24 * 60 * 60 * 1000)), // Within 1 year
    
    // Custom date validation
    scheduleDate: Joi.date()
        .custom((value, helpers) => {
            const dayOfWeek = value.getDay();
            if (dayOfWeek === 0 || dayOfWeek === 6) { // Sunday or Saturday
                return helpers.error('any.invalid');
            }
            return value;
        })
        .messages({
            'any.invalid': 'Schedule date must be a weekday'
        })
});
```

### Array Validations
```javascript
const arraySchema = Joi.object({
    // Array of strings
    tags: Joi.array()
        .items(Joi.string().min(2).max(20))  // Each item validation
        .min(1)                              // Minimum array length
        .max(10)                             // Maximum array length
        .unique()                            // No duplicates
        .required(),
    
    // Array of numbers
    scores: Joi.array()
        .items(Joi.number().min(0).max(100))
        .length(5)                           // Exact length
        .required(),
    
    // Array of objects
    addresses: Joi.array()
        .items(Joi.object({
            street: Joi.string().required(),
            city: Joi.string().required(),
            zipCode: Joi.string().pattern(/^\d{5}$/).required(),
            isDefault: Joi.boolean().default(false)
        }))
        .min(1)
        .required(),
    
    // Mixed array types
    permissions: Joi.array()
        .items(
            Joi.string().valid('read', 'write', 'delete'),
            Joi.object({
                resource: Joi.string().required(),
                actions: Joi.array().items(Joi.string()).required()
            })
        )
        .unique((a, b) => {
            if (typeof a === 'string' && typeof b === 'string') {
                return a === b;
            }
            if (typeof a === 'object' && typeof b === 'object') {
                return a.resource === b.resource;
            }
            return false;
        })
});
```

### Object Validations
```javascript
const objectSchema = Joi.object({
    // Nested objects
    profile: Joi.object({
        firstName: Joi.string().required(),
        lastName: Joi.string().required(),
        bio: Joi.string().max(500),
        socialMedia: Joi.object({
            twitter: Joi.string().uri(),
            linkedin: Joi.string().uri(),
            github: Joi.string().uri()
        }).optional()
    }).required(),
    
    // Dynamic keys
    metadata: Joi.object()
        .pattern(/^[a-zA-Z0-9_]+$/, Joi.string()) // Key pattern and value type
        .min(1)
        .max(10),
    
    // Unknown keys handling
    settings: Joi.object({
        theme: Joi.string().valid('light', 'dark').default('light'),
        language: Joi.string().default('en')
    }).unknown(false),                       // Reject unknown keys
    
    // Conditional validation
    user: Joi.object({
        type: Joi.string().valid('individual', 'company').required(),
        name: Joi.string().when('type', {
            is: 'individual',
            then: Joi.string().max(50),
            otherwise: Joi.string().max(100)
        }).required(),
        taxId: Joi.string().when('type', {
            is: 'company',
            then: Joi.string().required(),
            otherwise: Joi.string().optional()
        })
    })
});
```

---

## 🛠️ Express Middleware Integration

### Basic Validation Middleware
```javascript
// middleware/validation.js
const Joi = require('joi');

function validate(schema, property = 'body') {
    return (req, res, next) => {
        const { error, value } = schema.validate(req[property], {
            abortEarly: false,      // Return all errors
            allowUnknown: false,    // Strip unknown properties
            stripUnknown: true      // Remove unknown properties
        });
        
        if (error) {
            const errorDetails = error.details.map(detail => ({
                field: detail.path.join('.'),
                message: detail.message,
                value: detail.context.value
            }));
            
            return res.status(400).json({
                error: 'Validation failed',
                details: errorDetails
            });
        }
        
        // Replace request data with validated data
        req[property] = value;
        next();
    };
}

// Validate different parts of request
function validateBody(schema) {
    return validate(schema, 'body');
}

function validateQuery(schema) {
    return validate(schema, 'query');
}

function validateParams(schema) {
    return validate(schema, 'params');
}

function validateHeaders(schema) {
    return validate(schema, 'headers');
}

module.exports = {
    validate,
    validateBody,
    validateQuery,
    validateParams,
    validateHeaders
};
```

### Advanced Validation Middleware
```javascript
// middleware/advancedValidation.js
const Joi = require('joi');

class ValidationMiddleware {
    static createValidator(schemas) {
        return async (req, res, next) => {
            const errors = {};
            const validatedData = {};
            
            // Validate each part of the request
            for (const [key, schema] of Object.entries(schemas)) {
                if (req[key]) {
                    const { error, value } = schema.validate(req[key], {
                        abortEarly: false,
                        allowUnknown: key === 'headers', // Allow unknown headers
                        stripUnknown: key !== 'headers'  // Strip unknown except headers
                    });
                    
                    if (error) {
                        errors[key] = error.details.map(detail => ({
                            field: detail.path.join('.'),
                            message: detail.message,
                            value: detail.context.value
                        }));
                    } else {
                        validatedData[key] = value;
                    }
                }
            }
            
            if (Object.keys(errors).length > 0) {
                return res.status(400).json({
                    error: 'Validation failed',
                    details: errors
                });
            }
            
            // Replace request data with validated data
            Object.assign(req, validatedData);
            next();
        };
    }
    
    static async validateAsync(schema, data, options = {}) {
        try {
            const value = await schema.validateAsync(data, {
                abortEarly: false,
                ...options
            });
            return { value, error: null };
        } catch (error) {
            return { value: null, error };
        }
    }
    
    static createConditionalValidator(condition, schemas) {
        return (req, res, next) => {
            const shouldValidate = typeof condition === 'function' 
                ? condition(req) 
                : req[condition.property] === condition.value;
            
            if (!shouldValidate) {
                return next();
            }
            
            const validator = ValidationMiddleware.createValidator(schemas);
            return validator(req, res, next);
        };
    }
}

module.exports = ValidationMiddleware;
```

---

## 🔧 Real-World Validation Examples

### User Registration Validation
```javascript
// schemas/userSchemas.js
const Joi = require('joi');

const registerSchema = Joi.object({
    username: Joi.string()
        .min(3)
        .max(20)
        .pattern(/^[a-zA-Z0-9_]+$/)
        .required()
        .messages({
            'string.pattern.base': 'Username can only contain letters, numbers, and underscores'
        }),
    
    email: Joi.string()
        .email({ tlds: { allow: false } })  // Allow any TLD
        .lowercase()
        .required(),
    
    password: Joi.string()
        .min(8)
        .max(128)
        .pattern(new RegExp('^(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#\$%\^&\*])'))
        .required()
        .messages({
            'string.pattern.base': 'Password must contain uppercase, lowercase, number, and special character'
        }),
    
    confirmPassword: Joi.string()
        .valid(Joi.ref('password'))
        .required()
        .messages({
            'any.only': 'Passwords do not match'
        }),
    
    firstName: Joi.string()
        .min(2)
        .max(50)
        .pattern(/^[a-zA-Z\s]+$/)
        .required(),
    
    lastName: Joi.string()
        .min(2)
        .max(50)
        .pattern(/^[a-zA-Z\s]+$/)
        .required(),
    
    dateOfBirth: Joi.date()
        .max(new Date(Date.now() - 13 * 365 * 24 * 60 * 60 * 1000)) // At least 13 years old
        .required(),
    
    phoneNumber: Joi.string()
        .pattern(/^\+?[1-9]\d{1,14}$/)
        .optional(),
    
    agreeToTerms: Joi.boolean()
        .valid(true)
        .required()
        .messages({
            'any.only': 'You must agree to the terms and conditions'
        })
});

const loginSchema = Joi.object({
    email: Joi.string()
        .email()
        .required(),
    
    password: Joi.string()
        .required(),
    
    rememberMe: Joi.boolean()
        .default(false)
});

const updateProfileSchema = Joi.object({
    firstName: Joi.string()
        .min(2)
        .max(50)
        .pattern(/^[a-zA-Z\s]+$/),
    
    lastName: Joi.string()
        .min(2)
        .max(50)
        .pattern(/^[a-zA-Z\s]+$/),
    
    bio: Joi.string()
        .max(500)
        .allow(''),
    
    website: Joi.string()
        .uri()
        .allow(''),
    
    location: Joi.string()
        .max(100)
        .allow(''),
    
    phoneNumber: Joi.string()
        .pattern(/^\+?[1-9]\d{1,14}$/)
        .allow('')
});

module.exports = {
    registerSchema,
    loginSchema,
    updateProfileSchema
};
```

### E-commerce Product Validation
```javascript
// schemas/productSchemas.js
const createProductSchema = Joi.object({
    name: Joi.string()
        .min(3)
        .max(200)
        .required(),
    
    description: Joi.string()
        .min(10)
        .max(2000)
        .required(),
    
    price: Joi.number()
        .precision(2)
        .positive()
        .max(999999.99)
        .required(),
    
    salePrice: Joi.number()
        .precision(2)
        .positive()
        .max(Joi.ref('price'))
        .optional(),
    
    categoryId: Joi.string()
        .pattern(/^[0-9a-fA-F]{24}$/)       // MongoDB ObjectId pattern
        .required(),
    
    subcategoryId: Joi.string()
        .pattern(/^[0-9a-fA-F]{24}$/)
        .optional(),
    
    brandId: Joi.string()
        .pattern(/^[0-9a-fA-F]{24}$/)
        .optional(),
    
    tags: Joi.array()
        .items(Joi.string().min(2).max(30))
        .max(10)
        .unique()
        .default([]),
    
    specifications: Joi.object()
        .pattern(/^[a-zA-Z0-9_]+$/, Joi.alternatives().try(
            Joi.string().max(200),
            Joi.number(),
            Joi.boolean()
        ))
        .max(20),
    
    dimensions: Joi.object({
        length: Joi.number().positive(),
        width: Joi.number().positive(),
        height: Joi.number().positive(),
        weight: Joi.number().positive(),
        unit: Joi.string().valid('cm', 'inch', 'mm').default('cm')
    }).optional(),
    
    inventory: Joi.object({
        stockQuantity: Joi.number().integer().min(0).required(),
        lowStockThreshold: Joi.number().integer().min(0).default(5),
        trackInventory: Joi.boolean().default(true),
        allowBackorders: Joi.boolean().default(false)
    }).required(),
    
    shipping: Joi.object({
        freeShipping: Joi.boolean().default(false),
        shippingWeight: Joi.number().positive(),
        shippingClass: Joi.string().valid('standard', 'expedited', 'overnight').default('standard')
    }).optional(),
    
    seo: Joi.object({
        metaTitle: Joi.string().max(60),
        metaDescription: Joi.string().max(160),
        slug: Joi.string().pattern(/^[a-z0-9-]+$/).max(100)
    }).optional(),
    
    status: Joi.string()
        .valid('draft', 'published', 'archived')
        .default('draft'),
    
    featured: Joi.boolean()
        .default(false),
    
    images: Joi.array()
        .items(Joi.object({
            url: Joi.string().uri().required(),
            alt: Joi.string().max(100),
            isPrimary: Joi.boolean().default(false)
        }))
        .max(10)
        .default([])
});

const updateProductSchema = createProductSchema.fork(
    ['name', 'description', 'price', 'categoryId', 'inventory'],
    (schema) => schema.optional()
);

const productQuerySchema = Joi.object({
    page: Joi.number().integer().min(1).default(1),
    limit: Joi.number().integer().min(1).max(100).default(20),
    sort: Joi.string().valid('name', 'price', 'created_at', 'rating').default('created_at'),
    order: Joi.string().valid('asc', 'desc').default('desc'),
    category: Joi.string().pattern(/^[0-9a-fA-F]{24}$/),
    brand: Joi.string().pattern(/^[0-9a-fA-F]{24}$/),
    minPrice: Joi.number().min(0),
    maxPrice: Joi.number().min(0),
    inStock: Joi.boolean(),
    featured: Joi.boolean(),
    status: Joi.string().valid('published', 'draft', 'archived').default('published'),
    search: Joi.string().max(100)
});

module.exports = {
    createProductSchema,
    updateProductSchema,
    productQuerySchema
};
```

### API Request Validation with Headers
```javascript
// schemas/apiSchemas.js
const apiKeyHeaderSchema = Joi.object({
    'x-api-key': Joi.string()
        .pattern(/^[a-zA-Z0-9]{32}$/)
        .required()
        .messages({
            'string.pattern.base': 'Invalid API key format'
        }),
    
    'content-type': Joi.string()
        .valid('application/json')
        .insensitive()
        .when('$method', {
            is: Joi.valid('POST', 'PUT', 'PATCH'),
            then: Joi.required(),
            otherwise: Joi.optional()
        }),
    
    'user-agent': Joi.string()
        .max(200)
        .optional(),
    
    'accept': Joi.string()
        .valid('application/json', '*/*')
        .insensitive()
        .default('application/json')
}).unknown(true); // Allow other headers

const paginationQuerySchema = Joi.object({
    page: Joi.number()
        .integer()
        .min(1)
        .max(1000)
        .default(1),
    
    limit: Joi.number()
        .integer()
        .min(1)
        .max(100)
        .default(20),
    
    sort: Joi.string()
        .pattern(/^[a-zA-Z_][a-zA-Z0-9_]*$/)
        .max(50)
        .optional(),
    
    order: Joi.string()
        .valid('asc', 'desc')
        .insensitive()
        .default('asc')
});

// Usage in routes
const { validateBody, validateQuery, validateHeaders } = require('../middleware/validation');

router.post('/products',
    validateHeaders(apiKeyHeaderSchema),
    validateBody(createProductSchema),
    async (req, res) => {
        // Handle validated request
    }
);

router.get('/products',
    validateQuery(Joi.object().concat(productQuerySchema).concat(paginationQuerySchema)),
    async (req, res) => {
        // Handle validated request
    }
);
```

---

## 🔧 Custom Validation Functions

### Custom Validators
```javascript
// utils/customValidators.js
const Joi = require('joi');

// Custom extension for advanced validations
const JoiExtended = Joi.extend(
    {
        type: 'string',
        base: Joi.string(),
        messages: {
            'string.creditCard': '{{#label}} must be a valid credit card number',
            'string.strongPassword': '{{#label}} must contain at least 8 characters with uppercase, lowercase, number and special character'
        },
        rules: {
            creditCard: {
                method() {
                    return this.$_addRule('creditCard');
                },
                validate(value, helpers) {
                    // Luhn algorithm for credit card validation
                    const cleanedValue = value.replace(/\s+/g, '');
                    if (!/^\d+$/.test(cleanedValue)) {
                        return helpers.error('string.creditCard');
                    }
                    
                    let sum = 0;
                    let isEven = false;
                    
                    for (let i = cleanedValue.length - 1; i >= 0; i--) {
                        let digit = parseInt(cleanedValue[i], 10);
                        
                        if (isEven) {
                            digit *= 2;
                            if (digit > 9) {
                                digit -= 9;
                            }
                        }
                        
                        sum += digit;
                        isEven = !isEven;
                    }
                    
                    if (sum % 10 !== 0) {
                        return helpers.error('string.creditCard');
                    }
                    
                    return value;
                }
            },
            strongPassword: {
                method() {
                    return this.$_addRule('strongPassword');
                },
                validate(value, helpers) {
                    const hasUpper = /[A-Z]/.test(value);
                    const hasLower = /[a-z]/.test(value);
                    const hasNumber = /\d/.test(value);
                    const hasSpecial = /[!@#$%^&*(),.?":{}|<>]/.test(value);
                    const isLongEnough = value.length >= 8;
                    
                    if (!hasUpper || !hasLower || !hasNumber || !hasSpecial || !isLongEnough) {
                        return helpers.error('string.strongPassword');
                    }
                    
                    return value;
                }
            }
        }
    }
);

// Custom async validator for database checks
const createAsyncValidator = (checkFunction, errorMessage) => {
    return Joi.string().external(async (value) => {
        const exists = await checkFunction(value);
        if (exists) {
            throw new Error(errorMessage);
        }
        return value;
    });
};

// Usage examples
const paymentSchema = Joi.object({
    cardNumber: JoiExtended.string()
        .creditCard()
        .required(),
    
    expiryDate: Joi.string()
        .pattern(/^(0[1-9]|1[0-2])\/\d{2}$/)
        .custom((value, helpers) => {
            const [month, year] = value.split('/');
            const expiry = new Date(2000 + parseInt(year), parseInt(month) - 1);
            const now = new Date();
            
            if (expiry < now) {
                return helpers.error('any.invalid');
            }
            
            return value;
        })
        .messages({
            'any.invalid': 'Card has expired'
        })
        .required(),
    
    cvv: Joi.string()
        .pattern(/^\d{3,4}$/)
        .required()
});

const userRegistrationSchema = Joi.object({
    username: createAsyncValidator(
        async (username) => {
            const user = await User.findOne({ username });
            return !!user;
        },
        'Username already exists'
    ).required(),
    
    email: createAsyncValidator(
        async (email) => {
            const user = await User.findOne({ email });
            return !!user;
        },
        'Email already registered'
    ).required(),
    
    password: JoiExtended.string()
        .strongPassword()
        .required()
});

module.exports = {
    JoiExtended,
    createAsyncValidator,
    paymentSchema,
    userRegistrationSchema
};
```

### File Upload Validation
```javascript
// schemas/fileSchemas.js
const Joi = require('joi');

const fileUploadSchema = Joi.object({
    fieldname: Joi.string().required(),
    originalname: Joi.string().required(),
    encoding: Joi.string().required(),
    mimetype: Joi.string()
        .valid(
            'image/jpeg',
            'image/png',
            'image/gif',
            'image/webp',
            'application/pdf',
            'text/plain',
            'application/msword',
            'application/vnd.openxmlformats-officedocument.wordprocessingml.document'
        )
        .required(),
    size: Joi.number()
        .max(5 * 1024 * 1024) // 5MB max
        .required(),
    buffer: Joi.binary().required()
});

const imageUploadSchema = fileUploadSchema.fork('mimetype', (schema) =>
    schema.valid('image/jpeg', 'image/png', 'image/gif', 'image/webp')
).append({
    size: Joi.number().max(2 * 1024 * 1024) // 2MB for images
});

// Middleware for file validation
function validateFileUpload(schema) {
    return (req, res, next) => {
        if (!req.file && !req.files) {
            return res.status(400).json({ error: 'No file uploaded' });
        }
        
        const files = req.files || [req.file];
        const errors = [];
        
        files.forEach((file, index) => {
            const { error } = schema.validate(file);
            if (error) {
                errors.push({
                    file: index,
                    errors: error.details.map(detail => detail.message)
                });
            }
        });
        
        if (errors.length > 0) {
            return res.status(400).json({
                error: 'File validation failed',
                details: errors
            });
        }
        
        next();
    };
}

module.exports = {
    fileUploadSchema,
    imageUploadSchema,
    validateFileUpload
};
```

---

## 🔗 Related Topics
- [[15-Query-Parameters]] - Validating URL parameters
- [[17-Express-Sessions]] - Session data validation
- [[11-Authentication-and-Authorization]] - User input validation
- [[32-Development-vs-Production]] - Environment-based validation

---

*Back to [[00-Main-Index]]*
