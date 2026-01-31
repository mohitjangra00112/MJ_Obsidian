# Caching Strategies

**Navigation**: [[25-Security-Best-Practices]] | [[00-Main-Index]] | Next: [[27-Performance-Monitoring]]

---

## 🚀 Caching Strategies for Node.js Backend

**Caching** is essential for improving application performance and reducing database load. This guide covers various caching strategies from memory cache to distributed Redis implementations.

---

## 💾 In-Memory Caching

### Node.js Memory Cache
```javascript
// utils/MemoryCache.js
class MemoryCache {
    constructor(defaultTTL = 300000) { // 5 minutes default
        this.cache = new Map();
        this.timers = new Map();
        this.defaultTTL = defaultTTL;
        this.stats = {
            hits: 0,
            misses: 0,
            sets: 0,
            deletes: 0
        };
    }
    
    // Set cache item with TTL
    set(key, value, ttl = this.defaultTTL) {
        try {
            // Clear existing timer if key exists
            if (this.timers.has(key)) {
                clearTimeout(this.timers.get(key));
            }
            
            // Store value
            this.cache.set(key, {
                value,
                createdAt: Date.now(),
                ttl,
                accessCount: 0
            });
            
            // Set expiration timer
            if (ttl > 0) {
                const timer = setTimeout(() => {
                    this.delete(key);
                }, ttl);
                
                this.timers.set(key, timer);
            }
            
            this.stats.sets++;
            return true;
        } catch (error) {
            console.error('Memory cache set error:', error);
            return false;
        }
    }
    
    // Get cache item
    get(key) {
        try {
            const item = this.cache.get(key);
            
            if (!item) {
                this.stats.misses++;
                return null;
            }
            
            // Check if expired
            if (item.ttl > 0 && Date.now() - item.createdAt > item.ttl) {
                this.delete(key);
                this.stats.misses++;
                return null;
            }
            
            // Update access count
            item.accessCount++;
            this.stats.hits++;
            
            return item.value;
        } catch (error) {
            console.error('Memory cache get error:', error);
            return null;
        }
    }
    
    // Delete cache item
    delete(key) {
        try {
            const deleted = this.cache.delete(key);
            
            if (this.timers.has(key)) {
                clearTimeout(this.timers.get(key));
                this.timers.delete(key);
            }
            
            if (deleted) {
                this.stats.deletes++;
            }
            
            return deleted;
        } catch (error) {
            console.error('Memory cache delete error:', error);
            return false;
        }
    }
    
    // Check if key exists
    has(key) {
        const item = this.cache.get(key);
        if (!item) return false;
        
        // Check if expired
        if (item.ttl > 0 && Date.now() - item.createdAt > item.ttl) {
            this.delete(key);
            return false;
        }
        
        return true;
    }
    
    // Clear all cache
    clear() {
        // Clear all timers
        for (const timer of this.timers.values()) {
            clearTimeout(timer);
        }
        
        this.cache.clear();
        this.timers.clear();
        
        // Reset stats
        this.stats = {
            hits: 0,
            misses: 0,
            sets: 0,
            deletes: 0
        };
    }
    
    // Get cache statistics
    getStats() {
        const total = this.stats.hits + this.stats.misses;
        const hitRate = total > 0 ? (this.stats.hits / total * 100).toFixed(2) : 0;
        
        return {
            ...this.stats,
            size: this.cache.size,
            hitRate: `${hitRate}%`,
            memoryUsage: this.getMemoryUsage()
        };
    }
    
    // Get memory usage estimation
    getMemoryUsage() {
        let size = 0;
        
        for (const [key, item] of this.cache) {
            size += JSON.stringify(key).length;
            size += JSON.stringify(item.value).length;
            size += 64; // Overhead estimation
        }
        
        return {
            bytes: size,
            kb: (size / 1024).toFixed(2),
            mb: (size / 1024 / 1024).toFixed(2)
        };
    }
    
    // Get all keys
    keys() {
        return Array.from(this.cache.keys());
    }
    
    // Get cache item info
    getInfo(key) {
        const item = this.cache.get(key);
        if (!item) return null;
        
        return {
            key,
            createdAt: new Date(item.createdAt),
            ttl: item.ttl,
            accessCount: item.accessCount,
            timeLeft: item.ttl > 0 ? Math.max(0, item.ttl - (Date.now() - item.createdAt)) : 'infinite'
        };
    }
    
    // Cleanup expired items
    cleanup() {
        let cleaned = 0;
        const now = Date.now();
        
        for (const [key, item] of this.cache) {
            if (item.ttl > 0 && now - item.createdAt > item.ttl) {
                this.delete(key);
                cleaned++;
            }
        }
        
        return cleaned;
    }
    
    // Auto cleanup interval
    startAutoCleanup(interval = 60000) { // 1 minute
        setInterval(() => {
            const cleaned = this.cleanup();
            if (cleaned > 0) {
                console.log(`Memory cache cleaned ${cleaned} expired items`);
            }
        }, interval);
    }
}

// Create global instance
const memoryCache = new MemoryCache();
memoryCache.startAutoCleanup();

module.exports = memoryCache;
```

