# XMLHttpRequest

## Overview
XMLHttpRequest (XHR) is the traditional way to make HTTP requests in JavaScript. While modern applications often use the Fetch API, understanding XHR is important for maintaining legacy code and understanding the foundations of AJAX programming.

## Basic XMLHttpRequest Usage

### Simple GET Request
```javascript
// Basic GET request
function makeGetRequest(url, callback) {
    const xhr = new XMLHttpRequest();
    
    // Configure the request
    xhr.open('GET', url, true); // method, url, async
    
    // Set up event handlers
    xhr.onreadystatechange = function() {
        if (xhr.readyState === XMLHttpRequest.DONE) {
            if (xhr.status === 200) {
                callback(null, xhr.responseText);
            } else {
                callback(new Error(`Request failed with status: ${xhr.status}`));
            }
        }
    };
    
    // Send the request
    xhr.send();
}

// Usage
makeGetRequest('https://jsonplaceholder.typicode.com/posts/1', (error, data) => {
    if (error) {
        console.error('Error:', error.message);
    } else {
        console.log('Response:', JSON.parse(data));
    }
});

// More detailed GET request with headers
function detailedGetRequest(url) {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        
        xhr.open('GET', url, true);
        
        // Set request headers
        xhr.setRequestHeader('Accept', 'application/json');
        xhr.setRequestHeader('Content-Type', 'application/json');
        
        // Handle different ready states
        xhr.onreadystatechange = function() {
            console.log('Ready state:', xhr.readyState);
            
            switch(xhr.readyState) {
                case 0: // UNSENT
                    console.log('Request not initialized');
                    break;
                case 1: // OPENED
                    console.log('Server connection established');
                    break;
                case 2: // HEADERS_RECEIVED
                    console.log('Request received');
                    break;
                case 3: // LOADING
                    console.log('Processing request');
                    break;
                case 4: // DONE
                    console.log('Request finished and response ready');
                    
                    if (xhr.status >= 200 && xhr.status < 300) {
                        try {
                            const data = JSON.parse(xhr.responseText);
                            resolve(data);
                        } catch (parseError) {
                            reject(new Error('Failed to parse JSON response'));
                        }
                    } else {
                        reject(new Error(`HTTP Error: ${xhr.status} ${xhr.statusText}`));
                    }
                    break;
            }
        };
        
        // Handle network errors
        xhr.onerror = function() {
            reject(new Error('Network error occurred'));
        };
        
        // Handle timeout
        xhr.ontimeout = function() {
            reject(new Error('Request timed out'));
        };
        
        // Set timeout (5 seconds)
        xhr.timeout = 5000;
        
        xhr.send();
    });
}

// Usage with Promise
detailedGetRequest('https://jsonplaceholder.typicode.com/posts/1')
    .then(data => console.log('Success:', data))
    .catch(error => console.error('Error:', error.message));
```

