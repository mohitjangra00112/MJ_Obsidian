# Performance Monitoring

**Navigation**: [[26-Caching-Strategies]] | [[00-Main-Index]] | Next: [[28-Database-Integration]]

---

## 📊 Performance Monitoring for Node.js Backend

**Performance monitoring** is crucial for maintaining application health, identifying bottlenecks, and ensuring optimal user experience. This guide covers comprehensive monitoring strategies from basic metrics to advanced APM solutions.

---

## 🔍 Application Performance Monitoring (APM)

### Custom APM Service
```javascript
// services/APMService.js
const EventEmitter = require('events');
const os = require('os');
const v8 = require('v8');

class APMService extends EventEmitter {
    constructor() {
        super();
        this.metrics = {
            requests: new Map(),
            errors: new Map(),
            performance: [],
            system: []
        };
        
        this.startTime = Date.now();
        this.requestCount = 0;
        this.errorCount = 0;
        
        // Start system monitoring
        this.startSystemMonitoring();
        
        // Cleanup old data every hour
        setInterval(() => {
            this.cleanupOldData();
        }, 60 * 60 * 1000);
    }
    
    // Start request tracking
    startRequest(req) {
        const requestId = this.generateRequestId();
        const startTime = process.hrtime.bigint();
        
        req.apmRequestId = requestId;
        req.apmStartTime = startTime;
        
        this.requestCount++;
        
        return {
            requestId,
            startTime: Date.now(),
            method: req.method,
            url: req.originalUrl,
            userAgent: req.get('User-Agent'),
            ip: req.ip
        };
    }
    
    // End request tracking
    endRequest(req, res) {
        if (!req.apmRequestId || !req.apmStartTime) {
            return null;
        }
        
        const endTime = process.hrtime.bigint();
        const duration = Number(endTime - req.apmStartTime) / 1e6; // Convert to milliseconds
        
        const requestData = {
            requestId: req.apmRequestId,
            method: req.method,
            url: req.originalUrl,
            statusCode: res.statusCode,
            duration,
            timestamp: Date.now(),
            memoryUsage: process.memoryUsage(),
            userAgent: req.get('User-Agent'),
            ip: req.ip,
            userId: req.user?.id || null
        };
        
        // Store request data
        this.recordRequest(requestData);
        
        // Check for errors
        if (res.statusCode >= 400) {
            this.recordError(requestData);
        }
        
        // Check for slow requests
        if (duration > 1000) { // Slower than 1 second
            this.recordSlowRequest(requestData);
        }
        
        this.emit('requestCompleted', requestData);
        
        return requestData;
    }
    
    // Record request metrics
    recordRequest(requestData) {
        const minute = Math.floor(requestData.timestamp / 60000) * 60000;
        const key = `${requestData.method}:${this.getRoutePattern(requestData.url)}`;
        
        if (!this.metrics.requests.has(minute)) {
            this.metrics.requests.set(minute, new Map());
        }
        
        const minuteMetrics = this.metrics.requests.get(minute);
        
        if (!minuteMetrics.has(key)) {
            minuteMetrics.set(key, {
                count: 0,
                totalDuration: 0,
                durations: [],
                statusCodes: new Map(),
                errors: 0
            });
        }
        
        const routeMetrics = minuteMetrics.get(key);
        routeMetrics.count++;
        routeMetrics.totalDuration += requestData.duration;
        routeMetrics.durations.push(requestData.duration);
        
        // Limit stored durations to last 100 per route per minute
        if (routeMetrics.durations.length > 100) {
            routeMetrics.durations = routeMetrics.durations.slice(-100);
        }
        
        // Track status codes
        const statusCode = requestData.statusCode.toString();
        routeMetrics.statusCodes.set(
            statusCode,
            (routeMetrics.statusCodes.get(statusCode) || 0) + 1
        );
        
        if (requestData.statusCode >= 400) {
            routeMetrics.errors++;
        }
    }
    
    // Record error details
    recordError(requestData) {
        this.errorCount++;
        
        const errorKey = `${requestData.statusCode}:${requestData.method}:${this.getRoutePattern(requestData.url)}`;
        const minute = Math.floor(requestData.timestamp / 60000) * 60000;
        
        if (!this.metrics.errors.has(minute)) {
            this.metrics.errors.set(minute, new Map());
        }
        
        const minuteErrors = this.metrics.errors.get(minute);
        minuteErrors.set(errorKey, (minuteErrors.get(errorKey) || 0) + 1);
        
        this.emit('errorRecorded', requestData);
    }
    
    // Record slow request
    recordSlowRequest(requestData) {
        this.emit('slowRequest', requestData);
        
        // Log slow request details
        console.warn(`Slow request detected: ${requestData.method} ${requestData.url} took ${requestData.duration.toFixed(2)}ms`);
    }
    
    // Start system monitoring
    startSystemMonitoring() {
        setInterval(() => {
            this.recordSystemMetrics();
        }, 30000); // Every 30 seconds
    }
    
    // Record system metrics
    recordSystemMetrics() {
        const memoryUsage = process.memoryUsage();
        const cpuUsage = process.cpuUsage();
        const heapStats = v8.getHeapStatistics();
        
        const systemData = {
            timestamp: Date.now(),
            memory: {
                rss: memoryUsage.rss,
                heapUsed: memoryUsage.heapUsed,
                heapTotal: memoryUsage.heapTotal,
                external: memoryUsage.external,
                arrayBuffers: memoryUsage.arrayBuffers
            },
            cpu: {
                user: cpuUsage.user,
                system: cpuUsage.system
            },
            heap: {
                totalHeapSize: heapStats.total_heap_size,
                totalHeapSizeExecutable: heapStats.total_heap_size_executable,
                totalPhysicalSize: heapStats.total_physical_size,
                totalAvailableSize: heapStats.total_available_size,
                usedHeapSize: heapStats.used_heap_size,
                heapSizeLimit: heapStats.heap_size_limit,
                mallocedMemory: heapStats.malloced_memory,
                peakMallocedMemory: heapStats.peak_malloced_memory
            },
            system: {
                loadAverage: os.loadavg(),
                freeMemory: os.freemem(),
                totalMemory: os.totalmem(),
                uptime: os.uptime()
            },
            eventLoop: this.getEventLoopLag()
        };
        
        this.metrics.system.push(systemData);
        
        // Keep only last 1000 system metrics (about 8.3 hours)
        if (this.metrics.system.length > 1000) {
            this.metrics.system = this.metrics.system.slice(-1000);
        }
        
        this.emit('systemMetrics', systemData);
        
        // Check for memory leaks
        this.checkMemoryLeak(systemData);
        
        // Check for high CPU usage
        this.checkHighCPU(systemData);
    }
    
    // Get event loop lag
    getEventLoopLag() {
        const start = process.hrtime.bigint();
        
        setImmediate(() => {
            const lag = Number(process.hrtime.bigint() - start) / 1e6;
            this.lastEventLoopLag = lag;
        });
        
        return this.lastEventLoopLag || 0;
    }
    
    // Check for memory leaks
    checkMemoryLeak(systemData) {
        const heapUsedMB = systemData.memory.heapUsed / (1024 * 1024);
        const threshold = 500; // 500MB threshold
        
        if (heapUsedMB > threshold) {
            this.emit('memoryAlert', {
                type: 'high_memory_usage',
                heapUsedMB,
                threshold,
                timestamp: systemData.timestamp
            });
        }
        
        // Check for continuously growing memory
        if (this.metrics.system.length >= 10) {
            const recent = this.metrics.system.slice(-10);
            const trend = this.calculateMemoryTrend(recent);
            
            if (trend > 10) { // Growing more than 10MB per minute
                this.emit('memoryAlert', {
                    type: 'memory_leak_suspected',
                    trendMBPerMinute: trend,
                    currentHeapMB: heapUsedMB,
                    timestamp: systemData.timestamp
                });
            }
        }
    }
    
    // Check for high CPU usage
    checkHighCPU(systemData) {
        const loadAverage = systemData.system.loadAverage[0];
        const cpuCount = os.cpus().length;
        const cpuUsagePercent = (loadAverage / cpuCount) * 100;
        
        if (cpuUsagePercent > 80) {
            this.emit('cpuAlert', {
                type: 'high_cpu_usage',
                cpuUsagePercent,
                loadAverage,
                cpuCount,
                timestamp: systemData.timestamp
            });
        }
    }
    
    // Calculate memory trend
    calculateMemoryTrend(systemMetrics) {
        if (systemMetrics.length < 2) return 0;
        
        const first = systemMetrics[0];
        const last = systemMetrics[systemMetrics.length - 1];
        
        const timeDiffMinutes = (last.timestamp - first.timestamp) / 60000;
        const memoryDiffMB = (last.memory.heapUsed - first.memory.heapUsed) / (1024 * 1024);
        
        return timeDiffMinutes > 0 ? memoryDiffMB / timeDiffMinutes : 0;
    }
    
    // Get performance summary
    getPerformanceSummary(timeRangeMinutes = 60) {
        const now = Date.now();
        const startTime = now - (timeRangeMinutes * 60 * 1000);
        
        const summary = {
            timeRange: `${timeRangeMinutes} minutes`,
            totalRequests: 0,
            totalErrors: 0,
            averageResponseTime: 0,
            requestsPerMinute: 0,
            errorRate: 0,
            slowRequests: 0,
            topRoutes: [],
            topErrors: [],
            systemHealth: this.getSystemHealth()
        };
        
        let totalDuration = 0;
        let slowRequestCount = 0;
        const routeStats = new Map();
        const errorStats = new Map();
        
        // Analyze request metrics
        for (const [minute, routes] of this.metrics.requests) {
            if (minute >= startTime) {
                for (const [route, metrics] of routes) {
                    summary.totalRequests += metrics.count;
                    totalDuration += metrics.totalDuration;
                    summary.totalErrors += metrics.errors;
                    
                    // Count slow requests (>1s)
                    slowRequestCount += metrics.durations.filter(d => d > 1000).length;
                    
                    // Aggregate route stats
                    if (!routeStats.has(route)) {
                        routeStats.set(route, {
                            count: 0,
                            totalDuration: 0,
                            errors: 0
                        });
                    }
                    
                    const routeStat = routeStats.get(route);
                    routeStat.count += metrics.count;
                    routeStat.totalDuration += metrics.totalDuration;
                    routeStat.errors += metrics.errors;
                }
            }
        }
        
        // Analyze error metrics
        for (const [minute, errors] of this.metrics.errors) {
            if (minute >= startTime) {
                for (const [errorKey, count] of errors) {
                    errorStats.set(errorKey, (errorStats.get(errorKey) || 0) + count);
                }
            }
        }
        
        // Calculate derived metrics
        if (summary.totalRequests > 0) {
            summary.averageResponseTime = totalDuration / summary.totalRequests;
            summary.errorRate = (summary.totalErrors / summary.totalRequests) * 100;
        }
        
        summary.requestsPerMinute = summary.totalRequests / timeRangeMinutes;
        summary.slowRequests = slowRequestCount;
        
        // Top routes by request count
        summary.topRoutes = Array.from(routeStats.entries())
            .map(([route, stats]) => ({
                route,
                count: stats.count,
                averageResponseTime: stats.count > 0 ? stats.totalDuration / stats.count : 0,
                errorRate: stats.count > 0 ? (stats.errors / stats.count) * 100 : 0
            }))
            .sort((a, b) => b.count - a.count)
            .slice(0, 10);
        
        // Top errors by count
        summary.topErrors = Array.from(errorStats.entries())
            .map(([errorKey, count]) => ({ errorKey, count }))
            .sort((a, b) => b.count - a.count)
            .slice(0, 10);
        
        return summary;
    }
    
    // Get system health status
    getSystemHealth() {
        if (this.metrics.system.length === 0) {
            return { status: 'unknown', details: 'No system metrics available' };
        }
        
        const latest = this.metrics.system[this.metrics.system.length - 1];
        const heapUsedMB = latest.memory.heapUsed / (1024 * 1024);
        const loadAverage = latest.system.loadAverage[0];
        const cpuCount = os.cpus().length;
        const cpuUsagePercent = (loadAverage / cpuCount) * 100;
        
        let status = 'healthy';
        const issues = [];
        
        if (heapUsedMB > 500) {
            status = 'warning';
            issues.push(`High memory usage: ${heapUsedMB.toFixed(2)}MB`);
        }
        
        if (cpuUsagePercent > 80) {
            status = 'critical';
            issues.push(`High CPU usage: ${cpuUsagePercent.toFixed(2)}%`);
        }
        
        if (latest.eventLoop > 100) {
            status = status === 'critical' ? 'critical' : 'warning';
            issues.push(`High event loop lag: ${latest.eventLoop.toFixed(2)}ms`);
        }
        
        return {
            status,
            details: issues.length > 0 ? issues.join(', ') : 'All systems operating normally',
            metrics: {
                memoryUsageMB: heapUsedMB.toFixed(2),
                cpuUsagePercent: cpuUsagePercent.toFixed(2),
                eventLoopLagMs: latest.eventLoop.toFixed(2),
                uptime: process.uptime()
            }
        };
    }
    
    // Get route pattern from URL
    getRoutePattern(url) {
        // Remove query parameters
        const path = url.split('?')[0];
        
        // Replace IDs and UUIDs with placeholders
        return path
            .replace(/\/\d+/g, '/:id')
            .replace(/\/[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}/gi, '/:uuid')
            .replace(/\/[a-f0-9]{24}/g, '/:objectid');
    }
    
    // Generate unique request ID
    generateRequestId() {
        return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
    }
    
    // Cleanup old data
    cleanupOldData() {
        const oneHourAgo = Date.now() - 60 * 60 * 1000;
        
        // Clean old request metrics
        for (const [minute] of this.metrics.requests) {
            if (minute < oneHourAgo) {
                this.metrics.requests.delete(minute);
            }
        }
        
        // Clean old error metrics
        for (const [minute] of this.metrics.errors) {
            if (minute < oneHourAgo) {
                this.metrics.errors.delete(minute);
            }
        }
        
        // Clean old performance data
        this.metrics.performance = this.metrics.performance.filter(
            item => item.timestamp > oneHourAgo
        );
    }
    
    // Export metrics for external monitoring
    exportMetrics() {
        return {
            timestamp: new Date().toISOString(),
            uptime: process.uptime(),
            totalRequests: this.requestCount,
            totalErrors: this.errorCount,
            performanceSummary: this.getPerformanceSummary(60),
            systemHealth: this.getSystemHealth(),
            memory: process.memoryUsage(),
            pid: process.pid
        };
    }
}

module.exports = new APMService();
```