### LRU Cache Implementation
```javascript
// utils/LRUCache.js
class LRUCache {
    constructor(maxSize = 1000, defaultTTL = 300000) {
        this.maxSize = maxSize;
        this.defaultTTL = defaultTTL;
        this.cache = new Map();
        this.stats = {
            hits: 0,
            misses: 0,
            evictions: 0
        };
    }
    
    set(key, value, ttl = this.defaultTTL) {
        const now = Date.now();
        
        // Remove existing key to update position
        if (this.cache.has(key)) {
            this.cache.delete(key);
        }
        
        // Evict least recently used if at capacity
        if (this.cache.size >= this.maxSize) {
            const firstKey = this.cache.keys().next().value;
            this.cache.delete(firstKey);
            this.stats.evictions++;
        }
        
        // Add new item (will be at the end = most recent)
        this.cache.set(key, {
            value,
            createdAt: now,
            expiresAt: ttl > 0 ? now + ttl : Infinity,
            accessCount: 0
        });
        
        return true;
    }
    
    get(key) {
        const item = this.cache.get(key);
        
        if (!item) {
            this.stats.misses++;
            return null;
        }
        
        // Check expiration
        if (Date.now() > item.expiresAt) {
            this.cache.delete(key);
            this.stats.misses++;
            return null;
        }
        
        // Move to end (most recent)
        this.cache.delete(key);
        this.cache.set(key, {
            ...item,
            accessCount: item.accessCount + 1
        });
        
        this.stats.hits++;
        return item.value;
    }
    
    delete(key) {
        return this.cache.delete(key);
    }
    
    has(key) {
        const item = this.cache.get(key);
        if (!item) return false;
        
        if (Date.now() > item.expiresAt) {
            this.cache.delete(key);
            return false;
        }
        
        return true;
    }
    
    clear() {
        this.cache.clear();
        this.stats = { hits: 0, misses: 0, evictions: 0 };
    }
    
    getStats() {
        const total = this.stats.hits + this.stats.misses;
        const hitRate = total > 0 ? (this.stats.hits / total * 100).toFixed(2) : 0;
        
        return {
            ...this.stats,
            size: this.cache.size,
            maxSize: this.maxSize,
            hitRate: `${hitRate}%`,
            usage: `${((this.cache.size / this.maxSize) * 100).toFixed(1)}%`
        };
    }
    
    // Get least recently used items
    getLRUItems(count = 5) {
        const items = [];
        let i = 0;
        
        for (const [key, item] of this.cache) {
            if (i >= count) break;
            
            items.push({
                key,
                accessCount: item.accessCount,
                createdAt: new Date(item.createdAt),
                position: i + 1
            });
            i++;
        }
        
        return items;
    }
}

module.exports = LRUCache;
```

---

## 🔄 Redis Caching

