# Axios

## Overview
Axios is a popular promise-based HTTP client for JavaScript that works in browsers and Node.js. It provides a more convenient and feature-rich alternative to the native Fetch API and XMLHttpRequest, with automatic JSON parsing, request/response interceptors, and comprehensive error handling.

## Installation and Setup

### CDN Installation
```html
<!-- CDN via unpkg -->
<script src="https://unpkg.com/axios/dist/axios.min.js"></script>

<!-- CDN via jsDelivr -->
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

### NPM Installation
```bash
# Install via npm
npm install axios

# Install via yarn
yarn add axios
```

### ES6 Import
```javascript
// ES6 import
import axios from 'axios';

// CommonJS require
const axios = require('axios');

// Browser global (when using CDN)
// axios is available as global variable
```

## Basic Usage

### Simple GET Request
```javascript
// Basic GET request
axios.get('https://jsonplaceholder.typicode.com/posts/1')
    .then(response => {
        console.log('Status:', response.status);
        console.log('Data:', response.data);
        console.log('Headers:', response.headers);
    })
    .catch(error => {
        console.error('Error:', error.message);
    });

// GET request with parameters
axios.get('https://jsonplaceholder.typicode.com/posts', {
    params: {
        userId: 1,
        _limit: 5
    }
})
    .then(response => {
        console.log('Posts:', response.data);
    })
    .catch(error => {
        console.error('Error:', error);
    });

// GET request with custom headers
axios.get('https://api.example.com/data', {
    headers: {
        'Authorization': 'Bearer your-token-here',
        'Content-Type': 'application/json',
        'X-Custom-Header': 'custom-value'
    },
    timeout: 5000 // 5 second timeout
})
    .then(response => {
        console.log('Data with auth:', response.data);
    })
    .catch(error => {
        if (error.code === 'ECONNABORTED') {
            console.error('Request timed out');
        } else {
            console.error('Error:', error.message);
        }
    });

// Async/await syntax
async function fetchUser(userId) {
    try {
        const response = await axios.get(`https://jsonplaceholder.typicode.com/users/${userId}`);
        return response.data;
    } catch (error) {
        console.error('Failed to fetch user:', error.message);
        throw error;
    }
}

// Usage
fetchUser(1)
    .then(user => console.log('User:', user))
    .catch(error => console.error('Error:', error));
```

### POST Requests
```javascript
// Basic POST request
const newPost = {
    title: 'My New Post',
    body: 'This is the content of my new post',
    userId: 1
};

axios.post('https://jsonplaceholder.typicode.com/posts', newPost)
    .then(response => {
        console.log('Created post:', response.data);
        console.log('Status:', response.status);
    })
    .catch(error => {
        console.error('Error creating post:', error.response?.data || error.message);
    });

// POST with custom headers and config
axios.post('https://api.example.com/users', {
    name: 'John Doe',
    email: 'john@example.com',
    role: 'admin'
}, {
    headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer token123'
    },
    timeout: 10000,
    validateStatus: function (status) {
        return status < 500; // Accept any status code less than 500
    }
})
    .then(response => {
        console.log('User created:', response.data);
    })
    .catch(error => {
        console.error('Failed to create user:', error);
    });

// Form data POST
const formData = new FormData();
formData.append('name', 'John Doe');
formData.append('email', 'john@example.com');
formData.append('avatar', fileInput.files[0]); // File from input

axios.post('https://api.example.com/upload', formData, {
    headers: {
        'Content-Type': 'multipart/form-data'
    },
    onUploadProgress: (progressEvent) => {
        const percentCompleted = Math.round((progressEvent.loaded * 100) / progressEvent.total);
        console.log(`Upload progress: ${percentCompleted}%`);
    }
})
    .then(response => {
        console.log('Upload successful:', response.data);
    })
    .catch(error => {
        console.error('Upload failed:', error);
    });

// URL-encoded form POST
const params = new URLSearchParams();
params.append('username', 'john_doe');
params.append('password', 'secret123');

axios.post('https://api.example.com/login', params, {
    headers: {
        'Content-Type': 'application/x-www-form-urlencoded'
    }
})
    .then(response => {
        console.log('Login successful:', response.data);
    })
    .catch(error => {
        console.error('Login failed:', error.response?.data);
    });
