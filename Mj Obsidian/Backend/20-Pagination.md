# Pagination

**Navigation**: [[19-EJS-Templating]] | [[00-Main-Index]] | Next: [[21-Middlewares]]

---

## 📄 Pagination in Node.js Backend

**Pagination** is essential for handling large datasets efficiently. It divides data into smaller, manageable chunks and provides navigation between pages, improving both **performance** and **user experience**.

---

## 🔧 Backend Pagination Implementation

### Basic Pagination Logic
```javascript
// utils/pagination.js
class Pagination {
    constructor(page = 1, limit = 10, total = 0) {
        this.page = Math.max(1, parseInt(page));
        this.limit = Math.max(1, parseInt(limit));
        this.total = parseInt(total);
        this.offset = (this.page - 1) * this.limit;
        this.totalPages = Math.ceil(this.total / this.limit);
        this.hasNextPage = this.page < this.totalPages;
        this.hasPrevPage = this.page > 1;
        this.nextPage = this.hasNextPage ? this.page + 1 : null;
        this.prevPage = this.hasPrevPage ? this.page - 1 : null;
        this.startIndex = this.offset + 1;
        this.endIndex = Math.min(this.offset + this.limit, this.total);
    }
    
    // Generate page numbers for pagination UI
    getPageNumbers(maxPages = 5) {
        const pages = [];
        let start = Math.max(1, this.page - Math.floor(maxPages / 2));
        let end = Math.min(this.totalPages, start + maxPages - 1);
        
        // Adjust start if we're near the end
        if (end - start + 1 < maxPages) {
            start = Math.max(1, end - maxPages + 1);
        }
        
        for (let i = start; i <= end; i++) {
            pages.push(i);
        }
        
        return pages;
    }
    
    // Get pagination info object
    getInfo() {
        return {
            page: this.page,
            limit: this.limit,
            total: this.total,
            totalPages: this.totalPages,
            hasNextPage: this.hasNextPage,
            hasPrevPage: this.hasPrevPage,
            nextPage: this.nextPage,
            prevPage: this.prevPage,
            startIndex: this.startIndex,
            endIndex: this.endIndex,
            pageNumbers: this.getPageNumbers()
        };
    }
    
    // Get SQL LIMIT and OFFSET values
    getSqlParams() {
        return {
            limit: this.limit,
            offset: this.offset
        };
    }
    
    // Get MongoDB skip and limit values
    getMongoParams() {
        return {
            skip: this.offset,
            limit: this.limit
        };
    }
}

module.exports = Pagination;
```

### Pagination Middleware
```javascript
// middleware/pagination.js
const Pagination = require('../utils/pagination');

function paginationMiddleware(options = {}) {
    const {
        defaultLimit = 10,
        maxLimit = 100,
        defaultPage = 1
    } = options;
    
    return (req, res, next) => {
        const page = parseInt(req.query.page) || defaultPage;
        const limit = Math.min(parseInt(req.query.limit) || defaultLimit, maxLimit);
        
        // Add pagination helper to request
        req.pagination = {
            page,
            limit,
            offset: (page - 1) * limit,
            
            // Helper method to create pagination object
            create: (total) => new Pagination(page, limit, total)
        };
        
        // Add pagination helper to response locals for templates
        res.locals.createPagination = (total) => req.pagination.create(total);
        
        next();
    };
}

module.exports = paginationMiddleware;
```

---

## 🗄️ Database Pagination