### POST Request with Data
```javascript
// POST request with JSON data
function makePostRequest(url, data) {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        
        xhr.open('POST', url, true);
        
        // Set headers for JSON
        xhr.setRequestHeader('Content-Type', 'application/json');
        xhr.setRequestHeader('Accept', 'application/json');
        
        xhr.onreadystatechange = function() {
            if (xhr.readyState === XMLHttpRequest.DONE) {
                if (xhr.status >= 200 && xhr.status < 300) {
                    try {
                        const responseData = JSON.parse(xhr.responseText);
                        resolve(responseData);
                    } catch (error) {
                        resolve(xhr.responseText); // Return raw text if not JSON
                    }
                } else {
                    reject(new Error(`Request failed: ${xhr.status} ${xhr.statusText}`));
                }
            }
        };
        
        xhr.onerror = () => reject(new Error('Network error'));
        xhr.ontimeout = () => reject(new Error('Request timeout'));
        
        xhr.timeout = 10000; // 10 seconds
        
        // Send JSON data
        xhr.send(JSON.stringify(data));
    });
}

// Usage
const postData = {
    title: 'New Post',
    body: 'This is the content of the new post',
    userId: 1
};

makePostRequest('https://jsonplaceholder.typicode.com/posts', postData)
    .then(response => console.log('Created:', response))
    .catch(error => console.error('Error:', error.message));

// POST request with FormData
function postFormData(url, formData) {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        
        xhr.open('POST', url, true);
        
        // Don't set Content-Type header for FormData
        // Browser will set it automatically with boundary
        
        xhr.onreadystatechange = function() {
            if (xhr.readyState === XMLHttpRequest.DONE) {
                if (xhr.status >= 200 && xhr.status < 300) {
                    resolve(xhr.responseText);
                } else {
                    reject(new Error(`Upload failed: ${xhr.status}`));
                }
            }
        };
        
        xhr.onerror = () => reject(new Error('Upload error'));
        
        xhr.send(formData);
    });
}

// Usage with file upload
function uploadFile(file, url) {
    const formData = new FormData();
    formData.append('file', file);
    formData.append('description', 'Uploaded via XHR');
    
    return postFormData(url, formData);
}

// Example file upload
const fileInput = document.getElementById('fileInput');
fileInput.addEventListener('change', (event) => {
    const file = event.target.files[0];
    if (file) {
        uploadFile(file, '/api/upload')
            .then(result => console.log('Upload successful:', result))
            .catch(error => console.error('Upload failed:', error));
    }
});
```