```

### PUT, PATCH, and DELETE Requests
```javascript
// PUT request (complete resource update)
const updatedPost = {
    id: 1,
    title: 'Updated Post Title',
    body: 'This is the updated content',
    userId: 1
};

axios.put('https://jsonplaceholder.typicode.com/posts/1', updatedPost)
    .then(response => {
        console.log('Post updated:', response.data);
    })
    .catch(error => {
        console.error('Update failed:', error);
    });

// PATCH request (partial resource update)
axios.patch('https://jsonplaceholder.typicode.com/posts/1', {
    title: 'Partially Updated Title'
})
    .then(response => {
        console.log('Post partially updated:', response.data);
    })
    .catch(error => {
        console.error('Partial update failed:', error);
    });

// DELETE request
axios.delete('https://jsonplaceholder.typicode.com/posts/1')
    .then(response => {
        console.log('Post deleted:', response.status);
    })
    .catch(error => {
        console.error('Delete failed:', error);
    });

// DELETE with data (some APIs support this)
axios.delete('https://api.example.com/posts/1', {
    data: {
        reason: 'Inappropriate content'
    },
    headers: {
        'Authorization': 'Bearer token123'
    }
})
    .then(response => {
        console.log('Post deleted with reason:', response.data);
    })
    .catch(error => {
        console.error('Delete failed:', error);
    });
```

## Axios Configuration

### Default Configuration
```javascript
// Set default base URL
axios.defaults.baseURL = 'https://api.example.com';

// Set default headers
axios.defaults.headers.common['Authorization'] = 'Bearer token123';
axios.defaults.headers.post['Content-Type'] = 'application/json';

// Set default timeout
axios.defaults.timeout = 10000;

// Set default retry behavior (with axios-retry plugin)
axios.defaults.retry = 3;
axios.defaults.retryDelay = 1000;

// Custom default config
axios.defaults.validateStatus = function (status) {
    return status >= 200 && status < 300; // Default behavior
};

// Now all requests will use these defaults
axios.get('/users') // Will request https://api.example.com/users
    .then(response => console.log(response.data));

axios.post('/users', { name: 'John' }) // Will include Authorization header
    .then(response => console.log(response.data));
```

### Custom Axios Instances
```javascript
// Create custom instance with specific config
const apiClient = axios.create({
    baseURL: 'https://api.example.com',
    timeout: 5000,
    headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json'
    }
});

// Create another instance for different API
const authClient = axios.create({
    baseURL: 'https://auth.example.com',
    timeout: 3000,
    headers: {
        'Content-Type': 'application/json'
    }
});

// Use instances
apiClient.get('/users')
    .then(response => console.log('API users:', response.data));

authClient.post('/login', { username: 'user', password: 'pass' })
    .then(response => console.log('Auth response:', response.data));

// Instance-specific configuration
apiClient.defaults.headers.common['X-API-Key'] = 'your-api-key';

// Override instance config for specific request
apiClient.get('/public-data', {
    timeout: 1000, // Override default timeout
    headers: {
        'Authorization': null // Remove authorization for public endpoint
    }
})
    .then(response => console.log('Public data:', response.data));
```

### Advanced Configuration
```javascript
// Comprehensive axios configuration
const advancedClient = axios.create({
    baseURL: 'https://api.example.com',
    timeout: 10000,
    
    // Headers
    headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
        'X-Requested-With': 'XMLHttpRequest'
    },
    
    // Request/response data transformation
    transformRequest: [
        function (data, headers) {
            // Transform request data
            if (data && typeof data === 'object') {
                // Add timestamp to all requests
                data._timestamp = Date.now();
            }
            return JSON.stringify(data);
        }
    ],
    
    transformResponse: [
        function (data) {
            // Transform response data
            try {
                const parsed = JSON.parse(data);
                // Add processed timestamp
                if (parsed && typeof parsed === 'object') {
                    parsed._processed = new Date().toISOString();
                }
                return parsed;
            } catch (error) {
                return data;
            }
        }
    ],
    
    // Validate status codes
    validateStatus: function (status) {
        return status >= 200 && status < 300;
    },
    
    // Maximum content length
    maxContentLength: 2000000, // 2MB
    
    // Maximum redirects
    maxRedirects: 5,
    
    // Proxy configuration (Node.js only)
    proxy: {
        host: '127.0.0.1',
        port: 9000,
        auth: {
            username: 'proxy-user',
            password: 'proxy-pass'
        }
    },
    
    // Custom HTTP adapter (advanced)
    adapter: function (config) {
        // Custom request logic
        return new Promise((resolve, reject) => {
            // Implementation would go here
        });
    }
});