### Redis Cache Service
```javascript
// services/RedisCacheService.js
const Redis = require('ioredis');

class RedisCacheService {
    constructor() {
        this.redis = new Redis({
            host: process.env.REDIS_HOST || 'localhost',
            port: process.env.REDIS_PORT || 6379,
            password: process.env.REDIS_PASSWORD,
            db: process.env.REDIS_DB || 0,
            retryDelayOnFailover: 100,
            maxRetriesPerRequest: 3,
            lazyConnect: true,
            keepAlive: 30000,
            connectTimeout: 10000,
            commandTimeout: 5000
        });
        
        this.redis.on('connect', () => {
            console.log('Redis connected successfully');
        });
        
        this.redis.on('error', (error) => {
            console.error('Redis connection error:', error);
        });
        
        this.redis.on('close', () => {
            console.log('Redis connection closed');
        });
        
        this.defaultTTL = 300; // 5 minutes in seconds
        this.keyPrefix = process.env.CACHE_KEY_PREFIX || 'app:cache:';
    }
    
    // Generate cache key with prefix
    getKey(key) {
        return `${this.keyPrefix}${key}`;
    }
    
    // Set cache item
    async set(key, value, ttl = this.defaultTTL) {
        try {
            const cacheKey = this.getKey(key);
            const serializedValue = JSON.stringify(value);
            
            if (ttl > 0) {
                await this.redis.setex(cacheKey, ttl, serializedValue);
            } else {
                await this.redis.set(cacheKey, serializedValue);
            }
            
            return true;
        } catch (error) {
            console.error('Redis set error:', error);
            return false;
        }
    }
    
    // Get cache item
    async get(key) {
        try {
            const cacheKey = this.getKey(key);
            const value = await this.redis.get(cacheKey);
            
            if (value === null) {
                return null;
            }
            
            return JSON.parse(value);
        } catch (error) {
            console.error('Redis get error:', error);
            return null;
        }
    }
    
    // Delete cache item
    async delete(key) {
        try {
            const cacheKey = this.getKey(key);
            const result = await this.redis.del(cacheKey);
            return result > 0;
        } catch (error) {
            console.error('Redis delete error:', error);
            return false;
        }
    }
    
    // Check if key exists
    async has(key) {
        try {
            const cacheKey = this.getKey(key);
            const exists = await this.redis.exists(cacheKey);
            return exists === 1;
        } catch (error) {
            console.error('Redis exists error:', error);
            return false;
        }
    }
    
    // Set multiple items
    async setMultiple(items, ttl = this.defaultTTL) {
        try {
            const pipeline = this.redis.pipeline();
            
            for (const [key, value] of Object.entries(items)) {
                const cacheKey = this.getKey(key);
                const serializedValue = JSON.stringify(value);
                
                if (ttl > 0) {
                    pipeline.setex(cacheKey, ttl, serializedValue);
                } else {
                    pipeline.set(cacheKey, serializedValue);
                }
            }
            
            await pipeline.exec();
            return true;
        } catch (error) {
            console.error('Redis setMultiple error:', error);
            return false;
        }
    }
    
    // Get multiple items
    async getMultiple(keys) {
        try {
            const cacheKeys = keys.map(key => this.getKey(key));
            const values = await this.redis.mget(...cacheKeys);
            
            const result = {};
            keys.forEach((key, index) => {
                if (values[index] !== null) {
                    result[key] = JSON.parse(values[index]);
                }
            });
            
            return result;
        } catch (error) {
            console.error('Redis getMultiple error:', error);
            return {};
        }
    }
    
    // Delete multiple items
    async deleteMultiple(keys) {
        try {
            const cacheKeys = keys.map(key => this.getKey(key));
            const result = await this.redis.del(...cacheKeys);
            return result;
        } catch (error) {
            console.error('Redis deleteMultiple error:', error);
            return 0;
        }
    }
    
    // Clear all cache items with prefix
    async clear() {
        try {
            const pattern = `${this.keyPrefix}*`;
            const keys = await this.redis.keys(pattern);
            
            if (keys.length > 0) {
                await this.redis.del(...keys);
            }
            
            return keys.length;
        } catch (error) {
            console.error('Redis clear error:', error);
            return 0;
        }
    }
    
    // Get cache statistics
    async getStats() {
        try {
            const info = await this.redis.info('memory');
            const dbInfo = await this.redis.info('keyspace');
            
            const pattern = `${this.keyPrefix}*`;
            const keys = await this.redis.keys(pattern);
            
            return {
                totalKeys: keys.length,
                memoryUsed: this.parseMemoryInfo(info),
                keyspace: this.parseKeyspaceInfo(dbInfo),
                connection: {
                    status: this.redis.status,
                    host: this.redis.options.host,
                    port: this.redis.options.port,
                    db: this.redis.options.db
                }
            };
        } catch (error) {
            console.error('Redis stats error:', error);
            return null;
        }
    }
    
    // Parse memory info from Redis INFO command
    parseMemoryInfo(info) {
        const lines = info.split('\r\n');
        const memory = {};
        
        lines.forEach(line => {
            if (line.includes('used_memory_human:')) {
                memory.used = line.split(':')[1];
            }
            if (line.includes('used_memory_peak_human:')) {
                memory.peak = line.split(':')[1];
            }
        });
        
        return memory;
    }
    
    // Parse keyspace info
    parseKeyspaceInfo(info) {
        const lines = info.split('\r\n');
        const keyspace = {};
        
        lines.forEach(line => {
            if (line.startsWith('db')) {
                const [db, stats] = line.split(':');
                keyspace[db] = stats;
            }
        });
        
        return keyspace;
    }
    
    // Get TTL for a key
    async getTTL(key) {
        try {
            const cacheKey = this.getKey(key);
            const ttl = await this.redis.ttl(cacheKey);
            return ttl;
        } catch (error) {
            console.error('Redis TTL error:', error);
            return -1;
        }
    }
    
    // Extend TTL for a key
    async extendTTL(key, additionalSeconds) {
        try {
            const cacheKey = this.getKey(key);
            const currentTTL = await this.redis.ttl(cacheKey);
            
            if (currentTTL > 0) {
                const newTTL = currentTTL + additionalSeconds;
                await this.redis.expire(cacheKey, newTTL);
                return newTTL;
            }
            
            return currentTTL;
        } catch (error) {
            console.error('Redis extend TTL error:', error);
            return -1;
        }
    }
    
    // Increment counter
    async increment(key, amount = 1, ttl = this.defaultTTL) {
        try {
            const cacheKey = this.getKey(key);
            const value = await this.redis.incrby(cacheKey, amount);
            
            // Set TTL if this is a new key
            if (value === amount && ttl > 0) {
                await this.redis.expire(cacheKey, ttl);
            }
            
            return value;
        } catch (error) {
            console.error('Redis increment error:', error);
            return null;
        }
    }
    
    // Decrement counter
    async decrement(key, amount = 1) {
        try {
            const cacheKey = this.getKey(key);
            const value = await this.redis.decrby(cacheKey, amount);
            return value;
        } catch (error) {
            console.error('Redis decrement error:', error);
            return null;
        }
    }
    
    // Add to set
    async addToSet(key, ...members) {
        try {
            const cacheKey = this.getKey(key);
            const result = await this.redis.sadd(cacheKey, ...members);
            return result;
        } catch (error) {
            console.error('Redis add to set error:', error);
            return 0;
        }
    }
    
    // Get set members
    async getSetMembers(key) {
        try {
            const cacheKey = this.getKey(key);
            const members = await this.redis.smembers(cacheKey);
            return members;
        } catch (error) {
            console.error('Redis get set members error:', error);
            return [];
        }
    }
    
    // Disconnect from Redis
    async disconnect() {
        try {
            await this.redis.disconnect();
            console.log('Redis disconnected');
        } catch (error) {
            console.error('Redis disconnect error:', error);
        }
    }
}

module.exports = new RedisCacheService();
```

---

## 🎯 Cache Strategies