### Advanced Request Methods
```javascript
// Complete HTTP client using XMLHttpRequest
class XHRClient {
    constructor(baseURL = '', defaultHeaders = {}) {
        this.baseURL = baseURL;
        this.defaultHeaders = {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
            ...defaultHeaders
        };
        this.interceptors = {
            request: [],
            response: []
        };
    }
    
    // Add request interceptor
    addRequestInterceptor(interceptor) {
        this.interceptors.request.push(interceptor);
    }
    
    // Add response interceptor
    addResponseInterceptor(interceptor) {
        this.interceptors.response.push(interceptor);
    }
    
    // Make HTTP request
    request(method, url, data = null, options = {}) {
        return new Promise((resolve, reject) => {
            const xhr = new XMLHttpRequest();
            const fullURL = this.baseURL + url;
            
            // Apply request interceptors
            let requestConfig = {
                method,
                url: fullURL,
                data,
                headers: { ...this.defaultHeaders, ...options.headers },
                timeout: options.timeout || 30000
            };
            
            this.interceptors.request.forEach(interceptor => {
                requestConfig = interceptor(requestConfig) || requestConfig;
            });
            
            xhr.open(requestConfig.method, requestConfig.url, true);
            
            // Set headers
            Object.keys(requestConfig.headers).forEach(key => {
                xhr.setRequestHeader(key, requestConfig.headers[key]);
            });
            
            // Set timeout
            xhr.timeout = requestConfig.timeout;
            
            // Progress tracking for uploads
            if (xhr.upload && options.onUploadProgress) {
                xhr.upload.addEventListener('progress', (event) => {
                    if (event.lengthComputable) {
                        const percentComplete = (event.loaded / event.total) * 100;
                        options.onUploadProgress(percentComplete, event);
                    }
                });
            }
            
            // Progress tracking for downloads
            if (options.onDownloadProgress) {
                xhr.addEventListener('progress', (event) => {
                    if (event.lengthComputable) {
                        const percentComplete = (event.loaded / event.total) * 100;
                        options.onDownloadProgress(percentComplete, event);
                    }
                });
            }
            
            xhr.onreadystatechange = function() {
                if (xhr.readyState === XMLHttpRequest.DONE) {
                    let response = {
                        data: xhr.responseText,
                        status: xhr.status,
                        statusText: xhr.statusText,
                        headers: xhr.getAllResponseHeaders()
                    };
                    
                    // Try to parse JSON
                    try {
                        response.data = JSON.parse(xhr.responseText);
                    } catch (e) {
                        // Keep as text if not JSON
                    }
                    
                    // Apply response interceptors
                    this.interceptors.response.forEach(interceptor => {
                        response = interceptor(response) || response;
                    });
                    
                    if (xhr.status >= 200 && xhr.status < 300) {
                        resolve(response);
                    } else {
                        reject(new Error(`Request failed: ${xhr.status} ${xhr.statusText}`));
                    }
                }
            }.bind(this);
            
            xhr.onerror = () => reject(new Error('Network error'));
            xhr.ontimeout = () => reject(new Error('Request timeout'));
            
            // Send request
            let requestBody = requestConfig.data;
            
            if (requestBody && typeof requestBody === 'object' && !(requestBody instanceof FormData)) {
                requestBody = JSON.stringify(requestBody);
            }
            
            xhr.send(requestBody);
        });
    }
    
    // Convenience methods
    get(url, options = {}) {
        return this.request('GET', url, null, options);
    }
    
    post(url, data, options = {}) {
        return this.request('POST', url, data, options);
    }
    
    put(url, data, options = {}) {
        return this.request('PUT', url, data, options);
    }
    
    patch(url, data, options = {}) {
        return this.request('PATCH', url, data, options);
    }
    
    delete(url, options = {}) {
        return this.request('DELETE', url, null, options);
    }
    
    // Upload file with progress
    uploadFile(url, file, options = {}) {
        const formData = new FormData();
        formData.append('file', file);
        
        if (options.additionalData) {
            Object.keys(options.additionalData).forEach(key => {
                formData.append(key, options.additionalData[key]);
            });
        }
        
        return this.request('POST', url, formData, {
            headers: {}, // Let browser set Content-Type for FormData
            ...options
        });
    }
}

// Usage
const apiClient = new XHRClient('https://jsonplaceholder.typicode.com');

// Add authentication interceptor
apiClient.addRequestInterceptor((config) => {
    const token = localStorage.getItem('authToken');
    if (token) {
        config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
});

// Add logging interceptor
apiClient.addResponseInterceptor((response) => {
    console.log('Response received:', response.status, response.statusText);
    return response;
});

// Make requests
apiClient.get('/posts')
    .then(response => console.log('Posts:', response.data))
    .catch(error => console.error('Error:', error));

apiClient.post('/posts', {
    title: 'New Post',
    body: 'Content',
    userId: 1
})
    .then(response => console.log('Created post:', response.data))
    .catch(error => console.error('Error:', error));

// File upload with progress
const fileInputEl = document.getElementById('file');
fileInputEl.addEventListener('change', (event) => {
    const file = event.target.files[0];
    if (file) {
        apiClient.uploadFile('/upload', file, {
            additionalData: { description: 'My file' },
            onUploadProgress: (percent) => {
                console.log(`Upload progress: ${percent.toFixed(1)}%`);
            }
        })
        .then(response => console.log('Upload complete:', response.data))
        .catch(error => console.error('Upload error:', error));
    }
});
```

## Error Handling and Status Codes

