# Fetch API

## Overview
The Fetch API provides a modern, promise-based interface for making HTTP requests. It's more powerful and flexible than XMLHttpRequest and has become the standard way to make network requests in modern JavaScript.

## Basic Fetch Syntax

### Simple GET Request
```javascript
// Basic fetch
fetch('https://api.example.com/users')
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error('Error:', error));

// With async/await
async function fetchUsers() {
    try {
        const response = await fetch('https://api.example.com/users');
        const data = await response.json();
        console.log(data);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

### Fetch Options
```javascript
const options = {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer token123'
    },
    body: JSON.stringify({
        name: 'John Doe',
        email: 'john@example.com'
    })
};

fetch('https://api.example.com/users', options)
    .then(response => response.json())
    .then(data => console.log(data));
```

## HTTP Methods

### GET Request
```javascript
// Simple GET
async function getUser(id) {
    const response = await fetch(`/api/users/${id}`);
    
    if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return await response.json();
}

// GET with query parameters
function buildURL(base, params) {
    const url = new URL(base);
    Object.keys(params).forEach(key => {
        url.searchParams.append(key, params[key]);
    });
    return url.toString();
}

async function searchUsers(filters) {
    const url = buildURL('/api/users', {
        name: filters.name,
        page: filters.page || 1,
        limit: filters.limit || 10
    });
    
    const response = await fetch(url);
    return await response.json();
}
```

### POST Request
```javascript
async function createUser(userData) {
    const response = await fetch('/api/users', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(userData)
    });
    
    if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.message || 'Failed to create user');
    }
    
    return await response.json();
}

// Usage
const newUser = {
    name: 'Jane Doe',
    email: 'jane@example.com',
    age: 28
};

createUser(newUser)
    .then(user => console.log('Created user:', user))
    .catch(error => console.error('Error:', error));
```

### PUT Request
```javascript
async function updateUser(id, userData) {
    const response = await fetch(`/api/users/${id}`, {
        method: 'PUT',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(userData)
    });
    
    return await response.json();
}

// Partial update with PATCH
async function patchUser(id, updates) {
    const response = await fetch(`/api/users/${id}`, {
        method: 'PATCH',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify(updates)
    });
    
    return await response.json();
}
```

### DELETE Request
```javascript
async function deleteUser(id) {
    const response = await fetch(`/api/users/${id}`, {
        method: 'DELETE'
    });
    
    if (!response.ok) {
        throw new Error(`Failed to delete user: ${response.status}`);
    }
    
    // Some APIs return 204 No Content for successful deletion
    if (response.status === 204) {
        return { success: true };
    }
    
    return await response.json();
}
```

## Request Configuration

### Headers
```javascript
// Common headers
const commonHeaders = {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
    'Authorization': 'Bearer your-token-here',
    'X-API-Key': 'your-api-key',
    'User-Agent': 'MyApp/1.0'
};

// Dynamic headers
function createAuthHeader(token) {
    return {
        'Authorization': `Bearer ${token}`
    };
}

// Custom headers
const customRequest = await fetch('/api/data', {
    headers: {
        ...commonHeaders,
        'X-Custom-Header': 'custom-value'
    }
});
```

### Request Body Types
```javascript
// JSON data
fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ name: 'John', age: 30 })
});

// Form data
const formData = new FormData();
formData.append('name', 'John');
formData.append('file', fileInput.files[0]);

fetch('/api/upload', {
    method: 'POST',
    body: formData // Don't set Content-Type, browser will set it with boundary
});

// URL-encoded data
const params = new URLSearchParams();
params.append('name', 'John');
params.append('age', '30');

fetch('/api/users', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: params
});

// Plain text
fetch('/api/notes', {
    method: 'POST',
    headers: { 'Content-Type': 'text/plain' },
    body: 'This is a plain text note'
});

