# Async Await

## Overview
Async/await is syntactic sugar built on top of Promises that makes asynchronous code look and behave more like synchronous code. It provides a cleaner and more readable way to handle asynchronous operations.

## Basic Syntax

### async Functions
```javascript
// Function declaration
async function fetchData() {
    return "Hello World";
}

// Function expression
const fetchDataExpression = async function() {
    return "Hello World";
};

// Arrow function
const fetchDataArrow = async () => {
    return "Hello World";
};

// Method in object
const obj = {
    async getData() {
        return "Hello World";
    }
};

// Method in class
class DataService {
    async fetchData() {
        return "Hello World";
    }
}
```

### await Keyword
```javascript
async function example() {
    // await can only be used inside async functions
    const result = await Promise.resolve("Hello");
    console.log(result); // "Hello"
    
    // await pauses execution until promise resolves
    const delayed = await new Promise(resolve => {
        setTimeout(() => resolve("Delayed"), 1000);
    });
    console.log(delayed); // "Delayed" (after 1 second)
}

example();
```

## Converting Promises to Async/Await

### Promise Chain
```javascript
// Using Promises
function fetchUserWithPromises(id) {
    return fetch(`/api/users/${id}`)
        .then(response => response.json())
        .then(user => {
            console.log("User:", user);
            return fetch(`/api/users/${user.id}/posts`);
        })
        .then(response => response.json())
        .then(posts => {
            console.log("Posts:", posts);
            return { user, posts }; // Error: user not in scope
        })
        .catch(error => {
            console.error("Error:", error);
            throw error;
        });
}

// Using Async/Await
async function fetchUserWithAsync(id) {
    try {
        const userResponse = await fetch(`/api/users/${id}`);
        const user = await userResponse.json();
        console.log("User:", user);
        
        const postsResponse = await fetch(`/api/users/${user.id}/posts`);
        const posts = await postsResponse.json();
        console.log("Posts:", posts);
        
        return { user, posts };
    } catch (error) {
        console.error("Error:", error);
        throw error;
    }
}
```

## Error Handling

### try/catch Blocks
```javascript
async function handleErrors() {
    try {
        const response = await fetch('/api/data');
        
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}: ${response.statusText}`);
        }
        
        const data = await response.json();
        return data;
    } catch (error) {
        if (error.name === 'TypeError') {
            console.error("Network error:", error.message);
        } else {
            console.error("API error:", error.message);
        }
        throw error; // Re-throw if needed
    }
}
```

### Multiple try/catch Blocks
```javascript
async function processData() {
    let userData;
    let userPosts;
    
    // Handle user fetch separately
    try {
        const userResponse = await fetch('/api/user/1');
        userData = await userResponse.json();
    } catch (error) {
        console.error("Failed to fetch user:", error);
        userData = { name: "Unknown User" }; // Fallback
    }
    
    // Handle posts fetch separately
    try {
        const postsResponse = await fetch('/api/posts');
        userPosts = await postsResponse.json();
    } catch (error) {
        console.error("Failed to fetch posts:", error);
        userPosts = []; // Fallback
    }
    
    return { user: userData, posts: userPosts };
}
```

### Error Recovery
```javascript
async function withRetry(operation, maxRetries = 3) {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            return await operation();
        } catch (error) {
            if (attempt === maxRetries) {
                throw new Error(`Failed after ${maxRetries} attempts: ${error.message}`);
            }
            
            console.log(`Attempt ${attempt} failed, retrying...`);
            await new Promise(resolve => setTimeout(resolve, 1000 * attempt));
        }
    }
}

// Usage
async function unreliableOperation() {
    const response = await fetch('/api/unreliable');
    if (!response.ok) {
        throw new Error('Request failed');
    }
    return response.json();
}