### Cache-Aside Pattern
```javascript
// services/CacheAsideService.js
const RedisCache = require('./RedisCacheService');

class CacheAsideService {
    constructor() {
        this.cache = RedisCache;
        this.defaultTTL = 300; // 5 minutes
    }
    
    // Get data with cache-aside pattern
    async get(key, dataLoader, options = {}) {
        const {
            ttl = this.defaultTTL,
            forceRefresh = false,
            fallbackOnError = true
        } = options;
        
        try {
            // Check cache first (unless force refresh)
            if (!forceRefresh) {
                const cachedData = await this.cache.get(key);
                if (cachedData !== null) {
                    return {
                        data: cachedData,
                        source: 'cache',
                        cached: true
                    };
                }
            }
            
            // Load data from source
            const freshData = await dataLoader();
            
            // Store in cache
            await this.cache.set(key, freshData, ttl);
            
            return {
                data: freshData,
                source: 'database',
                cached: false
            };
            
        } catch (error) {
            console.error('Cache-aside get error:', error);
            
            if (fallbackOnError) {
                // Try to get stale data from cache
                const staleData = await this.cache.get(key);
                if (staleData !== null) {
                    return {
                        data: staleData,
                        source: 'cache_stale',
                        cached: true,
                        error: error.message
                    };
                }
            }
            
            throw error;
        }
    }
    
    // Update data and cache
    async update(key, data, options = {}) {
        const { ttl = this.defaultTTL } = options;
        
        try {
            // Update cache
            await this.cache.set(key, data, ttl);
            
            return {
                success: true,
                cached: true
            };
        } catch (error) {
            console.error('Cache-aside update error:', error);
            throw error;
        }
    }
    
    // Delete from cache
    async delete(key) {
        try {
            await this.cache.delete(key);
            return { success: true };
        } catch (error) {
            console.error('Cache-aside delete error:', error);
            throw error;
        }
    }
    
    // Get multiple items with cache-aside
    async getMultiple(keys, dataLoader, options = {}) {
        const { ttl = this.defaultTTL } = options;
        
        try {
            // Get all from cache first
            const cachedData = await this.cache.getMultiple(keys);
            const missingKeys = keys.filter(key => !cachedData.hasOwnProperty(key));
            
            // If all data is cached, return it
            if (missingKeys.length === 0) {
                return {
                    data: cachedData,
                    source: 'cache',
                    cached: true
                };
            }
            
            // Load missing data
            const freshData = await dataLoader(missingKeys);
            
            // Cache the fresh data
            if (Object.keys(freshData).length > 0) {
                await this.cache.setMultiple(freshData, ttl);
            }
            
            // Combine cached and fresh data
            const combinedData = { ...cachedData, ...freshData };
            
            return {
                data: combinedData,
                source: 'mixed',
                cached: Object.keys(cachedData).length > 0,
                cacheHits: Object.keys(cachedData).length,
                cacheMisses: missingKeys.length
            };
            
        } catch (error) {
            console.error('Cache-aside getMultiple error:', error);
            throw error;
        }
    }
}

module.exports = new CacheAsideService();
```

### Write-Through Cache
```javascript
// services/WriteThroughService.js
const RedisCache = require('./RedisCacheService');

class WriteThroughService {
    constructor() {
        this.cache = RedisCache;
        this.defaultTTL = 300;
    }
    
    // Write through cache - write to both cache and database
    async set(key, data, dataWriter, options = {}) {
        const { ttl = this.defaultTTL } = options;
        
        try {
            // Write to database first
            const dbResult = await dataWriter(data);
            
            // If database write succeeds, write to cache
            await this.cache.set(key, dbResult, ttl);
            
            return {
                success: true,
                data: dbResult,
                source: 'database',
                cached: true
            };
            
        } catch (error) {
            console.error('Write-through error:', error);
            throw error;
        }
    }
    
    // Read from cache first, then database
    async get(key, dataLoader, options = {}) {
        const { ttl = this.defaultTTL } = options;
        
        try {
            // Try cache first
            const cachedData = await this.cache.get(key);
            if (cachedData !== null) {
                return {
                    data: cachedData,
                    source: 'cache',
                    cached: true
                };
            }
            
            // Load from database
            const dbData = await dataLoader();
            
            // Cache the result
            await this.cache.set(key, dbData, ttl);
            
            return {
                data: dbData,
                source: 'database',
                cached: false
            };
            
        } catch (error) {
            console.error('Write-through get error:', error);
            throw error;
        }
    }
    
    // Update both cache and database
    async update(key, data, dataUpdater, options = {}) {
        const { ttl = this.defaultTTL } = options;
        
        try {
            // Update database first
            const dbResult = await dataUpdater(data);
            
            // Update cache
            await this.cache.set(key, dbResult, ttl);
            
            return {
                success: true,
                data: dbResult,
                updated: true
            };
            
        } catch (error) {
            console.error('Write-through update error:', error);
            throw error;
        }
    }
    
    // Delete from both cache and database
    async delete(key, dataDeleter) {
        try {
            // Delete from database first
            await dataDeleter();
            
            // Delete from cache
            await this.cache.delete(key);
            
            return { success: true };
            
        } catch (error) {
            console.error('Write-through delete error:', error);
            throw error;
        }
    }
}

module.exports = new WriteThroughService();
```

