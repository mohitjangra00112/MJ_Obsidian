# Query Parameters in Express

**Navigation**: [[14-Environment-Variables]] | [[00-Main-Index]] | Next: [[16-Joi-Validation]]

---

## 📝 Understanding Query Parameters

Query parameters are **key-value pairs** appended to URLs after a `?` symbol, separated by `&`. They provide a way to pass data to the server through the URL.

**URL Structure**: `http://localhost:3000/products?category=electronics&page=2&limit=10&sort=price`

---

## 🔧 Basic Query Parameter Handling

### Simple Query Parameters
```javascript
const express = require('express');
const app = express();

// Basic query parameter access
app.get('/search', (req, res) => {
    // URL: /search?q=nodejs&category=tutorial
    const query = req.query.q;
    const category = req.query.category;
    
    res.json({
        searchQuery: query,
        category: category,
        allParams: req.query
    });
});

// Multiple query parameters
app.get('/products', (req, res) => {
    // URL: /products?category=electronics&minPrice=100&maxPrice=500&sort=price&order=asc
    const {
        category,
        minPrice,
        maxPrice,
        sort = 'name',      // Default value
        order = 'asc',      // Default value
        page = 1,           // Default pagination
        limit = 10          // Default limit
    } = req.query;
    
    res.json({
        filters: {
            category,
            priceRange: {
                min: parseInt(minPrice) || 0,
                max: parseInt(maxPrice) || Infinity
            }
        },
        sorting: {
            field: sort,
            direction: order
        },
        pagination: {
            page: parseInt(page),
            limit: parseInt(limit),
            offset: (parseInt(page) - 1) * parseInt(limit)
        }
    });
});

app.listen(3000, () => {
    console.log('Server running on port 3000');
});
```

### Query Parameter Types and Conversion
```javascript
// utils/queryHelpers.js
class QueryHelper {
    static parseBoolean(value) {
        if (typeof value === 'boolean') return value;
        if (typeof value === 'string') {
            return value.toLowerCase() === 'true' || value === '1';
        }
        return false;
    }
    
    static parseNumber(value, defaultValue = 0) {
        const parsed = Number(value);
        return isNaN(parsed) ? defaultValue : parsed;
    }
    
    static parseArray(value, separator = ',') {
        if (!value) return [];
        if (Array.isArray(value)) return value;
        return String(value).split(separator).map(item => item.trim());
    }
    
    static parseDate(value) {
        if (!value) return null;
        const date = new Date(value);
        return isNaN(date.getTime()) ? null : date;
    }
    
    static sanitizeString(value, maxLength = 255) {
        if (!value) return '';
        return String(value).trim().substring(0, maxLength);
    }
}

// Using query helpers
app.get('/advanced-search', (req, res) => {
    const {
        q,
        categories,
        minPrice,
        maxPrice,
        inStock,
        featured,
        createdAfter,
        tags,
        page,
        limit
    } = req.query;
    
    const searchParams = {
        query: QueryHelper.sanitizeString(q, 100),
        categories: QueryHelper.parseArray(categories),
        priceRange: {
            min: QueryHelper.parseNumber(minPrice, 0),
            max: QueryHelper.parseNumber(maxPrice, Infinity)
        },
        filters: {
            inStock: QueryHelper.parseBoolean(inStock),
            featured: QueryHelper.parseBoolean(featured)
        },
        dateFilters: {
            createdAfter: QueryHelper.parseDate(createdAfter)
        },
        tags: QueryHelper.parseArray(tags),
        pagination: {
            page: Math.max(1, QueryHelper.parseNumber(page, 1)),
            limit: Math.min(100, Math.max(1, QueryHelper.parseNumber(limit, 10)))
        }
    };
    
    res.json({
        searchParams,
        sql: buildSearchQuery(searchParams)
    });
});

function buildSearchQuery(params) {
    const conditions = [];
    const values = [];
    
    if (params.query) {
        conditions.push('name ILIKE $' + (values.length + 1));
        values.push(`%${params.query}%`);
    }
    
    if (params.categories.length > 0) {
        conditions.push('category = ANY($' + (values.length + 1) + ')');
        values.push(params.categories);
    }
    
    if (params.priceRange.min > 0) {
        conditions.push('price >= $' + (values.length + 1));
        values.push(params.priceRange.min);
    }
    
    if (params.priceRange.max < Infinity) {
        conditions.push('price <= $' + (values.length + 1));
        values.push(params.priceRange.max);
    }
    
    if (params.filters.inStock) {
        conditions.push('stock_quantity > 0');
    }
    
    if (params.filters.featured) {
        conditions.push('featured = true');
    }
    
    const whereClause = conditions.length > 0 ? 'WHERE ' + conditions.join(' AND ') : '';
    const offset = (params.pagination.page - 1) * params.pagination.limit;
    
    return {
        query: `SELECT * FROM products ${whereClause} ORDER BY name LIMIT $${values.length + 1} OFFSET $${values.length + 2}`,
        values: [...values, params.pagination.limit, offset]
    };
}

module.exports = QueryHelper;
```