try {
    const result = await withRetry(unreliableOperation, 3);
    console.log("Success:", result);
} catch (error) {
    console.error("Final failure:", error.message);
}
```

## Parallel vs Sequential Execution

### Sequential Execution (One After Another)
```javascript
async function sequentialExecution() {
    console.time('Sequential');
    
    // These run one after another (slower)
    const result1 = await fetch('/api/data1').then(r => r.json());
    const result2 = await fetch('/api/data2').then(r => r.json());
    const result3 = await fetch('/api/data3').then(r => r.json());
    
    console.timeEnd('Sequential');
    return [result1, result2, result3];
}
```

### Parallel Execution (All at Once)
```javascript
async function parallelExecution() {
    console.time('Parallel');
    
    // Start all requests simultaneously
    const promise1 = fetch('/api/data1').then(r => r.json());
    const promise2 = fetch('/api/data2').then(r => r.json());
    const promise3 = fetch('/api/data3').then(r => r.json());
    
    // Wait for all to complete
    const [result1, result2, result3] = await Promise.all([
        promise1,
        promise2,
        promise3
    ]);
    
    console.timeEnd('Parallel');
    return [result1, result2, result3];
}

// Alternative syntax
async function parallelAlternative() {
    const results = await Promise.all([
        fetch('/api/data1').then(r => r.json()),
        fetch('/api/data2').then(r => r.json()),
        fetch('/api/data3').then(r => r.json())
    ]);
    
    return results;
}
```

### Mixed Approach
```javascript
async function mixedExecution() {
    // First, get user (must happen first)
    const user = await fetch('/api/user/1').then(r => r.json());
    
    // Then, get user's data in parallel
    const [posts, comments, followers] = await Promise.all([
        fetch(`/api/users/${user.id}/posts`).then(r => r.json()),
        fetch(`/api/users/${user.id}/comments`).then(r => r.json()),
        fetch(`/api/users/${user.id}/followers`).then(r => r.json())
    ]);
    
    return {
        user,
        posts,
        comments,
        followers
    };
}
```

## Advanced Patterns

### forEach with async/await (Common Mistake)
```javascript
// WRONG - doesn't work as expected
async function wrongForEach() {
    const urls = ['/api/data1', '/api/data2', '/api/data3'];
    const results = [];
    
    urls.forEach(async (url) => {
        const response = await fetch(url);
        const data = await response.json();
        results.push(data); // This won't work as expected
    });
    
    return results; // Returns empty array!
}

// CORRECT - using for...of
async function correctForOf() {
    const urls = ['/api/data1', '/api/data2', '/api/data3'];
    const results = [];
    
    for (const url of urls) {
        const response = await fetch(url);
        const data = await response.json();
        results.push(data);
    }
    
    return results; // Works correctly
}

// CORRECT - using Promise.all for parallel
async function correctParallel() {
    const urls = ['/api/data1', '/api/data2', '/api/data3'];
    
    const results = await Promise.all(
        urls.map(async (url) => {
            const response = await fetch(url);
            return response.json();
        })
    );
    
    return results;
}

// CORRECT - using reduce for sequential
async function correctSequential() {
    const urls = ['/api/data1', '/api/data2', '/api/data3'];
    
    const results = await urls.reduce(async (accumulator, url) => {
        const acc = await accumulator;
        const response = await fetch(url);
        const data = await response.json();
        acc.push(data);
        return acc;
    }, Promise.resolve([]));
    
    return results;
}
```

### Async Generators
```javascript
async function* asyncGenerator() {
    const data = ['a', 'b', 'c'];
    
    for (const item of data) {
        // Simulate async operation
        await new Promise(resolve => setTimeout(resolve, 1000));
        yield item.toUpperCase();
    }
}

// Usage
async function useAsyncGenerator() {
    for await (const value of asyncGenerator()) {
        console.log(value); // A, B, C (with 1 second delay each)
    }
}

useAsyncGenerator();
```

### Pipeline Pattern
```javascript
const pipe = (...functions) => (value) =>
    functions.reduce(async (accumulator, fn) => fn(await accumulator), value);

// Async pipeline functions
const fetchData = async (url) => {
    const response = await fetch(url);
    return response.json();
};

const validateData = async (data) => {
    if (!data || !data.id) {
        throw new Error('Invalid data');
    }
    return data;
};

const transformData = async (data) => {
    return {
        ...data,
        processedAt: new Date().toISOString()
    };
};

