# MVC Architecture

**Navigation**: [[17-Express-Sessions]] | [[00-Main-Index]] | Next: [[19-EJS-Templating]]

---

## 🏗️ MVC Architecture in Node.js

**MVC (Model-View-Controller)** is a design pattern that separates application logic into three interconnected components, making code more **organized**, **maintainable**, and **scalable**.

---

## 📁 MVC Project Structure

### Complete Project Structure
```
project/
├── app.js                      # Main application file
├── server.js                   # Server startup
├── package.json
├── .env
├── config/
│   ├── database.js             # Database configuration
│   ├── config.js               # App configuration
│   └── passport.js             # Authentication config
├── controllers/                # Controllers (Business Logic)
│   ├── userController.js
│   ├── productController.js
│   ├── authController.js
│   └── adminController.js
├── models/                     # Models (Data Layer)
│   ├── User.js
│   ├── Product.js
│   ├── Order.js
│   └── index.js
├── views/                      # Views (Presentation Layer)
│   ├── layouts/
│   │   └── main.ejs
│   ├── partials/
│   │   ├── header.ejs
│   │   ├── footer.ejs
│   │   └── navbar.ejs
│   ├── users/
│   │   ├── profile.ejs
│   │   ├── login.ejs
│   │   └── register.ejs
│   ├── products/
│   │   ├── index.ejs
│   │   ├── show.ejs
│   │   └── create.ejs
│   └── errors/
│       ├── 404.ejs
│       └── 500.ejs
├── routes/                     # Route Definitions
│   ├── index.js
│   ├── userRoutes.js
│   ├── productRoutes.js
│   └── authRoutes.js
├── middleware/                 # Custom Middleware
│   ├── auth.js
│   ├── validation.js
│   ├── upload.js
│   └── errorHandler.js
├── services/                   # Business Services
│   ├── userService.js
│   ├── emailService.js
│   ├── paymentService.js
│   └── uploadService.js
├── utils/                      # Utility Functions
│   ├── helpers.js
│   ├── validators.js
│   └── constants.js
├── public/                     # Static Files
│   ├── css/
│   ├── js/
│   ├── images/
│   └── uploads/
└── tests/                      # Test Files
    ├── controllers/
    ├── models/
    └── services/
```

---

## 🗄️ Models (Data Layer)

### User Model Example
```javascript
// models/User.js
const mongoose = require('mongoose');
const bcrypt = require('bcrypt');
const validator = require('validator');

const userSchema = new mongoose.Schema({
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
    email: {
        type: String,
        required: [true, 'Email is required'],
        unique: true,
        lowercase: true,
        validate: [validator.isEmail, 'Please provide a valid email']
    },
    password: {
        type: String,
        required: [true, 'Password is required'],
        minlength: [8, 'Password must be at least 8 characters'],
        select: false // Don't include password in queries by default
    },
    role: {
        type: String,
        enum: ['user', 'admin', 'moderator'],
        default: 'user'
    },
    profile: {
        avatar: String,
        bio: String,
        dateOfBirth: Date,
        phoneNumber: String,
        address: {
            street: String,
            city: String,
            state: String,
            zipCode: String,
            country: String
        }
    },
    preferences: {
        emailNotifications: {
            type: Boolean,
            default: true
        },
        theme: {
            type: String,
            enum: ['light', 'dark'],
            default: 'light'
        },
        language: {
            type: String,
            default: 'en'
        }
    },
    status: {
        type: String,
        enum: ['active', 'inactive', 'suspended'],
        default: 'active'
    },
    emailVerified: {
        type: Boolean,
        default: false
    },
    emailVerificationToken: String,
    passwordResetToken: String,
    passwordResetExpires: Date,
    loginAttempts: {
        type: Number,
        default: 0
    },
    lockUntil: Date,
    lastLogin: Date,
    lastActivity: Date
}, {
    timestamps: true,
    toJSON: { virtuals: true },
    toObject: { virtuals: true }
});

// Virtual fields
userSchema.virtual('fullName').get(function() {
    return `${this.firstName} ${this.lastName}`;
});

userSchema.virtual('isLocked').get(function() {
    return !!(this.lockUntil && this.lockUntil > Date.now());
});

// Indexes
userSchema.index({ email: 1 });
userSchema.index({ role: 1 });
userSchema.index({ status: 1 });
userSchema.index({ createdAt: -1 });

// Middleware - Hash password before saving
userSchema.pre('save', async function(next) {
    // Only hash password if it's modified
    if (!this.isModified('password')) return next();
    
    try {
        // Hash password with cost of 12
        this.password = await bcrypt.hash(this.password, 12);
        next();
    } catch (error) {
        next(error);
    }
});

// Middleware - Update lastActivity
userSchema.pre(/^find/, function(next) {
    // Add lastActivity update for find operations
    this.populate({
        path: 'orders',
        select: 'orderNumber total status createdAt'
    });
    next();
});

// Instance Methods
userSchema.methods.comparePassword = async function(candidatePassword) {
    return await bcrypt.compare(candidatePassword, this.password);
};

userSchema.methods.createPasswordResetToken = function() {
    const resetToken = crypto.randomBytes(32).toString('hex');
    
    this.passwordResetToken = crypto
        .createHash('sha256')
        .update(resetToken)
        .digest('hex');
    
    this.passwordResetExpires = Date.now() + 10 * 60 * 1000; // 10 minutes
    
    return resetToken;
};

userSchema.methods.incrementLoginAttempts = function() {
    // If previous lock has expired, restart at 1
    if (this.lockUntil && this.lockUntil < Date.now()) {
        return this.updateOne({
            $unset: { lockUntil: 1 },
            $set: { loginAttempts: 1 }
        });
    }
    
    const updates = { $inc: { loginAttempts: 1 } };
    
    // Lock account after 5 attempts for 2 hours
    if (this.loginAttempts + 1 >= 5 && !this.isLocked) {
        updates.$set = { lockUntil: Date.now() + 2 * 60 * 60 * 1000 };
    }
    
    return this.updateOne(updates);
};

userSchema.methods.resetLoginAttempts = function() {
    return this.updateOne({
        $unset: { loginAttempts: 1, lockUntil: 1 }
    });
};

// Static Methods
userSchema.statics.findByEmail = function(email) {
    return this.findOne({ email: email.toLowerCase() });
};

userSchema.statics.getActiveUsers = function() {
    return this.find({ status: 'active' });
};

userSchema.statics.getUserStats = async function() {
    const stats = await this.aggregate([
        {
            $group: {
                _id: '$status',
                count: { $sum: 1 }
            }
        }
    ]);
    
    const roleStats = await this.aggregate([
        {
            $group: {
                _id: '$role',
                count: { $sum: 1 }
            }
        }
    ]);
    
    return { statusStats: stats, roleStats };
};

module.exports = mongoose.model('User', userSchema);
```