---

## 🔍 Advanced Query Processing

### Query Parameter Middleware
```javascript
// middleware/queryProcessor.js
const QueryHelper = require('../utils/queryHelpers');

function queryProcessor(options = {}) {
    const {
        maxLimit = 100,
        defaultLimit = 10,
        allowedSortFields = [],
        defaultSort = 'created_at',
        allowedFilters = []
    } = options;
    
    return (req, res, next) => {
        // Pagination processing
        req.pagination = {
            page: Math.max(1, QueryHelper.parseNumber(req.query.page, 1)),
            limit: Math.min(maxLimit, Math.max(1, QueryHelper.parseNumber(req.query.limit, defaultLimit)))
        };
        req.pagination.offset = (req.pagination.page - 1) * req.pagination.limit;
        
        // Sorting processing
        const sortField = req.query.sort || defaultSort;
        const sortOrder = (req.query.order || 'asc').toLowerCase();
        
        req.sorting = {
            field: allowedSortFields.includes(sortField) ? sortField : defaultSort,
            order: ['asc', 'desc'].includes(sortOrder) ? sortOrder : 'asc'
        };
        
        // Filter processing
        req.filters = {};
        allowedFilters.forEach(filter => {
            if (req.query[filter] !== undefined) {
                switch (filter) {
                    case 'active':
                    case 'featured':
                    case 'published':
                        req.filters[filter] = QueryHelper.parseBoolean(req.query[filter]);
                        break;
                    case 'minPrice':
                    case 'maxPrice':
                    case 'minRating':
                    case 'maxRating':
                        req.filters[filter] = QueryHelper.parseNumber(req.query[filter]);
                        break;
                    case 'categories':
                    case 'tags':
                        req.filters[filter] = QueryHelper.parseArray(req.query[filter]);
                        break;
                    case 'createdAfter':
                    case 'createdBefore':
                        req.filters[filter] = QueryHelper.parseDate(req.query[filter]);
                        break;
                    default:
                        req.filters[filter] = QueryHelper.sanitizeString(req.query[filter]);
                }
            }
        });
        
        // Search processing
        if (req.query.q) {
            req.search = {
                query: QueryHelper.sanitizeString(req.query.q, 100),
                fields: QueryHelper.parseArray(req.query.searchFields) || ['name', 'description']
            };
        }
        
        next();
    };
}

// Usage with different endpoints
app.get('/products', 
    queryProcessor({
        allowedSortFields: ['name', 'price', 'created_at', 'rating'],
        defaultSort: 'name',
        allowedFilters: ['category', 'minPrice', 'maxPrice', 'featured', 'tags'],
        maxLimit: 50
    }),
    async (req, res) => {
        try {
            const products = await getProducts({
                pagination: req.pagination,
                sorting: req.sorting,
                filters: req.filters,
                search: req.search
            });
            
            res.json(products);
        } catch (error) {
            res.status(500).json({ error: error.message });
        }
    }
);

app.get('/users',
    queryProcessor({
        allowedSortFields: ['username', 'email', 'created_at', 'last_login'],
        defaultSort: 'created_at',
        allowedFilters: ['active', 'role', 'createdAfter'],
        maxLimit: 25
    }),
    async (req, res) => {
        try {
            const users = await getUsers({
                pagination: req.pagination,
                sorting: req.sorting,
                filters: req.filters
            });
            
            res.json(users);
        } catch (error) {
            res.status(500).json({ error: error.message });
        }
    }
);

module.exports = queryProcessor;
```