// Environment-specific configuration
const isDevelopment = process.env.NODE_ENV === 'development';

const environmentClient = axios.create({
    baseURL: isDevelopment 
        ? 'http://localhost:3000/api' 
        : 'https://api.production.com',
    timeout: isDevelopment ? 30000 : 10000,
    headers: {
        'X-Environment': isDevelopment ? 'development' : 'production'
    }
});
```

## Interceptors

### Request Interceptors
```javascript
// Request interceptor for authentication
axios.interceptors.request.use(
    function (config) {
        // Do something before request is sent
        console.log('Sending request:', config.method?.toUpperCase(), config.url);
        
        // Add auth token if available
        const token = localStorage.getItem('authToken');
        if (token) {
            config.headers.Authorization = `Bearer ${token}`;
        }
        
        // Add request timestamp
        config.metadata = { startTime: new Date() };
        
        return config;
    },
    function (error) {
        // Do something with request error
        console.error('Request error:', error);
        return Promise.reject(error);
    }
);

// Multiple request interceptors
axios.interceptors.request.use(config => {
    // Add correlation ID for tracking
    config.headers['X-Correlation-ID'] = Math.random().toString(36);
    return config;
});

axios.interceptors.request.use(config => {
    // Add user agent info
    config.headers['X-User-Agent'] = navigator.userAgent;
    return config;
});

// Conditional request interceptor
axios.interceptors.request.use(config => {
    // Only add API key for certain endpoints
    if (config.url?.includes('/api/')) {
        config.headers['X-API-Key'] = process.env.API_KEY;
    }
    
    // Log requests in development
    if (process.env.NODE_ENV === 'development') {
        console.log('Request config:', {
            method: config.method,
            url: config.url,
            data: config.data,
            headers: config.headers
        });
    }
    
    return config;
});
```

### Response Interceptors
```javascript
// Response interceptor for logging and error handling
axios.interceptors.response.use(
    function (response) {
        // Log response time
        const endTime = new Date();
        const startTime = response.config.metadata?.startTime;
        if (startTime) {
            const duration = endTime - startTime;
            console.log(`Request completed in ${duration}ms`);
        }
        
        // Log successful responses
        console.log('Response received:', response.status, response.statusText);
        
        return response;
    },
    function (error) {
        // Handle different types of errors
        if (error.response) {
            // Server responded with error status
            console.error('Response error:', error.response.status, error.response.data);
            
            // Handle specific status codes
            switch (error.response.status) {
                case 401:
                    console.warn('Unauthorized - redirecting to login');
                    // Redirect to login or refresh token
                    break;
                case 403:
                    console.warn('Forbidden - insufficient permissions');
                    break;
                case 404:
                    console.warn('Resource not found');
                    break;
                case 500:
                    console.error('Server error - please try again later');
                    break;
            }
        } else if (error.request) {
            // Request was made but no response received
            console.error('Network error - no response received');
        } else {
            // Something else happened
            console.error('Request setup error:', error.message);
        }
        
        return Promise.reject(error);
    }
);

// Token refresh interceptor
let isRefreshing = false;
let failedQueue = [];

const processQueue = (error, token = null) => {
    failedQueue.forEach(prom => {
        if (error) {
            prom.reject(error);
        } else {
            prom.resolve(token);
        }
    });
    
    failedQueue = [];
};