### Write-Behind Cache
```javascript
// services/WriteBehindService.js
const RedisCache = require('./RedisCacheService');
const EventEmitter = require('events');

class WriteBehindService extends EventEmitter {
    constructor() {
        super();
        this.cache = RedisCache;
        this.writeQueue = new Map();
        this.batchSize = 10;
        this.flushInterval = 5000; // 5 seconds
        this.defaultTTL = 300;
        
        // Start background flush process
        this.startBackgroundFlush();
    }
    
    // Write to cache immediately, queue for database write
    async set(key, data, dataWriter, options = {}) {
        const { ttl = this.defaultTTL, immediate = false } = options;
        
        try {
            // Write to cache immediately
            await this.cache.set(key, data, ttl);
            
            if (immediate) {
                // Immediate database write
                await dataWriter(data);
                return {
                    success: true,
                    data,
                    cached: true,
                    queued: false
                };
            } else {
                // Queue for background write
                this.queueWrite(key, data, dataWriter);
                return {
                    success: true,
                    data,
                    cached: true,
                    queued: true
                };
            }
            
        } catch (error) {
            console.error('Write-behind set error:', error);
            throw error;
        }
    }
    
    // Queue write operation
    queueWrite(key, data, dataWriter) {
        this.writeQueue.set(key, {
            data,
            dataWriter,
            timestamp: Date.now(),
            retries: 0
        });
        
        this.emit('writeQueued', { key, queueSize: this.writeQueue.size });
    }
    
    // Start background flush process
    startBackgroundFlush() {
        setInterval(async () => {
            await this.flushWrites();
        }, this.flushInterval);
    }
    
    // Flush queued writes to database
    async flushWrites() {
        if (this.writeQueue.size === 0) return;
        
        const batchEntries = Array.from(this.writeQueue.entries()).slice(0, this.batchSize);
        const promises = [];
        
        for (const [key, writeOp] of batchEntries) {
            promises.push(this.executeWrite(key, writeOp));
        }
        
        await Promise.allSettled(promises);
    }
    
    // Execute individual write operation
    async executeWrite(key, writeOp) {
        try {
            await writeOp.dataWriter(writeOp.data);
            
            // Remove from queue on success
            this.writeQueue.delete(key);
            
            this.emit('writeSuccess', { key, data: writeOp.data });
            
        } catch (error) {
            console.error(`Write-behind flush error for key ${key}:`, error);
            
            writeOp.retries++;
            writeOp.lastError = error.message;
            
            // Remove after max retries
            if (writeOp.retries >= 3) {
                this.writeQueue.delete(key);
                this.emit('writeFailure', { key, error: error.message, retries: writeOp.retries });
            } else {
                this.emit('writeRetry', { key, error: error.message, retries: writeOp.retries });
            }
        }
    }
    
    // Force flush all queued writes
    async forceFlush() {
        const allEntries = Array.from(this.writeQueue.entries());
        const promises = [];
        
        for (const [key, writeOp] of allEntries) {
            promises.push(this.executeWrite(key, writeOp));
        }
        
        await Promise.allSettled(promises);
        
        return {
            processed: allEntries.length,
            remaining: this.writeQueue.size
        };
    }
    
    // Get from cache
    async get(key, dataLoader, options = {}) {
        const { ttl = this.defaultTTL } = options;
        
        try {
            const cachedData = await this.cache.get(key);
            if (cachedData !== null) {
                return {
                    data: cachedData,
                    source: 'cache',
                    cached: true
                };
            }
            
            // Load from database and cache
            const dbData = await dataLoader();
            await this.cache.set(key, dbData, ttl);
            
            return {
                data: dbData,
                source: 'database',
                cached: false
            };
            
        } catch (error) {
            console.error('Write-behind get error:', error);
            throw error;
        }
    }
    
    // Get queue statistics
    getQueueStats() {
        const now = Date.now();
        let oldestWrite = now;
        let newestWrite = 0;
        let totalRetries = 0;
        
        for (const writeOp of this.writeQueue.values()) {
            if (writeOp.timestamp < oldestWrite) {
                oldestWrite = writeOp.timestamp;
            }
            if (writeOp.timestamp > newestWrite) {
                newestWrite = writeOp.timestamp;
            }
            totalRetries += writeOp.retries;
        }
        
        return {
            queueSize: this.writeQueue.size,
            oldestAge: this.writeQueue.size > 0 ? now - oldestWrite : 0,
            newestAge: this.writeQueue.size > 0 ? now - newestWrite : 0,
            totalRetries,
            batchSize: this.batchSize,
            flushInterval: this.flushInterval
        };
    }
}

module.exports = new WriteBehindService();
```

---

## 🔄 Cache Invalidation