### Dynamic Query Builder
```javascript
// utils/queryBuilder.js
class QueryBuilder {
    constructor(tableName) {
        this.table = tableName;
        this.selectFields = ['*'];
        this.whereConditions = [];
        this.joinClauses = [];
        this.orderClauses = [];
        this.limitClause = null;
        this.offsetClause = null;
        this.parameters = [];
    }
    
    select(fields) {
        this.selectFields = Array.isArray(fields) ? fields : [fields];
        return this;
    }
    
    where(field, operator, value) {
        if (value !== undefined && value !== null && value !== '') {
            const paramIndex = this.parameters.length + 1;
            this.whereConditions.push(`${field} ${operator} $${paramIndex}`);
            this.parameters.push(value);
        }
        return this;
    }
    
    whereIn(field, values) {
        if (Array.isArray(values) && values.length > 0) {
            const paramIndex = this.parameters.length + 1;
            this.whereConditions.push(`${field} = ANY($${paramIndex})`);
            this.parameters.push(values);
        }
        return this;
    }
    
    whereRange(field, min, max) {
        if (min !== undefined && min !== null) {
            this.where(field, '>=', min);
        }
        if (max !== undefined && max !== null) {
            this.where(field, '<=', max);
        }
        return this;
    }
    
    whereDate(field, operator, date) {
        if (date instanceof Date) {
            this.where(field, operator, date.toISOString());
        }
        return this;
    }
    
    whereSearch(fields, query) {
        if (query && fields.length > 0) {
            const searchConditions = fields.map(field => 
                `${field} ILIKE $${this.parameters.length + 1}`
            );
            this.whereConditions.push(`(${searchConditions.join(' OR ')})`);
            this.parameters.push(`%${query}%`);
        }
        return this;
    }
    
    join(table, condition) {
        this.joinClauses.push(`JOIN ${table} ON ${condition}`);
        return this;
    }
    
    leftJoin(table, condition) {
        this.joinClauses.push(`LEFT JOIN ${table} ON ${condition}`);
        return this;
    }
    
    orderBy(field, direction = 'ASC') {
        this.orderClauses.push(`${field} ${direction.toUpperCase()}`);
        return this;
    }
    
    limit(count) {
        this.limitClause = count;
        return this;
    }
    
    offset(count) {
        this.offsetClause = count;
        return this;
    }
    
    build() {
        let query = `SELECT ${this.selectFields.join(', ')} FROM ${this.table}`;
        
        if (this.joinClauses.length > 0) {
            query += ' ' + this.joinClauses.join(' ');
        }
        
        if (this.whereConditions.length > 0) {
            query += ' WHERE ' + this.whereConditions.join(' AND ');
        }
        
        if (this.orderClauses.length > 0) {
            query += ' ORDER BY ' + this.orderClauses.join(', ');
        }
        
        if (this.limitClause !== null) {
            const paramIndex = this.parameters.length + 1;
            query += ` LIMIT $${paramIndex}`;
            this.parameters.push(this.limitClause);
        }
        
        if (this.offsetClause !== null) {
            const paramIndex = this.parameters.length + 1;
            query += ` OFFSET $${paramIndex}`;
            this.parameters.push(this.offsetClause);
        }
        
        return {
            text: query,
            values: this.parameters
        };
    }
    
    buildCount() {
        let query = `SELECT COUNT(*) FROM ${this.table}`;
        
        if (this.joinClauses.length > 0) {
            query += ' ' + this.joinClauses.join(' ');
        }
        
        if (this.whereConditions.length > 0) {
            query += ' WHERE ' + this.whereConditions.join(' AND ');
        }
        
        return {
            text: query,
            values: this.parameters.slice(0, -2) // Remove limit and offset parameters
        };
    }
}

// Usage with query parameters
async function getProducts(req, res) {
    const queryBuilder = new QueryBuilder('products')
        .select(['id', 'name', 'price', 'category', 'featured', 'created_at']);
    
    // Apply filters from query parameters
    if (req.filters.category) {
        queryBuilder.where('category', '=', req.filters.category);
    }
    
    if (req.filters.categories && req.filters.categories.length > 0) {
        queryBuilder.whereIn('category', req.filters.categories);
    }
    
    if (req.filters.minPrice || req.filters.maxPrice) {
        queryBuilder.whereRange('price', req.filters.minPrice, req.filters.maxPrice);
    }
    
    if (req.filters.featured !== undefined) {
        queryBuilder.where('featured', '=', req.filters.featured);
    }
    
    if (req.filters.createdAfter) {
        queryBuilder.whereDate('created_at', '>=', req.filters.createdAfter);
    }
    
    if (req.search && req.search.query) {
        queryBuilder.whereSearch(req.search.fields, req.search.query);
    }
    
    // Apply sorting and pagination
    queryBuilder
        .orderBy(req.sorting.field, req.sorting.order)
        .limit(req.pagination.limit)
        .offset(req.pagination.offset);
    
    try {
        const [dataResult, countResult] = await Promise.all([
            db.query(queryBuilder.build()),
            db.query(queryBuilder.buildCount())
        ]);
        
        const products = dataResult.rows;
        const totalCount = parseInt(countResult.rows[0].count);
        const totalPages = Math.ceil(totalCount / req.pagination.limit);
        
        res.json({
            data: products,
            pagination: {
                page: req.pagination.page,
                limit: req.pagination.limit,
                totalCount,
                totalPages,
                hasNextPage: req.pagination.page < totalPages,
                hasPrevPage: req.pagination.page > 1
            },
            filters: req.filters,
            sorting: req.sorting
        });
    } catch (error) {
        res.status(500).json({ error: error.message });
    }
}

module.exports = QueryBuilder;
```