### APM Middleware
```javascript
// middleware/apmMiddleware.js
const APMService = require('../services/APMService');

// Request tracking middleware
const apmMiddleware = (req, res, next) => {
    // Start request tracking
    const requestData = APMService.startRequest(req);
    
    // Override res.end to capture response data
    const originalEnd = res.end;
    
    res.end = function(...args) {
        // End request tracking
        APMService.endRequest(req, res);
        
        // Call original end method
        originalEnd.apply(res, args);
    };
    
    // Add request ID to response headers
    res.set('X-Request-ID', requestData.requestId);
    
    next();
};

// Error tracking middleware
const errorTrackingMiddleware = (error, req, res, next) => {
    // Record error details
    APMService.emit('errorOccurred', {
        error: {
            message: error.message,
            stack: error.stack,
            name: error.name
        },
        request: {
            method: req.method,
            url: req.originalUrl,
            headers: req.headers,
            body: req.body,
            params: req.params,
            query: req.query,
            ip: req.ip,
            userAgent: req.get('User-Agent'),
            userId: req.user?.id
        },
        timestamp: new Date().toISOString()
    });
    
    next(error);
};

module.exports = {
    apmMiddleware,
    errorTrackingMiddleware
};
```

---

## 🔧 Custom Metrics Collection

### Business Metrics Service
```javascript
// services/BusinessMetricsService.js
const EventEmitter = require('events');

class BusinessMetricsService extends EventEmitter {
    constructor() {
        super();
        this.counters = new Map();
        this.gauges = new Map();
        this.histograms = new Map();
        this.timers = new Map();
    }
    
    // Increment counter
    incrementCounter(name, value = 1, tags = {}) {
        const key = this.getMetricKey(name, tags);
        
        if (!this.counters.has(key)) {
            this.counters.set(key, {
                name,
                value: 0,
                tags,
                timestamp: Date.now()
            });
        }
        
        const counter = this.counters.get(key);
        counter.value += value;
        counter.timestamp = Date.now();
        
        this.emit('counterIncremented', { name, value, tags, total: counter.value });
        
        return counter.value;
    }
    
    // Set gauge value
    setGauge(name, value, tags = {}) {
        const key = this.getMetricKey(name, tags);
        
        this.gauges.set(key, {
            name,
            value,
            tags,
            timestamp: Date.now()
        });
        
        this.emit('gaugeSet', { name, value, tags });
        
        return value;
    }
    
    // Record histogram value
    recordHistogram(name, value, tags = {}) {
        const key = this.getMetricKey(name, tags);
        
        if (!this.histograms.has(key)) {
            this.histograms.set(key, {
                name,
                values: [],
                tags,
                count: 0,
                sum: 0,
                min: Infinity,
                max: -Infinity
            });
        }
        
        const histogram = this.histograms.get(key);
        histogram.values.push({ value, timestamp: Date.now() });
        histogram.count++;
        histogram.sum += value;
        histogram.min = Math.min(histogram.min, value);
        histogram.max = Math.max(histogram.max, value);
        
        // Keep only last 1000 values
        if (histogram.values.length > 1000) {
            const removed = histogram.values.shift();
            histogram.sum -= removed.value;
            histogram.count--;
        }
        
        this.emit('histogramRecorded', { name, value, tags });
        
        return histogram;
    }
    
    // Start timer
    startTimer(name, tags = {}) {
        const timerId = `${name}_${Date.now()}_${Math.random()}`;
        
        this.timers.set(timerId, {
            name,
            tags,
            startTime: process.hrtime.bigint(),
            timestamp: Date.now()
        });
        
        return {
            timerId,
            stop: () => this.stopTimer(timerId)
        };
    }
    
    // Stop timer and record duration
    stopTimer(timerId) {
        const timer = this.timers.get(timerId);
        if (!timer) {
            throw new Error(`Timer ${timerId} not found`);
        }
        
        const endTime = process.hrtime.bigint();
        const duration = Number(endTime - timer.startTime) / 1e6; // Convert to milliseconds
        
        this.timers.delete(timerId);
        
        // Record as histogram
        this.recordHistogram(`${timer.name}_duration`, duration, timer.tags);
        
        this.emit('timerStopped', { 
            name: timer.name, 
            duration, 
            tags: timer.tags 
        });
        
        return duration;
    }
    
    // Business metric: User registration
    recordUserRegistration(userId, source = 'web') {
        this.incrementCounter('user_registrations', 1, { source });
        this.setGauge('total_users', this.getUserCount(), {});
        
        this.emit('userRegistered', { userId, source });
    }
    
    // Business metric: Order processing
    recordOrder(orderId, amount, currency = 'USD') {
        this.incrementCounter('orders_total', 1);
        this.recordHistogram('order_amount', amount, { currency });
        this.incrementCounter('revenue_total', amount, { currency });
        
        this.emit('orderProcessed', { orderId, amount, currency });
    }
    
    // Business metric: API endpoint usage
    recordAPIUsage(endpoint, method, statusCode, duration) {
        const tags = { endpoint, method, status: statusCode.toString() };
        
        this.incrementCounter('api_requests_total', 1, tags);
        this.recordHistogram('api_request_duration', duration, tags);
        
        if (statusCode >= 400) {
            this.incrementCounter('api_errors_total', 1, tags);
        }
        
        this.emit('apiUsage', { endpoint, method, statusCode, duration });
    }
    
    // Business metric: Database operations
    recordDatabaseOperation(operation, table, duration, success = true) {
        const tags = { operation, table, success: success.toString() };
        
        this.incrementCounter('db_operations_total', 1, tags);
        this.recordHistogram('db_operation_duration', duration, tags);
        
        if (!success) {
            this.incrementCounter('db_errors_total', 1, tags);
        }
        
        this.emit('databaseOperation', { operation, table, duration, success });
    }
    
    // Business metric: Cache operations
    recordCacheOperation(operation, hit = true) {
        const tags = { operation, result: hit ? 'hit' : 'miss' };
        
        this.incrementCounter('cache_operations_total', 1, tags);
        
        if (hit) {
            this.incrementCounter('cache_hits_total', 1, { operation });
        } else {
            this.incrementCounter('cache_misses_total', 1, { operation });
        }
        
        this.emit('cacheOperation', { operation, hit });
    }
    
    // Get metric key with tags
    getMetricKey(name, tags) {
        const tagString = Object.entries(tags)
            .sort(([a], [b]) => a.localeCompare(b))
            .map(([key, value]) => `${key}=${value}`)
            .join(',');
        
        return tagString ? `${name}{${tagString}}` : name;
    }
    
    // Get histogram statistics
    getHistogramStats(name, tags = {}) {
        const key = this.getMetricKey(name, tags);
        const histogram = this.histograms.get(key);
        
        if (!histogram || histogram.values.length === 0) {
            return null;
        }
        
        const values = histogram.values.map(v => v.value).sort((a, b) => a - b);
        const count = values.length;
        
        return {
            count,
            sum: histogram.sum,
            avg: histogram.sum / count,
            min: histogram.min,
            max: histogram.max,
            p50: this.percentile(values, 0.5),
            p90: this.percentile(values, 0.9),
            p95: this.percentile(values, 0.95),
            p99: this.percentile(values, 0.99)
        };
    }
    
    // Calculate percentile
    percentile(sortedValues, p) {
        const index = Math.ceil(sortedValues.length * p) - 1;
        return sortedValues[Math.max(0, index)];
    }
    
    // Get all metrics
    getAllMetrics() {
        const metrics = {
            counters: {},
            gauges: {},
            histograms: {}
        };
        
        // Export counters
        for (const [key, counter] of this.counters) {
            metrics.counters[key] = {
                name: counter.name,
                value: counter.value,
                tags: counter.tags,
                timestamp: counter.timestamp
            };
        }
        
        // Export gauges
        for (const [key, gauge] of this.gauges) {
            metrics.gauges[key] = {
                name: gauge.name,
                value: gauge.value,
                tags: gauge.tags,
                timestamp: gauge.timestamp
            };
        }
        
        // Export histogram stats
        for (const [key, histogram] of this.histograms) {
            metrics.histograms[key] = {
                name: histogram.name,
                tags: histogram.tags,
                stats: this.getHistogramStats(histogram.name, histogram.tags)
            };
        }
        
        return metrics;
    }
    
    // Reset all metrics
    reset() {
        this.counters.clear();
        this.gauges.clear();
        this.histograms.clear();
        this.timers.clear();
        
        this.emit('metricsReset');
    }
    
    // Export metrics in Prometheus format
    exportPrometheusMetrics() {
        let output = '';
        
        // Export counters
        for (const [key, counter] of this.counters) {
            const tagString = Object.entries(counter.tags)
                .map(([k, v]) => `${k}="${v}"`)
                .join(',');
            
            output += `# TYPE ${counter.name} counter\n`;
            output += `${counter.name}${tagString ? `{${tagString}}` : ''} ${counter.value}\n`;
        }
        
        // Export gauges
        for (const [key, gauge] of this.gauges) {
            const tagString = Object.entries(gauge.tags)
                .map(([k, v]) => `${k}="${v}"`)
                .join(',');
            
            output += `# TYPE ${gauge.name} gauge\n`;
            output += `${gauge.name}${tagString ? `{${tagString}}` : ''} ${gauge.value}\n`;
        }
        
        // Export histogram summaries
        for (const [key, histogram] of this.histograms) {
            const stats = this.getHistogramStats(histogram.name, histogram.tags);
            if (!stats) continue;
            
            const tagString = Object.entries(histogram.tags)
                .map(([k, v]) => `${k}="${v}"`)
                .join(',');
            
            output += `# TYPE ${histogram.name} histogram\n`;
            output += `${histogram.name}_count${tagString ? `{${tagString}}` : ''} ${stats.count}\n`;
            output += `${histogram.name}_sum${tagString ? `{${tagString}}` : ''} ${stats.sum}\n`;
            
            const percentiles = [0.5, 0.9, 0.95, 0.99];
            for (const p of percentiles) {
                const quantileTag = tagString ? 
                    `{${tagString},quantile="${p}"}` : 
                    `{quantile="${p}"}`;
                output += `${histogram.name}${quantileTag} ${this.percentile(
                    histogram.values.map(v => v.value).sort((a, b) => a - b), p
                )}\n`;
            }
        }
        
        return output;
    }
    
    // Helper method to get user count (placeholder)
    getUserCount() {
        // This would typically query your database
        return this.counters.get('user_registrations')?.value || 0;
    }
}