### Cache Invalidation Service
```javascript
// services/CacheInvalidationService.js
const RedisCache = require('./RedisCacheService');
const EventEmitter = require('events');

class CacheInvalidationService extends EventEmitter {
    constructor() {
        super();
        this.cache = RedisCache;
        this.dependencies = new Map(); // key -> Set of dependent keys
        this.tags = new Map(); // tag -> Set of keys
    }
    
    // Set cache with dependencies
    async setWithDependencies(key, value, dependencies = [], tags = [], ttl = 300) {
        try {
            // Set the cache value
            await this.cache.set(key, value, ttl);
            
            // Register dependencies
            for (const dep of dependencies) {
                if (!this.dependencies.has(dep)) {
                    this.dependencies.set(dep, new Set());
                }
                this.dependencies.get(dep).add(key);
            }
            
            // Register tags
            for (const tag of tags) {
                if (!this.tags.has(tag)) {
                    this.tags.set(tag, new Set());
                }
                this.tags.get(tag).add(key);
            }
            
            this.emit('cacheSet', { key, dependencies, tags });
            
            return true;
        } catch (error) {
            console.error('Cache set with dependencies error:', error);
            return false;
        }
    }
    
    // Invalidate cache key and its dependents
    async invalidate(key) {
        try {
            const invalidatedKeys = new Set([key]);
            
            // Recursively find all dependent keys
            this.findDependentKeys(key, invalidatedKeys);
            
            // Delete all invalidated keys
            const keysArray = Array.from(invalidatedKeys);
            if (keysArray.length > 0) {
                await this.cache.deleteMultiple(keysArray);
            }
            
            // Clean up dependency maps
            this.cleanupDependencies(invalidatedKeys);
            
            this.emit('cacheInvalidated', { 
                originalKey: key, 
                invalidatedKeys: keysArray 
            });
            
            return keysArray;
        } catch (error) {
            console.error('Cache invalidation error:', error);
            throw error;
        }
    }
    
    // Find all keys that depend on the given key
    findDependentKeys(key, invalidatedKeys) {
        if (this.dependencies.has(key)) {
            const dependentKeys = this.dependencies.get(key);
            
            for (const depKey of dependentKeys) {
                if (!invalidatedKeys.has(depKey)) {
                    invalidatedKeys.add(depKey);
                    // Recursively find dependencies of dependent keys
                    this.findDependentKeys(depKey, invalidatedKeys);
                }
            }
        }
    }
    
    // Invalidate by tag
    async invalidateByTag(tag) {
        try {
            if (!this.tags.has(tag)) {
                return [];
            }
            
            const taggedKeys = Array.from(this.tags.get(tag));
            const allInvalidatedKeys = new Set();
            
            // Invalidate each key and its dependents
            for (const key of taggedKeys) {
                const invalidatedKeys = await this.invalidate(key);
                invalidatedKeys.forEach(k => allInvalidatedKeys.add(k));
            }
            
            // Clean up tag mapping
            this.tags.delete(tag);
            
            const result = Array.from(allInvalidatedKeys);
            
            this.emit('tagInvalidated', { 
                tag, 
                invalidatedKeys: result 
            });
            
            return result;
        } catch (error) {
            console.error('Tag invalidation error:', error);
            throw error;
        }
    }
    
    // Invalidate multiple tags
    async invalidateByTags(tags) {
        const allInvalidatedKeys = new Set();
        
        for (const tag of tags) {
            const invalidatedKeys = await this.invalidateByTag(tag);
            invalidatedKeys.forEach(key => allInvalidatedKeys.add(key));
        }
        
        return Array.from(allInvalidatedKeys);
    }
    
    // Clean up dependency maps
    cleanupDependencies(invalidatedKeys) {
        // Remove invalidated keys from dependency sets
        for (const [depKey, dependentKeys] of this.dependencies) {
            for (const invalidatedKey of invalidatedKeys) {
                dependentKeys.delete(invalidatedKey);
            }
            
            // Remove empty dependency sets
            if (dependentKeys.size === 0) {
                this.dependencies.delete(depKey);
            }
        }
        
        // Remove invalidated keys from tag sets
        for (const [tag, taggedKeys] of this.tags) {
            for (const invalidatedKey of invalidatedKeys) {
                taggedKeys.delete(invalidatedKey);
            }
            
            // Remove empty tag sets
            if (taggedKeys.size === 0) {
                this.tags.delete(tag);
            }
        }
    }
    
    // Time-based invalidation
    async scheduleInvalidation(key, delay) {
        setTimeout(async () => {
            try {
                await this.invalidate(key);
                this.emit('scheduledInvalidation', { key, delay });
            } catch (error) {
                console.error('Scheduled invalidation error:', error);
            }
        }, delay);
    }
    
    // Pattern-based invalidation
    async invalidateByPattern(pattern) {
        try {
            const keys = await this.cache.redis.keys(`${this.cache.keyPrefix}${pattern}`);
            const invalidatedKeys = [];
            
            for (const fullKey of keys) {
                const key = fullKey.replace(this.cache.keyPrefix, '');
                const dependentKeys = await this.invalidate(key);
                invalidatedKeys.push(...dependentKeys);
            }
            
            const uniqueKeys = [...new Set(invalidatedKeys)];
            
            this.emit('patternInvalidated', { 
                pattern, 
                invalidatedKeys: uniqueKeys 
            });
            
            return uniqueKeys;
        } catch (error) {
            console.error('Pattern invalidation error:', error);
            throw error;
        }
    }
    
    // Get dependency information
    getDependencyInfo(key) {
        return {
            dependencies: this.dependencies.get(key) ? 
                Array.from(this.dependencies.get(key)) : [],
            dependsOn: this.findKeysThatDependOn(key),
            tags: this.findTagsForKey(key)
        };
    }
    
    // Find keys that depend on this key
    findKeysThatDependOn(targetKey) {
        const dependsOn = [];
        
        for (const [key, dependentKeys] of this.dependencies) {
            if (dependentKeys.has(targetKey)) {
                dependsOn.push(key);
            }
        }
        
        return dependsOn;
    }
    
    // Find tags for a key
    findTagsForKey(targetKey) {
        const tags = [];
        
        for (const [tag, taggedKeys] of this.tags) {
            if (taggedKeys.has(targetKey)) {
                tags.push(tag);
            }
        }
        
        return tags;
    }
    
    // Get cache statistics
    getStats() {
        return {
            totalDependencies: this.dependencies.size,
            totalTags: this.tags.size,
            dependencyMappings: Array.from(this.dependencies.entries()).map(([key, deps]) => ({
                key,
                dependentCount: deps.size
            })),
            tagMappings: Array.from(this.tags.entries()).map(([tag, keys]) => ({
                tag,
                keyCount: keys.size
            }))
        };
    }
}

module.exports = new CacheInvalidationService();
```

---

## 🚀 HTTP Caching