### PostgreSQL Pagination
```javascript
// services/productService.js
const { Pool } = require('pg');
const Pagination = require('../utils/pagination');

class ProductService {
    constructor() {
        this.pool = new Pool({
            connectionString: process.env.DATABASE_URL
        });
    }
    
    async getProducts(options = {}) {
        const {
            page = 1,
            limit = 10,
            search,
            category,
            minPrice,
            maxPrice,
            featured,
            sortBy = 'created_at',
            sortOrder = 'DESC'
        } = options;
        
        const pagination = new Pagination(page, limit);
        
        // Build WHERE clause
        const conditions = ['p.status = $1'];
        const values = ['published'];
        let paramCount = 1;
        
        if (search) {
            paramCount++;
            conditions.push(`(p.name ILIKE $${paramCount} OR p.description ILIKE $${paramCount})`);
            values.push(`%${search}%`);
        }
        
        if (category) {
            paramCount++;
            conditions.push(`p.category_id = $${paramCount}`);
            values.push(category);
        }
        
        if (minPrice !== undefined) {
            paramCount++;
            conditions.push(`p.price >= $${paramCount}`);
            values.push(minPrice);
        }
        
        if (maxPrice !== undefined) {
            paramCount++;
            conditions.push(`p.price <= $${paramCount}`);
            values.push(maxPrice);
        }
        
        if (featured !== undefined) {
            paramCount++;
            conditions.push(`p.featured = $${paramCount}`);
            values.push(featured);
        }
        
        const whereClause = conditions.length > 0 ? `WHERE ${conditions.join(' AND ')}` : '';
        
        // Validate sort column
        const allowedSortColumns = ['name', 'price', 'created_at', 'rating'];
        const sortColumn = allowedSortColumns.includes(sortBy) ? sortBy : 'created_at';
        const orderDirection = ['ASC', 'DESC'].includes(sortOrder.toUpperCase()) ? sortOrder : 'DESC';
        
        // Count total records
        const countQuery = `
            SELECT COUNT(*) as total
            FROM products p
            LEFT JOIN categories c ON p.category_id = c.id
            ${whereClause}
        `;
        
        // Get paginated data
        const dataQuery = `
            SELECT 
                p.id, p.name, p.slug, p.description, p.price, p.sale_price,
                p.featured, p.rating, p.created_at, p.updated_at,
                c.name as category_name, c.slug as category_slug,
                (
                    SELECT json_agg(
                        json_build_object(
                            'id', pi.id,
                            'url', pi.url,
                            'alt', pi.alt_text,
                            'is_primary', pi.is_primary
                        )
                    )
                    FROM product_images pi 
                    WHERE pi.product_id = p.id 
                    ORDER BY pi.is_primary DESC, pi.created_at ASC
                ) as images
            FROM products p
            LEFT JOIN categories c ON p.category_id = c.id
            ${whereClause}
            ORDER BY p.${sortColumn} ${orderDirection}
            LIMIT $${paramCount + 1} OFFSET $${paramCount + 2}
        `;
        
        values.push(pagination.limit, pagination.offset);
        
        try {
            const [countResult, dataResult] = await Promise.all([
                this.pool.query(countQuery, values.slice(0, -2)), // Remove limit and offset for count
                this.pool.query(dataQuery, values)
            ]);
            
            const total = parseInt(countResult.rows[0].total);
            const products = dataResult.rows;
            
            // Update pagination with total count
            const paginationInfo = new Pagination(page, limit, total);
            
            return {
                products,
                pagination: paginationInfo.getInfo(),
                filters: {
                    search,
                    category,
                    minPrice,
                    maxPrice,
                    featured,
                    sortBy,
                    sortOrder
                }
            };
            
        } catch (error) {
            throw new Error(`Failed to fetch products: ${error.message}`);
        }
    }
    
    // Advanced pagination with cursor-based approach for large datasets
    async getProductsCursor(options = {}) {
        const {
            limit = 10,
            cursor, // Last item's ID from previous page
            search,
            category,
            sortBy = 'created_at',
            sortOrder = 'DESC'
        } = options;
        
        const conditions = ['p.status = $1'];
        const values = ['published'];
        let paramCount = 1;
        
        // Add cursor condition for pagination
        if (cursor) {
            paramCount++;
            if (sortOrder.toUpperCase() === 'DESC') {
                conditions.push(`p.${sortBy} < (SELECT ${sortBy} FROM products WHERE id = $${paramCount})`);
            } else {
                conditions.push(`p.${sortBy} > (SELECT ${sortBy} FROM products WHERE id = $${paramCount})`);
            }
            values.push(cursor);
        }
        
        // Add other filters
        if (search) {
            paramCount++;
            conditions.push(`(p.name ILIKE $${paramCount} OR p.description ILIKE $${paramCount})`);
            values.push(`%${search}%`);
        }
        
        if (category) {
            paramCount++;
            conditions.push(`p.category_id = $${paramCount}`);
            values.push(category);
        }
        
        const whereClause = `WHERE ${conditions.join(' AND ')}`;
        
        const query = `
            SELECT 
                p.id, p.name, p.slug, p.price, p.sale_price,
                p.${sortBy}, p.created_at,
                c.name as category_name
            FROM products p
            LEFT JOIN categories c ON p.category_id = c.id
            ${whereClause}
            ORDER BY p.${sortBy} ${sortOrder}
            LIMIT $${paramCount + 1}
        `;
        
        values.push(limit + 1); // Fetch one extra to check if there's a next page
        
        try {
            const result = await this.pool.query(query, values);
            const products = result.rows;
            
            const hasNextPage = products.length > limit;
            if (hasNextPage) {
                products.pop(); // Remove the extra item
            }
            
            const nextCursor = hasNextPage && products.length > 0 
                ? products[products.length - 1].id 
                : null;
            
            return {
                products,
                pagination: {
                    hasNextPage,
                    nextCursor,
                    limit
                }
            };
            
        } catch (error) {
            throw new Error(`Failed to fetch products: ${error.message}`);
        }
    }
}

module.exports = ProductService;
```