module.exports = new BusinessMetricsService();
```

---

## 📈 Health Check System

### Health Check Service
```javascript
// services/HealthCheckService.js
const EventEmitter = require('events');

class HealthCheckService extends EventEmitter {
    constructor() {
        super();
        this.checks = new Map();
        this.results = new Map();
        this.globalHealth = 'unknown';
        this.checkInterval = 30000; // 30 seconds
        this.timeout = 5000; // 5 seconds
        
        // Start periodic health checks
        this.startPeriodicChecks();
    }
    
    // Register a health check
    register(name, checkFunction, options = {}) {
        const {
            critical = false,
            timeout = this.timeout,
            interval = this.checkInterval,
            tags = {}
        } = options;
        
        this.checks.set(name, {
            name,
            checkFunction,
            critical,
            timeout,
            interval,
            tags,
            lastRun: null,
            enabled: true
        });
        
        this.emit('checkRegistered', { name, critical, tags });
    }
    
    // Run a single health check
    async runCheck(name) {
        const check = this.checks.get(name);
        if (!check || !check.enabled) {
            return null;
        }
        
        const startTime = Date.now();
        let result = {
            name,
            status: 'unknown',
            message: '',
            duration: 0,
            timestamp: startTime,
            critical: check.critical,
            tags: check.tags
        };
        
        try {
            // Run check with timeout
            const checkPromise = Promise.resolve(check.checkFunction());
            const timeoutPromise = new Promise((_, reject) => {
                setTimeout(() => reject(new Error('Health check timeout')), check.timeout);
            });
            
            const checkResult = await Promise.race([checkPromise, timeoutPromise]);
            
            result.duration = Date.now() - startTime;
            check.lastRun = Date.now();
            
            if (typeof checkResult === 'boolean') {
                result.status = checkResult ? 'healthy' : 'unhealthy';
                result.message = checkResult ? 'Check passed' : 'Check failed';
            } else if (typeof checkResult === 'object') {
                result = { ...result, ...checkResult };
            } else {
                result.status = 'healthy';
                result.message = checkResult?.toString() || 'Check passed';
            }
            
        } catch (error) {
            result.status = 'unhealthy';
            result.message = error.message;
            result.duration = Date.now() - startTime;
            check.lastRun = Date.now();
            
            this.emit('checkFailed', { name, error: error.message });
        }
        
        this.results.set(name, result);
        this.updateGlobalHealth();
        
        this.emit('checkCompleted', result);
        
        return result;
    }
    