// Binary data
fetch('/api/upload', {
    method: 'POST',
    headers: { 'Content-Type': 'application/octet-stream' },
    body: new Uint8Array([1, 2, 3, 4])
});
```

## Response Handling

### Response Object
```javascript
async function handleResponse() {
    const response = await fetch('/api/data');
    
    // Response properties
    console.log(response.status);        // 200, 404, 500, etc.
    console.log(response.statusText);    // "OK", "Not Found", etc.
    console.log(response.ok);           // true if status 200-299
    console.log(response.headers);      // Headers object
    console.log(response.url);          // Final URL (after redirects)
    console.log(response.type);         // "basic", "cors", "opaque"
    console.log(response.redirected);   // true if response is redirect
    
    // Response methods (each can only be called once)
    if (response.ok) {
        const data = await response.json();    // Parse as JSON
        // const text = await response.text();     // Parse as text
        // const blob = await response.blob();     // Parse as Blob
        // const buffer = await response.arrayBuffer(); // Parse as ArrayBuffer
    }
}
```

### Different Response Types
```javascript
async function handleDifferentResponses(url) {
    const response = await fetch(url);
    
    const contentType = response.headers.get('content-type');
    
    if (contentType && contentType.includes('application/json')) {
        return await response.json();
    } else if (contentType && contentType.includes('text/')) {
        return await response.text();
    } else if (contentType && contentType.includes('image/')) {
        return await response.blob();
    } else {
        return await response.arrayBuffer();
    }
}

// Specific response handlers
async function downloadFile(url) {
    const response = await fetch(url);
    const blob = await response.blob();
    
    // Create download link
    const downloadUrl = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = downloadUrl;
    a.download = 'file.pdf';
    a.click();
    URL.revokeObjectURL(downloadUrl);
}

async function loadImage(url) {
    const response = await fetch(url);
    const blob = await response.blob();
    const imageUrl = URL.createObjectURL(blob);
    
    const img = new Image();
    img.src = imageUrl;
    return img;
}
```

## Error Handling

### Network vs HTTP Errors
```javascript
async function fetchWithErrorHandling(url) {
    try {
        const response = await fetch(url);
        
        // Fetch only rejects on network errors, not HTTP errors
        if (!response.ok) {
            // Handle HTTP errors (4xx, 5xx)
            if (response.status >= 400 && response.status < 500) {
                throw new Error(`Client error: ${response.status} ${response.statusText}`);
            } else if (response.status >= 500) {
                throw new Error(`Server error: ${response.status} ${response.statusText}`);
            }
        }
        
        return await response.json();
    } catch (error) {
        if (error instanceof TypeError) {
            // Network error (no internet, CORS, etc.)
            throw new Error('Network error: Please check your connection');
        }
        throw error; // Re-throw other errors
    }
}
```

### Detailed Error Handling
```javascript
class APIError extends Error {
    constructor(message, status, response) {
        super(message);
        this.name = 'APIError';
        this.status = status;
        this.response = response;
    }
}

async function apiCall(url, options = {}) {
    try {
        const response = await fetch(url, options);
        
        if (!response.ok) {
            let errorMessage = `HTTP ${response.status}: ${response.statusText}`;
            let errorBody;
            
            try {
                errorBody = await response.json();
                errorMessage = errorBody.message || errorMessage;
            } catch {
                // Response is not JSON
                errorMessage = await response.text() || errorMessage;
            }
            
            throw new APIError(errorMessage, response.status, errorBody);
        }
        
        return await response.json();
    } catch (error) {
        if (error instanceof APIError) {
            throw error;
        }
        
        // Network or parsing error
        throw new Error(`Request failed: ${error.message}`);
    }
}

