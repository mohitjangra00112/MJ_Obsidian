# EJS Templating

**Navigation**: [[18-MVC-Architecture]] | [[00-Main-Index]] | Next: [[20-Pagination]]

---

## 🎨 EJS (Embedded JavaScript) Templating

**EJS** is a simple templating engine that lets you **generate HTML markup** with plain JavaScript. It's one of the most popular templating engines for Node.js applications.

---

## 📦 Installation and Setup

### Basic Installation
```bash
npm install ejs express-ejs-layouts
```

### Express Setup with EJS
```javascript
// app.js
const express = require('express');
const path = require('path');
const expressLayouts = require('express-ejs-layouts');

const app = express();

// Set EJS as the view engine
app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// Use express-ejs-layouts
app.use(expressLayouts);
app.set('layout', 'layouts/main'); // Default layout

// Serve static files
app.use(express.static(path.join(__dirname, 'public')));

// Body parsing middleware
app.use(express.urlencoded({ extended: true }));
app.use(express.json());

// Sample routes
app.get('/', (req, res) => {
    res.render('home', {
        title: 'Home Page',
        message: 'Welcome to our website!',
        user: req.user || null
    });
});

app.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

---

## 📂 Project Structure for EJS

### Recommended Directory Structure
```
views/
├── layouts/
│   ├── main.ejs              # Main layout
│   ├── admin.ejs             # Admin layout
│   └── auth.ejs              # Authentication layout
├── partials/
│   ├── header.ejs            # Header partial
│   ├── footer.ejs            # Footer partial
│   ├── navbar.ejs            # Navigation partial
│   ├── sidebar.ejs           # Sidebar partial
│   ├── flash.ejs             # Flash messages
│   └── pagination.ejs        # Pagination component
├── pages/
│   ├── home.ejs              # Home page
│   ├── about.ejs             # About page
│   └── contact.ejs           # Contact page
├── users/
│   ├── profile.ejs           # User profile
│   ├── edit-profile.ejs      # Edit profile
│   ├── login.ejs             # Login page
│   └── register.ejs          # Registration page
├── products/
│   ├── index.ejs             # Product listing
│   ├── show.ejs              # Single product
│   ├── create.ejs            # Create product
│   └── edit.ejs              # Edit product
├── admin/
│   ├── dashboard.ejs         # Admin dashboard
│   ├── users.ejs             # User management
│   └── products.ejs          # Product management
└── errors/
    ├── 404.ejs               # Not found page
    ├── 500.ejs               # Server error page
    └── error.ejs             # Generic error page
```

---

## 🔧 EJS Syntax and Features

### Basic EJS Tags
```html
<!-- views/examples/syntax.ejs -->
<!DOCTYPE html>
<html>
<head>
    <title>EJS Syntax Examples</title>
</head>
<body>
    <!-- 1. Output (escaped) -->
    <h1><%= title %></h1>
    <p>Welcome, <%= user.name %>!</p>
    
    <!-- 2. Output (unescaped - dangerous, use carefully) -->
    <div><%- htmlContent %></div>
    
    <!-- 3. JavaScript code (no output) -->
    <% 
        const currentYear = new Date().getFullYear();
        const isLoggedIn = user && user.id;
    %>
    
    <!-- 4. Conditional rendering -->
    <% if (isLoggedIn) { %>
        <p>Hello, <%= user.name %>!</p>
        <a href="/logout">Logout</a>
    <% } else { %>
        <p>Please log in</p>
        <a href="/login">Login</a>
    <% } %>
    
    <!-- 5. Loops -->
    <% if (products && products.length > 0) { %>
        <ul>
        <% products.forEach(product => { %>
            <li>
                <strong><%= product.name %></strong> - $<%= product.price %>
                <% if (product.onSale) { %>
                    <span class="sale-badge">ON SALE!</span>
                <% } %>
            </li>
        <% }) %>
        </ul>
    <% } else { %>
        <p>No products available</p>
    <% } %>
    
    <!-- 6. Include partials -->
    <%- include('partials/header') %>
    <%- include('partials/product-card', { product: featuredProduct }) %>
    
    <!-- 7. Comments -->
    <%# This is a comment and won't be rendered %>
    
    <!-- 8. Literal template strings -->
    <%% This will output: <% literally %>
    
    <footer>
        <p>&copy; <%= currentYear %> My Company</p>
    </footer>