### MongoDB Pagination with Mongoose
```javascript
// models/Product.js (MongoDB/Mongoose)
const mongoose = require('mongoose');
const mongoosePaginate = require('mongoose-paginate-v2');

const productSchema = new mongoose.Schema({
    name: { type: String, required: true },
    description: { type: String, required: true },
    price: { type: Number, required: true },
    category: { type: mongoose.Schema.ObjectId, ref: 'Category' },
    featured: { type: Boolean, default: false },
    status: { type: String, enum: ['draft', 'published'], default: 'published' }
}, { timestamps: true });

// Add pagination plugin
productSchema.plugin(mongoosePaginate);

// Custom pagination method
productSchema.statics.paginateProducts = async function(options = {}) {
    const {
        page = 1,
        limit = 10,
        search,
        category,
        featured,
        sortBy = 'createdAt',
        sortOrder = 'desc'
    } = options;
    
    // Build query
    const query = { status: 'published' };
    
    if (search) {
        query.$or = [
            { name: { $regex: search, $options: 'i' } },
            { description: { $regex: search, $options: 'i' } }
        ];
    }
    
    if (category) query.category = category;
    if (featured !== undefined) query.featured = featured;
    
    // Pagination options
    const paginateOptions = {
        page: parseInt(page),
        limit: parseInt(limit),
        sort: { [sortBy]: sortOrder === 'desc' ? -1 : 1 },
        populate: [
            { path: 'category', select: 'name slug' },
            { path: 'images', select: 'url alt isPrimary' }
        ],
        lean: true // Return plain objects instead of Mongoose documents for better performance
    };
    
    try {
        const result = await this.paginate(query, paginateOptions);
        
        return {
            products: result.docs,
            pagination: {
                page: result.page,
                limit: result.limit,
                total: result.totalDocs,
                totalPages: result.totalPages,
                hasNextPage: result.hasNextPage,
                hasPrevPage: result.hasPrevPage,
                nextPage: result.nextPage,
                prevPage: result.prevPage,
                startIndex: ((result.page - 1) * result.limit) + 1,
                endIndex: Math.min(result.page * result.limit, result.totalDocs)
            }
        };
    } catch (error) {
        throw new Error(`Pagination failed: ${error.message}`);
    }
};

module.exports = mongoose.model('Product', productSchema);
```

---

## 🎯 Controller Implementation