// Usage with specific error handling
try {
    const data = await apiCall('/api/users');
    console.log(data);
} catch (error) {
    if (error instanceof APIError) {
        switch (error.status) {
            case 401:
                console.log('Unauthorized - redirect to login');
                break;
            case 403:
                console.log('Forbidden - insufficient permissions');
                break;
            case 404:
                console.log('Not found');
                break;
            case 429:
                console.log('Rate limited - try again later');
                break;
            default:
                console.log(`API error: ${error.message}`);
        }
    } else {
        console.log(`Network error: ${error.message}`);
    }
}
```

## Advanced Features

### Request Timeout
```javascript
function fetchWithTimeout(url, options = {}, timeout = 5000) {
    return Promise.race([
        fetch(url, options),
        new Promise((_, reject) => {
            setTimeout(() => reject(new Error('Request timeout')), timeout);
        })
    ]);
}

// Using AbortController for cancellation
function fetchWithAbort(url, options = {}, timeout = 5000) {
    const controller = new AbortController();
    
    const timeoutId = setTimeout(() => {
        controller.abort();
    }, timeout);
    
    return fetch(url, {
        ...options,
        signal: controller.signal
    }).finally(() => {
        clearTimeout(timeoutId);
    });
}

// Manual cancellation
const controller = new AbortController();

fetch('/api/data', { signal: controller.signal })
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => {
        if (error.name === 'AbortError') {
            console.log('Request was cancelled');
        } else {
            console.error('Request failed:', error);
        }
    });

// Cancel the request
setTimeout(() => {
    controller.abort();
}, 3000);
```

### Request Interceptors
```javascript
class APIClient {
    constructor(baseURL, defaultOptions = {}) {
        this.baseURL = baseURL;
        this.defaultOptions = defaultOptions;
        this.interceptors = {
            request: [],
            response: []
        };
    }
    
    addRequestInterceptor(interceptor) {
        this.interceptors.request.push(interceptor);
    }
    
    addResponseInterceptor(interceptor) {
        this.interceptors.response.push(interceptor);
    }
    
    async request(endpoint, options = {}) {
        let url = `${this.baseURL}${endpoint}`;
        let config = { ...this.defaultOptions, ...options };
        
        // Apply request interceptors
        for (const interceptor of this.interceptors.request) {
            const result = await interceptor(url, config);
            if (result) {
                url = result.url || url;
                config = result.config || config;
            }
        }
        
        let response = await fetch(url, config);
        
        // Apply response interceptors
        for (const interceptor of this.interceptors.response) {
            response = await interceptor(response);
        }
        
        return response;
    }
}

// Usage
const api = new APIClient('https://api.example.com');

// Add auth token to all requests
api.addRequestInterceptor(async (url, config) => {
    const token = localStorage.getItem('authToken');
    if (token) {
        config.headers = {
            ...config.headers,
            'Authorization': `Bearer ${token}`
        };
    }
    return { url, config };
});

// Log all responses
api.addResponseInterceptor(async (response) => {
    console.log(`Response from ${response.url}:`, response.status);
    return response;
});
```

### Retry Logic
```javascript
async function fetchWithRetry(url, options = {}, maxRetries = 3, delay = 1000) {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            const response = await fetch(url, options);
            
            // Only retry on 5xx errors or network errors
            if (response.ok || (response.status >= 400 && response.status < 500)) {
                return response;
            }
            
            if (attempt === maxRetries) {
                throw new Error(`Request failed after ${maxRetries} attempts`);
            }
            
        } catch (error) {
            if (attempt === maxRetries) {
                throw error;
            }
            
            console.log(`Attempt ${attempt} failed, retrying in ${delay}ms...`);
        }
        
        // Wait before retry with exponential backoff
        await new Promise(resolve => setTimeout(resolve, delay * attempt));
    }
}

