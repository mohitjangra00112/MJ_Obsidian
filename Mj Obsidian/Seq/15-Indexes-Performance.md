# Indexes & Performance Optimization

## 🎯 Understanding Database Indexes

Indexes are data structures that improve query performance by creating shortcuts to your data. Think of them as a book's index that helps you quickly find specific information.

```mermaid
graph TB
    A[Query Without Index] --> B[Full Table Scan]
    B --> C[Check Every Row]
    C --> D[O(n) Time Complexity]
    
    E[Query With Index] --> F[Index Lookup]
    F --> G[Direct Row Access]
    G --> H[O(log n) Time Complexity]
    
    style A fill:#fecaca
    style E fill:#a7f3d0
    style D fill:#fecaca
    style H fill:#a7f3d0
```

## 📊 Index Types and Performance Impact

### 1. B-Tree Indexes (Default)

```javascript
// models/User.js - B-Tree index examples
const { Model, DataTypes } = require('sequelize');

class User extends Model {
  static associate(models) {
    User.hasMany(models.Post, { foreignKey: 'userId' });
  }
}

User.init({
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true
    // Automatically creates unique B-tree index
  },
  email: {
    type: DataTypes.STRING(255),
    allowNull: false,
    unique: true // Creates unique B-tree index
  },
  username: {
    type: DataTypes.STRING(50),
    allowNull: false,
    unique: true
  },
  firstName: {
    type: DataTypes.STRING(50),
    allowNull: false
  },
  lastName: {
    type: DataTypes.STRING(50),
    allowNull: false
  },
  isActive: {
    type: DataTypes.BOOLEAN,
    defaultValue: true
  },
  createdAt: {
    type: DataTypes.DATE,
    allowNull: false
  },
  updatedAt: {
    type: DataTypes.DATE,
    allowNull: false
  }
}, {
  sequelize,
  modelName: 'User',
  tableName: 'users',
  underscored: true,
  timestamps: true,
  
  indexes: [
    // Single column indexes
    {
      name: 'users_email_idx',
      unique: true,
      fields: ['email']
    },
    {
      name: 'users_username_idx',
      unique: true,
      fields: ['username']
    },
    {
      name: 'users_is_active_idx',
      fields: ['is_active'] // Good for filtering active users
    },
    {
      name: 'users_created_at_idx',
      fields: ['created_at'] // Good for date range queries
    },
    
    // Composite indexes
    {
      name: 'users_name_composite_idx',
      fields: ['first_name', 'last_name'] // Good for full name searches
    },
    {
      name: 'users_active_created_idx',
      fields: ['is_active', 'created_at'] // Good for active users by date
    },
    
    // Partial indexes (PostgreSQL)
    {
      name: 'users_active_email_partial_idx',
      fields: ['email'],
      where: {
        is_active: true
      }
    }
  ]
});

module.exports = User;
```

### 2. GIN and GiST Indexes (PostgreSQL)

```javascript
// models/Post.js - Advanced PostgreSQL indexes
const { Model, DataTypes } = require('sequelize');

class Post extends Model {
  static associate(models) {
    Post.belongsTo(models.User, { foreignKey: 'userId', as: 'author' });
    Post.hasMany(models.Comment, { foreignKey: 'postId' });
  }
}

Post.init({
  id: {
    type: DataTypes.INTEGER,
    primaryKey: true,
    autoIncrement: true
  },
  title: {
    type: DataTypes.STRING(200),
    allowNull: false
  },
  content: {
    type: DataTypes.TEXT,
    allowNull: false
  },
  excerpt: {
    type: DataTypes.TEXT
  },
  tags: {
    type: DataTypes.ARRAY(DataTypes.STRING),
    defaultValue: []
  },
  metadata: {
    type: DataTypes.JSONB,
    defaultValue: {}
  },
  status: {
    type: DataTypes.ENUM('draft', 'published', 'archived'),
    defaultValue: 'draft'
  },
  publishedAt: {
    type: DataTypes.DATE,
    allowNull: true
  },
  userId: {
    type: DataTypes.INTEGER,
    allowNull: false,
    references: {
      model: 'users',
      key: 'id'
    }
  }
}, {
  sequelize,
  modelName: 'Post',
  tableName: 'posts',
  underscored: true,
  timestamps: true,
  
  indexes: [
    // B-tree indexes
    {
      name: 'posts_user_id_idx',
      fields: ['user_id']
    },
    {
      name: 'posts_status_idx',
      fields: ['status']
    },
    {
      name: 'posts_published_at_idx',
      fields: ['published_at']
    },
    
    // GIN indexes for arrays and JSONB
    {
      name: 'posts_tags_gin_idx',
      fields: ['tags'],
      using: 'gin' // For array operations and text search
    },
    {
      name: 'posts_metadata_gin_idx',
      fields: ['metadata'],
      using: 'gin' // For JSONB operations
    },
    
    // Full text search index
    {
      name: 'posts_fulltext_idx',
      fields: [
        sequelize.fn('to_tsvector', 'english', 
          sequelize.fn('concat', 
            sequelize.col('title'), ' ', 
            sequelize.col('content')
          )
        )
      ],
      using: 'gin'
    },
    
    // Composite indexes for common query patterns
    {
      name: 'posts_user_status_published_idx',
      fields: ['user_id', 'status', 'published_at']
    },
    {
      name: 'posts_status_published_title_idx',
      fields: ['status', 'published_at', 'title']
    },
    
    // Partial indexes
    {
      name: 'posts_published_recent_idx',
      fields: ['published_at'],
      where: {
        status: 'published',
        published_at: {
          [sequelize.Op.gte]: sequelize.literal("NOW() - INTERVAL '30 days'")
        }
      }
    }
  ]
});

module.exports = Post;
```