### Product Model Example
```javascript
// models/Product.js
const mongoose = require('mongoose');
const slugify = require('slugify');

const productSchema = new mongoose.Schema({
    name: {
        type: String,
        required: [true, 'Product name is required'],
        trim: true,
        maxlength: [200, 'Product name cannot exceed 200 characters']
    },
    slug: {
        type: String,
        unique: true
    },
    description: {
        type: String,
        required: [true, 'Product description is required'],
        maxlength: [2000, 'Description cannot exceed 2000 characters']
    },
    shortDescription: {
        type: String,
        maxlength: [500, 'Short description cannot exceed 500 characters']
    },
    price: {
        type: Number,
        required: [true, 'Product price is required'],
        min: [0, 'Price cannot be negative']
    },
    salePrice: {
        type: Number,
        min: [0, 'Sale price cannot be negative'],
        validate: {
            validator: function(value) {
                return !value || value < this.price;
            },
            message: 'Sale price must be less than regular price'
        }
    },
    category: {
        type: mongoose.Schema.ObjectId,
        ref: 'Category',
        required: [true, 'Product must belong to a category']
    },
    subcategory: {
        type: mongoose.Schema.ObjectId,
        ref: 'Subcategory'
    },
    brand: {
        type: mongoose.Schema.ObjectId,
        ref: 'Brand'
    },
    tags: [{
        type: String,
        trim: true
    }],
    sku: {
        type: String,
        required: [true, 'SKU is required'],
        unique: true,
        uppercase: true
    },
    images: [{
        public_id: String,
        url: String,
        alt: String,
        isPrimary: {
            type: Boolean,
            default: false
        }
    }],
    specifications: {
        type: Map,
        of: String
    },
    dimensions: {
        length: Number,
        width: Number,
        height: Number,
        weight: Number,
        unit: {
            type: String,
            enum: ['cm', 'inch', 'mm'],
            default: 'cm'
        }
    },
    inventory: {
        stockQuantity: {
            type: Number,
            required: [true, 'Stock quantity is required'],
            min: [0, 'Stock cannot be negative']
        },
        lowStockThreshold: {
            type: Number,
            default: 5
        },
        trackInventory: {
            type: Boolean,
            default: true
        },
        allowBackorders: {
            type: Boolean,
            default: false
        }
    },
    shipping: {
        freeShipping: {
            type: Boolean,
            default: false
        },
        shippingWeight: Number,
        shippingClass: {
            type: String,
            enum: ['standard', 'expedited', 'overnight'],
            default: 'standard'
        }
    },
    seo: {
        metaTitle: {
            type: String,
            maxlength: [60, 'Meta title cannot exceed 60 characters']
        },
        metaDescription: {
            type: String,
            maxlength: [160, 'Meta description cannot exceed 160 characters']
        },
        keywords: [String]
    },
    rating: {
        average: {
            type: Number,
            default: 0,
            min: [0, 'Rating cannot be below 0'],
            max: [5, 'Rating cannot be above 5']
        },
        count: {
            type: Number,
            default: 0
        }
    },
    status: {
        type: String,
        enum: ['draft', 'published', 'archived'],
        default: 'draft'
    },
    featured: {
        type: Boolean,
        default: false
    },
    createdBy: {
        type: mongoose.Schema.ObjectId,
        ref: 'User',
        required: true
    }
}, {
    timestamps: true,
    toJSON: { virtuals: true },
    toObject: { virtuals: true }
});

// Virtual fields
productSchema.virtual('finalPrice').get(function() {
    return this.salePrice || this.price;
});

productSchema.virtual('isOnSale').get(function() {
    return !!this.salePrice;
});

productSchema.virtual('discountPercentage').get(function() {
    if (!this.salePrice) return 0;
    return Math.round(((this.price - this.salePrice) / this.price) * 100);
});

productSchema.virtual('isLowStock').get(function() {
    return this.inventory.stockQuantity <= this.inventory.lowStockThreshold;
});

productSchema.virtual('isInStock').get(function() {
    return this.inventory.stockQuantity > 0 || this.inventory.allowBackorders;
});

productSchema.virtual('primaryImage').get(function() {
    const primary = this.images.find(img => img.isPrimary);
    return primary || this.images[0];
});

// Indexes
productSchema.index({ name: 'text', description: 'text' });
productSchema.index({ category: 1 });
productSchema.index({ status: 1 });
productSchema.index({ featured: 1 });
productSchema.index({ price: 1 });
productSchema.index({ 'rating.average': -1 });
productSchema.index({ createdAt: -1 });
productSchema.index({ slug: 1 });

// Middleware
productSchema.pre('save', function(next) {
    if (this.isModified('name')) {
        this.slug = slugify(this.name, { lower: true, strict: true });
    }
    
    // Ensure only one primary image
    const primaryImages = this.images.filter(img => img.isPrimary);
    if (primaryImages.length > 1) {
        this.images.forEach((img, index) => {
            if (index > 0) img.isPrimary = false;
        });
    }
    
    next();
});

// Static methods
productSchema.statics.findPublished = function() {
    return this.find({ status: 'published' });
};

productSchema.statics.findFeatured = function() {
    return this.find({ status: 'published', featured: true });
};

productSchema.statics.findInStock = function() {
    return this.find({
        status: 'published',
        $or: [
            { 'inventory.stockQuantity': { $gt: 0 } },
            { 'inventory.allowBackorders': true }
        ]
    });
};

productSchema.statics.searchProducts = function(query, options = {}) {
    const {
        category,
        minPrice,
        maxPrice,
        inStock,
        featured,
        sort = { createdAt: -1 },
        limit = 20,
        skip = 0
    } = options;
    
    const searchCriteria = {
        status: 'published',
        ...(query && { $text: { $search: query } }),
        ...(category && { category }),
        ...(minPrice !== undefined && { price: { $gte: minPrice } }),
        ...(maxPrice !== undefined && { price: { ...searchCriteria.price, $lte: maxPrice } }),
        ...(featured && { featured: true }),
        ...(inStock && {
            $or: [
                { 'inventory.stockQuantity': { $gt: 0 } },
                { 'inventory.allowBackorders': true }
            ]
        })
    };
    
    return this.find(searchCriteria)
        .populate('category brand')
        .sort(sort)
        .limit(limit)
        .skip(skip);
};

module.exports = mongoose.model('Product', productSchema);
```