### HTTP Cache Middleware
```javascript
// middleware/httpCacheMiddleware.js
const crypto = require('crypto');

class HTTPCacheMiddleware {
    constructor(defaultMaxAge = 300) {
        this.defaultMaxAge = defaultMaxAge;
    }
    
    // ETag generation
    generateETag(data) {
        return crypto
            .createHash('md5')
            .update(JSON.stringify(data))
            .digest('hex');
    }
    
    // Cache control headers
    setCacheHeaders(res, options = {}) {
        const {
            maxAge = this.defaultMaxAge,
            public = true,
            staleWhileRevalidate = 0,
            staleIfError = 0,
            mustRevalidate = false,
            noCache = false,
            noStore = false
        } = options;
        
        if (noStore) {
            res.set('Cache-Control', 'no-store');
            return;
        }
        
        if (noCache) {
            res.set('Cache-Control', 'no-cache, must-revalidate');
            return;
        }
        
        const directives = [];
        
        if (public) {
            directives.push('public');
        } else {
            directives.push('private');
        }
        
        directives.push(`max-age=${maxAge}`);
        
        if (staleWhileRevalidate > 0) {
            directives.push(`stale-while-revalidate=${staleWhileRevalidate}`);
        }
        
        if (staleIfError > 0) {
            directives.push(`stale-if-error=${staleIfError}`);
        }
        
        if (mustRevalidate) {
            directives.push('must-revalidate');
        }
        
        res.set('Cache-Control', directives.join(', '));
        
        // Set Expires header
        const expires = new Date(Date.now() + maxAge * 1000);
        res.set('Expires', expires.toUTCString());
    }
    
    // ETag middleware
    etag(options = {}) {
        return (req, res, next) => {
            const originalJson = res.json;
            
            res.json = function(data) {
                // Generate ETag
                const etag = this.generateETag(data);
                res.set('ETag', `"${etag}"`);
                
                // Check If-None-Match header
                const ifNoneMatch = req.get('If-None-Match');
                if (ifNoneMatch && ifNoneMatch.includes(etag)) {
                    return res.status(304).end();
                }
                
                // Set cache headers
                this.setCacheHeaders(res, options);
                
                return originalJson.call(res, data);
            }.bind(this);
            
            next();
        };
    }
    
    // Last-Modified middleware
    lastModified(getLastModified) {
        return (req, res, next) => {
            const lastModified = getLastModified(req);
            
            if (lastModified) {
                const lastModifiedDate = new Date(lastModified);
                res.set('Last-Modified', lastModifiedDate.toUTCString());
                
                // Check If-Modified-Since header
                const ifModifiedSince = req.get('If-Modified-Since');
                if (ifModifiedSince) {
                    const ifModifiedSinceDate = new Date(ifModifiedSince);
                    if (lastModifiedDate <= ifModifiedSinceDate) {
                        return res.status(304).end();
                    }
                }
            }
            
            next();
        };
    }
    
    // Static file caching
    static(options = {}) {
        const {
            maxAge = 86400, // 1 day
            immutable = false
        } = options;
        
        return (req, res, next) => {
            // Only apply to GET requests
            if (req.method !== 'GET') {
                return next();
            }
            
            // Set cache headers for static files
            const directives = ['public', `max-age=${maxAge}`];
            
            if (immutable) {
                directives.push('immutable');
            }
            
            res.set('Cache-Control', directives.join(', '));
            
            next();
        };
    }
    
    // No cache for sensitive routes
    noCache() {
        return (req, res, next) => {
            res.set({
                'Cache-Control': 'no-store, no-cache, must-revalidate, proxy-revalidate',
                'Pragma': 'no-cache',
                'Expires': '0'
            });
            
            next();
        };
    }
    
    // Conditional caching based on user authentication
    conditionalCache(options = {}) {
        return (req, res, next) => {
            if (req.user) {
                // Authenticated user - private cache
                this.setCacheHeaders(res, {
                    ...options,
                    public: false,
                    maxAge: options.authenticatedMaxAge || 60
                });
            } else {
                // Anonymous user - public cache
                this.setCacheHeaders(res, {
                    ...options,
                    public: true,
                    maxAge: options.anonymousMaxAge || this.defaultMaxAge
                });
            }
            
            next();
        };
    }
    
    // Vary header middleware
    vary(headers = []) {
        return (req, res, next) => {
            if (headers.length > 0) {
                res.vary(headers.join(', '));
            }
            next();
        };
    }
}

module.exports = new HTTPCacheMiddleware();
```

---

## 📊 Cache Performance Monitoring

