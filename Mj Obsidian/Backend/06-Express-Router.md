# Express Router

**Navigation**: [[05-Express-Framework]] | [[00-Main-Index]] | Next: [[07-Serving-Static-Files]]

---

## 🛤️ Express Router

Express Router is a **mini express application** that provides routing functionality. It helps organize routes into separate modules for better code structure.

---

## 🎯 Basic Router Setup

### Creating a Router Module
```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

// Middleware specific to this router
router.use((req, res, next) => {
    console.log('Users router middleware');
    console.log('Time:', Date.now());
    next();
});

// Routes
router.get('/', (req, res) => {
    res.json({ message: 'Users homepage' });
});

router.get('/:id', (req, res) => {
    res.json({ userId: req.params.id });
});

router.post('/', (req, res) => {
    res.json({ message: 'User created', data: req.body });
});

module.exports = router;
```

### Using Router in Main App
```javascript
// app.js
const express = require('express');
const userRoutes = require('./routes/users');

const app = express();

app.use(express.json());

// Mount the router
app.use('/users', userRoutes);

app.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

---

## 🏗️ Modular Route Organization

### File Structure
```
project/
├── app.js
├── routes/
│   ├── index.js
│   ├── users.js
│   ├── posts.js
│   ├── auth.js
│   └── admin.js
├── controllers/
│   ├── userController.js
│   ├── postController.js
│   └── authController.js
└── middleware/
    ├── auth.js
    ├── validation.js
    └── logger.js
```

### Main Routes Index
```javascript
// routes/index.js
const express = require('express');
const userRoutes = require('./users');
const postRoutes = require('./posts');
const authRoutes = require('./auth');
const adminRoutes = require('./admin');

const router = express.Router();

// Health check
router.get('/health', (req, res) => {
    res.json({ 
        status: 'OK', 
        timestamp: new Date().toISOString() 
    });
});

// API routes
router.use('/users', userRoutes);
router.use('/posts', postRoutes);
router.use('/auth', authRoutes);
router.use('/admin', adminRoutes);

module.exports = router;
```

### User Routes with Controller
```javascript
// routes/users.js
const express = require('express');
const userController = require('../controllers/userController');
const auth = require('../middleware/auth');
const validation = require('../middleware/validation');

const router = express.Router();

// Public routes
router.post('/register', validation.validateUser, userController.register);
router.post('/login', validation.validateLogin, userController.login);

// Protected routes (require authentication)
router.use(auth.authenticate); // Apply to all routes below

router.get('/', userController.getAllUsers);
router.get('/profile', userController.getProfile);
router.get('/:id', validation.validateUserId, userController.getUserById);
router.put('/:id', validation.validateUserId, validation.validateUserUpdate, userController.updateUser);
router.delete('/:id', validation.validateUserId, userController.deleteUser);

// Admin only routes
router.get('/admin/stats', auth.requireAdmin, userController.getUserStats);

module.exports = router;
```

---

## 🎮 Controller Pattern

### User Controller
```javascript
// controllers/userController.js
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');

// Mock database
let users = [
    { id: 1, name: 'John Doe', email: 'john@example.com', role: 'user' },
    { id: 2, name: 'Admin User', email: 'admin@example.com', role: 'admin' }
];
let nextId = 3;