</body>
</html>
```

### Advanced EJS Features
```html
<!-- views/examples/advanced.ejs -->
<!DOCTYPE html>
<html>
<head>
    <title><%= title || 'Default Title' %></title>
</head>
<body>
    <!-- Ternary operators -->
    <div class="<%= user.isAdmin ? 'admin-panel' : 'user-panel' %>">
        <h1><%= user.isAdmin ? 'Admin Dashboard' : 'User Dashboard' %></h1>
    </div>
    
    <!-- Array methods -->
    <% const activeUsers = users.filter(u => u.status === 'active') %>
    <% const userCount = activeUsers.length %>
    
    <p>Active Users: <%= userCount %></p>
    
    <!-- Object destructuring -->
    <% const { name, email, role } = user %>
    <div class="user-info">
        <h2><%= name %></h2>
        <p>Email: <%= email %></p>
        <p>Role: <%= role %></p>
    </div>
    
    <!-- Switch-like conditional -->
    <div class="status-badge">
        <% if (order.status === 'pending') { %>
            <span class="badge badge-warning">Pending</span>
        <% } else if (order.status === 'processing') { %>
            <span class="badge badge-info">Processing</span>
        <% } else if (order.status === 'shipped') { %>
            <span class="badge badge-primary">Shipped</span>
        <% } else if (order.status === 'delivered') { %>
            <span class="badge badge-success">Delivered</span>
        <% } else { %>
            <span class="badge badge-danger">Unknown</span>
        <% } %>
    </div>
    
    <!-- Nested loops with conditions -->
    <% categories.forEach(category => { %>
        <div class="category">
            <h3><%= category.name %></h3>
            <% const categoryProducts = products.filter(p => p.categoryId === category.id) %>
            <% if (categoryProducts.length > 0) { %>
                <div class="products-grid">
                    <% categoryProducts.forEach(product => { %>
                        <div class="product-card">
                            <h4><%= product.name %></h4>
                            <p>$<%= product.price.toFixed(2) %></p>
                        </div>
                    <% }) %>
                </div>
            <% } else { %>
                <p>No products in this category</p>
            <% } %>
        </div>
    <% }) %>
    
    <!-- Using helper functions -->
    <% 
    function formatDate(date) {
        return new Date(date).toLocaleDateString();
    }
    
    function truncateText(text, maxLength) {
        return text.length > maxLength ? text.substring(0, maxLength) + '...' : text;
    }
    %>
    
    <div class="article">
        <h3><%= article.title %></h3>
        <p class="meta">Published on <%= formatDate(article.createdAt) %></p>
        <p><%= truncateText(article.content, 150) %></p>
    </div>
</body>
</html>
```

---

## 🧩 Layouts and Partials

### Main Layout Template
```html
<!-- views/layouts/main.ejs -->
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><%= title %> | <%= siteName || 'My Website' %></title>
    
    <!-- Meta Tags -->
    <% if (typeof metaDescription !== 'undefined') { %>
        <meta name="description" content="<%= metaDescription %>">
    <% } %>
    <% if (typeof metaKeywords !== 'undefined') { %>
        <meta name="keywords" content="<%= metaKeywords %>">
    <% } %>
    
    <!-- CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <link href="/css/main.css" rel="stylesheet">
    
    <!-- Page-specific CSS -->
    <% if (typeof pageCSS !== 'undefined') { %>
        <%- pageCSS %>
    <% } %>