    // Run all health checks
    async runAllChecks() {
        const promises = [];
        
        for (const name of this.checks.keys()) {
            promises.push(this.runCheck(name));
        }
        
        const results = await Promise.allSettled(promises);
        
        return results
            .filter(result => result.status === 'fulfilled' && result.value !== null)
            .map(result => result.value);
    }
    
    // Start periodic health checks
    startPeriodicChecks() {
        setInterval(async () => {
            await this.runAllChecks();
        }, this.checkInterval);
    }
    
    // Update global health status
    updateGlobalHealth() {
        const results = Array.from(this.results.values());
        
        if (results.length === 0) {
            this.globalHealth = 'unknown';
            return;
        }
        
        // Check critical services first
        const criticalResults = results.filter(r => r.critical);
        const hasCriticalFailures = criticalResults.some(r => r.status === 'unhealthy');
        
        if (hasCriticalFailures) {
            this.globalHealth = 'unhealthy';
            this.emit('globalHealthChanged', { status: 'unhealthy', reason: 'Critical service failure' });
            return;
        }
        
        // Check all services
        const hasAnyFailures = results.some(r => r.status === 'unhealthy');
        const hasWarnings = results.some(r => r.status === 'warning');
        
        if (hasAnyFailures) {
            this.globalHealth = 'degraded';
            this.emit('globalHealthChanged', { status: 'degraded', reason: 'Non-critical service failure' });
        } else if (hasWarnings) {
            this.globalHealth = 'warning';
            this.emit('globalHealthChanged', { status: 'warning', reason: 'Service warnings detected' });
        } else {
            this.globalHealth = 'healthy';
        }
    }
    