---

## 🎮 Controllers (Business Logic)

### User Controller Example
```javascript
// controllers/userController.js
const User = require('../models/User');
const userService = require('../services/userService');
const emailService = require('../services/emailService');
const { validationResult } = require('express-validator');

class UserController {
    // Get all users (Admin only)
    static async getUsers(req, res, next) {
        try {
            const {
                page = 1,
                limit = 10,
                sort = '-createdAt',
                status,
                role,
                search
            } = req.query;
            
            const options = {
                page: parseInt(page),
                limit: parseInt(limit),
                sort,
                populate: 'orders'
            };
            
            const filter = {};
            if (status) filter.status = status;
            if (role) filter.role = role;
            if (search) {
                filter.$or = [
                    { firstName: { $regex: search, $options: 'i' } },
                    { lastName: { $regex: search, $options: 'i' } },
                    { email: { $regex: search, $options: 'i' } }
                ];
            }
            
            const result = await User.paginate(filter, options);
            
            res.render('admin/users/index', {
                title: 'Users Management',
                users: result.docs,
                pagination: {
                    page: result.page,
                    pages: result.totalPages,
                    total: result.totalDocs,
                    limit: result.limit
                },
                filters: { status, role, search }
            });
            
        } catch (error) {
            next(error);
        }
    }
    
    // Get single user
    static async getUser(req, res, next) {
        try {
            const { id } = req.params;
            const user = await User.findById(id).populate('orders');
            
            if (!user) {
                return res.status(404).render('errors/404', {
                    title: 'User Not Found',
                    message: 'The requested user could not be found.'
                });
            }
            
            res.render('admin/users/show', {
                title: `User: ${user.fullName}`,
                user
            });
            
        } catch (error) {
            next(error);
        }
    }
    
    // Show user profile
    static async showProfile(req, res, next) {
        try {
            const user = await User.findById(req.session.userId);
            
            res.render('users/profile', {
                title: 'My Profile',
                user,
                activeTab: 'profile'
            });
            
        } catch (error) {
            next(error);
        }
    }
    
    // Show edit profile form
    static async editProfile(req, res, next) {
        try {
            const user = await User.findById(req.session.userId);
            
            res.render('users/edit-profile', {
                title: 'Edit Profile',
                user,
                activeTab: 'profile'
            });
            
        } catch (error) {
            next(error);
        }
    }
    
    // Update user profile
    static async updateProfile(req, res, next) {
        try {
            // Check for validation errors
            const errors = validationResult(req);
            if (!errors.isEmpty()) {
                const user = await User.findById(req.session.userId);
                return res.status(400).render('users/edit-profile', {
                    title: 'Edit Profile',
                    user,
                    errors: errors.array(),
                    oldInput: req.body
                });
            }
            
            const {
                firstName,
                lastName,
                email,
                bio,
                phoneNumber,
                dateOfBirth,
                address
            } = req.body;
            
            // Check if email is already taken by another user
            if (email !== req.user.email) {
                const existingUser = await User.findOne({ 
                    email, 
                    _id: { $ne: req.session.userId } 
                });
                
                if (existingUser) {
                    const user = await User.findById(req.session.userId);
                    return res.status(400).render('users/edit-profile', {
                        title: 'Edit Profile',
                        user,
                        errors: [{ msg: 'Email is already taken' }],
                        oldInput: req.body
                    });
                }
            }
            
            const updateData = {
                firstName,
                lastName,
                email,
                'profile.bio': bio,
                'profile.phoneNumber': phoneNumber,
                'profile.dateOfBirth': dateOfBirth,
                'profile.address': address
            };
            
            // Handle avatar upload if present
            if (req.file) {
                const avatarUrl = await userService.uploadAvatar(req.file, req.session.userId);
                updateData['profile.avatar'] = avatarUrl;
            }
            
            const user = await User.findByIdAndUpdate(
                req.session.userId,
                updateData,
                { new: true, runValidators: true }
            );
            
            req.flash('success', 'Profile updated successfully');
            res.redirect('/profile');
            
        } catch (error) {
            next(error);
        }
    }
    
    // Change password
    static async changePassword(req, res, next) {
        try {
            const errors = validationResult(req);
            if (!errors.isEmpty()) {
                return res.status(400).json({
                    success: false,
                    errors: errors.array()
                });
            }
            
            const { currentPassword, newPassword } = req.body;
            const user = await User.findById(req.session.userId).select('+password');
            
            // Verify current password
            const isValidPassword = await user.comparePassword(currentPassword);
            if (!isValidPassword) {
                return res.status(400).json({
                    success: false,
                    errors: [{ msg: 'Current password is incorrect' }]
                });
            }
            
            // Update password
            user.password = newPassword;
            await user.save();
            
            // Send email notification
            await emailService.sendPasswordChangeNotification(user.email, user.firstName);
            
            res.json({
                success: true,
                message: 'Password changed successfully'
            });
            
        } catch (error) {
            next(error);
        }
    }
    
    // Delete user account
    static async deleteAccount(req, res, next) {
        try {
            const { password } = req.body;
            const user = await User.findById(req.session.userId).select('+password');
            
            // Verify password before deletion
            const isValidPassword = await user.comparePassword(password);
            if (!isValidPassword) {
                return res.status(400).json({
                    success: false,
                    error: 'Invalid password'
                });
            }
            
            // Perform account deletion
            await userService.deleteUserAccount(req.session.userId);
            
            // Destroy session
            req.session.destroy((err) => {
                if (err) {
                    return next(err);
                }
                
                res.json({
                    success: true,
                    message: 'Account deleted successfully',
                    redirect: '/'
                });
            });
            
        } catch (error) {
            next(error);
        }
    }
    
    // Admin: Update user status
    static async updateUserStatus(req, res, next) {
        try {
            const { id } = req.params;
            const { status } = req.body;
            
            const user = await User.findByIdAndUpdate(
                id,
                { status },
                { new: true }
            );
            
            if (!user) {
                return res.status(404).json({
                    success: false,
                    error: 'User not found'
                });
            }
            
            // Send notification email
            if (status === 'suspended') {
                await emailService.sendAccountSuspensionNotification(user.email, user.firstName);
            }
            
            res.json({
                success: true,
                message: `User status updated to ${status}`,
                user
            });
            
        } catch (error) {
            next(error);
        }
    }
    
    // Admin: Get user statistics
    static async getUserStats(req, res, next) {
        try {
            const stats = await User.getUserStats();
            
            const recentUsers = await User.find()
                .sort({ createdAt: -1 })
                .limit(10)
                .select('firstName lastName email createdAt status');
            
            res.render('admin/users/stats', {
                title: 'User Statistics',
                stats,
                recentUsers
            });
            
        } catch (error) {
            next(error);
        }
    }
}

module.exports = UserController;
```