### Comprehensive Error Handling
```javascript
// Error handling utility for XHR
class XHRError extends Error {
    constructor(message, status, statusText, response) {
        super(message);
        this.name = 'XHRError';
        this.status = status;
        this.statusText = statusText;
        this.response = response;
    }
}

function handleXHRResponse(xhr) {
    const response = {
        data: xhr.responseText,
        status: xhr.status,
        statusText: xhr.statusText,
        headers: parseResponseHeaders(xhr.getAllResponseHeaders())
    };
    
    // Try to parse JSON response
    try {
        response.data = JSON.parse(xhr.responseText);
    } catch (e) {
        // Keep as text if not valid JSON
    }
    
    // Handle different status codes
    switch (true) {
        case xhr.status >= 200 && xhr.status < 300:
            return response; // Success
            
        case xhr.status === 400:
            throw new XHRError('Bad Request - Invalid parameters', xhr.status, xhr.statusText, response);
            
        case xhr.status === 401:
            throw new XHRError('Unauthorized - Authentication required', xhr.status, xhr.statusText, response);
            
        case xhr.status === 403:
            throw new XHRError('Forbidden - Access denied', xhr.status, xhr.statusText, response);
            
        case xhr.status === 404:
            throw new XHRError('Not Found - Resource does not exist', xhr.status, xhr.statusText, response);
            
        case xhr.status === 409:
            throw new XHRError('Conflict - Resource already exists', xhr.status, xhr.statusText, response);
            
        case xhr.status === 422:
            throw new XHRError('Unprocessable Entity - Validation failed', xhr.status, xhr.statusText, response);
            
        case xhr.status >= 400 && xhr.status < 500:
            throw new XHRError(`Client Error: ${xhr.status} ${xhr.statusText}`, xhr.status, xhr.statusText, response);
            
        case xhr.status >= 500:
            throw new XHRError(`Server Error: ${xhr.status} ${xhr.statusText}`, xhr.status, xhr.statusText, response);
            
        default:
            throw new XHRError(`Unexpected status: ${xhr.status} ${xhr.statusText}`, xhr.status, xhr.statusText, response);
    }
}

function parseResponseHeaders(headerString) {
    const headers = {};
    if (!headerString) return headers;
    
    headerString.split('\r\n').forEach(line => {
        const parts = line.split(': ');
        if (parts.length === 2) {
            headers[parts[0].toLowerCase()] = parts[1];
        }
    });
    
    return headers;
}

// Robust request function with retry logic
function robustXHRRequest(url, options = {}) {
    const {
        method = 'GET',
        data = null,
        headers = {},
        timeout = 30000,
        retries = 3,
        retryDelay = 1000
    } = options;
    
    return new Promise((resolve, reject) => {
        let attempt = 0;
        
        function makeRequest() {
            attempt++;
            const xhr = new XMLHttpRequest();
            
            xhr.open(method, url, true);
            xhr.timeout = timeout;
            
            // Set headers
            Object.keys(headers).forEach(key => {
                xhr.setRequestHeader(key, headers[key]);
            });
            
            xhr.onreadystatechange = function() {
                if (xhr.readyState === XMLHttpRequest.DONE) {
                    try {
                        const response = handleXHRResponse(xhr);
                        resolve(response);
                    } catch (error) {
                        if (attempt < retries && shouldRetry(error)) {
                            console.warn(`Request failed (attempt ${attempt}/${retries}), retrying in ${retryDelay}ms:`, error.message);
                            setTimeout(makeRequest, retryDelay * attempt);
                        } else {
                            reject(error);
                        }
                    }
                }
            };
            
            xhr.onerror = function() {
                const error = new Error('Network error occurred');
                if (attempt < retries) {
                    console.warn(`Network error (attempt ${attempt}/${retries}), retrying in ${retryDelay}ms`);
                    setTimeout(makeRequest, retryDelay * attempt);
                } else {
                    reject(error);
                }
            };
            
            xhr.ontimeout = function() {
                const error = new Error('Request timed out');
                if (attempt < retries) {
                    console.warn(`Timeout (attempt ${attempt}/${retries}), retrying in ${retryDelay}ms`);
                    setTimeout(makeRequest, retryDelay * attempt);
                } else {
                    reject(error);
                }
            };
            
            // Send request
            let requestBody = data;
            if (data && typeof data === 'object' && !(data instanceof FormData)) {
                requestBody = JSON.stringify(data);
            }
            
            xhr.send(requestBody);
        }
        
        makeRequest();
    });
}

function shouldRetry(error) {
    // Retry on server errors or network issues, but not client errors
    return error.status >= 500 || error.status === 0 || !error.status;
}

// Usage with error handling
robustXHRRequest('https://jsonplaceholder.typicode.com/posts/1', {
    method: 'GET',
    headers: { 'Accept': 'application/json' },
    retries: 3,
    retryDelay: 1000
})
.then(response => {
    console.log('Success:', response.data);
})
.catch(error => {
    if (error instanceof XHRError) {
        console.error('XHR Error:', error.message);
        console.error('Status:', error.status);
        console.error('Response:', error.response);
    } else {
        console.error('General Error:', error.message);
    }
});
```