const userController = {
    // GET /users
    getAllUsers: (req, res) => {
        try {
            const page = parseInt(req.query.page) || 1;
            const limit = parseInt(req.query.limit) || 10;
            const search = req.query.search;
            
            let filteredUsers = users;
            
            // Search functionality
            if (search) {
                filteredUsers = users.filter(user => 
                    user.name.toLowerCase().includes(search.toLowerCase()) ||
                    user.email.toLowerCase().includes(search.toLowerCase())
                );
            }
            
            // Pagination
            const startIndex = (page - 1) * limit;
            const endIndex = page * limit;
            const paginatedUsers = filteredUsers.slice(startIndex, endIndex);
            
            res.json({
                users: paginatedUsers.map(user => ({
                    id: user.id,
                    name: user.name,
                    email: user.email,
                    role: user.role
                })),
                pagination: {
                    page,
                    limit,
                    total: filteredUsers.length,
                    pages: Math.ceil(filteredUsers.length / limit)
                }
            });
        } catch (error) {
            res.status(500).json({ error: 'Internal server error' });
        }
    },
    
    // GET /users/:id
    getUserById: (req, res) => {
        try {
            const id = parseInt(req.params.id);
            const user = users.find(u => u.id === id);
            
            if (!user) {
                return res.status(404).json({ error: 'User not found' });
            }
            
            res.json({
                user: {
                    id: user.id,
                    name: user.name,
                    email: user.email,
                    role: user.role
                }
            });
        } catch (error) {
            res.status(500).json({ error: 'Internal server error' });
        }
    },
    
    // GET /users/profile
    getProfile: (req, res) => {
        try {
            const user = req.user; // From auth middleware
            res.json({
                user: {
                    id: user.id,
                    name: user.name,
                    email: user.email,
                    role: user.role
                }
            });
        } catch (error) {
            res.status(500).json({ error: 'Internal server error' });
        }
    },
    
    // POST /users/register
    register: async (req, res) => {
        try {
            const { name, email, password } = req.body;
            
            // Check if user exists
            const existingUser = users.find(u => u.email === email);
            if (existingUser) {
                return res.status(409).json({ error: 'Email already exists' });
            }
            
            // Hash password
            const hashedPassword = await bcrypt.hash(password, 10);
            
            // Create user
            const newUser = {
                id: nextId++,
                name,
                email,
                password: hashedPassword,
                role: 'user',
                createdAt: new Date().toISOString()
            };
            
            users.push(newUser);
            
            // Generate token
            const token = jwt.sign(
                { id: newUser.id, email: newUser.email },
                process.env.JWT_SECRET || 'secret',
                { expiresIn: '7d' }
            );
            
            res.status(201).json({
                message: 'User registered successfully',
                user: {
                    id: newUser.id,
                    name: newUser.name,
                    email: newUser.email,
                    role: newUser.role
                },
                token
            });
        } catch (error) {
            res.status(500).json({ error: 'Internal server error' });
        }
    },
    
    // POST /users/login
    login: async (req, res) => {
        try {
            const { email, password } = req.body;
            
            // Find user
            const user = users.find(u => u.email === email);
            if (!user) {
                return res.status(401).json({ error: 'Invalid credentials' });
            }
            
            // Check password
            const isValidPassword = await bcrypt.compare(password, user.password);
            if (!isValidPassword) {
                return res.status(401).json({ error: 'Invalid credentials' });
            }
            
            // Generate token
            const token = jwt.sign(
                { id: user.id, email: user.email },
                process.env.JWT_SECRET || 'secret',
                { expiresIn: '7d' }
            );
            
            res.json({
                message: 'Login successful',
                user: {
                    id: user.id,
                    name: user.name,
                    email: user.email,
                    role: user.role
                },
                token
            });
        } catch (error) {
            res.status(500).json({ error: 'Internal server error' });
        }
    },
    
    // PUT /users/:id
    updateUser: (req, res) => {
        try {
            const id = parseInt(req.params.id);
            const userIndex = users.findIndex(u => u.id === id);
            
            if (userIndex === -1) {
                return res.status(404).json({ error: 'User not found' });
            }
            
            const { name, email } = req.body;
            
            // Check if email exists for another user
            const emailExists = users.find(u => u.email === email && u.id !== id);
            if (emailExists) {
                return res.status(409).json({ error: 'Email already exists' });
            }
            
            users[userIndex] = {
                ...users[userIndex],
                name,
                email,
                updatedAt: new Date().toISOString()
            };
            
            res.json({
                message: 'User updated successfully',
                user: {
                    id: users[userIndex].id,
                    name: users[userIndex].name,
                    email: users[userIndex].email,
                    role: users[userIndex].role
                }
            });
        } catch (error) {
            res.status(500).json({ error: 'Internal server error' });
        }
    },
    
    // DELETE /users/:id
    deleteUser: (req, res) => {
        try {
            const id = parseInt(req.params.id);
            const userIndex = users.findIndex(u => u.id === id);
            
            if (userIndex === -1) {
                return res.status(404).json({ error: 'User not found' });
            }
            
            const deletedUser = users.splice(userIndex, 1)[0];
            
            res.json({
                message: 'User deleted successfully',
                user: {
                    id: deletedUser.id,
                    name: deletedUser.name,
                    email: deletedUser.email
                }
            });
        } catch (error) {
            res.status(500).json({ error: 'Internal server error' });
        }
    },
    
    // GET /users/admin/stats
    getUserStats: (req, res) => {
        try {
            const stats = {
                totalUsers: users.length,
                adminUsers: users.filter(u => u.role === 'admin').length,
                regularUsers: users.filter(u => u.role === 'user').length,
                recentUsers: users.filter(u => {
                    if (!u.createdAt) return false;
                    const userDate = new Date(u.createdAt);
                    const weekAgo = new Date(Date.now() - 7 * 24 * 60 * 60 * 1000);
                    return userDate > weekAgo;
                }).length
            };
            
            res.json({ stats });
        } catch (error) {
            res.status(500).json({ error: 'Internal server error' });
        }
    }
};