### Product Controller with Pagination
```javascript
// controllers/productController.js
const ProductService = require('../services/productService');
const paginationMiddleware = require('../middleware/pagination');

class ProductController {
    // Get paginated products list
    static async index(req, res, next) {
        try {
            const {
                search,
                category,
                minPrice,
                maxPrice,
                featured,
                sortBy = 'created_at',
                sortOrder = 'desc'
            } = req.query;
            
            const options = {
                page: req.pagination.page,
                limit: req.pagination.limit,
                search,
                category,
                minPrice: minPrice ? parseFloat(minPrice) : undefined,
                maxPrice: maxPrice ? parseFloat(maxPrice) : undefined,
                featured: featured === 'true' ? true : featured === 'false' ? false : undefined,
                sortBy,
                sortOrder
            };
            
            const productService = new ProductService();
            const result = await productService.getProducts(options);
            
            // For API requests, return JSON
            if (req.xhr || req.headers.accept?.includes('application/json')) {
                return res.json({
                    success: true,
                    data: result.products,
                    pagination: result.pagination,
                    filters: result.filters
                });
            }
            
            // For regular requests, render template
            res.render('products/index', {
                title: 'Products',
                products: result.products,
                pagination: result.pagination,
                filters: result.filters,
                currentPage: 'products'
            });
            
        } catch (error) {
            next(error);
        }
    }
    
    // API endpoint for AJAX pagination
    static async getProductsAPI(req, res, next) {
        try {
            const productService = new ProductService();
            const result = await productService.getProducts({
                page: req.pagination.page,
                limit: req.pagination.limit,
                ...req.query
            });
            
            res.json({
                success: true,
                products: result.products,
                pagination: result.pagination
            });
            
        } catch (error) {
            res.status(500).json({
                success: false,
                error: error.message
            });
        }
    }
    
    // Infinite scroll endpoint
    static async getProductsInfinite(req, res, next) {
        try {
            const productService = new ProductService();
            const result = await productService.getProductsCursor({
                limit: req.pagination.limit,
                cursor: req.query.cursor,
                ...req.query
            });
            
            res.json({
                success: true,
                products: result.products,
                pagination: result.pagination
            });
            
        } catch (error) {
            res.status(500).json({
                success: false,
                error: error.message
            });
        }
    }
}

module.exports = ProductController;
```

---

## 🎨 Frontend Pagination Templates

### EJS Pagination Component
```html
<!-- views/partials/pagination.ejs -->
<% if (pagination.totalPages > 1) { %>
    <nav aria-label="Page navigation">
        <ul class="pagination justify-content-center">
            <!-- Previous Page -->
            <li class="page-item <%= !pagination.hasPrevPage ? 'disabled' : '' %>">
                <% if (pagination.hasPrevPage) { %>
                    <a class="page-link" href="?page=<%= pagination.prevPage %><%= queryString || '' %>">
                        <i class="fas fa-chevron-left"></i>
                        <span class="d-none d-sm-inline ms-1">Previous</span>
                    </a>
                <% } else { %>
                    <span class="page-link">
                        <i class="fas fa-chevron-left"></i>
                        <span class="d-none d-sm-inline ms-1">Previous</span>
                    </span>
                <% } %>
            </li>
            
            <!-- First Page -->
            <% if (pagination.pageNumbers[0] > 1) { %>
                <li class="page-item">
                    <a class="page-link" href="?page=1<%= queryString || '' %>">1</a>
                </li>
                <% if (pagination.pageNumbers[0] > 2) { %>
                    <li class="page-item disabled">
                        <span class="page-link">...</span>
                    </li>
                <% } %>
            <% } %>
            
            <!-- Page Numbers -->
            <% pagination.pageNumbers.forEach(pageNum => { %>
                <li class="page-item <%= pageNum === pagination.page ? 'active' : '' %>">
                    <% if (pageNum === pagination.page) { %>
                        <span class="page-link"><%= pageNum %></span>
                    <% } else { %>
                        <a class="page-link" href="?page=<%= pageNum %><%= queryString || '' %>">
                            <%= pageNum %>
                        </a>
                    <% } %>
                </li>
            <% }) %>
            
            <!-- Last Page -->
            <% const lastPageNum = pagination.pageNumbers[pagination.pageNumbers.length - 1] %>
            <% if (lastPageNum < pagination.totalPages) { %>
                <% if (lastPageNum < pagination.totalPages - 1) { %>
                    <li class="page-item disabled">
                        <span class="page-link">...</span>
                    </li>
                <% } %>
                <li class="page-item">
                    <a class="page-link" href="?page=<%= pagination.totalPages %><%= queryString || '' %>">
                        <%= pagination.totalPages %>
                    </a>
                </li>
            <% } %>
            
            <!-- Next Page -->
            <li class="page-item <%= !pagination.hasNextPage ? 'disabled' : '' %>">
                <% if (pagination.hasNextPage) { %>
                    <a class="page-link" href="?page=<%= pagination.nextPage %><%= queryString || '' %>">
                        <span class="d-none d-sm-inline me-1">Next</span>
                        <i class="fas fa-chevron-right"></i>
                    </a>
                <% } else { %>
                    <span class="page-link">
                        <span class="d-none d-sm-inline me-1">Next</span>
                        <i class="fas fa-chevron-right"></i>
                    </span>
                <% } %>
            </li>
        </ul>
        
        <!-- Pagination Info -->
        <div class="pagination-info text-center mt-2">
            <small class="text-muted">
                Showing <%= pagination.startIndex %> to <%= pagination.endIndex %> 
                of <%= pagination.total %> results
            </small>
        </div>
    </nav>
<% } %>
```