// Advanced retry with conditions
async function smartRetry(url, options = {}, retryConfig = {}) {
    const {
        maxRetries = 3,
        baseDelay = 1000,
        maxDelay = 10000,
        retryCondition = (error, response) => true
    } = retryConfig;
    
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            const response = await fetch(url, options);
            
            if (response.ok) {
                return response;
            }
            
            if (!retryCondition(null, response)) {
                return response;
            }
            
            if (attempt === maxRetries) {
                return response;
            }
            
        } catch (error) {
            if (!retryCondition(error, null) || attempt === maxRetries) {
                throw error;
            }
        }
        
        // Exponential backoff with jitter
        const delay = Math.min(
            baseDelay * Math.pow(2, attempt - 1) + Math.random() * 1000,
            maxDelay
        );
        
        await new Promise(resolve => setTimeout(resolve, delay));
    }
}
```

### Parallel Requests
```javascript
// Multiple independent requests
async function fetchUserData(userId) {
    const [user, posts, comments] = await Promise.all([
        fetch(`/api/users/${userId}`).then(r => r.json()),
        fetch(`/api/users/${userId}/posts`).then(r => r.json()),
        fetch(`/api/users/${userId}/comments`).then(r => r.json())
    ]);
    
    return { user, posts, comments };
}

// Batch requests with error handling
async function fetchMultipleUsers(userIds) {
    const requests = userIds.map(async (id) => {
        try {
            const response = await fetch(`/api/users/${id}`);
            return { id, data: await response.json(), error: null };
        } catch (error) {
            return { id, data: null, error: error.message };
        }
    });
    
    return await Promise.all(requests);
}

// Rate-limited parallel requests
async function fetchWithRateLimit(urls, limit = 3) {
    const results = [];
    
    for (let i = 0; i < urls.length; i += limit) {
        const batch = urls.slice(i, i + limit);
        const batchResults = await Promise.all(
            batch.map(url => fetch(url).then(r => r.json()))
        );
        results.push(...batchResults);
    }
    
    return results;
}
```

## Real-World Examples

### Complete API Client
```javascript
class RESTClient {
    constructor(baseURL, options = {}) {
        this.baseURL = baseURL.replace(/\/$/, '');
        this.defaultOptions = {
            headers: {
                'Content-Type': 'application/json'
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
        
        const response = await fetch(url, config);
        
        if (!response.ok) {
            const error = await this.parseError(response);
            throw error;
        }
        
        return this.parseResponse(response);
    }
    
    async parseResponse(response) {
        const contentType = response.headers.get('content-type');
        
        if (contentType && contentType.includes('application/json')) {
            return await response.json();
        }
        return await response.text();
    }
    
    async parseError(response) {
        let message = `HTTP ${response.status}: ${response.statusText}`;
        
        try {
            const errorData = await response.json();
            message = errorData.message || errorData.error || message;
        } catch {
            // Not JSON, use status text
        }
        
        const error = new Error(message);
        error.status = response.status;
        return error;
    }
    
    get(endpoint, params = {}) {
        const url = new URL(endpoint, this.baseURL);
        Object.keys(params).forEach(key => {
            url.searchParams.append(key, params[key]);
        });
        
        return this.request(url.pathname + url.search);
    }
    
    post(endpoint, data) {
        return this.request(endpoint, {
            method: 'POST',
            body: JSON.stringify(data)
        });
    }
    
    put(endpoint, data) {
        return this.request(endpoint, {
            method: 'PUT',
            body: JSON.stringify(data)
        });
    }
    
    patch(endpoint, data) {
        return this.request(endpoint, {
            method: 'PATCH',
            body: JSON.stringify(data)
        });
    }
    
    delete(endpoint) {
        return this.request(endpoint, {
            method: 'DELETE'
        });
    }
}

// Usage
const api = new RESTClient('https://api.example.com', {
    headers: {
        'Authorization': 'Bearer token123'
    }
});

try {
    const users = await api.get('/users', { page: 1, limit: 10 });
    const newUser = await api.post('/users', { name: 'John', email: 'john@example.com' });
    await api.delete(`/users/${newUser.id}`);
} catch (error) {
    console.error('API error:', error.message);
}
```

---

## Related Topics
- [[19 - Promises]]
- [[20 - Async Await]]
- [[21 - XMLHttpRequest]]
- [[23 - Axios]]
- [[42 - Error Handling]]

---

*Next: [[23 - Axios]]*