const saveData = async (data) => {
    // Simulate save operation
    console.log('Saving:', data);
    return { ...data, saved: true };
};

// Create pipeline
const processPipeline = pipe(
    fetchData,
    validateData,
    transformData,
    saveData
);

// Usage
try {
    const result = await processPipeline('/api/data');
    console.log('Pipeline result:', result);
} catch (error) {
    console.error('Pipeline failed:', error);
}
```

## Real-World Examples

### API Client with Async/Await
```javascript
class APIClient {
    constructor(baseURL, options = {}) {
        this.baseURL = baseURL;
        this.defaultOptions = {
            headers: {
                'Content-Type': 'application/json',
                ...options.headers
            },
            ...options
        };
    }
    
    async request(endpoint, options = {}) {
        const url = `${this.baseURL}${endpoint}`;
        const config = {
            ...this.defaultOptions,
            ...options,
            headers: {
                ...this.defaultOptions.headers,
                ...options.headers
            }
        };
        
        try {
            const response = await fetch(url, config);
            
            if (!response.ok) {
                const errorData = await response.text();
                throw new Error(`HTTP ${response.status}: ${errorData}`);
            }
            
            const contentType = response.headers.get('content-type');
            if (contentType && contentType.includes('application/json')) {
                return await response.json();
            }
            
            return await response.text();
        } catch (error) {
            console.error(`API request failed: ${error.message}`);
            throw error;
        }
    }
    
    async get(endpoint) {
        return this.request(endpoint, { method: 'GET' });
    }
    
    async post(endpoint, data) {
        return this.request(endpoint, {
            method: 'POST',
            body: JSON.stringify(data)
        });
    }
    
    async put(endpoint, data) {
        return this.request(endpoint, {
            method: 'PUT',
            body: JSON.stringify(data)
        });
    }
    
    async delete(endpoint) {
        return this.request(endpoint, { method: 'DELETE' });
    }
}

// Usage
const api = new APIClient('https://api.example.com');

async function manageUser() {
    try {
        // Get user
        const user = await api.get('/users/1');
        console.log('User:', user);
        
        // Update user
        const updatedUser = await api.put('/users/1', {
            ...user,
            lastLogin: new Date().toISOString()
        });
        console.log('Updated user:', updatedUser);
        
        // Get user's posts
        const posts = await api.get(`/users/${user.id}/posts`);
        console.log('User posts:', posts);
        
    } catch (error) {
        console.error('User management failed:', error);
    }
}
```

### Database Operations
```javascript
class DatabaseService {
    constructor(connectionString) {
        this.connectionString = connectionString;
    }
    
    async connect() {
        // Simulate database connection
        console.log('Connecting to database...');
        await new Promise(resolve => setTimeout(resolve, 1000));
        console.log('Connected to database');
    }
    
    async transaction(operations) {
        console.log('Starting transaction...');
        
        try {
            // Begin transaction
            await this.query('BEGIN');
            
            const results = [];
            for (const operation of operations) {
                const result = await operation();
                results.push(result);
            }
            
            // Commit transaction
            await this.query('COMMIT');
            console.log('Transaction committed');
            
            return results;
        } catch (error) {
            // Rollback on error
            await this.query('ROLLBACK');
            console.log('Transaction rolled back');
            throw error;
        }
    }
    
    async query(sql, params = []) {
        // Simulate database query
        console.log(`Executing: ${sql}`, params);
        await new Promise(resolve => setTimeout(resolve, 100));
        return { sql, params, result: 'success' };
    }
    
    async createUser(userData) {
        return this.query(
            'INSERT INTO users (name, email) VALUES (?, ?)',
            [userData.name, userData.email]
        );
    }
    
    async updateUser(id, userData) {
        return this.query(
            'UPDATE users SET name = ?, email = ? WHERE id = ?',
            [userData.name, userData.email, id]
        );
    }
    
    async deleteUser(id) {
        return this.query('DELETE FROM users WHERE id = ?', [id]);
    }
}