### Product Listing with Pagination
```html
<!-- views/products/index.ejs -->
<% layout('layouts/main') -%>

<div class="container mt-4">
    <!-- Header -->
    <div class="row mb-4">
        <div class="col-md-8">
            <h1>Products</h1>
            <% if (filters.search) { %>
                <p class="text-muted">Search results for "<%= filters.search %>"</p>
            <% } %>
        </div>
        <div class="col-md-4">
            <!-- Items per page selector -->
            <div class="d-flex align-items-center">
                <label for="itemsPerPage" class="form-label me-2 mb-0">Show:</label>
                <select id="itemsPerPage" class="form-select" onchange="changeItemsPerPage(this.value)">
                    <option value="10" <%= pagination.limit === 10 ? 'selected' : '' %>>10</option>
                    <option value="20" <%= pagination.limit === 20 ? 'selected' : '' %>>20</option>
                    <option value="50" <%= pagination.limit === 50 ? 'selected' : '' %>>50</option>
                </select>
            </div>
        </div>
    </div>
    
    <!-- Filters -->
    <div class="row mb-4">
        <div class="col-12">
            <form method="GET" class="row g-3">
                <div class="col-md-3">
                    <input type="text" class="form-control" name="search" 
                           placeholder="Search products..." value="<%= filters.search || '' %>">
                </div>
                <div class="col-md-2">
                    <select name="category" class="form-select">
                        <option value="">All Categories</option>
                        <!-- Category options would be populated here -->
                    </select>
                </div>
                <div class="col-md-2">
                    <select name="sortBy" class="form-select">
                        <option value="created_at" <%= filters.sortBy === 'created_at' ? 'selected' : '' %>>Newest</option>
                        <option value="name" <%= filters.sortBy === 'name' ? 'selected' : '' %>>Name</option>
                        <option value="price" <%= filters.sortBy === 'price' ? 'selected' : '' %>>Price</option>
                        <option value="rating" <%= filters.sortBy === 'rating' ? 'selected' : '' %>>Rating</option>
                    </select>
                </div>
                <div class="col-md-2">
                    <select name="sortOrder" class="form-select">
                        <option value="desc" <%= filters.sortOrder === 'desc' ? 'selected' : '' %>>Descending</option>
                        <option value="asc" <%= filters.sortOrder === 'asc' ? 'selected' : '' %>>Ascending</option>
                    </select>
                </div>
                <div class="col-md-3">
                    <div class="d-flex gap-2">
                        <button type="submit" class="btn btn-primary">
                            <i class="fas fa-search me-1"></i>Filter
                        </button>
                        <a href="/products" class="btn btn-outline-secondary">
                            <i class="fas fa-times me-1"></i>Clear
                        </a>
                    </div>
                </div>
                
                <!-- Preserve page and limit -->
                <input type="hidden" name="page" value="1">
                <input type="hidden" name="limit" value="<%= pagination.limit %>">
            </form>
        </div>
    </div>
    
    <!-- Products Grid -->
    <div id="productsContainer">
        <% if (products.length === 0) { %>
            <div class="text-center py-5">
                <i class="fas fa-search fa-3x text-muted mb-3"></i>
                <h4>No products found</h4>
                <p class="text-muted">Try adjusting your search or filter criteria.</p>
            </div>
        <% } else { %>
            <div class="row" id="productsGrid">
                <% products.forEach(product => { %>
                    <div class="col-lg-3 col-md-4 col-sm-6 mb-4">
                        <%- include('../partials/product-card', { product }) %>
                    </div>
                <% }) %>
            </div>
            
            <!-- Pagination -->
            <div class="mt-4">
                <% 
                // Build query string for pagination links
                const queryParams = new URLSearchParams();
                Object.entries(filters).forEach(([key, value]) => {
                    if (value !== undefined && value !== null && value !== '') {
                        queryParams.append(key, value);
                    }
                });
                queryParams.append('limit', pagination.limit);
                const queryString = queryParams.toString() ? '&' + queryParams.toString() : '';
                %>
                <%- include('../partials/pagination', { pagination, queryString }) %>
            </div>
        <% } %>
    </div>
    
    <!-- Loading overlay for AJAX -->
    <div id="loadingOverlay" class="d-none">
        <div class="d-flex justify-content-center">
            <div class="spinner-border" role="status">
                <span class="visually-hidden">Loading...</span>
            </div>
        </div>
    </div>
</div>

<script>
function changeItemsPerPage(limit) {
    const url = new URL(window.location);
    url.searchParams.set('limit', limit);
    url.searchParams.set('page', '1'); // Reset to first page
    window.location = url.toString();
}

// AJAX pagination (optional)
function loadPage(page) {
    const url = new URL(window.location);
    url.searchParams.set('page', page);
    
    fetch(url.toString(), {
        headers: {
            'Accept': 'application/json',
            'X-Requested-With': 'XMLHttpRequest'
        }
    })
    .then(response => response.json())
    .then(data => {
        if (data.success) {
            // Update products grid
            document.getElementById('productsGrid').innerHTML = renderProducts(data.data);
            
            // Update pagination
            updatePagination(data.pagination);
            
            // Update URL without page reload
            history.pushState({}, '', url.toString());
        }
    })
    .catch(error => {
        console.error('Error loading page:', error);
    });
}

function renderProducts(products) {
    return products.map(product => `
        <div class="col-lg-3 col-md-4 col-sm-6 mb-4">
            <!-- Product card HTML -->
        </div>
    `).join('');
}

function updatePagination(pagination) {
    // Update pagination component
    // This would require generating the pagination HTML in JavaScript
}
</script>
```