    // Get health status
    getHealthStatus() {
        const results = Array.from(this.results.values());
        const summary = {
            status: this.globalHealth,
            timestamp: new Date().toISOString(),
            uptime: process.uptime(),
            checks: results,
            summary: {
                total: results.length,
                healthy: results.filter(r => r.status === 'healthy').length,
                unhealthy: results.filter(r => r.status === 'unhealthy').length,
                warning: results.filter(r => r.status === 'warning').length,
                unknown: results.filter(r => r.status === 'unknown').length
            }
        };
        
        return summary;
    }
    
    // Get detailed health report
    getDetailedReport() {
        const status = this.getHealthStatus();
        const checks = Array.from(this.checks.values());
        
        return {
            ...status,
            configuration: {
                totalRegisteredChecks: checks.length,
                enabledChecks: checks.filter(c => c.enabled).length,
                criticalChecks: checks.filter(c => c.critical).length,
                checkInterval: this.checkInterval,
                timeout: this.timeout
            },
            performance: {
                averageCheckDuration: this.getAverageCheckDuration(),
                slowestCheck: this.getSlowestCheck(),
                lastCheckTime: Math.max(...Array.from(this.results.values()).map(r => r.timestamp))
            }
        };
    }
    
    // Get average check duration
    getAverageCheckDuration() {
        const results = Array.from(this.results.values());
        if (results.length === 0) return 0;
        
        const totalDuration = results.reduce((sum, r) => sum + r.duration, 0);
        return totalDuration / results.length;
    }
    