### Product Controller Example
```javascript
// controllers/productController.js
const Product = require('../models/Product');
const Category = require('../models/Category');
const Brand = require('../models/Brand');
const productService = require('../services/productService');
const { validationResult } = require('express-validator');

class ProductController {
    // Display products listing
    static async index(req, res, next) {
        try {
            const {
                page = 1,
                limit = 12,
                sort = '-createdAt',
                category,
                brand,
                minPrice,
                maxPrice,
                search,
                featured,
                inStock
            } = req.query;
            
            const options = {
                sort,
                limit: parseInt(limit),
                skip: (parseInt(page) - 1) * parseInt(limit)
            };
            
            const products = await Product.searchProducts(search, {
                category,
                brand,
                minPrice: minPrice ? parseFloat(minPrice) : undefined,
                maxPrice: maxPrice ? parseFloat(maxPrice) : undefined,
                featured: featured === 'true',
                inStock: inStock === 'true',
                ...options
            });
            
            const totalProducts = await Product.countDocuments({
                status: 'published',
                ...(category && { category }),
                ...(brand && { brand })
            });
            
            const totalPages = Math.ceil(totalProducts / parseInt(limit));
            
            // Get filter options
            const [categories, brands] = await Promise.all([
                Category.find({ status: 'active' }).sort('name'),
                Brand.find({ status: 'active' }).sort('name')
            ]);
            
            res.render('products/index', {
                title: 'Products',
                products,
                categories,
                brands,
                pagination: {
                    currentPage: parseInt(page),
                    totalPages,
                    totalProducts,
                    hasNextPage: page < totalPages,
                    hasPrevPage: page > 1,
                    nextPage: parseInt(page) + 1,
                    prevPage: parseInt(page) - 1
                },
                filters: {
                    category,
                    brand,
                    minPrice,
                    maxPrice,
                    search,
                    featured,
                    inStock,
                    sort
                }
            });
            
        } catch (error) {
            next(error);
        }
    }
    
    // Display single product
    static async show(req, res, next) {
        try {
            const { slug } = req.params;
            
            const product = await Product.findOne({ slug, status: 'published' })
                .populate('category brand')
                .populate({
                    path: 'reviews',
                    populate: {
                        path: 'user',
                        select: 'firstName lastName'
                    }
                });
            
            if (!product) {
                return res.status(404).render('errors/404', {
                    title: 'Product Not Found',
                    message: 'The product you are looking for could not be found.'
                });
            }
            
            // Get related products
            const relatedProducts = await Product.find({
                category: product.category,
                _id: { $ne: product._id },
                status: 'published'
            })
            .limit(4)
            .select('name slug price salePrice images rating');
            
            // Track product view
            await productService.trackProductView(product._id, req.ip);
            
            res.render('products/show', {
                title: product.name,
                product,
                relatedProducts,
                metaDescription: product.seo?.metaDescription || product.shortDescription
            });
            
        } catch (error) {
            next(error);
        }
    }
    
    // Admin: Show create product form
    static async create(req, res, next) {
        try {
            const [categories, brands] = await Promise.all([
                Category.find({ status: 'active' }).sort('name'),
                Brand.find({ status: 'active' }).sort('name')
            ]);
            
            res.render('admin/products/create', {
                title: 'Create Product',
                categories,
                brands
            });
            
        } catch (error) {
            next(error);
        }
    }
    
    // Admin: Store new product
    static async store(req, res, next) {
        try {
            const errors = validationResult(req);
            if (!errors.isEmpty()) {
                const [categories, brands] = await Promise.all([
                    Category.find({ status: 'active' }).sort('name'),
                    Brand.find({ status: 'active' }).sort('name')
                ]);
                
                return res.status(400).render('admin/products/create', {
                    title: 'Create Product',
                    categories,
                    brands,
                    errors: errors.array(),
                    oldInput: req.body
                });
            }
            
            const productData = {
                ...req.body,
                createdBy: req.session.userId
            };
            
            // Handle image uploads
            if (req.files && req.files.length > 0) {
                productData.images = await productService.uploadProductImages(req.files);
            }
            
            const product = await Product.create(productData);
            
            req.flash('success', 'Product created successfully');
            res.redirect(`/admin/products/${product._id}`);
            
        } catch (error) {
            next(error);
        }
    }
    
    // Admin: Show edit product form
    static async edit(req, res, next) {
        try {
            const { id } = req.params;
            
            const [product, categories, brands] = await Promise.all([
                Product.findById(id),
                Category.find({ status: 'active' }).sort('name'),
                Brand.find({ status: 'active' }).sort('name')
            ]);
            
            if (!product) {
                return res.status(404).render('errors/404', {
                    title: 'Product Not Found'
                });
            }
            
            res.render('admin/products/edit', {
                title: `Edit: ${product.name}`,
                product,
                categories,
                brands
            });
            
        } catch (error) {
            next(error);
        }
    }
    
    // Admin: Update product
    static async update(req, res, next) {
        try {
            const { id } = req.params;
            const errors = validationResult(req);
            
            if (!errors.isEmpty()) {
                const [product, categories, brands] = await Promise.all([
                    Product.findById(id),
                    Category.find({ status: 'active' }).sort('name'),
                    Brand.find({ status: 'active' }).sort('name')
                ]);
                
                return res.status(400).render('admin/products/edit', {
                    title: `Edit: ${product.name}`,
                    product,
                    categories,
                    brands,
                    errors: errors.array(),
                    oldInput: req.body
                });
            }
            
            const updateData = req.body;
            
            // Handle new image uploads
            if (req.files && req.files.length > 0) {
                const newImages = await productService.uploadProductImages(req.files);
                updateData.images = [...updateData.images || [], ...newImages];
            }
            
            const product = await Product.findByIdAndUpdate(
                id,
                updateData,
                { new: true, runValidators: true }
            );
            
            if (!product) {
                return res.status(404).render('errors/404', {
                    title: 'Product Not Found'
                });
            }
            
            req.flash('success', 'Product updated successfully');
            res.redirect(`/admin/products/${product._id}`);
            
        } catch (error) {
            next(error);
        }
    }
    
    // Admin: Delete product
    static async destroy(req, res, next) {
        try {
            const { id } = req.params;
            
            const product = await Product.findById(id);
            if (!product) {
                return res.status(404).json({
                    success: false,
                    error: 'Product not found'
                });
            }
            
            // Delete associated images
            await productService.deleteProductImages(product.images);
            
            // Delete product
            await Product.findByIdAndDelete(id);
            
            res.json({
                success: true,
                message: 'Product deleted successfully'
            });
            
        } catch (error) {
            next(error);
        }
    }
    
    // API: Get products (JSON response)
    static async getProductsAPI(req, res, next) {
        try {
            const {
                page = 1,
                limit = 10,
                sort = '-createdAt',
                search,
                category,
                featured
            } = req.query;
            
            const options = {
                sort,
                limit: parseInt(limit),
                skip: (parseInt(page) - 1) * parseInt(limit)
            };
            
            const products = await Product.searchProducts(search, {
                category,
                featured: featured === 'true',
                ...options
            });
            
            const totalProducts = await Product.countDocuments({
                status: 'published',
                ...(category && { category })
            });
            
            res.json({
                success: true,
                data: products,
                pagination: {
                    currentPage: parseInt(page),
                    totalPages: Math.ceil(totalProducts / parseInt(limit)),
                    totalProducts,
                    limit: parseInt(limit)
                }
            });
            
        } catch (error) {
            next(error);
        }
    }
}

module.exports = ProductController;
```