### 3. Expression and Functional Indexes

```javascript
// migrations/create-expression-indexes.js - Advanced index patterns
'use strict';

module.exports = {
  async up(queryInterface, Sequelize) {
    // Case-insensitive search index
    await queryInterface.addIndex('users', {
      name: 'users_email_lower_idx',
      fields: [
        Sequelize.fn('LOWER', Sequelize.col('email'))
      ]
    });

    // Full name search index
    await queryInterface.addIndex('users', {
      name: 'users_full_name_idx',
      fields: [
        Sequelize.fn('CONCAT', 
          Sequelize.col('first_name'), ' ', 
          Sequelize.col('last_name')
        )
      ]
    });

    // Date part indexes
    await queryInterface.addIndex('posts', {
      name: 'posts_published_year_month_idx',
      fields: [
        Sequelize.fn('EXTRACT', 'YEAR FROM published_at'),
        Sequelize.fn('EXTRACT', 'MONTH FROM published_at')
      ]
    });

    // JSONB path index
    await queryInterface.addIndex('posts', {
      name: 'posts_metadata_category_idx',
      fields: [
        "((metadata->>'category'))"
      ]
    });

    // Trigram index for fuzzy text search
    await queryInterface.sequelize.query(`
      CREATE EXTENSION IF NOT EXISTS pg_trgm;
    `);
    
    await queryInterface.addIndex('posts', {
      name: 'posts_title_trgm_idx',
      fields: ['title'],
      using: 'gin',
      operator: 'gin_trgm_ops'
    });

    // Array length index
    await queryInterface.addIndex('posts', {
      name: 'posts_tags_length_idx',
      fields: [
        Sequelize.fn('ARRAY_LENGTH', Sequelize.col('tags'), 1)
      ]
    });
  },

  async down(queryInterface, Sequelize) {
    await queryInterface.removeIndex('users', 'users_email_lower_idx');
    await queryInterface.removeIndex('users', 'users_full_name_idx');
    await queryInterface.removeIndex('posts', 'posts_published_year_month_idx');
    await queryInterface.removeIndex('posts', 'posts_metadata_category_idx');
    await queryInterface.removeIndex('posts', 'posts_title_trgm_idx');
    await queryInterface.removeIndex('posts', 'posts_tags_length_idx');
  }
};
```

## 🚀 Query Optimization Strategies

### 1. Analyzing Query Performance