    // Get slowest check
    getSlowestCheck() {
        const results = Array.from(this.results.values());
        if (results.length === 0) return null;
        
        return results.reduce((slowest, current) => 
            current.duration > slowest.duration ? current : slowest
        );
    }
    
    // Enable/disable a check
    setCheckEnabled(name, enabled) {
        const check = this.checks.get(name);
        if (check) {
            check.enabled = enabled;
            this.emit('checkToggled', { name, enabled });
        }
    }
    
    // Remove a check
    unregister(name) {
        const removed = this.checks.delete(name);
        this.results.delete(name);
        
        if (removed) {
            this.updateGlobalHealth();
            this.emit('checkUnregistered', { name });
        }
        
        return removed;
    }
}

// Create health check service instance
const healthCheck = new HealthCheckService();

// Register default system health checks
healthCheck.register('memory', async () => {
    const memUsage = process.memoryUsage();
    const heapUsedMB = memUsage.heapUsed / 1024 / 1024;
    const heapTotalMB = memUsage.heapTotal / 1024 / 1024;
    const usage = (heapUsedMB / heapTotalMB) * 100;
    
    if (usage > 90) {
        return {
            status: 'unhealthy',
            message: `Memory usage critical: ${usage.toFixed(2)}%`,
            details: { heapUsedMB, heapTotalMB, usagePercent: usage }
        };
    } else if (usage > 75) {
        return {
            status: 'warning',
            message: `Memory usage high: ${usage.toFixed(2)}%`,
            details: { heapUsedMB, heapTotalMB, usagePercent: usage }
        };
    }
    
    return {
        status: 'healthy',
        message: `Memory usage normal: ${usage.toFixed(2)}%`,
        details: { heapUsedMB, heapTotalMB, usagePercent: usage }
    };
}, { critical: true });

healthCheck.register('eventLoop', async () => {
    return new Promise((resolve) => {
        const start = process.hrtime.bigint();
        
        setImmediate(() => {
            const lag = Number(process.hrtime.bigint() - start) / 1e6;
            
            if (lag > 100) {
                resolve({
                    status: 'unhealthy',
                    message: `Event loop lag critical: ${lag.toFixed(2)}ms`,
                    details: { lagMs: lag }
                });
            } else if (lag > 50) {
                resolve({
                    status: 'warning',
                    message: `Event loop lag high: ${lag.toFixed(2)}ms`,
                    details: { lagMs: lag }
                });
            } else {
                resolve({
                    status: 'healthy',
                    message: `Event loop lag normal: ${lag.toFixed(2)}ms`,
                    details: { lagMs: lag }
                });
            }
        });
    });
}, { critical: true });