</head>
<body class="<%= bodyClass || '' %>">
    <!-- Navigation -->
    <%- include('../partials/navbar', { 
        user: typeof user !== 'undefined' ? user : null,
        currentPage: typeof currentPage !== 'undefined' ? currentPage : ''
    }) %>
    
    <!-- Flash Messages -->
    <%- include('../partials/flash', {
        success: typeof success !== 'undefined' ? success : null,
        error: typeof error !== 'undefined' ? error : null,
        warning: typeof warning !== 'undefined' ? warning : null,
        info: typeof info !== 'undefined' ? info : null
    }) %>
    
    <!-- Main Content -->
    <main class="main-content">
        <%- body %>
    </main>
    
    <!-- Footer -->
    <%- include('../partials/footer') %>
    
    <!-- JavaScript -->
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.1.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="/js/main.js"></script>
    
    <!-- Page-specific JavaScript -->
    <% if (typeof pageJS !== 'undefined') { %>
        <%- pageJS %>
    <% } %>
</body>
</html>
```

### Navigation Partial
```html
<!-- views/partials/navbar.ejs -->
<nav class="navbar navbar-expand-lg navbar-dark bg-dark">
    <div class="container">
        <a class="navbar-brand" href="/">
            <i class="fas fa-store me-2"></i>My Store
        </a>
        
        <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav">
            <span class="navbar-toggler-icon"></span>
        </button>
        
        <div class="collapse navbar-collapse" id="navbarNav">
            <ul class="navbar-nav me-auto">
                <li class="nav-item">
                    <a class="nav-link <%= currentPage === 'home' ? 'active' : '' %>" href="/">
                        <i class="fas fa-home me-1"></i>Home
                    </a>
                </li>
                <li class="nav-item">
                    <a class="nav-link <%= currentPage === 'products' ? 'active' : '' %>" href="/products">
                        <i class="fas fa-box me-1"></i>Products
                    </a>
                </li>
                <li class="nav-item">
                    <a class="nav-link <%= currentPage === 'about' ? 'active' : '' %>" href="/about">
                        <i class="fas fa-info-circle me-1"></i>About
                    </a>
                </li>
                <li class="nav-item">
                    <a class="nav-link <%= currentPage === 'contact' ? 'active' : '' %>" href="/contact">
                        <i class="fas fa-envelope me-1"></i>Contact
                    </a>
                </li>
            </ul>
            
            <ul class="navbar-nav">
                <% if (user) { %>
                    <!-- Cart Icon -->
                    <li class="nav-item">
                        <a class="nav-link position-relative" href="/cart">
                            <i class="fas fa-shopping-cart"></i>
                            <span class="position-absolute top-0 start-100 translate-middle badge rounded-pill bg-danger" id="cartCount">
                                <%= typeof cartCount !== 'undefined' ? cartCount : 0 %>
                            </span>
                        </a>
                    </li>
                    
                    <!-- User Dropdown -->
                    <li class="nav-item dropdown">
                        <a class="nav-link dropdown-toggle" href="#" id="userDropdown" role="button" data-bs-toggle="dropdown">
                            <% if (user.avatar) { %>
                                <img src="<%= user.avatar %>" alt="Avatar" class="rounded-circle me-1" width="24" height="24">
                            <% } else { %>
                                <i class="fas fa-user-circle me-1"></i>
                            <% } %>
                            <%= user.firstName %>
                        </a>
                        <ul class="dropdown-menu">
                            <li><a class="dropdown-item" href="/profile">
                                <i class="fas fa-user me-2"></i>Profile
                            </a></li>
                            <li><a class="dropdown-item" href="/orders">
                                <i class="fas fa-shopping-bag me-2"></i>My Orders
                            </a></li>
                            <li><a class="dropdown-item" href="/settings">
                                <i class="fas fa-cog me-2"></i>Settings
                            </a></li>
                            <% if (user.role === 'admin') { %>
                                <li><hr class="dropdown-divider"></li>
                                <li><a class="dropdown-item" href="/admin">
                                    <i class="fas fa-shield-alt me-2"></i>Admin Panel
                                </a></li>
                            <% } %>
                            <li><hr class="dropdown-divider"></li>
                            <li><a class="dropdown-item" href="/logout">
                                <i class="fas fa-sign-out-alt me-2"></i>Logout
                            </a></li>
                        </ul>
                    </li>
                <% } else { %>
                    <!-- Guest User -->
                    <li class="nav-item">
                        <a class="nav-link" href="/login">
                            <i class="fas fa-sign-in-alt me-1"></i>Login
                        </a>
                    </li>
                    <li class="nav-item">
                        <a class="nav-link" href="/register">
                            <i class="fas fa-user-plus me-1"></i>Register
                        </a>
                    </li>
                <% } %>
            </ul>
        </div>
    </div>