---

## 👁️ Views (Presentation Layer)

### Main Layout (EJS)
```html
<!-- views/layouts/main.ejs -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><%= title %> | My App</title>
    
    <!-- Meta tags -->
    <% if (typeof metaDescription !== 'undefined') { %>
        <meta name="description" content="<%= metaDescription %>">
    <% } %>
    
    <!-- CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <link href="/css/main.css" rel="stylesheet">
    
    <!-- Additional CSS -->
    <% if (typeof additionalCSS !== 'undefined') { %>
        <%- additionalCSS %>
    <% } %>
</head>
<body>
    <!-- Navigation -->
    <%- include('../partials/navbar') %>
    
    <!-- Flash Messages -->
    <%- include('../partials/flash') %>
    
    <!-- Main Content -->
    <main class="main-content">
        <%- body %>
    </main>
    
    <!-- Footer -->
    <%- include('../partials/footer') %>
    
    <!-- JavaScript -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="/js/main.js"></script>
    
    <!-- Additional JS -->
    <% if (typeof additionalJS !== 'undefined') { %>
        <%- additionalJS %>
    <% } %>
</body>
</html>
```

### Product Listing View
```html
<!-- views/products/index.ejs -->
<% layout('layouts/main') -%>

<div class="container mt-4">
    <!-- Page Header -->
    <div class="row mb-4">
        <div class="col-md-8">
            <h1 class="page-title">Products</h1>
            <p class="text-muted">
                Showing <%= pagination.totalProducts %> products
            </p>
        </div>
        <div class="col-md-4">
            <!-- Sort Options -->
            <select class="form-select" id="sortSelect" onchange="applySorting()">
                <option value="-createdAt" <%= filters.sort === '-createdAt' ? 'selected' : '' %>>
                    Newest First
                </option>
                <option value="price" <%= filters.sort === 'price' ? 'selected' : '' %>>
                    Price: Low to High
                </option>
                <option value="-price" <%= filters.sort === '-price' ? 'selected' : '' %>>
                    Price: High to Low
                </option>
                <option value="-rating.average" <%= filters.sort === '-rating.average' ? 'selected' : '' %>>
                    Highest Rated
                </option>
            </select>
        </div>
    </div>
    
    <div class="row">
        <!-- Filters Sidebar -->
        <div class="col-md-3">
            <div class="card">
                <div class="card-header">
                    <h5 class="mb-0">Filters</h5>
                </div>
                <div class="card-body">
                    <form id="filterForm" method="GET">
                        <!-- Search -->
                        <div class="mb-3">
                            <label for="search" class="form-label">Search</label>
                            <input type="text" class="form-control" id="search" name="search" 
                                   value="<%= filters.search || '' %>" placeholder="Search products...">
                        </div>
                        
                        <!-- Category Filter -->
                        <div class="mb-3">
                            <label for="category" class="form-label">Category</label>
                            <select class="form-select" id="category" name="category">
                                <option value="">All Categories</option>
                                <% categories.forEach(cat => { %>
                                    <option value="<%= cat._id %>" 
                                            <%= filters.category === cat._id.toString() ? 'selected' : '' %>>
                                        <%= cat.name %>
                                    </option>
                                <% }) %>
                            </select>
                        </div>
                        
                        <!-- Brand Filter -->
                        <div class="mb-3">
                            <label for="brand" class="form-label">Brand</label>
                            <select class="form-select" id="brand" name="brand">
                                <option value="">All Brands</option>
                                <% brands.forEach(brand => { %>
                                    <option value="<%= brand._id %>" 
                                            <%= filters.brand === brand._id.toString() ? 'selected' : '' %>>
                                        <%= brand.name %>
                                    </option>
                                <% }) %>
                            </select>
                        </div>
                        
                        <!-- Price Range -->
                        <div class="mb-3">
                            <label class="form-label">Price Range</label>
                            <div class="row">
                                <div class="col">
                                    <input type="number" class="form-control" name="minPrice" 
                                           placeholder="Min" value="<%= filters.minPrice || '' %>">
                                </div>
                                <div class="col">
                                    <input type="number" class="form-control" name="maxPrice" 
                                           placeholder="Max" value="<%= filters.maxPrice || '' %>">
                                </div>
                            </div>
                        </div>
                        
                        <!-- Checkboxes -->
                        <div class="mb-3">
                            <div class="form-check">
                                <input class="form-check-input" type="checkbox" id="featured" 
                                       name="featured" value="true" <%= filters.featured ? 'checked' : '' %>>
                                <label class="form-check-label" for="featured">
                                    Featured Only
                                </label>
                            </div>
                            <div class="form-check">
                                <input class="form-check-input" type="checkbox" id="inStock" 
                                       name="inStock" value="true" <%= filters.inStock ? 'checked' : '' %>>
                                <label class="form-check-label" for="inStock">
                                    In Stock Only
                                </label>
                            </div>
                        </div>
                        
                        <button type="submit" class="btn btn-primary w-100">Apply Filters</button>
                        <a href="/products" class="btn btn-outline-secondary w-100 mt-2">Clear Filters</a>
                    </form>
                </div>
            </div>
        </div>
        
        <!-- Products Grid -->
        <div class="col-md-9">
            <% if (products.length === 0) { %>
                <div class="text-center py-5">
                    <i class="fas fa-search fa-3x text-muted mb-3"></i>
                    <h4>No products found</h4>
                    <p class="text-muted">Try adjusting your filters or search terms.</p>
                </div>
            <% } else { %>
                <div class="row">
                    <% products.forEach(product => { %>
                        <div class="col-md-4 col-sm-6 mb-4">
                            <%- include('../partials/product-card', { product }) %>
                        </div>
                    <% }) %>
                </div>
                
                <!-- Pagination -->
                <% if (pagination.totalPages > 1) { %>
                    <%- include('../partials/pagination', { pagination, currentUrl: '/products' }) %>
                <% } %>
            <% } %>
        </div>
    </div>
</div>

<script>
function applySorting() {
    const sortValue = document.getElementById('sortSelect').value;
    const url = new URL(window.location);
    url.searchParams.set('sort', sortValue);
    url.searchParams.set('page', '1'); // Reset to first page
    window.location = url.toString();
}
</script>
```