// Usage
async function userOperations() {
    const db = new DatabaseService('connection-string');
    
    try {
        await db.connect();
        
        const results = await db.transaction([
            () => db.createUser({ name: 'John', email: 'john@example.com' }),
            () => db.createUser({ name: 'Jane', email: 'jane@example.com' }),
            () => db.updateUser(1, { name: 'John Updated', email: 'john.updated@example.com' })
        ]);
        
        console.log('All operations completed:', results);
    } catch (error) {
        console.error('Database operations failed:', error);
    }
}
```

### File Processing
```javascript
async function processFiles(filePaths) {
    const results = [];
    
    for (const filePath of filePaths) {
        try {
            console.log(`Processing ${filePath}...`);
            
            // Read file
            const content = await readFile(filePath);
            
            // Process content
            const processed = await processContent(content);
            
            // Save processed content
            const outputPath = filePath.replace('.txt', '.processed.txt');
            await writeFile(outputPath, processed);
            
            results.push({
                input: filePath,
                output: outputPath,
                status: 'success'
            });
            
        } catch (error) {
            console.error(`Failed to process ${filePath}:`, error.message);
            results.push({
                input: filePath,
                status: 'error',
                error: error.message
            });
        }
    }
    
    return results;
}

// Simulate file operations
async function readFile(path) {
    await new Promise(resolve => setTimeout(resolve, 100));
    return `Content of ${path}`;
}

async function writeFile(path, content) {
    await new Promise(resolve => setTimeout(resolve, 100));
    console.log(`Wrote to ${path}: ${content}`);
}

async function processContent(content) {
    await new Promise(resolve => setTimeout(resolve, 50));
    return content.toUpperCase();
}

// Usage
const files = ['file1.txt', 'file2.txt', 'file3.txt'];
processFiles(files).then(results => {
    console.log('Processing complete:', results);
});
```

## Best Practices

### 1. Always Handle Errors
```javascript
// Bad - unhandled promise rejection
async function badExample() {
    const data = await fetch('/api/data').then(r => r.json());
    return data; // Error could crash the application
}

// Good - proper error handling
async function goodExample() {
    try {
        const response = await fetch('/api/data');
        if (!response.ok) {
            throw new Error(`HTTP ${response.status}`);
        }
        const data = await response.json();
        return data;
    } catch (error) {
        console.error('Failed to fetch data:', error);
        throw error; // Or handle gracefully
    }
}
```

### 2. Be Mindful of Parallel vs Sequential
```javascript
// Bad - unnecessary sequential execution
async function inefficient() {
    const user = await fetchUser();
    const settings = await fetchSettings(); // Could run in parallel
    const preferences = await fetchPreferences(); // Could run in parallel
    return { user, settings, preferences };
}

// Good - parallel where possible
async function efficient() {
    const [user, settings, preferences] = await Promise.all([
        fetchUser(),
        fetchSettings(),
        fetchPreferences()
    ]);
    return { user, settings, preferences };
}
```

### 3. Don't Mix async/await with .then()
```javascript
// Bad - mixing patterns
async function mixedPattern() {
    const response = await fetch('/api/data');
    return response.json().then(data => data.items); // Don't mix!
}

// Good - consistent pattern
async function consistentPattern() {
    const response = await fetch('/api/data');
    const data = await response.json();
    return data.items;
}
```

### 4. Use Proper Error Boundaries
```javascript
async function withProperErrorHandling() {
    try {
        const criticalData = await fetchCriticalData();
        
        // Non-critical operations with separate error handling
        let optionalData;
        try {
            optionalData = await fetchOptionalData();
        } catch (error) {
            console.warn('Optional data failed:', error);
            optionalData = null; // Graceful degradation
        }
        
        return { criticalData, optionalData };
    } catch (error) {
        // Only critical errors bubble up
        throw new Error(`Critical operation failed: ${error.message}`);
    }
}
```

---

## Related Topics
- [[19 - Promises]]
- [[18 - Callbacks]]
- [[22 - Fetch API]]
- [[42 - Error Handling]]

---

*Next: [[21 - XMLHttpRequest]]*