</nav>
```

### Flash Messages Partial
```html
<!-- views/partials/flash.ejs -->
<% if (success) { %>
    <div class="alert alert-success alert-dismissible fade show m-3" role="alert">
        <i class="fas fa-check-circle me-2"></i>
        <%= success %>
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
<% } %>

<% if (error) { %>
    <div class="alert alert-danger alert-dismissible fade show m-3" role="alert">
        <i class="fas fa-exclamation-triangle me-2"></i>
        <%= error %>
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
<% } %>

<% if (warning) { %>
    <div class="alert alert-warning alert-dismissible fade show m-3" role="alert">
        <i class="fas fa-exclamation-circle me-2"></i>
        <%= warning %>
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
<% } %>

<% if (info) { %>
    <div class="alert alert-info alert-dismissible fade show m-3" role="alert">
        <i class="fas fa-info-circle me-2"></i>
        <%= info %>
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
<% } %>
```

---

## 🏠 Real-World Page Examples

### Home Page
```html
<!-- views/home.ejs -->
<% layout('layouts/main') -%>

<!-- Hero Section -->
<section class="hero bg-primary text-white py-5">
    <div class="container">
        <div class="row align-items-center">
            <div class="col-lg-6">
                <h1 class="display-4 fw-bold mb-3">Welcome to <%= siteName || 'Our Store' %></h1>
                <p class="lead mb-4">Discover amazing products at unbeatable prices. Shop now and save big!</p>
                <a href="/products" class="btn btn-light btn-lg">
                    <i class="fas fa-shopping-bag me-2"></i>Shop Now
                </a>
            </div>
            <div class="col-lg-6">
                <img src="/images/hero-image.jpg" alt="Hero Image" class="img-fluid rounded">
            </div>
        </div>
    </div>
</section>

<!-- Featured Products -->
<section class="py-5">
    <div class="container">
        <div class="row mb-4">
            <div class="col-12 text-center">
                <h2 class="display-5 fw-bold">Featured Products</h2>
                <p class="lead text-muted">Check out our most popular items</p>
            </div>
        </div>
        
        <% if (featuredProducts && featuredProducts.length > 0) { %>
            <div class="row">
                <% featuredProducts.forEach(product => { %>
                    <div class="col-lg-3 col-md-6 mb-4">
                        <%- include('partials/product-card', { product }) %>
                    </div>
                <% }) %>
            </div>
            
            <div class="text-center mt-4">
                <a href="/products?featured=true" class="btn btn-outline-primary">
                    View All Featured Products
                </a>
            </div>
        <% } else { %>
            <div class="text-center">
                <p class="text-muted">No featured products available at the moment.</p>
            </div>
        <% } %>
    </div>
</section>