---

## ⚡ Advanced Pagination Techniques

### Infinite Scroll Implementation
```html
<!-- views/products/infinite-scroll.ejs -->
<div class="container mt-4">
    <h1>Products (Infinite Scroll)</h1>
    
    <div id="productsContainer">
        <div class="row" id="productsGrid">
            <% products.forEach(product => { %>
                <div class="col-lg-3 col-md-4 col-sm-6 mb-4">
                    <%- include('../partials/product-card', { product }) %>
                </div>
            <% }) %>
        </div>
    </div>
    
    <!-- Loading indicator -->
    <div id="loadingIndicator" class="text-center py-4 d-none">
        <div class="spinner-border" role="status">
            <span class="visually-hidden">Loading more products...</span>
        </div>
    </div>
    
    <!-- End message -->
    <div id="endMessage" class="text-center py-4 d-none">
        <p class="text-muted">You've reached the end of the products list.</p>
    </div>
</div>

<script>
class InfiniteScroll {
    constructor() {
        this.loading = false;
        this.hasNextPage = <%= pagination.hasNextPage ? 'true' : 'false' %>;
        this.nextCursor = '<%= pagination.nextCursor || "" %>';
        this.init();
    }
    
    init() {
        window.addEventListener('scroll', this.handleScroll.bind(this));
    }
    
    handleScroll() {
        if (this.loading || !this.hasNextPage) return;
        
        const { scrollTop, scrollHeight, clientHeight } = document.documentElement;
        
        // Load more when user is 100px from bottom
        if (scrollTop + clientHeight >= scrollHeight - 100) {
            this.loadMore();
        }
    }
    
    async loadMore() {
        if (this.loading || !this.hasNextPage) return;
        
        this.loading = true;
        this.showLoading();
        
        try {
            const response = await fetch(`/api/products/infinite?cursor=${this.nextCursor}&limit=12`);
            const data = await response.json();
            
            if (data.success) {
                this.appendProducts(data.products);
                this.hasNextPage = data.pagination.hasNextPage;
                this.nextCursor = data.pagination.nextCursor;
                
                if (!this.hasNextPage) {
                    this.showEndMessage();
                }
            }
        } catch (error) {
            console.error('Error loading more products:', error);
        } finally {
            this.loading = false;
            this.hideLoading();
        }
    }
    
    appendProducts(products) {
        const grid = document.getElementById('productsGrid');
        products.forEach(product => {
            const productElement = this.createProductElement(product);
            grid.appendChild(productElement);
        });
    }
    
    createProductElement(product) {
        const div = document.createElement('div');
        div.className = 'col-lg-3 col-md-4 col-sm-6 mb-4';
        div.innerHTML = `
            <div class="card product-card h-100">
                <img src="${product.images?.[0]?.url || '/images/placeholder.jpg'}" 
                     class="card-img-top" alt="${product.name}">
                <div class="card-body">
                    <h5 class="card-title">${product.name}</h5>
                    <p class="card-text text-muted">${product.description.substring(0, 100)}...</p>
                    <div class="d-flex justify-content-between align-items-center">
                        <span class="h6 mb-0">$${product.price.toFixed(2)}</span>
                        <button class="btn btn-primary btn-sm">Add to Cart</button>
                    </div>
                </div>
            </div>
        `;
        return div;
    }
    
    showLoading() {
        document.getElementById('loadingIndicator').classList.remove('d-none');
    }
    
    hideLoading() {
        document.getElementById('loadingIndicator').classList.add('d-none');
    }
    
    showEndMessage() {
        document.getElementById('endMessage').classList.remove('d-none');
    }
}

// Initialize infinite scroll
new InfiniteScroll();
</script>
```