```javascript
// utils/performance-analyzer.js - Query performance analysis tools

class PerformanceAnalyzer {
  static async analyzeQuery(model, queryOptions, description = '') {
    console.log(`\n🔍 Analyzing Query: ${description}`);
    console.log('📋 Query Options:', JSON.stringify(queryOptions, null, 2));
    
    const startTime = process.hrtime.bigint();
    
    try {
      // Execute query with explain
      const explainResult = await sequelize.query(
        `EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON) ${this.generateSQL(model, queryOptions)}`,
        { type: sequelize.QueryTypes.SELECT }
      );
      
      // Execute actual query
      const result = await model.findAll(queryOptions);
      
      const endTime = process.hrtime.bigint();
      const executionTime = Number(endTime - startTime) / 1000000; // Convert to milliseconds
      
      // Analyze execution plan
      const plan = explainResult[0]['QUERY PLAN'][0];
      this.analyzeExecutionPlan(plan);
      
      console.log(`⚡ Execution Time: ${executionTime.toFixed(2)}ms`);
      console.log(`📊 Rows Returned: ${result.length}`);
      
      return {
        result,
        executionTime,
        plan,
        rowCount: result.length
      };
      
    } catch (error) {
      console.error('❌ Query analysis failed:', error.message);
      throw error;
    }
  }

  static analyzeExecutionPlan(plan) {
    console.log('\n📈 Execution Plan Analysis:');
    console.log(`🔧 Node Type: ${plan['Node Type']}`);
    console.log(`⏱️ Actual Time: ${plan['Actual Total Time']?.toFixed(2)}ms`);
    console.log(`📊 Rows: ${plan['Actual Rows']} (estimated: ${plan['Plan Rows']})`);
    
    if (plan['Actual Total Time'] > 100) {
      console.warn('⚠️ Slow query detected (>100ms)');
    }
    
    if (plan['Node Type'] === 'Seq Scan') {
      console.warn('⚠️ Sequential scan detected - consider adding index');
    }
    
    if (plan['Plans']) {
      plan['Plans'].forEach((subPlan, index) => {
        console.log(`\n  └─ Sub-plan ${index + 1}:`);
        console.log(`     Type: ${subPlan['Node Type']}`);
        console.log(`     Time: ${subPlan['Actual Total Time']?.toFixed(2)}ms`);
        console.log(`     Rows: ${subPlan['Actual Rows']}`);
      });
    }
  }

  static generateSQL(model, options) {
    // This is a simplified version - you'd need more sophisticated SQL generation
    const tableName = model.tableName;
    let sql = `SELECT * FROM ${tableName}`;
    
    if (options.where) {
      // Convert Sequelize where to SQL
      sql += ' WHERE ' + this.convertWhereToSQL(options.where);
    }
    
    if (options.order) {
      sql += ' ORDER BY ' + this.convertOrderToSQL(options.order);
    }
    
    if (options.limit) {
      sql += ` LIMIT ${options.limit}`;
    }
    
    return sql;
  }

  static convertWhereToSQL(where) {
    // Simplified conversion - real implementation would be more complex
    return Object.entries(where)
      .map(([key, value]) => `${key} = '${value}'`)
      .join(' AND ');
  }

  static convertOrderToSQL(order) {
    if (Array.isArray(order[0])) {
      return order.map(([field, direction]) => `${field} ${direction || 'ASC'}`).join(', ');
    }
    return `${order[0]} ${order[1] || 'ASC'}`;
  }

  static async findSlowQueries(durationThreshold = 100) {
    console.log(`🐌 Finding queries slower than ${durationThreshold}ms`);
    
    // PostgreSQL-specific query to find slow queries
    const slowQueries = await sequelize.query(`
      SELECT 
        query,
        calls,
        total_time,
        mean_time,
        max_time,
        min_time,
        stddev_time
      FROM pg_stat_statements 
      WHERE mean_time > $1
      ORDER BY total_time DESC
      LIMIT 20;
    `, {
      bind: [durationThreshold],
      type: sequelize.QueryTypes.SELECT
    });
    
    console.log('🐌 Slow Queries Found:');
    slowQueries.forEach((query, index) => {
      console.log(`\n${index + 1}. Query: ${query.query.substring(0, 100)}...`);
      console.log(`   Calls: ${query.calls}`);
      console.log(`   Avg Time: ${query.mean_time.toFixed(2)}ms`);
      console.log(`   Max Time: ${query.max_time.toFixed(2)}ms`);
      console.log(`   Total Time: ${query.total_time.toFixed(2)}ms`);
    });
    
    return slowQueries;
  }

  static async analyzeIndexUsage() {
    console.log('📊 Analyzing Index Usage');
    
    const indexStats = await sequelize.query(`
      SELECT 
        schemaname,
        tablename,
        indexname,
        idx_scan as index_scans,
        idx_tup_read as tuples_read,
        idx_tup_fetch as tuples_fetched
      FROM pg_stat_user_indexes
      ORDER BY idx_scan DESC;
    `, {
      type: sequelize.QueryTypes.SELECT
    });
    
    console.log('📈 Index Usage Statistics:');
    indexStats.forEach(stat => {
      console.log(`\n📋 ${stat.tablename}.${stat.indexname}`);
      console.log(`   Scans: ${stat.index_scans}`);
      console.log(`   Tuples Read: ${stat.tuples_read}`);
      console.log(`   Tuples Fetched: ${stat.tuples_fetched}`);
      
      if (stat.index_scans === 0) {
        console.warn('   ⚠️ Unused index - consider dropping');
      }
    });
    
    return indexStats;
  }

  static async findMissingIndexes() {
    console.log('🔍 Finding Missing Indexes');
    
    // Find tables with sequential scans
    const seqScans = await sequelize.query(`
      SELECT 
        schemaname,
        tablename,
        seq_scan,
        seq_tup_read,
        n_tup_ins + n_tup_upd + n_tup_del as total_writes,
        seq_tup_read / GREATEST(seq_scan, 1) as avg_seq_read
      FROM pg_stat_user_tables
      WHERE seq_scan > 0
      ORDER BY seq_tup_read DESC;
    `, {
      type: sequelize.QueryTypes.SELECT
    });
    
    console.log('🔍 Tables with Sequential Scans:');
    seqScans.forEach(scan => {
      if (scan.avg_seq_read > 1000) {
        console.log(`\n⚠️ ${scan.tablename}:`);
        console.log(`   Sequential Scans: ${scan.seq_scan}`);
        console.log(`   Avg Rows per Scan: ${scan.avg_seq_read}`);
        console.log(`   Total Writes: ${scan.total_writes}`);
        console.log('   💡 Consider adding indexes for frequent WHERE clauses');
      }
    });
    
    return seqScans;
  }
}

module.exports = PerformanceAnalyzer;
```

### 2. Query Optimization Examples