---

## 🛡️ Query Parameter Validation

### Joi Validation Schema
```javascript
const Joi = require('joi');

// Query parameter validation schemas
const productQuerySchema = Joi.object({
    // Search
    q: Joi.string().max(100).trim(),
    searchFields: Joi.string().pattern(/^[a-zA-Z_,]+$/),
    
    // Filters
    category: Joi.string().valid('electronics', 'clothing', 'books', 'sports'),
    categories: Joi.string().pattern(/^[a-zA-Z,]+$/),
    minPrice: Joi.number().min(0),
    maxPrice: Joi.number().min(0),
    featured: Joi.boolean(),
    inStock: Joi.boolean(),
    tags: Joi.string().pattern(/^[a-zA-Z0-9,_-]+$/),
    
    // Date filters
    createdAfter: Joi.date().iso(),
    createdBefore: Joi.date().iso(),
    
    // Sorting
    sort: Joi.string().valid('name', 'price', 'created_at', 'rating', 'popularity'),
    order: Joi.string().valid('asc', 'desc'),
    
    // Pagination
    page: Joi.number().integer().min(1).max(1000),
    limit: Joi.number().integer().min(1).max(100)
});

const userQuerySchema = Joi.object({
    q: Joi.string().max(50).trim(),
    role: Joi.string().valid('admin', 'user', 'moderator'),
    active: Joi.boolean(),
    createdAfter: Joi.date().iso(),
    sort: Joi.string().valid('username', 'email', 'created_at', 'last_login'),
    order: Joi.string().valid('asc', 'desc'),
    page: Joi.number().integer().min(1).max(500),
    limit: Joi.number().integer().min(1).max(50)
});

// Validation middleware
function validateQuery(schema) {
    return (req, res, next) => {
        const { error, value } = schema.validate(req.query, {
            allowUnknown: false,
            stripUnknown: true,
            convert: true
        });
        
        if (error) {
            return res.status(400).json({
                error: 'Invalid query parameters',
                details: error.details.map(detail => ({
                    field: detail.path.join('.'),
                    message: detail.message,
                    value: detail.context.value
                }))
            });
        }
        
        // Replace req.query with validated and converted values
        req.query = value;
        next();
    };
}

// Usage
app.get('/products', 
    validateQuery(productQuerySchema),
    queryProcessor({
        allowedSortFields: ['name', 'price', 'created_at', 'rating'],
        allowedFilters: ['category', 'minPrice', 'maxPrice', 'featured']
    }),
    getProducts
);

app.get('/users',
    validateQuery(userQuerySchema),
    queryProcessor({
        allowedSortFields: ['username', 'email', 'created_at'],
        allowedFilters: ['active', 'role', 'createdAfter']
    }),
    getUsers
);
```