### Load More Button Implementation
```html
<!-- views/products/load-more.ejs -->
<div class="container mt-4">
    <h1>Products</h1>
    
    <div id="productsContainer">
        <div class="row" id="productsGrid">
            <% products.forEach(product => { %>
                <div class="col-lg-3 col-md-4 col-sm-6 mb-4">
                    <%- include('../partials/product-card', { product }) %>
                </div>
            <% }) %>
        </div>
    </div>
    
    <!-- Load More Button -->
    <% if (pagination.hasNextPage) { %>
        <div class="text-center mt-4">
            <button id="loadMoreBtn" class="btn btn-outline-primary btn-lg">
                <span id="btnText">Load More Products</span>
                <span id="btnSpinner" class="spinner-border spinner-border-sm ms-2 d-none"></span>
            </button>
        </div>
    <% } %>
    
    <!-- Products Count -->
    <div class="text-center mt-3">
        <small class="text-muted">
            Showing <span id="currentCount"><%= products.length %></span> of 
            <span id="totalCount"><%= pagination.total %></span> products
        </small>
    </div>
</div>

<script>
class LoadMorePagination {
    constructor() {
        this.currentPage = <%= pagination.page %>;
        this.hasNextPage = <%= pagination.hasNextPage ? 'true' : 'false' %>;
        this.loading = false;
        this.init();
    }
    
    init() {
        const loadMoreBtn = document.getElementById('loadMoreBtn');
        if (loadMoreBtn) {
            loadMoreBtn.addEventListener('click', this.loadMore.bind(this));
        }
    }
    
    async loadMore() {
        if (this.loading || !this.hasNextPage) return;
        
        this.loading = true;
        this.setLoadingState(true);
        
        try {
            const nextPage = this.currentPage + 1;
            const response = await fetch(`/api/products?page=${nextPage}&limit=12`);
            const data = await response.json();
            
            if (data.success) {
                this.appendProducts(data.products);
                this.currentPage = data.pagination.page;
                this.hasNextPage = data.pagination.hasNextPage;
                
                // Update counts
                const currentCount = document.querySelectorAll('#productsGrid .col-lg-3').length;
                document.getElementById('currentCount').textContent = currentCount;
                
                // Hide button if no more pages
                if (!this.hasNextPage) {
                    document.getElementById('loadMoreBtn').style.display = 'none';
                }
            }
        } catch (error) {
            console.error('Error loading more products:', error);
            alert('Failed to load more products. Please try again.');
        } finally {
            this.loading = false;
            this.setLoadingState(false);
        }
    }
    
    appendProducts(products) {
        const grid = document.getElementById('productsGrid');
        products.forEach(product => {
            const productElement = this.createProductElement(product);
            grid.appendChild(productElement);
        });
        
        // Add fade-in animation
        const newElements = grid.querySelectorAll('.col-lg-3:nth-last-child(-n+' + products.length + ')');
        newElements.forEach((element, index) => {
            element.style.opacity = '0';
            element.style.transform = 'translateY(20px)';
            setTimeout(() => {
                element.style.transition = 'opacity 0.3s ease, transform 0.3s ease';
                element.style.opacity = '1';
                element.style.transform = 'translateY(0)';
            }, index * 100);
        });
    }
    
    createProductElement(product) {
        const div = document.createElement('div');
        div.className = 'col-lg-3 col-md-4 col-sm-6 mb-4';
        div.innerHTML = `
            <div class="card product-card h-100">
                <img src="${product.images?.[0]?.url || '/images/placeholder.jpg'}" 
                     class="card-img-top" alt="${product.name}" loading="lazy">
                <div class="card-body d-flex flex-column">
                    <h5 class="card-title">${product.name}</h5>
                    <p class="card-text text-muted flex-grow-1">
                        ${product.description.substring(0, 100)}...
                    </p>
                    <div class="mt-auto">
                        <div class="d-flex justify-content-between align-items-center">
                            <span class="h6 mb-0">$${product.price.toFixed(2)}</span>
                            <button class="btn btn-primary btn-sm" onclick="addToCart('${product.id}')">
                                Add to Cart
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        `;
        return div;
    }
    
    setLoadingState(loading) {
        const btnText = document.getElementById('btnText');
        const btnSpinner = document.getElementById('btnSpinner');
        const loadMoreBtn = document.getElementById('loadMoreBtn');
        
        if (loading) {
            btnText.textContent = 'Loading...';
            btnSpinner.classList.remove('d-none');
            loadMoreBtn.disabled = true;
        } else {
            btnText.textContent = 'Load More Products';
            btnSpinner.classList.add('d-none');
            loadMoreBtn.disabled = false;
        }
    }
}

// Initialize load more pagination
new LoadMorePagination();

function addToCart(productId) {
    // Add to cart functionality
    console.log('Adding product to cart:', productId);
}
</script>
```

---

## 🔧 Route Configuration

### Product Routes with Pagination
```javascript
// routes/productRoutes.js
const express = require('express');
const router = express.Router();
const ProductController = require('../controllers/productController');
const paginationMiddleware = require('../middleware/pagination');

// Apply pagination middleware
router.use(paginationMiddleware({
    defaultLimit: 12,
    maxLimit: 50
}));

// Product listing with pagination
router.get('/', ProductController.index);

// API endpoints for AJAX pagination
router.get('/api', ProductController.getProductsAPI);

// Infinite scroll endpoint
router.get('/api/infinite', ProductController.getProductsInfinite);

module.exports = router;
```

---

## 🔗 Related Topics
- [[19-EJS-Templating]] - Template pagination components
- [[15-Query-Parameters]] - Pagination query parameters
- [[28-PostgreSQL-Integration]] - Database pagination
- [[30-Database-Pagination]] - Advanced database pagination

---

*Back to [[00-Main-Index]]*