## File Upload and Progress Tracking

### File Upload with Progress
```javascript
// File upload utility with progress tracking
class FileUploader {
    constructor(uploadUrl, options = {}) {
        this.uploadUrl = uploadUrl;
        this.options = {
            timeout: 300000, // 5 minutes
            chunkSize: 1024 * 1024, // 1MB chunks for large files
            ...options
        };
    }
    
    // Upload single file
    uploadFile(file, additionalData = {}) {
        return new Promise((resolve, reject) => {
            const xhr = new XMLHttpRequest();
            const formData = new FormData();
            
            // Add file to form data
            formData.append('file', file);
            
            // Add additional data
            Object.keys(additionalData).forEach(key => {
                formData.append(key, additionalData[key]);
            });
            
            xhr.open('POST', this.uploadUrl, true);
            xhr.timeout = this.options.timeout;
            
            // Upload progress
            xhr.upload.addEventListener('progress', (event) => {
                if (event.lengthComputable) {
                    const percentComplete = (event.loaded / event.total) * 100;
                    this.onProgress?.(percentComplete, event.loaded, event.total);
                }
            });
            
            // Upload started
            xhr.upload.addEventListener('loadstart', () => {
                this.onStart?.(file);
            });
            
            // Upload completed
            xhr.upload.addEventListener('load', () => {
                this.onComplete?.(file);
            });
            
            xhr.onreadystatechange = function() {
                if (xhr.readyState === XMLHttpRequest.DONE) {
                    if (xhr.status >= 200 && xhr.status < 300) {
                        try {
                            const response = JSON.parse(xhr.responseText);
                            resolve(response);
                        } catch (e) {
                            resolve(xhr.responseText);
                        }
                    } else {
                        reject(new Error(`Upload failed: ${xhr.status} ${xhr.statusText}`));
                    }
                }
            };
            
            xhr.onerror = () => {
                reject(new Error('Upload failed due to network error'));
            };
            
            xhr.ontimeout = () => {
                reject(new Error('Upload timed out'));
            };
            
            xhr.send(formData);
        });
    }
    
    // Upload multiple files
    uploadFiles(files, additionalData = {}) {
        const uploads = Array.from(files).map(file => {
            return this.uploadFile(file, additionalData);
        });
        
        return Promise.all(uploads);
    }
    
    // Chunked upload for large files
    uploadLargeFile(file, additionalData = {}) {
        const chunkSize = this.options.chunkSize;
        const totalChunks = Math.ceil(file.size / chunkSize);
        
        return new Promise((resolve, reject) => {
            let uploadedChunks = 0;
            const uploadId = Date.now().toString(36) + Math.random().toString(36);
            
            const uploadChunk = (chunkIndex) => {
                const start = chunkIndex * chunkSize;
                const end = Math.min(start + chunkSize, file.size);
                const chunk = file.slice(start, end);
                
                const xhr = new XMLHttpRequest();
                const formData = new FormData();
                
                formData.append('chunk', chunk);
                formData.append('chunkIndex', chunkIndex);
                formData.append('totalChunks', totalChunks);
                formData.append('uploadId', uploadId);
                formData.append('fileName', file.name);
                formData.append('fileSize', file.size);
                
                // Add additional data to first chunk
                if (chunkIndex === 0) {
                    Object.keys(additionalData).forEach(key => {
                        formData.append(key, additionalData[key]);
                    });
                }
                
                xhr.open('POST', this.uploadUrl, true);
                
                xhr.onreadystatechange = function() {
                    if (xhr.readyState === XMLHttpRequest.DONE) {
                        if (xhr.status >= 200 && xhr.status < 300) {
                            uploadedChunks++;
                            
                            // Report progress
                            const progress = (uploadedChunks / totalChunks) * 100;
                            this.onProgress?.(progress, uploadedChunks * chunkSize, file.size);
                            
                            if (uploadedChunks === totalChunks) {
                                // All chunks uploaded
                                try {
                                    const response = JSON.parse(xhr.responseText);
                                    resolve(response);
                                } catch (e) {
                                    resolve(xhr.responseText);
                                }
                            } else {
                                // Upload next chunk
                                uploadChunk(chunkIndex + 1);
                            }
                        } else {
                            reject(new Error(`Chunk upload failed: ${xhr.status}`));
                        }
                    }
                }.bind(this);
                
                xhr.onerror = () => {
                    reject(new Error('Chunk upload network error'));
                };
                
                xhr.send(formData);
            };
            
            // Start uploading first chunk
            uploadChunk(0);
        });
    }
    
    // Set event handlers
    onProgress(callback) {
        this.onProgress = callback;
        return this;
    }
    
    onStart(callback) {
        this.onStart = callback;
        return this;
    }
    
    onComplete(callback) {
        this.onComplete = callback;
        return this;
    }
}

// Usage example
const uploader = new FileUploader('/api/upload')
    .onStart((file) => {
        console.log('Upload started:', file.name);
    })
    .onProgress((percent, loaded, total) => {
        console.log(`Upload progress: ${percent.toFixed(1)}% (${loaded}/${total} bytes)`);
        
        // Update progress bar
        const progressBar = document.getElementById('progressBar');
        if (progressBar) {
            progressBar.style.width = `${percent}%`;
        }
    })
    .onComplete((file) => {
        console.log('Upload completed:', file.name);
    });

// File input handler
const fileInput = document.getElementById('fileUpload');
fileInput.addEventListener('change', (event) => {
    const files = event.target.files;
    
    if (files.length === 1) {
        const file = files[0];
        
        // Use chunked upload for large files (> 10MB)
        if (file.size > 10 * 1024 * 1024) {
            uploader.uploadLargeFile(file, { description: 'Large file upload' })
                .then(response => console.log('Large file uploaded:', response))
                .catch(error => console.error('Large file upload error:', error));
        } else {
            uploader.uploadFile(file, { description: 'Regular file upload' })
                .then(response => console.log('File uploaded:', response))
                .catch(error => console.error('File upload error:', error));
        }
    } else if (files.length > 1) {
        // Multiple file upload
        uploader.uploadFiles(files, { description: 'Multiple file upload' })
            .then(responses => console.log('All files uploaded:', responses))
            .catch(error => console.error('Multiple file upload error:', error));
    }
});
```