<!-- Statistics -->
<section class="bg-light py-5">
    <div class="container">
        <div class="row text-center">
            <div class="col-md-3 mb-4">
                <div class="stat-item">
                    <i class="fas fa-users fa-3x text-primary mb-3"></i>
                    <h3 class="fw-bold"><%= stats.totalUsers || 0 %></h3>
                    <p class="text-muted">Happy Customers</p>
                </div>
            </div>
            <div class="col-md-3 mb-4">
                <div class="stat-item">
                    <i class="fas fa-box fa-3x text-primary mb-3"></i>
                    <h3 class="fw-bold"><%= stats.totalProducts || 0 %></h3>
                    <p class="text-muted">Products</p>
                </div>
            </div>
            <div class="col-md-3 mb-4">
                <div class="stat-item">
                    <i class="fas fa-shopping-cart fa-3x text-primary mb-3"></i>
                    <h3 class="fw-bold"><%= stats.totalOrders || 0 %></h3>
                    <p class="text-muted">Orders Delivered</p>
                </div>
            </div>
            <div class="col-md-3 mb-4">
                <div class="stat-item">
                    <i class="fas fa-star fa-3x text-primary mb-3"></i>
                    <h3 class="fw-bold">4.8</h3>
                    <p class="text-muted">Average Rating</p>
                </div>
            </div>
        </div>
    </div>
</section>

<!-- Newsletter Signup -->
<section class="py-5">
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-lg-6 text-center">
                <h3 class="fw-bold mb-3">Stay Updated</h3>
                <p class="text-muted mb-4">Subscribe to our newsletter for the latest updates and exclusive offers.</p>
                <form action="/newsletter/subscribe" method="POST" class="d-flex">
                    <input type="email" name="email" class="form-control me-2" placeholder="Enter your email" required>
                    <button type="submit" class="btn btn-primary">
                        <i class="fas fa-paper-plane me-1"></i>Subscribe
                    </button>
                </form>
            </div>
        </div>
    </div>
</section>
```

### Product Page with Form
```html
<!-- views/products/show.ejs -->
<% layout('layouts/main') -%>