module.exports = healthCheck;
```

---

## 🔗 Alerting System

### Alert Manager
```javascript
// services/AlertManager.js
const EventEmitter = require('events');

class AlertManager extends EventEmitter {
    constructor() {
        super();
        this.rules = new Map();
        this.alerts = new Map();
        this.channels = new Map();
        this.suppressions = new Set();
        this.cooldowns = new Map();
        
        // Default cooldown period (5 minutes)
        this.defaultCooldown = 5 * 60 * 1000;
    }
    
    // Register alert rule
    addRule(name, condition, options = {}) {
        const {
            severity = 'warning',
            message = `Alert: ${name}`,
            cooldown = this.defaultCooldown,
            channels = ['console'],
            tags = {},
            enabled = true
        } = options;
        
        this.rules.set(name, {
            name,
            condition,
            severity,
            message,
            cooldown,
            channels,
            tags,
            enabled,
            lastTriggered: null,
            triggerCount: 0
        });
        
        this.emit('ruleAdded', { name, severity, channels });
    }
    
    // Check alert conditions
    async checkAlerts(metrics) {
        const triggeredAlerts = [];
        
        for (const [name, rule] of this.rules) {
            if (!rule.enabled) continue;
            
            try {
                const shouldAlert = await this.evaluateCondition(rule.condition, metrics);
                
                if (shouldAlert) {
                    const alert = await this.triggerAlert(name, rule, metrics);
                    if (alert) {
                        triggeredAlerts.push(alert);
                    }
                }
            } catch (error) {
                console.error(`Error evaluating alert rule ${name}:`, error);
            }
        }
        
        return triggeredAlerts;
    }
    
    // Evaluate alert condition
    async evaluateCondition(condition, metrics) {
        if (typeof condition === 'function') {
            return await condition(metrics);
        }
        
        // Simple condition evaluation
        if (typeof condition === 'object') {
            const { metric, operator, threshold } = condition;
            const value = this.getMetricValue(metrics, metric);
            
            switch (operator) {
                case '>': return value > threshold;
                case '<': return value < threshold;
                case '>=': return value >= threshold;
                case '<=': return value <= threshold;
                case '==': return value === threshold;
                case '!=': return value !== threshold;
                default: return false;
            }
        }
        
        return false;
    }
    
    // Get metric value from metrics object
    getMetricValue(metrics, metricPath) {
        const parts = metricPath.split('.');
        let value = metrics;
        
        for (const part of parts) {
            if (value && typeof value === 'object') {
                value = value[part];
            } else {
                return undefined;
            }
        }
        
        return value;
    }
    
    // Trigger alert
    async triggerAlert(ruleName, rule, metrics) {
        const now = Date.now();
        const alertKey = `${ruleName}_${rule.severity}`;
        
        // Check cooldown
        if (this.cooldowns.has(alertKey)) {
            const lastAlert = this.cooldowns.get(alertKey);
            if (now - lastAlert < rule.cooldown) {
                return null; // Still in cooldown
            }
        }
        
        // Check suppression
        if (this.suppressions.has(ruleName)) {
            return null; // Alert suppressed
        }
        
        const alert = {
            id: this.generateAlertId(),
            rule: ruleName,
            severity: rule.severity,
            message: typeof rule.message === 'function' ? 
                rule.message(metrics) : rule.message,
            timestamp: new Date().toISOString(),
            metrics: this.extractRelevantMetrics(metrics, rule.tags),
            tags: rule.tags,
            status: 'firing'
        };
        
        // Store alert
        this.alerts.set(alert.id, alert);
        
        // Update rule statistics
        rule.lastTriggered = now;
        rule.triggerCount++;
        
        // Set cooldown
        this.cooldowns.set(alertKey, now);
        
        // Send to channels
        await this.sendAlert(alert, rule.channels);
        
        this.emit('alertTriggered', alert);
        
        return alert;
    }
    
    // Send alert to channels
    async sendAlert(alert, channels) {
        const promises = [];
        
        for (const channelName of channels) {
            const channel = this.channels.get(channelName);
            if (channel) {
                promises.push(this.sendToChannel(channel, alert));
            }
        }
        
        await Promise.allSettled(promises);
    }
    
    // Send alert to specific channel
    async sendToChannel(channel, alert) {
        try {
            await channel.send(alert);
        } catch (error) {
            console.error(`Failed to send alert to channel ${channel.name}:`, error);
            this.emit('channelError', { channel: channel.name, error: error.message });
        }
    }
    
    // Register notification channel
    addChannel(name, channel) {
        this.channels.set(name, { name, ...channel });
        this.emit('channelAdded', { name });
    }
    
    // Suppress alert rule
    suppressAlert(ruleName, duration = 60 * 60 * 1000) { // 1 hour default
        this.suppressions.add(ruleName);
        
        setTimeout(() => {
            this.suppressions.delete(ruleName);
            this.emit('alertUnsuppressed', { ruleName });
        }, duration);
        
        this.emit('alertSuppressed', { ruleName, duration });
    }
    
    // Resolve alert
    resolveAlert(alertId, reason = 'Manual resolution') {
        const alert = this.alerts.get(alertId);
        if (alert) {
            alert.status = 'resolved';
            alert.resolvedAt = new Date().toISOString();
            alert.resolution = reason;
            
            this.emit('alertResolved', alert);
        }
    }
    
    // Get active alerts
    getActiveAlerts() {
        return Array.from(this.alerts.values())
            .filter(alert => alert.status === 'firing')
            .sort((a, b) => {
                const severityOrder = { critical: 4, error: 3, warning: 2, info: 1 };
                return severityOrder[b.severity] - severityOrder[a.severity];
            });
    }
    