### Cache Metrics Service
```javascript
// services/CacheMetricsService.js
const EventEmitter = require('events');

class CacheMetricsService extends EventEmitter {
    constructor() {
        super();
        this.metrics = {
            hits: 0,
            misses: 0,
            sets: 0,
            deletes: 0,
            errors: 0,
            totalRequests: 0
        };
        
        this.performanceData = [];
        this.keyMetrics = new Map(); // Per-key metrics
        this.startTime = Date.now();
        
        // Clean old performance data every hour
        setInterval(() => {
            this.cleanOldPerformanceData();
        }, 60 * 60 * 1000);
    }
    
    // Record cache hit
    recordHit(key, responseTime = 0) {
        this.metrics.hits++;
        this.metrics.totalRequests++;
        
        this.recordKeyMetric(key, 'hit', responseTime);
        this.recordPerformanceData('hit', responseTime);
        
        this.emit('hit', { key, responseTime });
    }
    
    // Record cache miss
    recordMiss(key, responseTime = 0) {
        this.metrics.misses++;
        this.metrics.totalRequests++;
        
        this.recordKeyMetric(key, 'miss', responseTime);
        this.recordPerformanceData('miss', responseTime);
        
        this.emit('miss', { key, responseTime });
    }
    
    // Record cache set operation
    recordSet(key, responseTime = 0) {
        this.metrics.sets++;
        
        this.recordKeyMetric(key, 'set', responseTime);
        this.recordPerformanceData('set', responseTime);
        
        this.emit('set', { key, responseTime });
    }
    
    // Record cache delete operation
    recordDelete(key, responseTime = 0) {
        this.metrics.deletes++;
        
        this.recordKeyMetric(key, 'delete', responseTime);
        this.recordPerformanceData('delete', responseTime);
        
        this.emit('delete', { key, responseTime });
    }
    
    // Record cache error
    recordError(operation, error, key = null) {
        this.metrics.errors++;
        
        if (key) {
            this.recordKeyMetric(key, 'error', 0);
        }
        
        this.recordPerformanceData('error', 0);
        
        this.emit('error', { operation, error, key });
    }
    
    // Record per-key metrics
    recordKeyMetric(key, operation, responseTime) {
        if (!this.keyMetrics.has(key)) {
            this.keyMetrics.set(key, {
                hits: 0,
                misses: 0,
                sets: 0,
                deletes: 0,
                errors: 0,
                totalResponseTime: 0,
                requestCount: 0,
                lastAccessed: null
            });
        }
        
        const keyMetric = this.keyMetrics.get(key);
        keyMetric[operation === 'hit' || operation === 'miss' ? operation : 'sets']++;
        keyMetric.totalResponseTime += responseTime;
        keyMetric.requestCount++;
        keyMetric.lastAccessed = new Date();
    }
    
    // Record performance data points
    recordPerformanceData(operation, responseTime) {
        this.performanceData.push({
            operation,
            responseTime,
            timestamp: Date.now()
        });
        
        // Keep only last 1000 data points
        if (this.performanceData.length > 1000) {
            this.performanceData = this.performanceData.slice(-1000);
        }
    }
    
    // Get cache hit rate
    getHitRate() {
        const total = this.metrics.hits + this.metrics.misses;
        return total > 0 ? (this.metrics.hits / total * 100).toFixed(2) : 0;
    }
    
    // Get overall metrics
    getMetrics() {
        const uptime = Date.now() - this.startTime;
        const requestsPerSecond = this.metrics.totalRequests / (uptime / 1000);
        
        return {
            ...this.metrics,
            hitRate: `${this.getHitRate()}%`,
            uptime: {
                milliseconds: uptime,
                seconds: Math.floor(uptime / 1000),
                minutes: Math.floor(uptime / 60000),
                hours: Math.floor(uptime / 3600000)
            },
            requestsPerSecond: requestsPerSecond.toFixed(2)
        };
    }
    
    // Get performance statistics
    getPerformanceStats() {
        if (this.performanceData.length === 0) {
            return null;
        }
        
        const responseTimes = this.performanceData
            .filter(d => d.responseTime > 0)
            .map(d => d.responseTime);
        
        if (responseTimes.length === 0) {
            return null;
        }
        
        responseTimes.sort((a, b) => a - b);
        
        const avg = responseTimes.reduce((sum, time) => sum + time, 0) / responseTimes.length;
        const median = responseTimes[Math.floor(responseTimes.length / 2)];
        const p95 = responseTimes[Math.floor(responseTimes.length * 0.95)];
        const p99 = responseTimes[Math.floor(responseTimes.length * 0.99)];
        
        return {
            average: avg.toFixed(2),
            median: median.toFixed(2),
            p95: p95.toFixed(2),
            p99: p99.toFixed(2),
            min: responseTimes[0].toFixed(2),
            max: responseTimes[responseTimes.length - 1].toFixed(2),
            sampleSize: responseTimes.length
        };
    }
    
    // Get top performing keys
    getTopKeys(limit = 10) {
        const sortedKeys = Array.from(this.keyMetrics.entries())
            .sort(([, a], [, b]) => b.requestCount - a.requestCount)
            .slice(0, limit);
        
        return sortedKeys.map(([key, metrics]) => ({
            key,
            ...metrics,
            hitRate: metrics.hits + metrics.misses > 0 ? 
                (metrics.hits / (metrics.hits + metrics.misses) * 100).toFixed(2) : 0,
            avgResponseTime: metrics.requestCount > 0 ? 
                (metrics.totalResponseTime / metrics.requestCount).toFixed(2) : 0
        }));
    }
    
    // Get slow keys
    getSlowKeys(limit = 10) {
        const slowKeys = Array.from(this.keyMetrics.entries())
            .filter(([, metrics]) => metrics.requestCount > 0)
            .map(([key, metrics]) => ({
                key,
                avgResponseTime: metrics.totalResponseTime / metrics.requestCount,
                requestCount: metrics.requestCount
            }))
            .sort((a, b) => b.avgResponseTime - a.avgResponseTime)
            .slice(0, limit);
        
        return slowKeys.map(item => ({
            ...item,
            avgResponseTime: item.avgResponseTime.toFixed(2)
        }));
    }
    
    // Clean old performance data
    cleanOldPerformanceData() {
        const oneHourAgo = Date.now() - 60 * 60 * 1000;
        this.performanceData = this.performanceData.filter(
            data => data.timestamp > oneHourAgo
        );
    }
    
    // Reset metrics
    reset() {
        this.metrics = {
            hits: 0,
            misses: 0,
            sets: 0,
            deletes: 0,
            errors: 0,
            totalRequests: 0
        };
        
        this.performanceData = [];
        this.keyMetrics.clear();
        this.startTime = Date.now();
        
        this.emit('reset');
    }
    
    // Export metrics for external monitoring
    exportMetrics() {
        return {
            timestamp: new Date().toISOString(),
            metrics: this.getMetrics(),
            performance: this.getPerformanceStats(),
            topKeys: this.getTopKeys(5),
            slowKeys: this.getSlowKeys(5)
        };
    }
}

module.exports = new CacheMetricsService();
```

---

## 🔗 Related Topics
- [[25-Security-Best-Practices]] - Cache security considerations
- [[20-Database-Performance]] - Database caching strategies
- [[27-Performance-Monitoring]] - Performance monitoring
- [[17-Rate-Limiting]] - Rate limiting with caching

---

*Back to [[00-Main-Index]]*