axios.interceptors.response.use(
    response => response,
    async error => {
        const originalRequest = error.config;
        
        if (error.response?.status === 401 && !originalRequest._retry) {
            if (isRefreshing) {
                // Queue the request
                return new Promise((resolve, reject) => {
                    failedQueue.push({ resolve, reject });
                }).then(token => {
                    originalRequest.headers.Authorization = `Bearer ${token}`;
                    return axios(originalRequest);
                }).catch(err => {
                    return Promise.reject(err);
                });
            }
            
            originalRequest._retry = true;
            isRefreshing = true;
            
            try {
                const refreshToken = localStorage.getItem('refreshToken');
                const response = await axios.post('/auth/refresh', {
                    refresh_token: refreshToken
                });
                
                const { access_token } = response.data;
                localStorage.setItem('authToken', access_token);
                
                // Update default header
                axios.defaults.headers.common.Authorization = `Bearer ${access_token}`;
                
                processQueue(null, access_token);
                
                // Retry original request
                originalRequest.headers.Authorization = `Bearer ${access_token}`;
                return axios(originalRequest);
                
            } catch (refreshError) {
                processQueue(refreshError, null);
                
                // Refresh failed - redirect to login
                localStorage.removeItem('authToken');
                localStorage.removeItem('refreshToken');
                window.location.href = '/login';
                
                return Promise.reject(refreshError);
            } finally {
                isRefreshing = false;
            }
        }
        
        return Promise.reject(error);
    }
);

// Response transformation interceptor
axios.interceptors.response.use(response => {
    // Normalize API responses
    if (response.data && typeof response.data === 'object') {
        // Convert snake_case to camelCase
        response.data = convertKeysToCamelCase(response.data);
        
        // Add metadata
        response.data._meta = {
            status: response.status,
            timestamp: new Date().toISOString(),
            duration: response.config.metadata?.startTime 
                ? new Date() - response.config.metadata.startTime 
                : null
        };
    }
    
    return response;
});

function convertKeysToCamelCase(obj) {
    if (Array.isArray(obj)) {
        return obj.map(convertKeysToCamelCase);
    } else if (obj !== null && typeof obj === 'object') {
        const result = {};
        for (const key in obj) {
            if (obj.hasOwnProperty(key)) {
                const camelKey = key.replace(/_([a-z])/g, (match, letter) => letter.toUpperCase());
                result[camelKey] = convertKeysToCamelCase(obj[key]);
            }
        }
        return result;
    }
    return obj;
}
```

### Removing Interceptors
```javascript
// Store interceptor ID for later removal
const requestInterceptorId = axios.interceptors.request.use(
    config => {
        console.log('Request interceptor');
        return config;
    }
);

const responseInterceptorId = axios.interceptors.response.use(
    response => {
        console.log('Response interceptor');
        return response;
    }
);

// Remove interceptors
axios.interceptors.request.eject(requestInterceptorId);
axios.interceptors.response.eject(responseInterceptorId);

// Clear all interceptors
axios.interceptors.request.clear();
axios.interceptors.response.clear();
```

## Error Handling

### Comprehensive Error Handling
```javascript
// Custom error class
class APIError extends Error {
    constructor(message, status, data, config) {
        super(message);
        this.name = 'APIError';
        this.status = status;
        this.data = data;
        this.config = config;
    }
}

// Error handling utility
function handleAxiosError(error) {
    if (error.response) {
        // Server responded with error status
        const { status, data, headers } = error.response;
        const message = data?.message || data?.error || `HTTP ${status} Error`;
        
        console.error('API Error:', {
            status,
            message,
            data,
            url: error.config?.url,
            method: error.config?.method
        });
        
        throw new APIError(message, status, data, error.config);
        
    } else if (error.request) {
        // Network error or no response
        const message = 'Network error - please check your connection';
        console.error('Network Error:', error.request);
        throw new APIError(message, 0, null, error.config);
        
    } else {
        // Request setup error
        const message = `Request Error: ${error.message}`;
        console.error('Request Setup Error:', error.message);
        throw new APIError(message, -1, null, error.config);
    }
}

// Usage with proper error handling
async function fetchUserWithErrorHandling(userId) {
    try {
        const response = await axios.get(`/users/${userId}`);
        return response.data;
        
    } catch (error) {
        handleAxiosError(error);
    }
}