```javascript
// services/optimized-queries.js - Real-world optimization examples

class OptimizedQueries {
  /**
   * Example 1: Optimizing user search
   * Before: Full table scan on multiple columns
   * After: Composite index + proper query structure
   */
  static async searchUsers(searchTerm, filters = {}) {
    const { isActive = true, limit = 20, offset = 0 } = filters;
    
    // ❌ BAD: This query is slow and inefficient
    const badQuery = async () => {
      return await User.findAll({
        where: {
          [sequelize.Op.or]: [
            { firstName: { [sequelize.Op.iLike]: `%${searchTerm}%` } },
            { lastName: { [sequelize.Op.iLike]: `%${searchTerm}%` } },
            { email: { [sequelize.Op.iLike]: `%${searchTerm}%` } },
            { username: { [sequelize.Op.iLike]: `%${searchTerm}%` } }
          ],
          isActive
        },
        limit,
        offset,
        order: [['createdAt', 'DESC']]
      });
    };
    
    // ✅ GOOD: Optimized query with proper indexing
    const optimizedQuery = async () => {
      // Use full-text search for better performance
      if (searchTerm.length >= 3) {
        return await sequelize.query(`
          SELECT u.* FROM users u
          WHERE 
            is_active = :isActive
            AND (
              to_tsvector('english', first_name || ' ' || last_name || ' ' || email || ' ' || username) 
              @@ plainto_tsquery('english', :searchTerm)
              OR first_name ILIKE :searchTermPrefix
              OR last_name ILIKE :searchTermPrefix
              OR email ILIKE :searchTermPrefix
              OR username ILIKE :searchTermPrefix
            )
          ORDER BY 
            CASE 
              WHEN username ILIKE :searchTermPrefix THEN 1
              WHEN email ILIKE :searchTermPrefix THEN 2
              WHEN first_name ILIKE :searchTermPrefix THEN 3
              WHEN last_name ILIKE :searchTermPrefix THEN 4
              ELSE 5
            END,
            created_at DESC
          LIMIT :limit OFFSET :offset
        `, {
          replacements: {
            searchTerm,
            searchTermPrefix: `${searchTerm}%`,
            isActive,
            limit,
            offset
          },
          type: sequelize.QueryTypes.SELECT,
          model: User,
          mapToModel: true
        });
      } else {
        // For short search terms, use prefix matching
        return await User.findAll({
          where: {
            isActive,
            [sequelize.Op.or]: [
              { firstName: { [sequelize.Op.iLike]: `${searchTerm}%` } },
              { lastName: { [sequelize.Op.iLike]: `${searchTerm}%` } },
              { username: { [sequelize.Op.iLike]: `${searchTerm}%` } }
            ]
          },
          limit,
          offset,
          order: [['createdAt', 'DESC']]
        });
      }
    };
    
    return await optimizedQuery();
  }

  /**
   * Example 2: Optimizing post feed with pagination
   * Avoiding OFFSET for large datasets
   */
  static async getUserFeed(userId, cursorId = null, limit = 20) {
    // ❌ BAD: OFFSET-based pagination is slow for large datasets
    const badPagination = async (page = 1) => {
      const offset = (page - 1) * limit;
      
      return await Post.findAll({
        where: {
          userId: {
            [sequelize.Op.in]: sequelize.literal(`
              (SELECT following_id FROM user_followers WHERE follower_id = ${userId})
            `)
          },
          status: 'published'
        },
        include: [
          { model: User, as: 'author', attributes: ['firstName', 'lastName', 'username'] }
        ],
        order: [['publishedAt', 'DESC']],
        limit,
        offset // This becomes slow for large offsets
      });
    };
    
    // ✅ GOOD: Cursor-based pagination
    const optimizedPagination = async () => {
      const whereClause = {
        userId: {
          [sequelize.Op.in]: sequelize.literal(`
            (SELECT following_id FROM user_followers WHERE follower_id = ${userId})
          `)
        },
        status: 'published'
      };
      
      // Add cursor condition for pagination
      if (cursorId) {
        whereClause.id = { [sequelize.Op.lt]: cursorId };
      }
      
      return await Post.findAll({
        where: whereClause,
        include: [
          { 
            model: User, 
            as: 'author', 
            attributes: ['id', 'firstName', 'lastName', 'username'] 
          }
        ],
        order: [['id', 'DESC']], // Use ID for consistent ordering
        limit: limit + 1 // Get one extra to check if there are more
      });
    };
    
    const posts = await optimizedPagination();
    
    // Check if there are more posts
    const hasMore = posts.length > limit;
    if (hasMore) {
      posts.pop(); // Remove the extra post
    }
    
    const nextCursor = posts.length > 0 ? posts[posts.length - 1].id : null;
    
    return {
      posts,
      hasMore,
      nextCursor
    };
  }

  /**
   * Example 3: Optimizing complex aggregations
   * Using window functions and proper indexing
   */
  static async getPostStatistics(userId, dateRange) {
    const { startDate, endDate } = dateRange;
    
    // ❌ BAD: Multiple separate queries
    const badApproach = async () => {
      const totalPosts = await Post.count({
        where: { userId, createdAt: { [sequelize.Op.between]: [startDate, endDate] } }
      });
      
      const publishedPosts = await Post.count({
        where: { 
          userId, 
          status: 'published',
          createdAt: { [sequelize.Op.between]: [startDate, endDate] }
        }
      });
      
      const avgViews = await Post.findOne({
        where: { userId, createdAt: { [sequelize.Op.between]: [startDate, endDate] } },
        attributes: [[sequelize.fn('AVG', sequelize.col('viewCount')), 'avgViews']]
      });
      
      // ... more separate queries
      
      return { totalPosts, publishedPosts, avgViews: avgViews.avgViews };
    };
    
    // ✅ GOOD: Single optimized query with window functions
    const optimizedApproach = async () => {
      const stats = await sequelize.query(`
        WITH post_stats AS (
          SELECT 
            COUNT(*) as total_posts,
            COUNT(*) FILTER (WHERE status = 'published') as published_posts,
            COUNT(*) FILTER (WHERE status = 'draft') as draft_posts,
            AVG(view_count) as avg_views,
            MAX(view_count) as max_views,
            SUM(view_count) as total_views,
            COUNT(DISTINCT DATE(created_at)) as active_days
          FROM posts
          WHERE 
            user_id = :userId
            AND created_at BETWEEN :startDate AND :endDate
        ),
        monthly_breakdown AS (
          SELECT 
            DATE_TRUNC('month', created_at) as month,
            COUNT(*) as posts_count,
            AVG(view_count) as avg_monthly_views
          FROM posts
          WHERE 
            user_id = :userId
            AND created_at BETWEEN :startDate AND :endDate
          GROUP BY DATE_TRUNC('month', created_at)
          ORDER BY month
        )
        SELECT 
          ps.*,
          COALESCE(
            ARRAY_AGG(
              JSON_BUILD_OBJECT(
                'month', mb.month,
                'posts_count', mb.posts_count,
                'avg_views', mb.avg_monthly_views
              )
            ) FILTER (WHERE mb.month IS NOT NULL),
            ARRAY[]::json[]
          ) as monthly_breakdown
        FROM post_stats ps
        LEFT JOIN monthly_breakdown mb ON true
        GROUP BY ps.total_posts, ps.published_posts, ps.draft_posts, 
                 ps.avg_views, ps.max_views, ps.total_views, ps.active_days
      `, {
        replacements: { userId, startDate, endDate },
        type: sequelize.QueryTypes.SELECT
      });
      
      return stats[0];
    };
    
    return await optimizedApproach();
  }

  /**
   * Example 4: Optimizing N+1 queries
   * Using proper includes and batch loading
   */
  static async getPostsWithComments(postIds) {
    // ❌ BAD: N+1 query problem
    const badApproach = async () => {
      const posts = await Post.findAll({
        where: { id: { [sequelize.Op.in]: postIds } }
      });
      
      // This creates N additional queries
      for (const post of posts) {
        post.comments = await Comment.findAll({
          where: { postId: post.id },
          include: [{ model: User, as: 'author' }]
        });
        
        post.commentCount = post.comments.length;
      }
      
      return posts;
    };
    
    // ✅ GOOD: Single query with proper includes
    const optimizedApproach = async () => {
      return await Post.findAll({
        where: { id: { [sequelize.Op.in]: postIds } },
        include: [
          {
            model: User,
            as: 'author',
            attributes: ['id', 'firstName', 'lastName', 'username']
          },
          {
            model: Comment,
            as: 'comments',
            include: [
              {
                model: User,
                as: 'author',
                attributes: ['id', 'firstName', 'lastName', 'username']
              }
            ],
            order: [['createdAt', 'DESC']],
            limit: 5 // Only get recent comments
          }
        ],
        order: [
          ['publishedAt', 'DESC'],
          [{ model: Comment, as: 'comments' }, 'createdAt', 'DESC']
        ]
      });
    };
    
    return await optimizedApproach();
  }

  /**
   * Example 5: Optimizing JSON queries
   * Proper indexing and querying of JSONB fields
   */
  static async searchPostsByMetadata(filters) {
    const { category, tags, author, dateRange } = filters;
    
    // ✅ Optimized JSONB queries with proper indexing
    const whereClause = {
      status: 'published'
    };
    
    // Use GIN index for JSONB operations
    if (category) {
      whereClause[sequelize.Op.and] = sequelize.where(
        sequelize.fn('jsonb_extract_path_text', sequelize.col('metadata'), 'category'),
        category
      );
    }
    
    // Use GIN index for array operations
    if (tags && tags.length > 0) {
      whereClause.tags = {
        [sequelize.Op.overlap]: tags // Uses GIN index
      };
    }
    
    if (dateRange) {
      whereClause.publishedAt = {
        [sequelize.Op.between]: [dateRange.start, dateRange.end]
      };
    }
    
    return await Post.findAll({
      where: whereClause,
      include: [
        {
          model: User,
          as: 'author',
          attributes: ['id', 'firstName', 'lastName', 'username'],
          where: author ? { username: { [sequelize.Op.iLike]: `%${author}%` } } : undefined
        }
      ],
      order: [['publishedAt', 'DESC']],
      limit: 50
    });
  }
}

module.exports = OptimizedQueries;
```