### Product Card Partial
```html
<!-- views/partials/product-card.ejs -->
<div class="card product-card h-100">
    <div class="position-relative">
        <% if (product.primaryImage) { %>
            <img src="<%= product.primaryImage.url %>" 
                 class="card-img-top" 
                 alt="<%= product.primaryImage.alt || product.name %>"
                 loading="lazy">
        <% } else { %>
            <div class="card-img-top bg-light d-flex align-items-center justify-content-center" 
                 style="height: 200px;">
                <i class="fas fa-image fa-3x text-muted"></i>
            </div>
        <% } %>
        
        <!-- Badges -->
        <% if (product.featured) { %>
            <span class="badge bg-warning position-absolute top-0 start-0 m-2">
                Featured
            </span>
        <% } %>
        
        <% if (product.isOnSale) { %>
            <span class="badge bg-danger position-absolute top-0 end-0 m-2">
                -<%= product.discountPercentage %>%
            </span>
        <% } %>
        
        <% if (!product.isInStock) { %>
            <span class="badge bg-secondary position-absolute top-0 start-50 translate-middle-x m-2">
                Out of Stock
            </span>
        <% } %>
    </div>
    
    <div class="card-body d-flex flex-column">
        <h5 class="card-title">
            <a href="/products/<%= product.slug %>" class="text-decoration-none">
                <%= product.name %>
            </a>
        </h5>
        
        <p class="card-text text-muted small flex-grow-1">
            <%= product.shortDescription || product.description.substring(0, 100) + '...' %>
        </p>
        
        <!-- Rating -->
        <% if (product.rating.count > 0) { %>
            <div class="mb-2">
                <% for (let i = 1; i <= 5; i++) { %>
                    <i class="fas fa-star <%= i <= product.rating.average ? 'text-warning' : 'text-muted' %>"></i>
                <% } %>
                <small class="text-muted">(<%= product.rating.count %>)</small>
            </div>
        <% } %>
        
        <!-- Price -->
        <div class="mb-3">
            <% if (product.isOnSale) { %>
                <span class="h5 text-danger">$<%= product.salePrice.toFixed(2) %></span>
                <span class="text-muted text-decoration-line-through ms-2">
                    $<%= product.price.toFixed(2) %>
                </span>
            <% } else { %>
                <span class="h5">$<%= product.price.toFixed(2) %></span>
            <% } %>
        </div>
        
        <!-- Actions -->
        <div class="mt-auto">
            <% if (product.isInStock) { %>
                <button class="btn btn-primary w-100" 
                        onclick="addToCart('<%= product._id %>')">
                    <i class="fas fa-cart-plus me-2"></i>Add to Cart
                </button>
            <% } else { %>
                <button class="btn btn-secondary w-100" disabled>
                    Out of Stock
                </button>
            <% } %>
        </div>
    </div>
</div>
```