// Retry mechanism for failed requests
async function retryRequest(requestFn, maxRetries = 3, delay = 1000) {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
        try {
            return await requestFn();
        } catch (error) {
            if (attempt === maxRetries) {
                throw error;
            }
            
            // Only retry on network errors or 5xx server errors
            if (error.response && error.response.status < 500) {
                throw error; // Don't retry client errors
            }
            
            console.warn(`Attempt ${attempt} failed, retrying in ${delay}ms...`);
            await new Promise(resolve => setTimeout(resolve, delay));
            delay *= 2; // Exponential backoff
        }
    }
}

// Usage with retry
async function robustApiCall() {
    try {
        const result = await retryRequest(
            () => axios.get('/unstable-endpoint'),
            3, // max retries
            1000 // initial delay
        );
        return result.data;
    } catch (error) {
        console.error('All retry attempts failed:', error);
        throw error;
    }
}

// Global error handler
axios.interceptors.response.use(
    response => response,
    error => {
        // Log all errors globally
        console.error('Axios Error:', {
            url: error.config?.url,
            method: error.config?.method,
            status: error.response?.status,
            message: error.message,
            timestamp: new Date().toISOString()
        });
        
        // Show user-friendly error messages
        if (error.response?.status >= 500) {
            showNotification('Server error - please try again later', 'error');
        } else if (error.response?.status === 404) {
            showNotification('Requested resource not found', 'warning');
        } else if (!error.response) {
            showNotification('Connection error - please check your internet', 'error');
        }
        
        return Promise.reject(error);
    }
);

function showNotification(message, type) {
    // Implementation depends on your notification system
    console.log(`[${type.toUpperCase()}] ${message}`);
}
```

## Advanced Features

### Request and Response Transformation
```javascript
// Custom transformations
const apiClient = axios.create({
    baseURL: 'https://api.example.com',
    
    // Transform request data
    transformRequest: [
        function (data, headers) {
            // Convert camelCase to snake_case for API
            if (data && typeof data === 'object') {
                data = convertKeysToSnakeCase(data);
            }
            
            // Add request metadata
            data = {
                ...data,
                _client_info: {
                    version: '1.0.0',
                    timestamp: new Date().toISOString(),
                    user_agent: navigator.userAgent
                }
            };
            
            return JSON.stringify(data);
        }
    ],
    
    // Transform response data
    transformResponse: [
        function (data) {
            try {
                const parsed = JSON.parse(data);
                
                // Convert snake_case to camelCase
                if (parsed && typeof parsed === 'object') {
                    parsed = convertKeysToCamelCase(parsed);
                }
                
                // Validate response structure
                if (parsed && !parsed.data && !parsed.error) {
                    return { data: parsed }; // Wrap in standard format
                }
                
                return parsed;
            } catch (error) {
                return data; // Return raw data if parsing fails
            }
        }
    ]
});

function convertKeysToSnakeCase(obj) {
    if (Array.isArray(obj)) {
        return obj.map(convertKeysToSnakeCase);
    } else if (obj !== null && typeof obj === 'object') {
        const result = {};
        for (const key in obj) {
            if (obj.hasOwnProperty(key)) {
                const snakeKey = key.replace(/[A-Z]/g, letter => `_${letter.toLowerCase()}`);
                result[snakeKey] = convertKeysToSnakeCase(obj[key]);
            }
        }
        return result;
    }
    return obj;
}
```

### File Upload and Download
```javascript
// File upload with progress tracking
async function uploadFile(file, onProgress) {
    const formData = new FormData();
    formData.append('file', file);
    formData.append('metadata', JSON.stringify({
        originalName: file.name,
        size: file.size,
        type: file.type,
        uploadedAt: new Date().toISOString()
    }));
    
    try {
        const response = await axios.post('/upload', formData, {
            headers: {
                'Content-Type': 'multipart/form-data'
            },
            onUploadProgress: (progressEvent) => {
                const percentCompleted = Math.round(
                    (progressEvent.loaded * 100) / progressEvent.total
                );
                onProgress?.(percentCompleted, progressEvent);
            },
            timeout: 30000 // 30 seconds for file upload
        });
        
        return response.data;
    } catch (error) {
        console.error('File upload failed:', error);
        throw error;
    }
}