<div class="container mt-4">
    <!-- Breadcrumb -->
    <nav aria-label="breadcrumb">
        <ol class="breadcrumb">
            <li class="breadcrumb-item"><a href="/">Home</a></li>
            <li class="breadcrumb-item"><a href="/products">Products</a></li>
            <% if (product.category) { %>
                <li class="breadcrumb-item">
                    <a href="/products?category=<%= product.category._id %>">
                        <%= product.category.name %>
                    </a>
                </li>
            <% } %>
            <li class="breadcrumb-item active"><%= product.name %></li>
        </ol>
    </nav>
    
    <div class="row">
        <!-- Product Images -->
        <div class="col-lg-6">
            <div class="product-images">
                <% if (product.images && product.images.length > 0) { %>
                    <!-- Main Image -->
                    <div class="main-image mb-3">
                        <img src="<%= product.images[0].url %>" 
                             alt="<%= product.images[0].alt || product.name %>"
                             class="img-fluid rounded shadow"
                             id="mainProductImage">
                    </div>
                    
                    <!-- Thumbnail Images -->
                    <% if (product.images.length > 1) { %>
                        <div class="thumbnail-images">
                            <div class="row">
                                <% product.images.forEach((image, index) => { %>
                                    <div class="col-3 mb-2">
                                        <img src="<%= image.url %>" 
                                             alt="<%= image.alt || product.name %>"
                                             class="img-fluid rounded shadow-sm thumbnail-img <%= index === 0 ? 'active' : '' %>"
                                             onclick="changeMainImage('<%= image.url %>', this)">
                                    </div>
                                <% }) %>
                            </div>
                        </div>
                    <% } %>
                <% } else { %>
                    <div class="no-image bg-light d-flex align-items-center justify-content-center rounded" style="height: 400px;">
                        <i class="fas fa-image fa-5x text-muted"></i>
                    </div>
                <% } %>
            </div>
        </div>
        
        <!-- Product Info -->
        <div class="col-lg-6">
            <div class="product-info">
                <h1 class="product-title mb-3"><%= product.name %></h1>
                
                <!-- Rating -->
                <% if (product.rating.count > 0) { %>
                    <div class="rating mb-3">
                        <% for (let i = 1; i <= 5; i++) { %>
                            <i class="fas fa-star <%= i <= product.rating.average ? 'text-warning' : 'text-muted' %>"></i>
                        <% } %>
                        <span class="ms-2 text-muted">
                            (<%= product.rating.count %> review<%= product.rating.count !== 1 ? 's' : '' %>)
                        </span>
                    </div>
                <% } %>
                
                <!-- Price -->
                <div class="price mb-3">
                    <% if (product.salePrice) { %>
                        <span class="current-price h3 text-danger fw-bold">
                            $<%= product.salePrice.toFixed(2) %>
                        </span>
                        <span class="original-price h5 text-muted text-decoration-line-through ms-2">
                            $<%= product.price.toFixed(2) %>
                        </span>
                        <span class="badge bg-danger ms-2">
                            Save <%= Math.round(((product.price - product.salePrice) / product.price) * 100) %>%
                        </span>
                    <% } else { %>
                        <span class="current-price h3 fw-bold">
                            $<%= product.price.toFixed(2) %>
                        </span>
                    <% } %>
                </div>
                
                <!-- Stock Status -->
                <div class="stock-status mb-3">
                    <% if (product.inventory.stockQuantity > 0) { %>
                        <span class="badge bg-success">
                            <i class="fas fa-check me-1"></i>In Stock (<%= product.inventory.stockQuantity %> available)
                        </span>
                    <% } else if (product.inventory.allowBackorders) { %>
                        <span class="badge bg-warning">
                            <i class="fas fa-clock me-1"></i>Available on Backorder
                        </span>
                    <% } else { %>
                        <span class="badge bg-danger">
                            <i class="fas fa-times me-1"></i>Out of Stock
                        </span>
                    <% } %>
                </div>
                
                <!-- Short Description -->
                <% if (product.shortDescription) { %>
                    <div class="short-description mb-4">
                        <p class="lead"><%= product.shortDescription %></p>
                    </div>
                <% } %>
                
                <!-- Add to Cart Form -->
                <% if (product.inventory.stockQuantity > 0 || product.inventory.allowBackorders) { %>
                    <form id="addToCartForm" class="mb-4">
                        <div class="row">
                            <div class="col-md-4 mb-3">
                                <label for="quantity" class="form-label">Quantity:</label>
                                <div class="input-group">
                                    <button class="btn btn-outline-secondary" type="button" onclick="decreaseQuantity()">
                                        <i class="fas fa-minus"></i>
                                    </button>
                                    <input type="number" class="form-control text-center" 
                                           id="quantity" name="quantity" value="1" min="1" 
                                           max="<%= product.inventory.stockQuantity || 999 %>">
                                    <button class="btn btn-outline-secondary" type="button" onclick="increaseQuantity()">
                                        <i class="fas fa-plus"></i>
                                    </button>
                                </div>
                            </div>
                            <div class="col-md-8 mb-3">
                                <label class="form-label">&nbsp;</label>
                                <div class="d-grid gap-2 d-md-flex">
                                    <button type="submit" class="btn btn-primary btn-lg flex-fill">
                                        <i class="fas fa-cart-plus me-2"></i>Add to Cart
                                    </button>
                                    <button type="button" class="btn btn-outline-danger btn-lg" onclick="addToWishlist('<%= product._id %>')">
                                        <i class="fas fa-heart"></i>
                                    </button>
                                </div>
                            </div>
                        </div>
                    </form>
                <% } %>
                
                <!-- Product Features -->
                <div class="product-features">
                    <ul class="list-unstyled">
                        <% if (product.shipping.freeShipping) { %>
                            <li class="mb-2">
                                <i class="fas fa-truck text-success me-2"></i>
                                Free shipping on this item
                            </li>
                        <% } %>
                        <li class="mb-2">
                            <i class="fas fa-undo text-info me-2"></i>
                            30-day return policy
                        </li>
                        <li class="mb-2">
                            <i class="fas fa-shield-alt text-warning me-2"></i>
                            1-year warranty included
                        </li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Product Details Tabs -->
    <div class="row mt-5">
        <div class="col-12">
            <ul class="nav nav-tabs" id="productTabs" role="tablist">
                <li class="nav-item" role="presentation">
                    <button class="nav-link active" id="description-tab" data-bs-toggle="tab" 
                            data-bs-target="#description" type="button" role="tab">
                        Description
                    </button>
                </li>
                <li class="nav-item" role="presentation">
                    <button class="nav-link" id="specifications-tab" data-bs-toggle="tab" 
                            data-bs-target="#specifications" type="button" role="tab">
                        Specifications
                    </button>
                </li>
                <li class="nav-item" role="presentation">
                    <button class="nav-link" id="reviews-tab" data-bs-toggle="tab" 
                            data-bs-target="#reviews" type="button" role="tab">
                        Reviews (<%= product.rating.count %>)
                    </button>
                </li>
            </ul>
            
            <div class="tab-content" id="productTabsContent">
                <!-- Description Tab -->
                <div class="tab-pane fade show active" id="description" role="tabpanel">
                    <div class="p-4">
                        <div class="product-description">
                            <%- product.description %>
                        </div>
                    </div>
                </div>
                
                <!-- Specifications Tab -->
                <div class="tab-pane fade" id="specifications" role="tabpanel">
                    <div class="p-4">
                        <% if (product.specifications && Object.keys(product.specifications).length > 0) { %>
                            <table class="table table-striped">
                                <% Object.entries(product.specifications).forEach(([key, value]) => { %>
                                    <tr>
                                        <td class="fw-bold"><%= key.replace(/([A-Z])/g, ' $1').replace(/^./, str => str.toUpperCase()) %></td>
                                        <td><%= value %></td>
                                    </tr>
                                <% }) %>
                            </table>
                        <% } else { %>
                            <p class="text-muted">No specifications available for this product.</p>
                        <% } %>
                    </div>
                </div>
                
                <!-- Reviews Tab -->
                <div class="tab-pane fade" id="reviews" role="tabpanel">
                    <div class="p-4">
                        <%- include('../partials/product-reviews', { product, reviews: product.reviews }) %>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