    // Get alert statistics
    getAlertStats() {
        const alerts = Array.from(this.alerts.values());
        const rules = Array.from(this.rules.values());
        
        return {
            totalAlerts: alerts.length,
            activeAlerts: alerts.filter(a => a.status === 'firing').length,
            resolvedAlerts: alerts.filter(a => a.status === 'resolved').length,
            severityBreakdown: {
                critical: alerts.filter(a => a.severity === 'critical').length,
                error: alerts.filter(a => a.severity === 'error').length,
                warning: alerts.filter(a => a.severity === 'warning').length,
                info: alerts.filter(a => a.severity === 'info').length
            },
            rules: {
                total: rules.length,
                enabled: rules.filter(r => r.enabled).length,
                disabled: rules.filter(r => !r.enabled).length
            },
            suppressions: this.suppressions.size,
            channels: this.channels.size
        };
    }
    
    // Generate unique alert ID
    generateAlertId() {
        return `alert_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    }
    
    // Extract relevant metrics based on tags
    extractRelevantMetrics(metrics, tags) {
        // This would extract specific metrics based on alert context
        return {
            timestamp: new Date().toISOString(),
            systemHealth: metrics.systemHealth,
            performance: metrics.performanceSummary
        };
    }
}

// Create alert manager instance
const alertManager = new AlertManager();

// Add console channel
alertManager.addChannel('console', {
    send: async (alert) => {
        const severity = alert.severity.toUpperCase();
        const timestamp = new Date(alert.timestamp).toLocaleString();
        console.log(`[${severity}] ${timestamp}: ${alert.message}`);
        console.log(`Alert ID: ${alert.id}`);
        if (alert.tags && Object.keys(alert.tags).length > 0) {
            console.log(`Tags:`, alert.tags);
        }
    }
});

// Add email channel (placeholder)
alertManager.addChannel('email', {
    send: async (alert) => {
        // Implement email sending logic
        console.log(`Would send email alert: ${alert.message}`);
    }
});

// Add Slack channel (placeholder)
alertManager.addChannel('slack', {
    send: async (alert) => {
        // Implement Slack webhook logic
        console.log(`Would send Slack alert: ${alert.message}`);
    }
});

module.exports = alertManager;
```

---

## 📊 Dashboard API

### Monitoring Dashboard API
```javascript
// routes/monitoring.js
const express = require('express');
const APMService = require('../services/APMService');
const BusinessMetricsService = require('../services/BusinessMetricsService');
const HealthCheckService = require('../services/HealthCheckService');
const AlertManager = require('../services/AlertManager');

const router = express.Router();

// Get performance summary
router.get('/performance', (req, res) => {
    const timeRange = parseInt(req.query.timeRange) || 60;
    const summary = APMService.getPerformanceSummary(timeRange);
    
    res.json({
        success: true,
        data: summary,
        timestamp: new Date().toISOString()
    });
});

// Get system health
router.get('/health', async (req, res) => {
    const health = HealthCheckService.getHealthStatus();
    
    res.status(health.status === 'healthy' ? 200 : 503)
        .json({
            success: health.status === 'healthy',
            data: health
        });
});

// Get detailed health report
router.get('/health/detailed', async (req, res) => {
    const report = HealthCheckService.getDetailedReport();
    
    res.json({
        success: true,
        data: report
    });
});

// Run health checks manually
router.post('/health/check', async (req, res) => {
    const results = await HealthCheckService.runAllChecks();
    
    res.json({
        success: true,
        data: results
    });
});

// Get business metrics
router.get('/metrics', (req, res) => {
    const metrics = BusinessMetricsService.getAllMetrics();
    
    res.json({
        success: true,
        data: metrics
    });
});

// Get Prometheus metrics
router.get('/metrics/prometheus', (req, res) => {
    const metrics = BusinessMetricsService.exportPrometheusMetrics();
    
    res.set('Content-Type', 'text/plain');
    res.send(metrics);
});

// Get active alerts
router.get('/alerts', (req, res) => {
    const alerts = AlertManager.getActiveAlerts();
    const stats = AlertManager.getAlertStats();
    
    res.json({
        success: true,
        data: {
            alerts,
            stats
        }
    });
});

// Resolve alert
router.post('/alerts/:alertId/resolve', (req, res) => {
    const { alertId } = req.params;
    const { reason } = req.body;
    
    AlertManager.resolveAlert(alertId, reason);
    
    res.json({
        success: true,
        message: 'Alert resolved'
    });
});

// Suppress alert rule
router.post('/alerts/rules/:ruleName/suppress', (req, res) => {
    const { ruleName } = req.params;
    const { duration } = req.body;
    
    AlertManager.suppressAlert(ruleName, duration);
    
    res.json({
        success: true,
        message: 'Alert rule suppressed'
    });
});

// Get system information
router.get('/system', (req, res) => {
    const systemInfo = {
        uptime: process.uptime(),
        memory: process.memoryUsage(),
        cpu: process.cpuUsage(),
        platform: process.platform,
        version: process.version,
        pid: process.pid,
        env: process.env.NODE_ENV || 'development'
    };
    
    res.json({
        success: true,
        data: systemInfo
    });
});

// Get APM export
router.get('/apm/export', (req, res) => {
    const metrics = APMService.exportMetrics();
    
    res.json({
        success: true,
        data: metrics
    });
});

module.exports = router;
```

---

## 🔗 Related Topics
- [[26-Caching-Strategies]] - Cache performance monitoring
- [[25-Security-Best-Practices]] - Security monitoring
- [[28-Database-Integration]] - Database performance
- [[18-Environment-Setup]] - Environment configuration

---

*Back to [[00-Main-Index]]*