// File download with progress tracking
async function downloadFile(url, filename, onProgress) {
    try {
        const response = await axios.get(url, {
            responseType: 'blob', // Important for file downloads
            onDownloadProgress: (progressEvent) => {
                if (progressEvent.lengthComputable) {
                    const percentCompleted = Math.round(
                        (progressEvent.loaded * 100) / progressEvent.total
                    );
                    onProgress?.(percentCompleted, progressEvent);
                }
            }
        });
        
        // Create download link
        const blob = new Blob([response.data]);
        const downloadUrl = window.URL.createObjectURL(blob);
        const link = document.createElement('a');
        link.href = downloadUrl;
        link.download = filename;
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
        window.URL.revokeObjectURL(downloadUrl);
        
        return response.data;
    } catch (error) {
        console.error('File download failed:', error);
        throw error;
    }
}

// Multiple file upload
async function uploadMultipleFiles(files, onProgress) {
    const uploads = Array.from(files).map((file, index) => {
        return uploadFile(file, (percent, event) => {
            onProgress?.({
                fileIndex: index,
                fileName: file.name,
                percent,
                event
            });
        });
    });
    
    try {
        const results = await Promise.all(uploads);
        return results;
    } catch (error) {
        console.error('Multiple file upload failed:', error);
        throw error;
    }
}

// Chunked file upload for large files
async function uploadLargeFile(file, chunkSize = 1024 * 1024) { // 1MB chunks
    const totalChunks = Math.ceil(file.size / chunkSize);
    const uploadId = Date.now().toString(36) + Math.random().toString(36);
    
    for (let chunkIndex = 0; chunkIndex < totalChunks; chunkIndex++) {
        const start = chunkIndex * chunkSize;
        const end = Math.min(start + chunkSize, file.size);
        const chunk = file.slice(start, end);
        
        const formData = new FormData();
        formData.append('chunk', chunk);
        formData.append('chunkIndex', chunkIndex);
        formData.append('totalChunks', totalChunks);
        formData.append('uploadId', uploadId);
        formData.append('fileName', file.name);
        
        try {
            await axios.post('/upload-chunk', formData, {
                headers: { 'Content-Type': 'multipart/form-data' }
            });
            
            console.log(`Uploaded chunk ${chunkIndex + 1}/${totalChunks}`);
        } catch (error) {
            console.error(`Failed to upload chunk ${chunkIndex}:`, error);
            throw error;
        }
    }
    
    // Finalize upload
    const response = await axios.post('/finalize-upload', {
        uploadId,
        fileName: file.name,
        totalChunks
    });
    
    return response.data;
}
```

### Request Cancellation
```javascript
// Cancel requests using AbortController (modern approach)
async function cancellableRequest() {
    const controller = new AbortController();
    
    try {
        const response = await axios.get('/slow-endpoint', {
            signal: controller.signal,
            timeout: 10000
        });
        
        return response.data;
    } catch (error) {
        if (error.name === 'AbortError') {
            console.log('Request was cancelled');
        } else {
            console.error('Request failed:', error);
        }
        throw error;
    }
}

// Cancel after 5 seconds
const controller = new AbortController();
setTimeout(() => {
    controller.abort();
}, 5000);

// Legacy cancellation using CancelToken (deprecated but still supported)
function legacyCancellableRequest() {
    const source = axios.CancelToken.source();
    
    const request = axios.get('/slow-endpoint', {
        cancelToken: source.token
    });
    
    // Cancel after 5 seconds
    setTimeout(() => {
        source.cancel('Request cancelled by user');
    }, 5000);
    
    return request.catch(error => {
        if (axios.isCancel(error)) {
            console.log('Request cancelled:', error.message);
        } else {
            console.error('Request failed:', error);
        }
        throw error;
    });
}

// Request queue with cancellation
class RequestQueue {
    constructor() {
        this.activeRequests = new Map();
    }
    
    async makeRequest(id, config) {
        // Cancel existing request with same ID
        this.cancelRequest(id);
        
        const controller = new AbortController();
        this.activeRequests.set(id, controller);
        
        try {
            const response = await axios({
                ...config,
                signal: controller.signal
            });
            
            this.activeRequests.delete(id);
            return response;
        } catch (error) {
            this.activeRequests.delete(id);
            throw error;
        }
    }
    