<!-- JavaScript for product page -->
<script>
function changeMainImage(newSrc, thumbnail) {
    document.getElementById('mainProductImage').src = newSrc;
    
    // Update active thumbnail
    document.querySelectorAll('.thumbnail-img').forEach(img => img.classList.remove('active'));
    thumbnail.classList.add('active');
}

function increaseQuantity() {
    const quantityInput = document.getElementById('quantity');
    const currentValue = parseInt(quantityInput.value);
    const maxValue = parseInt(quantityInput.max);
    
    if (currentValue < maxValue) {
        quantityInput.value = currentValue + 1;
    }
}

function decreaseQuantity() {
    const quantityInput = document.getElementById('quantity');
    const currentValue = parseInt(quantityInput.value);
    
    if (currentValue > 1) {
        quantityInput.value = currentValue - 1;
    }
}

document.getElementById('addToCartForm').addEventListener('submit', async function(e) {
    e.preventDefault();
    
    const quantity = document.getElementById('quantity').value;
    const productId = '<%= product._id %>';
    
    try {
        const response = await fetch('/cart/add', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
            },
            body: JSON.stringify({
                productId: productId,
                quantity: parseInt(quantity)
            })
        });
        
        const result = await response.json();
        
        if (result.success) {
            // Update cart count in navbar
            document.getElementById('cartCount').textContent = result.cartCount;
            
            // Show success message
            showAlert('success', 'Product added to cart successfully!');
        } else {
            showAlert('error', result.error || 'Failed to add product to cart');
        }
    } catch (error) {
        showAlert('error', 'An error occurred while adding the product to cart');
    }
});