---

## 🔧 Real-World Examples

### E-commerce Product Search
```javascript
// routes/products.js
const express = require('express');
const router = express.Router();
const Joi = require('joi');
const QueryBuilder = require('../utils/queryBuilder');

const productSearchSchema = Joi.object({
    // Text search
    q: Joi.string().max(100).trim(),
    
    // Category filters
    category: Joi.string().valid('electronics', 'clothing', 'books', 'home', 'sports'),
    subcategory: Joi.string().max(50),
    brand: Joi.string().max(50),
    
    // Price filters
    minPrice: Joi.number().min(0),
    maxPrice: Joi.number().min(0),
    
    // Rating filters
    minRating: Joi.number().min(1).max(5),
    
    // Availability
    inStock: Joi.boolean(),
    freeShipping: Joi.boolean(),
    onSale: Joi.boolean(),
    
    // Sorting
    sort: Joi.string().valid('relevance', 'price', 'rating', 'newest', 'popularity'),
    order: Joi.string().valid('asc', 'desc'),
    
    // Pagination
    page: Joi.number().integer().min(1).default(1),
    limit: Joi.number().integer().min(1).max(60).default(20)
});

router.get('/search', validateQuery(productSearchSchema), async (req, res) => {
    try {
        const {
            q, category, subcategory, brand,
            minPrice, maxPrice, minRating,
            inStock, freeShipping, onSale,
            sort, order, page, limit
        } = req.query;
        
        const queryBuilder = new QueryBuilder('products')
            .select([
                'p.id', 'p.name', 'p.description', 'p.price', 'p.sale_price',
                'p.rating', 'p.review_count', 'p.stock_quantity', 'p.free_shipping',
                'c.name as category_name', 'b.name as brand_name',
                'p.image_url', 'p.created_at'
            ])
            .leftJoin('categories c', 'p.category_id = c.id')
            .leftJoin('brands b', 'p.brand_id = b.id');
        
        // Text search
        if (q) {
            queryBuilder.whereSearch(['p.name', 'p.description'], q);
        }
        
        // Category filters
        if (category) {
            queryBuilder.where('c.slug', '=', category);
        }
        
        if (subcategory) {
            queryBuilder.where('p.subcategory', '=', subcategory);
        }
        
        if (brand) {
            queryBuilder.where('b.slug', '=', brand);
        }
        
        // Price filters
        if (minPrice !== undefined || maxPrice !== undefined) {
            const priceField = 'COALESCE(p.sale_price, p.price)';
            queryBuilder.whereRange(priceField, minPrice, maxPrice);
        }
        
        // Rating filter
        if (minRating) {
            queryBuilder.where('p.rating', '>=', minRating);
        }
        
        // Availability filters
        if (inStock) {
            queryBuilder.where('p.stock_quantity', '>', 0);
        }
        
        if (freeShipping) {
            queryBuilder.where('p.free_shipping', '=', true);
        }
        
        if (onSale) {
            queryBuilder.where('p.sale_price', 'IS NOT', null);
        }
        
        // Sorting
        switch (sort) {
            case 'price':
                queryBuilder.orderBy('COALESCE(p.sale_price, p.price)', order || 'asc');
                break;
            case 'rating':
                queryBuilder.orderBy('p.rating', order || 'desc');
                break;
            case 'newest':
                queryBuilder.orderBy('p.created_at', 'desc');
                break;
            case 'popularity':
                queryBuilder.orderBy('p.review_count', 'desc');
                break;
            default: // relevance
                if (q) {
                    queryBuilder.orderBy('ts_rank(p.search_vector, plainto_tsquery($' + (queryBuilder.parameters.length + 1) + '))', 'desc');
                    queryBuilder.parameters.push(q);
                } else {
                    queryBuilder.orderBy('p.created_at', 'desc');
                }
        }
        
        // Pagination
        const offset = (page - 1) * limit;
        queryBuilder.limit(limit).offset(offset);
        
        // Execute queries
        const [products, totalCount, filters] = await Promise.all([
            db.query(queryBuilder.build()),
            db.query(queryBuilder.buildCount()),
            getAvailableFilters(req.query)
        ]);
        
        res.json({
            products: products.rows,
            pagination: {
                page,
                limit,
                totalCount: parseInt(totalCount.rows[0].count),
                totalPages: Math.ceil(totalCount.rows[0].count / limit)
            },
            filters,
            appliedFilters: req.query
        });
        
    } catch (error) {
        console.error('Product search error:', error);
        res.status(500).json({ error: 'Internal server error' });
    }
});

async function getAvailableFilters(appliedFilters) {
    // Build base query without the filter being calculated
    const baseQuery = new QueryBuilder('products p')
        .leftJoin('categories c', 'p.category_id = c.id')
        .leftJoin('brands b', 'p.brand_id = b.id');
    
    // Apply all filters except the one being calculated
    // ... (implementation details)
    
    const [categories, brands, priceRange] = await Promise.all([
        db.query('SELECT c.slug, c.name, COUNT(*) as count FROM categories c JOIN products p ON c.id = p.category_id GROUP BY c.id, c.slug, c.name ORDER BY count DESC'),
        db.query('SELECT b.slug, b.name, COUNT(*) as count FROM brands b JOIN products p ON b.id = p.brand_id GROUP BY b.id, b.slug, b.name ORDER BY count DESC'),
        db.query('SELECT MIN(COALESCE(sale_price, price)) as min_price, MAX(COALESCE(sale_price, price)) as max_price FROM products')
    ]);
    
    return {
        categories: categories.rows,
        brands: brands.rows,
        priceRange: priceRange.rows[0]
    };
}

module.exports = router;
```