---

## 🚦 Routes (URL Mapping)

### Main Router Setup
```javascript
// routes/index.js
const express = require('express');
const router = express.Router();

// Import route modules
const authRoutes = require('./authRoutes');
const userRoutes = require('./userRoutes');
const productRoutes = require('./productRoutes');
const adminRoutes = require('./adminRoutes');
const apiRoutes = require('./apiRoutes');

// Middleware
const { requireAuth, optionalAuth } = require('../middleware/auth');

// Public routes
router.use('/auth', authRoutes);
router.use('/api', apiRoutes);

// Routes with optional authentication
router.use('/', optionalAuth, productRoutes);

// Protected routes
router.use('/profile', requireAuth, userRoutes);
router.use('/admin', requireAuth, adminRoutes);

// Home page
router.get('/', optionalAuth, async (req, res, next) => {
    try {
        const Product = require('../models/Product');
        
        const [featuredProducts, newProducts] = await Promise.all([
            Product.findFeatured().limit(8),
            Product.findPublished().sort({ createdAt: -1 }).limit(8)
        ]);
        
        res.render('home', {
            title: 'Welcome',
            featuredProducts,
            newProducts
        });
    } catch (error) {
        next(error);
    }
});

module.exports = router;
```

### Product Routes
```javascript
// routes/productRoutes.js
const express = require('express');
const router = express.Router();
const ProductController = require('../controllers/productController');

// Product listing and search
router.get('/products', ProductController.index);

// Single product view
router.get('/products/:slug', ProductController.show);

module.exports = router;
```

---

## 🔗 Related Topics
- [[19-EJS-Templating]] - View templating with EJS
- [[21-Middlewares]] - Express middleware
- [[28-PostgreSQL-Integration]] - Database integration
- [[30-Error-Handling]] - Error handling patterns

---

*Back to [[00-Main-Index]]*