function showAlert(type, message) {
    // Create and show bootstrap alert
    const alertDiv = document.createElement('div');
    alertDiv.className = `alert alert-${type === 'error' ? 'danger' : type} alert-dismissible fade show`;
    alertDiv.innerHTML = `
        ${message}
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    `;
    
    document.querySelector('.main-content').insertBefore(alertDiv, document.querySelector('.container'));
    
    // Auto dismiss after 5 seconds
    setTimeout(() => {
        alertDiv.remove();
    }, 5000);
}
</script>
```

---

## 🔧 Advanced EJS Techniques

### Custom Helper Functions
```javascript
// helpers/ejsHelpers.js
const helpers = {
    // Format currency
    formatCurrency: (amount, currency = 'USD') => {
        return new Intl.NumberFormat('en-US', {
            style: 'currency',
            currency: currency
        }).format(amount);
    },
    
    // Format date
    formatDate: (date, options = {}) => {
        const defaultOptions = {
            year: 'numeric',
            month: 'long',
            day: 'numeric'
        };
        return new Date(date).toLocaleDateString('en-US', { ...defaultOptions, ...options });
    },
    
    // Truncate text
    truncate: (text, length = 100) => {
        if (!text) return '';
        return text.length > length ? text.substring(0, length) + '...' : text;
    },
    
    // Pluralize
    pluralize: (count, singular, plural) => {
        return count === 1 ? singular : (plural || singular + 's');
    },
    
    // Calculate reading time
    readingTime: (text) => {
        const wordsPerMinute = 200;
        const words = text.split(' ').length;
        const minutes = Math.ceil(words / wordsPerMinute);
        return `${minutes} min read`;
    },
    
    // Generate excerpt
    excerpt: (content, length = 160) => {
        const text = content.replace(/<[^>]*>/g, ''); // Remove HTML tags
        return text.length > length ? text.substring(0, length) + '...' : text;
    },
    
    // Check if array contains value
    includes: (array, value) => {
        return Array.isArray(array) && array.includes(value);
    },
    
    // Get file extension
    getFileExtension: (filename) => {
        return filename.split('.').pop().toLowerCase();
    },
    
    // Format file size
    formatFileSize: (bytes) => {
        const sizes = ['Bytes', 'KB', 'MB', 'GB'];
        if (bytes === 0) return '0 Bytes';
        const i = Math.floor(Math.log(bytes) / Math.log(1024));
        return Math.round(bytes / Math.pow(1024, i) * 100) / 100 + ' ' + sizes[i];
    }
};

// Middleware to make helpers available in templates
function addHelpersToRes(req, res, next) {
    res.locals.helpers = helpers;
    next();
}

module.exports = { helpers, addHelpersToRes };
```

### Using Helpers in Templates
```html
<!-- Example usage of helpers -->
<div class="product-price">
    <%= helpers.formatCurrency(product.price) %>
</div>

<div class="article-meta">
    Published <%= helpers.formatDate(article.createdAt) %> • 
    <%= helpers.readingTime(article.content) %>
</div>

<div class="file-info">
    Size: <%= helpers.formatFileSize(file.size) %> • 
    Type: <%= helpers.getFileExtension(file.name).toUpperCase() %>
</div>

<div class="comment-count">
    <%= comments.length %> <%= helpers.pluralize(comments.length, 'comment') %>
</div>
```

---

## 🔗 Related Topics
- [[20-Pagination]] - Paginating EJS templates
- [[18-MVC-Architecture]] - Using EJS in MVC pattern
- [[23-EJS-Includes-and-Partials]] - Advanced includes
- [[24-Express-EJS-Layout]] - Layout management

---

*Back to [[00-Main-Index]]*