### Blog Post Filtering
```javascript
// routes/posts.js
const blogQuerySchema = Joi.object({
    // Search
    q: Joi.string().max(100).trim(),
    
    // Content filters
    category: Joi.string().max(50),
    tags: Joi.string().pattern(/^[a-zA-Z0-9,_-]+$/),
    author: Joi.string().max(50),
    status: Joi.string().valid('published', 'draft', 'archived'),
    
    // Date filters
    publishedAfter: Joi.date().iso(),
    publishedBefore: Joi.date().iso(),
    year: Joi.number().integer().min(2020).max(new Date().getFullYear()),
    month: Joi.number().integer().min(1).max(12),
    
    // Content type
    hasVideo: Joi.boolean(),
    hasImages: Joi.boolean(),
    featured: Joi.boolean(),
    
    // Sorting
    sort: Joi.string().valid('published_at', 'title', 'views', 'comments_count'),
    order: Joi.string().valid('asc', 'desc'),
    
    // Pagination
    page: Joi.number().integer().min(1).default(1),
    limit: Joi.number().integer().min(1).max(50).default(10)
});

router.get('/', validateQuery(blogQuerySchema), async (req, res) => {
    try {
        const queryBuilder = new QueryBuilder('posts p')
            .select([
                'p.id', 'p.title', 'p.excerpt', 'p.slug',
                'p.published_at', 'p.views', 'p.featured_image',
                'u.username as author', 'c.name as category',
                '(SELECT COUNT(*) FROM comments WHERE post_id = p.id) as comments_count'
            ])
            .leftJoin('users u', 'p.author_id = u.id')
            .leftJoin('categories c', 'p.category_id = c.id')
            .where('p.status', '=', 'published');
        
        // Apply filters from query
        const { q, category, tags, author, publishedAfter, publishedBefore, year, month, hasVideo, hasImages, featured, sort, order, page, limit } = req.query;
        
        if (q) {
            queryBuilder.whereSearch(['p.title', 'p.excerpt', 'p.content'], q);
        }
        
        if (category) {
            queryBuilder.where('c.slug', '=', category);
        }
        
        if (tags) {
            const tagList = tags.split(',');
            queryBuilder.whereIn('p.id', 
                `SELECT DISTINCT post_id FROM post_tags pt 
                 JOIN tags t ON pt.tag_id = t.id 
                 WHERE t.slug = ANY($${queryBuilder.parameters.length + 1})`
            );
            queryBuilder.parameters.push(tagList);
        }
        
        if (author) {
            queryBuilder.where('u.username', '=', author);
        }
        
        if (publishedAfter) {
            queryBuilder.whereDate('p.published_at', '>=', new Date(publishedAfter));
        }
        
        if (publishedBefore) {
            queryBuilder.whereDate('p.published_at', '<=', new Date(publishedBefore));
        }
        
        if (year) {
            queryBuilder.where('EXTRACT(YEAR FROM p.published_at)', '=', year);
        }
        
        if (month) {
            queryBuilder.where('EXTRACT(MONTH FROM p.published_at)', '=', month);
        }
        
        if (hasVideo) {
            queryBuilder.where('p.video_url', 'IS NOT', null);
        }
        
        if (hasImages) {
            queryBuilder.where('p.featured_image', 'IS NOT', null);
        }
        
        if (featured) {
            queryBuilder.where('p.featured', '=', true);
        }
        
        // Sorting
        const sortField = sort || 'published_at';
        const sortOrder = order || 'desc';
        queryBuilder.orderBy(`p.${sortField}`, sortOrder);
        
        // Pagination
        const offset = (page - 1) * limit;
        queryBuilder.limit(limit).offset(offset);
        
        const [posts, totalCount] = await Promise.all([
            db.query(queryBuilder.build()),
            db.query(queryBuilder.buildCount())
        ]);
        
        res.json({
            posts: posts.rows,
            pagination: {
                page,
                limit,
                totalCount: parseInt(totalCount.rows[0].count),
                totalPages: Math.ceil(totalCount.rows[0].count / limit)
            }
        });
        
    } catch (error) {
        console.error('Blog search error:', error);
        res.status(500).json({ error: 'Internal server error' });
    }
});
```

---

## 🔗 Related Topics
- [[15-Query-Parameters]] - This file
- [[16-Joi-Validation]] - Parameter validation
- [[28-PostgreSQL-Integration]] - Database queries
- [[20-Pagination]] - Paginated results

---

*Back to [[00-Main-Index]]*