module.exports = userController;
```

---

## 🔒 Middleware for Routes

### Authentication Middleware
```javascript
// middleware/auth.js
const jwt = require('jsonwebtoken');

const authMiddleware = {
    authenticate: (req, res, next) => {
        try {
            const token = req.header('Authorization')?.replace('Bearer ', '');
            
            if (!token) {
                return res.status(401).json({ error: 'Access denied. No token provided.' });
            }
            
            const decoded = jwt.verify(token, process.env.JWT_SECRET || 'secret');
            
            // Find user (in real app, this would be a database query)
            const users = require('../data/users'); // Mock data
            const user = users.find(u => u.id === decoded.id);
            
            if (!user) {
                return res.status(401).json({ error: 'Invalid token.' });
            }
            
            req.user = user;
            next();
        } catch (error) {
            res.status(401).json({ error: 'Invalid token.' });
        }
    },
    
    requireAdmin: (req, res, next) => {
        if (req.user.role !== 'admin') {
            return res.status(403).json({ error: 'Access denied. Admin role required.' });
        }
        next();
    },
    
    requireOwnershipOrAdmin: (req, res, next) => {
        const resourceUserId = parseInt(req.params.id);
        const currentUserId = req.user.id;
        const isAdmin = req.user.role === 'admin';
        
        if (resourceUserId !== currentUserId && !isAdmin) {
            return res.status(403).json({ error: 'Access denied.' });
        }
        
        next();
    }
};

module.exports = authMiddleware;
```

### Validation Middleware
```javascript
// middleware/validation.js
const validationMiddleware = {
    validateUser: (req, res, next) => {
        const { name, email, password } = req.body;
        const errors = [];
        
        if (!name || name.trim().length < 2) {
            errors.push('Name must be at least 2 characters long');
        }
        
        if (!email || !email.includes('@')) {
            errors.push('Valid email is required');
        }
        
        if (!password || password.length < 6) {
            errors.push('Password must be at least 6 characters long');
        }
        
        if (errors.length > 0) {
            return res.status(400).json({ errors });
        }
        
        next();
    },
    
    validateLogin: (req, res, next) => {
        const { email, password } = req.body;
        const errors = [];
        
        if (!email) {
            errors.push('Email is required');
        }
        
        if (!password) {
            errors.push('Password is required');
        }
        
        if (errors.length > 0) {
            return res.status(400).json({ errors });
        }
        
        next();
    },
    
    validateUserId: (req, res, next) => {
        const id = parseInt(req.params.id);
        
        if (isNaN(id) || id <= 0) {
            return res.status(400).json({ error: 'Invalid user ID' });
        }
        
        next();
    },
    
    validateUserUpdate: (req, res, next) => {
        const { name, email } = req.body;
        const errors = [];
        
        if (name && name.trim().length < 2) {
            errors.push('Name must be at least 2 characters long');
        }
        
        if (email && !email.includes('@')) {
            errors.push('Valid email is required');
        }
        
        if (errors.length > 0) {
            return res.status(400).json({ errors });
        }
        
        next();
    }
};