### 3. Connection Pool Optimization

```javascript
// config/database-optimized.js - Optimized database configuration

const config = {
  development: {
    username: process.env.DB_USERNAME,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    dialect: 'postgres',
    
    // Connection Pool Configuration
    pool: {
      max: 20,        // Maximum number of connections
      min: 5,         // Minimum number of connections
      acquire: 30000, // Maximum time to get connection (ms)
      idle: 10000,    // Maximum time connection can be idle (ms)
      evict: 10000,   // Time interval to run eviction (ms)
      handleDisconnects: true
    },
    
    // Query Optimization
    dialectOptions: {
      // Use read replicas for read operations
      readReplicas: [
        {
          host: process.env.DB_READ_HOST_1,
          username: process.env.DB_USERNAME,
          password: process.env.DB_PASSWORD
        },
        {
          host: process.env.DB_READ_HOST_2,
          username: process.env.DB_USERNAME,
          password: process.env.DB_PASSWORD
        }
      ],
      
      // PostgreSQL specific optimizations
      ssl: process.env.NODE_ENV === 'production' ? {
        require: true,
        rejectUnauthorized: false
      } : false,
      
      // Connection settings
      statement_timeout: 60000,   // 60 seconds
      query_timeout: 60000,       // 60 seconds
      connectionTimeoutMillis: 5000,
      
      // Performance settings
      application_name: 'blog_app',
      
      // Prepared statement cache
      preparedStatements: true,
      max_prepared_statements: 100
    },
    
    // Sequelize options
    logging: (sql, timing) => {
      if (timing > 100) { // Log slow queries
        console.log(`🐌 Slow Query (${timing}ms):`, sql);
      }
    },
    
    benchmark: true,              // Show execution time
    logQueryParameters: process.env.NODE_ENV === 'development',
    
    // Retry configuration
    retry: {
      max: 3,
      timeout: 5000,
      match: [
        'ETIMEDOUT',
        'EHOSTUNREACH',
        'ECONNRESET',
        'ECONNREFUSED',
        'EHOSTDOWN',
        'ENETDOWN',
        'ENETUNREACH',
        'EAI_AGAIN'
      ]
    },
    
    // Define models with optimizations
    define: {
      charset: 'utf8mb4',
      collate: 'utf8mb4_unicode_ci',
      underscored: true,
      freezeTableName: true,
      timestamps: true,
      paranoid: false,
      
      // Index hints
      indexes: [],
      
      // Validation options
      validate: {},
      
      // Hook optimizations
      hooks: {},
      
      // Scope optimizations
      scopes: {
        // Default scope for performance
        defaultScope: {
          attributes: { exclude: ['deletedAt'] }
        }
      }
    },
    
    // Transaction defaults
    transactionType: 'IMMEDIATE',
    isolationLevel: 'READ_COMMITTED'
  },

  test: {
    // Test database configuration
    username: process.env.DB_TEST_USERNAME,
    password: process.env.DB_TEST_PASSWORD,
    database: process.env.DB_TEST_NAME,
    host: process.env.DB_TEST_HOST,
    port: process.env.DB_TEST_PORT,
    dialect: 'postgres',
    
    // Faster settings for testing
    pool: {
      max: 5,
      min: 1,
      acquire: 10000,
      idle: 5000
    },
    
    logging: false, // Disable logging in tests
    
    dialectOptions: {
      statement_timeout: 10000,
      query_timeout: 10000
    }
  },

  production: {
    username: process.env.DB_USERNAME,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    dialect: 'postgres',
    
    // Production pool configuration
    pool: {
      max: 50,        // Higher for production traffic
      min: 10,
      acquire: 60000, // Longer timeout for production
      idle: 300000,   // 5 minutes idle timeout
      evict: 10000,
      handleDisconnects: true,
      
      // Health checks
      validate: (client) => {
        return client.query('SELECT 1').then(() => true).catch(() => false);
      }
    },
    
    dialectOptions: {
      ssl: {
        require: true,
        rejectUnauthorized: false
      },
      
      // Production-specific timeouts
      statement_timeout: 120000,  // 2 minutes
      query_timeout: 120000,
      idle_in_transaction_session_timeout: 300000, // 5 minutes
      
      // Connection limits
      max_connections: 200,
      
      // Performance settings
      shared_preload_libraries: 'pg_stat_statements',
      track_activity_query_size: 2048,
      log_min_duration_statement: 1000, // Log queries > 1 second
      
      // Memory settings
      shared_buffers: '256MB',
      effective_cache_size: '1GB',
      work_mem: '4MB',
      maintenance_work_mem: '64MB'
    },
    
    logging: (sql, timing) => {
      // Log slow queries and errors in production
      if (timing > 1000) {
        console.error(`🐌 Very Slow Query (${timing}ms):`, sql.substring(0, 200));
      }
    },
    
    benchmark: false, // Disable in production for performance
    
    // Retry configuration for production
    retry: {
      max: 5,
      timeout: 10000,
      match: [
        'ETIMEDOUT',
        'EHOSTUNREACH',
        'ECONNRESET',
        'ECONNREFUSED',
        'EHOSTDOWN',
        'ENETDOWN',
        'ENETUNREACH',
        'EAI_AGAIN',
        'SequelizeConnectionError',
        'SequelizeConnectionRefusedError',
        'SequelizeHostNotFoundError',
        'SequelizeHostNotReachableError',
        'SequelizeInvalidConnectionError',
        'SequelizeConnectionTimedOutError'
      ]
    }
  }
};

module.exports = config;
```