## CORS and Authentication

### CORS Handling
```javascript
// CORS-aware request utility
function makeCORSRequest(url, options = {}) {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        
        // Check if CORS is supported
        if (!('withCredentials' in xhr)) {
            reject(new Error('CORS not supported'));
            return;
        }
        
        const {
            method = 'GET',
            data = null,
            headers = {},
            withCredentials = false,
            timeout = 30000
        } = options;
        
        xhr.open(method, url, true);
        
        // Enable credentials if needed (cookies, auth headers)
        xhr.withCredentials = withCredentials;
        
        // Set headers
        Object.keys(headers).forEach(key => {
            xhr.setRequestHeader(key, headers[key]);
        });
        
        xhr.timeout = timeout;
        
        xhr.onreadystatechange = function() {
            if (xhr.readyState === XMLHttpRequest.DONE) {
                if (xhr.status >= 200 && xhr.status < 300) {
                    try {
                        const response = JSON.parse(xhr.responseText);
                        resolve(response);
                    } catch (e) {
                        resolve(xhr.responseText);
                    }
                } else if (xhr.status === 0) {
                    reject(new Error('CORS request blocked or network error'));
                } else {
                    reject(new Error(`Request failed: ${xhr.status} ${xhr.statusText}`));
                }
            }
        };
        
        xhr.onerror = function() {
            reject(new Error('CORS request failed'));
        };
        
        xhr.ontimeout = function() {
            reject(new Error('CORS request timed out'));
        };
        
        // Send request
        let requestBody = data;
        if (data && typeof data === 'object' && !(data instanceof FormData)) {
            requestBody = JSON.stringify(data);
        }
        
        xhr.send(requestBody);
    });
}

// Authentication helper
class AuthenticatedXHR {
    constructor(baseURL = '') {
        this.baseURL = baseURL;
        this.token = null;
        this.refreshToken = null;
    }
    
    // Set authentication token
    setToken(token) {
        this.token = token;
    }
    
    // Set refresh token
    setRefreshToken(refreshToken) {
        this.refreshToken = refreshToken;
    }
    
    // Make authenticated request
    request(url, options = {}) {
        const headers = {
            'Content-Type': 'application/json',
            ...options.headers
        };
        
        // Add auth header if token is available
        if (this.token) {
            headers.Authorization = `Bearer ${this.token}`;
        }
        
        return makeCORSRequest(this.baseURL + url, {
            ...options,
            headers
        }).catch(error => {
            // If 401 error and refresh token available, try to refresh
            if (error.message.includes('401') && this.refreshToken) {
                return this.refreshTokenAndRetry(url, options);
            }
            throw error;
        });
    }
    
    // Refresh token and retry request
    refreshTokenAndRetry(url, options) {
        return this.refreshAuthToken()
            .then(() => {
                // Retry original request with new token
                const headers = {
                    'Content-Type': 'application/json',
                    ...options.headers,
                    Authorization: `Bearer ${this.token}`
                };
                
                return makeCORSRequest(this.baseURL + url, {
                    ...options,
                    headers
                });
            });
    }
    
    // Refresh authentication token
    refreshAuthToken() {
        return makeCORSRequest(this.baseURL + '/auth/refresh', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                Authorization: `Bearer ${this.refreshToken}`
            }
        })
        .then(response => {
            this.token = response.access_token;
            if (response.refresh_token) {
                this.refreshToken = response.refresh_token;
            }
            return response;
        });
    }
    
    // Login
    login(credentials) {
        return makeCORSRequest(this.baseURL + '/auth/login', {
            method: 'POST',
            data: credentials
        })
        .then(response => {
            this.token = response.access_token;
            this.refreshToken = response.refresh_token;
            return response;
        });
    }
    
    // Logout
    logout() {
        return makeCORSRequest(this.baseURL + '/auth/logout', {
            method: 'POST',
            headers: {
                Authorization: `Bearer ${this.token}`
            }
        })
        .then(response => {
            this.token = null;
            this.refreshToken = null;
            return response;
        });
    }
    
    // Convenience methods
    get(url, options = {}) {
        return this.request(url, { ...options, method: 'GET' });
    }
    
    post(url, data, options = {}) {
        return this.request(url, { ...options, method: 'POST', data });
    }
    
    put(url, data, options = {}) {
        return this.request(url, { ...options, method: 'PUT', data });
    }
    
    delete(url, options = {}) {
        return this.request(url, { ...options, method: 'DELETE' });
    }
}

// Usage
const authClient = new AuthenticatedXHR('https://api.example.com');

// Login
authClient.login({ username: 'user', password: 'pass' })
    .then(response => {
        console.log('Logged in:', response);
        
        // Make authenticated requests
        return authClient.get('/user/profile');
    })
    .then(profile => {
        console.log('User profile:', profile);
    })
    .catch(error => {
        console.error('Error:', error);
    });
```

---

## Related Topics
- [[22 - Fetch API]]
- [[23 - Axios]]
- [[18 - Callbacks]]
- [[19 - Promises]]
- [[42 - Error Handling]]

---

*Next: [[23 - Axios]]*