module.exports = validationMiddleware;
```

---

## 🔄 Nested Routers

### Posts with Comments Router
```javascript
// routes/posts.js
const express = require('express');
const commentsRouter = require('./comments');

const router = express.Router();

// Post routes
router.get('/', (req, res) => {
    res.json({ message: 'All posts' });
});

router.get('/:id', (req, res) => {
    res.json({ postId: req.params.id });
});

// Nested router for comments
// This creates routes like: /posts/:postId/comments
router.use('/:postId/comments', (req, res, next) => {
    // Add postId to request object for nested router
    req.postId = req.params.postId;
    next();
}, commentsRouter);

module.exports = router;
```

### Comments Router
```javascript
// routes/comments.js
const express = require('express');
const router = express.Router({ mergeParams: true });

// GET /posts/:postId/comments
router.get('/', (req, res) => {
    const postId = req.params.postId;
    res.json({ 
        message: `Comments for post ${postId}`,
        postId: postId
    });
});

// GET /posts/:postId/comments/:commentId
router.get('/:commentId', (req, res) => {
    const { postId, commentId } = req.params;
    res.json({ 
        message: `Comment ${commentId} from post ${postId}`,
        postId: postId,
        commentId: commentId
    });
});

// POST /posts/:postId/comments
router.post('/', (req, res) => {
    const postId = req.params.postId;
    res.json({ 
        message: `New comment created for post ${postId}`,
        postId: postId,
        comment: req.body
    });
});

module.exports = router;
```

---

## 📊 Advanced Router Patterns

### Route Parameters and RegEx
```javascript
const express = require('express');
const router = express.Router();

// Route with regex pattern
router.get('/users/:id(\\d+)', (req, res) => {
    // Only matches numeric IDs
    res.json({ userId: req.params.id });
});

// Route with multiple patterns
router.get('/posts/:year(\\d{4})/:month(\\d{2})?', (req, res) => {
    const { year, month } = req.params;
    res.json({ year, month: month || 'all' });
});

// Route with custom parameter
router.param('id', (req, res, next, id) => {
    // This runs for any route with :id parameter
    console.log(`User ID: ${id}`);
    
    // You can add validation here
    if (isNaN(id)) {
        return res.status(400).json({ error: 'Invalid ID format' });
    }
    
    // Add to request object
    req.userId = parseInt(id);
    next();
});

router.get('/users/:id', (req, res) => {
    // req.userId is available from param middleware
    res.json({ userId: req.userId });
});

module.exports = router;
```

### Router with Error Handling
```javascript
const express = require('express');
const router = express.Router();

// Async wrapper for error handling
const asyncHandler = (fn) => (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
};

// Routes with async error handling
router.get('/users', asyncHandler(async (req, res) => {
    // Simulate async operation
    const users = await getUsersFromDatabase();
    res.json({ users });
}));

router.post('/users', asyncHandler(async (req, res) => {
    const user = await createUser(req.body);
    res.status(201).json({ user });
}));

// Error handling middleware for this router
router.use((err, req, res, next) => {
    console.error('Router error:', err);
    
    if (err.name === 'ValidationError') {
        return res.status(400).json({ error: err.message });
    }
    
    if (err.name === 'NotFoundError') {
        return res.status(404).json({ error: err.message });
    }
    
    // Pass to global error handler
    next(err);
});

module.exports = router;
```

---

## 🔗 Related Topics
- [[05-Express-Framework]] - Basic Express concepts
- [[20-Middlewares]] - Advanced middleware patterns
- [[19-MVC-Pattern]] - Organizing with MVC architecture
- [[11-Authentication-and-Authorization]] - Securing routes

---

*Back to [[00-Main-Index]]*