### 4. Monitoring and Alerting

```javascript
// utils/performance-monitor.js - Real-time performance monitoring

class PerformanceMonitor {
  constructor() {
    this.metrics = {
      queries: new Map(),
      connections: new Map(),
      slowQueries: [],
      errors: []
    };
    
    this.thresholds = {
      slowQuery: 100,     // ms
      verySlowQuery: 1000, // ms
      connectionPool: 0.8  // 80% of max connections
    };
    
    this.setupMonitoring();
  }

  setupMonitoring() {
    // Monitor Sequelize events
    sequelize.addHook('beforeQuery', (options, query) => {
      query.startTime = Date.now();
      query.id = this.generateQueryId();
    });

    sequelize.addHook('afterQuery', (options, query) => {
      const duration = Date.now() - query.startTime;
      this.recordQuery(query, duration, options);
    });

    // Monitor connection pool
    setInterval(() => {
      this.checkConnectionPool();
    }, 30000); // Every 30 seconds

    // Monitor slow queries
    setInterval(() => {
      this.analyzeSlowQueries();
    }, 300000); // Every 5 minutes
  }

  generateQueryId() {
    return Date.now() + '-' + Math.random().toString(36).substr(2, 9);
  }

  recordQuery(query, duration, options) {
    const queryType = this.getQueryType(query.sql);
    const queryKey = this.normalizeQuery(query.sql);
    
    // Update metrics
    if (!this.metrics.queries.has(queryKey)) {
      this.metrics.queries.set(queryKey, {
        count: 0,
        totalTime: 0,
        avgTime: 0,
        maxTime: 0,
        minTime: Infinity,
        type: queryType
      });
    }
    
    const metric = this.metrics.queries.get(queryKey);
    metric.count++;
    metric.totalTime += duration;
    metric.avgTime = metric.totalTime / metric.count;
    metric.maxTime = Math.max(metric.maxTime, duration);
    metric.minTime = Math.min(metric.minTime, duration);
    
    // Record slow queries
    if (duration > this.thresholds.slowQuery) {
      this.recordSlowQuery(query, duration, options);
    }
    
    // Alert on very slow queries
    if (duration > this.thresholds.verySlowQuery) {
      this.alertVerySlowQuery(query, duration);
    }
  }

  getQueryType(sql) {
    const normalizedSQL = sql.trim().toUpperCase();
    if (normalizedSQL.startsWith('SELECT')) return 'SELECT';
    if (normalizedSQL.startsWith('INSERT')) return 'INSERT';
    if (normalizedSQL.startsWith('UPDATE')) return 'UPDATE';
    if (normalizedSQL.startsWith('DELETE')) return 'DELETE';
    return 'OTHER';
  }

  normalizeQuery(sql) {
    // Normalize query by removing parameters and formatting
    return sql
      .replace(/\$\d+/g, '?')          // Replace $1, $2 with ?
      .replace(/'\w+'/g, '?')          // Replace string literals
      .replace(/\d+/g, '?')            // Replace numbers
      .replace(/\s+/g, ' ')            // Normalize whitespace
      .trim();
  }

  recordSlowQuery(query, duration, options) {
    this.metrics.slowQueries.push({
      sql: query.sql,
      duration,
      timestamp: new Date(),
      model: options.model?.name,
      type: this.getQueryType(query.sql)
    });
    
    // Keep only last 100 slow queries
    if (this.metrics.slowQueries.length > 100) {
      this.metrics.slowQueries.shift();
    }
    
    console.warn(`🐌 Slow Query (${duration}ms):`, query.sql.substring(0, 100));
  }

  alertVerySlowQuery(query, duration) {
    console.error(`🚨 VERY SLOW QUERY ALERT (${duration}ms):`, query.sql.substring(0, 200));
    
    // In production, you might want to send this to a monitoring service
    if (process.env.NODE_ENV === 'production') {
      this.sendToMonitoringService({
        type: 'very_slow_query',
        duration,
        sql: query.sql,
        timestamp: new Date()
      });
    }
  }

  async checkConnectionPool() {
    try {
      const pool = sequelize.connectionManager.pool;
      const totalConnections = pool.size;
      const idleConnections = pool.available;
      const activeConnections = pool.using;
      const waitingCount = pool.pending;
      
      const utilizationRatio = activeConnections / pool.options.max;
      
      console.log(`📊 Connection Pool Status:`);
      console.log(`   Total: ${totalConnections}/${pool.options.max}`);
      console.log(`   Active: ${activeConnections}`);
      console.log(`   Idle: ${idleConnections}`);
      console.log(`   Waiting: ${waitingCount}`);
      console.log(`   Utilization: ${(utilizationRatio * 100).toFixed(1)}%`);
      
      // Alert if utilization is too high
      if (utilizationRatio > this.thresholds.connectionPool) {
        console.warn(`⚠️ High connection pool utilization: ${(utilizationRatio * 100).toFixed(1)}%`);
        
        if (process.env.NODE_ENV === 'production') {
          this.sendToMonitoringService({
            type: 'high_connection_utilization',
            utilization: utilizationRatio,
            activeConnections,
            maxConnections: pool.options.max,
            timestamp: new Date()
          });
        }
      }
      
      this.metrics.connections.set(Date.now(), {
        total: totalConnections,
        active: activeConnections,
        idle: idleConnections,
        waiting: waitingCount,
        utilization: utilizationRatio
      });
      
    } catch (error) {
      console.error('❌ Failed to check connection pool:', error);
    }
  }

  analyzeSlowQueries() {
    if (this.metrics.slowQueries.length === 0) return;
    
    console.log('\n📊 Slow Query Analysis (Last 5 minutes):');
    
    // Group by query type
    const byType = this.metrics.slowQueries.reduce((acc, query) => {
      if (!acc[query.type]) acc[query.type] = [];
      acc[query.type].push(query);
      return acc;
    }, {});
    
    Object.entries(byType).forEach(([type, queries]) => {
      const avgDuration = queries.reduce((sum, q) => sum + q.duration, 0) / queries.length;
      const maxDuration = Math.max(...queries.map(q => q.duration));
      
      console.log(`\n${type} Queries:`);
      console.log(`  Count: ${queries.length}`);
      console.log(`  Avg Duration: ${avgDuration.toFixed(2)}ms`);
      console.log(`  Max Duration: ${maxDuration}ms`);
    });
    
    // Find most problematic queries
    const groupedQueries = this.metrics.slowQueries.reduce((acc, query) => {
      const normalized = this.normalizeQuery(query.sql);
      if (!acc[normalized]) acc[normalized] = [];
      acc[normalized].push(query);
      return acc;
    }, {});
    
    const problematicQueries = Object.entries(groupedQueries)
      .filter(([_, queries]) => queries.length > 3)
      .sort(([, a], [, b]) => b.length - a.length)
      .slice(0, 5);
    
    if (problematicQueries.length > 0) {
      console.log('\n🚨 Most Problematic Queries:');
      problematicQueries.forEach(([sql, queries], index) => {
        const avgDuration = queries.reduce((sum, q) => sum + q.duration, 0) / queries.length;
        console.log(`\n${index + 1}. Query (${queries.length} occurrences, avg ${avgDuration.toFixed(2)}ms):`);
        console.log(`   ${sql.substring(0, 100)}...`);
      });
    }
  }

  generateReport() {
    console.log('\n📈 Performance Report');
    console.log('===================');
    
    // Query statistics
    console.log('\n📊 Query Statistics:');
    const sortedQueries = Array.from(this.metrics.queries.entries())
      .sort(([, a], [, b]) => b.totalTime - a.totalTime)
      .slice(0, 10);
    
    sortedQueries.forEach(([sql, metric], index) => {
      console.log(`\n${index + 1}. ${metric.type} (${metric.count} executions)`);
      console.log(`   Avg: ${metric.avgTime.toFixed(2)}ms`);
      console.log(`   Max: ${metric.maxTime}ms`);
      console.log(`   Total: ${metric.totalTime.toFixed(2)}ms`);
      console.log(`   Query: ${sql.substring(0, 80)}...`);
    });
    
    // Connection pool trends
    const recentConnections = Array.from(this.metrics.connections.entries())
      .slice(-10)
      .map(([_, data]) => data);
    
    if (recentConnections.length > 0) {
      const avgUtilization = recentConnections.reduce((sum, conn) => sum + conn.utilization, 0) / recentConnections.length;
      console.log(`\n🔗 Connection Pool:`);
      console.log(`   Average Utilization: ${(avgUtilization * 100).toFixed(1)}%`);
      console.log(`   Current Active: ${recentConnections[recentConnections.length - 1].active}`);
    }
    
    // Recommendations
    this.generateRecommendations();
  }

  generateRecommendations() {
    console.log('\n💡 Performance Recommendations:');
    
    const recommendations = [];
    
    // Analyze slow queries
    const slowSelectQueries = this.metrics.slowQueries.filter(q => q.type === 'SELECT');
    if (slowSelectQueries.length > 10) {
      recommendations.push('Consider adding indexes for frequently used WHERE clauses');
    }
    
    // Analyze connection usage
    const recentConnections = Array.from(this.metrics.connections.values()).slice(-5);
    if (recentConnections.length > 0) {
      const avgUtilization = recentConnections.reduce((sum, conn) => sum + conn.utilization, 0) / recentConnections.length;
      
      if (avgUtilization > 0.8) {
        recommendations.push('Consider increasing connection pool size');
      } else if (avgUtilization < 0.2) {
        recommendations.push('Consider reducing connection pool size');
      }
    }
    
    // Query pattern analysis
    const insertQueries = Array.from(this.metrics.queries.values()).filter(q => q.type === 'INSERT');
    const totalInserts = insertQueries.reduce((sum, q) => sum + q.count, 0);
    
    if (totalInserts > 1000) {
      recommendations.push('Consider using bulk operations for multiple inserts');
    }
    
    if (recommendations.length === 0) {
      recommendations.push('No specific recommendations at this time');
    }
    
    recommendations.forEach((rec, index) => {
      console.log(`   ${index + 1}. ${rec}`);
    });
  }

  sendToMonitoringService(data) {
    // In a real application, you would send this to a monitoring service
    // like DataDog, New Relic, or custom alerting system
    console.log('📤 Sending to monitoring service:', data);
  }

  reset() {
    this.metrics = {
      queries: new Map(),
      connections: new Map(),
      slowQueries: [],
      errors: []
    };
  }
}

// Singleton instance
const performanceMonitor = new PerformanceMonitor();

module.exports = performanceMonitor;
```

## 🎯 Key Takeaways

1. **Index Strategy**: Create indexes based on your query patterns, not assumptions
2. **Composite Indexes**: Order columns by selectivity (most selective first)
3. **Avoid Over-Indexing**: Too many indexes slow down writes
4. **Monitor Performance**: Regularly analyze slow queries and index usage
5. **Connection Pooling**: Configure appropriate pool sizes for your workload
6. **Query Optimization**: Use EXPLAIN ANALYZE to understand query execution
7. **Caching**: Implement caching for frequently accessed data

## 🚀 What's Next?

Now that you understand performance optimization, let's explore [[16-Authentication|Authentication & Authorization]] to secure your high-performance application!

---

## 🔗 Related Topics
- [[14-Transactions|Transactions & Data Integrity]]
- [[16-Authentication|Authentication & Authorization]]
- [[18-Testing|Testing Strategies]]