    cancelRequest(id) {
        const controller = this.activeRequests.get(id);
        if (controller) {
            controller.abort();
            this.activeRequests.delete(id);
        }
    }
    
    cancelAllRequests() {
        for (const [id, controller] of this.activeRequests) {
            controller.abort();
        }
        this.activeRequests.clear();
    }
}

// Usage
const requestQueue = new RequestQueue();

// Search with automatic cancellation of previous search
async function search(query) {
    try {
        const response = await requestQueue.makeRequest('search', {
            method: 'GET',
            url: '/search',
            params: { q: query }
        });
        
        return response.data;
    } catch (error) {
        if (error.name === 'AbortError') {
            console.log('Search cancelled');
        } else {
            console.error('Search failed:', error);
        }
    }
}

// Auto-cancelling search input
const searchInput = document.getElementById('search');
searchInput.addEventListener('input', (event) => {
    const query = event.target.value;
    if (query.length > 2) {
        search(query).then(results => {
            console.log('Search results:', results);
        });
    }
});
```

### Concurrent Requests
```javascript
// Multiple concurrent requests
async function fetchUserData(userId) {
    try {
        // Make multiple requests concurrently
        const [profile, posts, followers, following] = await Promise.all([
            axios.get(`/users/${userId}`),
            axios.get(`/users/${userId}/posts`),
            axios.get(`/users/${userId}/followers`),
            axios.get(`/users/${userId}/following`)
        ]);
        
        return {
            profile: profile.data,
            posts: posts.data,
            followers: followers.data,
            following: following.data
        };
    } catch (error) {
        console.error('Failed to fetch user data:', error);
        throw error;
    }
}

// Concurrent requests with error handling
async function fetchUserDataRobust(userId) {
    const requests = {
        profile: axios.get(`/users/${userId}`),
        posts: axios.get(`/users/${userId}/posts`),
        followers: axios.get(`/users/${userId}/followers`),
        following: axios.get(`/users/${userId}/following`)
    };
    
    const results = await Promise.allSettled(Object.values(requests));
    const data = {};
    
    Object.keys(requests).forEach((key, index) => {
        const result = results[index];
        if (result.status === 'fulfilled') {
            data[key] = result.value.data;
        } else {
            console.error(`Failed to fetch ${key}:`, result.reason);
            data[key] = null; // or default value
        }
    });
    
    return data;
}

// Batch requests with rate limiting
class BatchRequestManager {
    constructor(concurrencyLimit = 5, delayBetweenBatches = 100) {
        this.concurrencyLimit = concurrencyLimit;
        this.delayBetweenBatches = delayBetweenBatches;
    }
    
    async executeBatch(requests) {
        const results = [];
        
        for (let i = 0; i < requests.length; i += this.concurrencyLimit) {
            const batch = requests.slice(i, i + this.concurrencyLimit);
            
            try {
                const batchResults = await Promise.all(
                    batch.map(request => axios(request))
                );
                results.push(...batchResults);
            } catch (error) {
                console.error(`Batch ${Math.floor(i / this.concurrencyLimit)} failed:`, error);
                results.push(...batch.map(() => null)); // Placeholder for failed requests
            }
            
            // Add delay between batches to avoid overwhelming the server
            if (i + this.concurrencyLimit < requests.length) {
                await new Promise(resolve => setTimeout(resolve, this.delayBetweenBatches));
            }
        }
        
        return results;
    }
}

// Usage
const batchManager = new BatchRequestManager(3, 200); // 3 concurrent, 200ms delay

const requests = [
    { method: 'GET', url: '/users/1' },
    { method: 'GET', url: '/users/2' },
    { method: 'GET', url: '/users/3' },
    { method: 'GET', url: '/users/4' },
    { method: 'GET', url: '/users/5' }
];

batchManager.executeBatch(requests)
    .then(results => {
        console.log('Batch results:', results);
    })
    .catch(error => {
        console.error('Batch execution failed:', error);
    });
```

---

## Related Topics
- [[21 - XMLHttpRequest]]
- [[22 - Fetch API]]
- [[19 - Promises]]
- [[20 - Async Await]]
- [[42 - Error Handling]]

---

*Next: [[30 - Deep Copy vs Shallow Copy]]*
