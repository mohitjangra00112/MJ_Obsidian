# Callbacks

## Overview
Callbacks are functions that are passed as arguments to other functions and are executed at a specific point in the future. They are fundamental to JavaScript's asynchronous nature and enable event-driven programming.

## Basic Callback Concepts

### Simple Callback Examples
```javascript
// Basic callback function
function greetUser(name, callback) {
    console.log(`Hello, ${name}!`);
    callback();
}

function afterGreeting() {
    console.log("Nice to meet you!");
}

// Using the callback
greetUser("John", afterGreeting);
// Output: "Hello, John!"
// Output: "Nice to meet you!"

// Inline callback function
greetUser("Jane", function() {
    console.log("How are you doing?");
});

// Arrow function callback
greetUser("Bob", () => {
    console.log("Have a great day!");
});

// Callback with parameters
function processData(data, successCallback, errorCallback) {
    if (data && data.length > 0) {
        successCallback(data);
    } else {
        errorCallback("No data provided");
    }
}

processData(
    [1, 2, 3],
    function(data) {
        console.log("Success! Data:", data);
    },
    function(error) {
        console.log("Error:", error);
    }
);
```

### Higher-Order Functions with Callbacks
```javascript
// Array methods are perfect examples of callbacks
const numbers = [1, 2, 3, 4, 5];

// map() with callback
const doubled = numbers.map(function(num) {
    return num * 2;
});
console.log(doubled); // [2, 4, 6, 8, 10]

// filter() with callback
const evens = numbers.filter(function(num) {
    return num % 2 === 0;
});
console.log(evens); // [2, 4]

// reduce() with callback
const sum = numbers.reduce(function(accumulator, current) {
    return accumulator + current;
}, 0);
console.log(sum); // 15

// forEach() with callback
numbers.forEach(function(num, index) {
    console.log(`Index ${index}: ${num}`);
});

// Custom higher-order function
function repeat(times, callback) {
    for (let i = 0; i < times; i++) {
        callback(i);
    }
}

repeat(3, function(index) {
    console.log(`Iteration ${index}`);
});
// Output: Iteration 0, Iteration 1, Iteration 2

// Function that returns a function (closure with callback)
function createMultiplier(factor) {
    return function(number, callback) {
        const result = number * factor;
        callback(result);
    };
}

const double = createMultiplier(2);
double(5, function(result) {
    console.log("Result:", result); // Result: 10
});
```

## Asynchronous Callbacks

### setTimeout and setInterval
```javascript
// setTimeout with callback
console.log("Before timeout");

setTimeout(function() {
    console.log("This runs after 2 seconds");
}, 2000);

console.log("After timeout call");

// setInterval with callback
let counter = 0;
const intervalId = setInterval(function() {
    counter++;
    console.log(`Counter: ${counter}`);
    
    if (counter >= 5) {
        clearInterval(intervalId);
        console.log("Interval cleared");
    }
}, 1000);

// Callback with parameters in setTimeout
function delayedGreeting(name, message, delay) {
    setTimeout(function() {
        console.log(`${message}, ${name}!`);
    }, delay);
}

delayedGreeting("Alice", "Good morning", 1000);

// Multiple nested timeouts
function sequence() {
    console.log("Step 1");
    
    setTimeout(function() {
        console.log("Step 2");
        
        setTimeout(function() {
            console.log("Step 3");
            
            setTimeout(function() {
                console.log("Step 4 - Done!");
            }, 1000);
        }, 1000);
    }, 1000);
}

sequence();
```

### Event Callbacks
```javascript
// DOM event callbacks
const button = document.getElementById('myButton');

// Event listener with callback
button.addEventListener('click', function(event) {
    console.log('Button clicked!');
    console.log('Event type:', event.type);
    console.log('Target:', event.target);
});

// Multiple event callbacks
function handleMouseOver(event) {
    console.log('Mouse over!');
    event.target.style.backgroundColor = '#f0f0f0';
}

function handleMouseOut(event) {
    console.log('Mouse out!');
    event.target.style.backgroundColor = '';
}

button.addEventListener('mouseover', handleMouseOver);
button.addEventListener('mouseout', handleMouseOut);

// Custom event system with callbacks
class EventEmitter {
    constructor() {
        this.events = {};
    }
    
    on(eventName, callback) {
        if (!this.events[eventName]) {
            this.events[eventName] = [];
        }
        this.events[eventName].push(callback);
    }
    
    emit(eventName, data) {
        if (this.events[eventName]) {
            this.events[eventName].forEach(callback => {
                callback(data);
            });
        }
    }
    
    off(eventName, callback) {
        if (this.events[eventName]) {
            this.events[eventName] = this.events[eventName].filter(cb => cb !== callback);
        }
    }
}

// Usage
const emitter = new EventEmitter();

function userLoginHandler(userData) {
    console.log('User logged in:', userData.name);
}

function analyticsHandler(userData) {
    console.log('Track login event for:', userData.id);
}

emitter.on('userLogin', userLoginHandler);
emitter.on('userLogin', analyticsHandler);

emitter.emit('userLogin', { id: 123, name: 'John Doe' });
```

## Callback Hell and Solutions

### Understanding Callback Hell
```javascript
// Example of callback hell (pyramid of doom)
function fetchUserData(userId, callback) {
    setTimeout(() => {
        console.log('Fetching user data...');
        callback(null, { id: userId, name: 'John Doe' });
    }, 1000);
}

function fetchUserPosts(userId, callback) {
    setTimeout(() => {
        console.log('Fetching user posts...');
        callback(null, [{ id: 1, title: 'First Post' }, { id: 2, title: 'Second Post' }]);
    }, 800);
}

function fetchPostComments(postId, callback) {
    setTimeout(() => {
        console.log('Fetching post comments...');
        callback(null, [{ id: 1, text: 'Great post!' }, { id: 2, text: 'Thanks for sharing!' }]);
    }, 600);
}

// Callback hell example
fetchUserData(123, function(error, user) {
    if (error) {
        console.error('Error fetching user:', error);
        return;
    }
    
    console.log('User:', user);
    
    fetchUserPosts(user.id, function(error, posts) {
        if (error) {
            console.error('Error fetching posts:', error);
            return;
        }
        
        console.log('Posts:', posts);
        
        if (posts.length > 0) {
            fetchPostComments(posts[0].id, function(error, comments) {
                if (error) {
                    console.error('Error fetching comments:', error);
                    return;
                }
                
                console.log('Comments:', comments);
                // More nesting could continue...
            });
        }
    });
});
```

### Solutions to Callback Hell

#### 1. Named Functions
```javascript
// Break down into named functions
function handleUserData(error, user) {
    if (error) {
        console.error('Error fetching user:', error);
        return;
    }
    
    console.log('User:', user);
    fetchUserPosts(user.id, handleUserPosts);
}

function handleUserPosts(error, posts) {
    if (error) {
        console.error('Error fetching posts:', error);
        return;
    }
    
    console.log('Posts:', posts);
    
    if (posts.length > 0) {
        fetchPostComments(posts[0].id, handlePostComments);
    }
}

function handlePostComments(error, comments) {
    if (error) {
        console.error('Error fetching comments:', error);
        return;
    }
    
    console.log('Comments:', comments);
}

// Much cleaner call
fetchUserData(123, handleUserData);
```

#### 2. Modular Functions
```javascript
// Create reusable modules
const UserService = {
    fetchUser(userId, callback) {
        setTimeout(() => {
            callback(null, { id: userId, name: 'John Doe' });
        }, 1000);
    },
    
    fetchPosts(userId, callback) {
        setTimeout(() => {
            callback(null, [{ id: 1, title: 'First Post' }]);
        }, 800);
    },
    
    fetchComments(postId, callback) {
        setTimeout(() => {
            callback(null, [{ id: 1, text: 'Great post!' }]);
        }, 600);
    }
};

// Error handling utility
function handleError(error, context) {
    if (error) {
        console.error(`Error in ${context}:`, error);
        return true;
    }
    return false;
}

// Improved flow
function loadUserContent(userId) {
    UserService.fetchUser(userId, (error, user) => {
        if (handleError(error, 'fetchUser')) return;
        
        console.log('User loaded:', user);
        
        UserService.fetchPosts(user.id, (error, posts) => {
            if (handleError(error, 'fetchPosts')) return;
            
            console.log('Posts loaded:', posts);
            
            if (posts.length > 0) {
                UserService.fetchComments(posts[0].id, (error, comments) => {
                    if (handleError(error, 'fetchComments')) return;
                    
                    console.log('Comments loaded:', comments);
                    displayUserContent(user, posts, comments);
                });
            }
        });
    });
}

function displayUserContent(user, posts, comments) {
    console.log('All data loaded successfully!');
    console.log({ user, posts, comments });
}

loadUserContent(123);
```

#### 3. Control Flow Libraries Pattern
```javascript
// Simplified control flow for callbacks
function series(tasks, finalCallback) {
    let currentIndex = 0;
    const results = [];
    
    function next(error, result) {
        if (error) {
            return finalCallback(error);
        }
        
        if (result !== undefined) {
            results.push(result);
        }
        
        if (currentIndex >= tasks.length) {
            return finalCallback(null, results);
        }
        
        const currentTask = tasks[currentIndex++];
        currentTask(next);
    }
    
    next();
}

// Usage with series control flow
function loadUserDataSeries(userId) {
    const tasks = [
        (callback) => {
            fetchUserData(userId, callback);
        },
        (callback) => {
            // Using userId from closure
            fetchUserPosts(userId, callback);
        },
        (callback) => {
            // This is simplified - in reality you'd need the postId from previous result
            fetchPostComments(1, callback);
        }
    ];
    
    series(tasks, (error, results) => {
        if (error) {
            console.error('Error in series:', error);
            return;
        }
        
        const [user, posts, comments] = results;
        console.log('All data loaded:', { user, posts, comments });
    });
}

// Parallel execution
function parallel(tasks, finalCallback) {
    let completed = 0;
    const results = [];
    let hasError = false;
    
    if (tasks.length === 0) {
        return finalCallback(null, results);
    }
    
    tasks.forEach((task, index) => {
        task((error, result) => {
            if (hasError) return;
            
            if (error) {
                hasError = true;
                return finalCallback(error);
            }
            
            results[index] = result;
            completed++;
            
            if (completed === tasks.length) {
                finalCallback(null, results);
            }
        });
    });
}

// Usage with parallel control flow
function loadMultipleUsers() {
    const tasks = [
        (callback) => fetchUserData(1, callback),
        (callback) => fetchUserData(2, callback),
        (callback) => fetchUserData(3, callback)
    ];
    
    parallel(tasks, (error, users) => {
        if (error) {
            console.error('Error loading users:', error);
            return;
        }
        
        console.log('All users loaded:', users);
    });
}

loadMultipleUsers();
```

## Practical Callback Patterns

### Retry Pattern
```javascript
function retry(fn, maxAttempts, delay, callback) {
    let attempts = 0;
    
    function attempt() {
        attempts++;
        
        fn((error, result) => {
            if (!error) {
                return callback(null, result);
            }
            
            if (attempts >= maxAttempts) {
                return callback(new Error(`Failed after ${maxAttempts} attempts: ${error.message}`));
            }
            
            console.log(`Attempt ${attempts} failed, retrying in ${delay}ms...`);
            setTimeout(attempt, delay);
        });
    }
    
    attempt();
}

// Usage
function unreliableAPI(callback) {
    // Simulate 70% failure rate
    if (Math.random() < 0.7) {
        setTimeout(() => callback(new Error('API Error')), 100);
    } else {
        setTimeout(() => callback(null, 'Success!'), 100);
    }
}

retry(unreliableAPI, 3, 1000, (error, result) => {
    if (error) {
        console.error('Final error:', error.message);
    } else {
        console.log('Success:', result);
    }
});
```

### Throttle and Debounce with Callbacks
```javascript
// Throttle function for callbacks
function throttle(func, limit) {
    let inThrottle;
    return function(...args) {
        if (!inThrottle) {
            func.apply(this, args);
            inThrottle = true;
            setTimeout(() => inThrottle = false, limit);
        }
    };
}

// Debounce function for callbacks
function debounce(func, delay) {
    let timeoutId;
    return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => func.apply(this, args), delay);
    };
}

// Usage examples
const searchInput = document.getElementById('search');

const throttledSearch = throttle(function(event) {
    console.log('Throttled search:', event.target.value);
}, 300);

const debouncedSearch = debounce(function(event) {
    console.log('Debounced search:', event.target.value);
    // Perform actual search
    performSearch(event.target.value);
}, 500);

searchInput.addEventListener('input', throttledSearch);
searchInput.addEventListener('input', debouncedSearch);

function performSearch(query) {
    console.log('Searching for:', query);
}
```

### Memoization with Callbacks
```javascript
function memoize(fn) {
    const cache = new Map();
    
    return function(...args) {
        const key = JSON.stringify(args);
        
        if (cache.has(key)) {
            console.log('Cache hit!');
            const callback = args[args.length - 1];
            const cachedResult = cache.get(key);
            
            // Simulate async callback
            setTimeout(() => {
                callback(null, cachedResult);
            }, 0);
            return;
        }
        
        // Store original callback
        const originalCallback = args[args.length - 1];
        
        // Replace with caching callback
        args[args.length - 1] = function(error, result) {
            if (!error) {
                cache.set(key, result);
            }
            originalCallback(error, result);
        };
        
        fn.apply(this, args);
    };
}

// Expensive function to memoize
function expensiveCalculation(x, y, callback) {
    console.log('Performing expensive calculation...');
    setTimeout(() => {
        const result = x * y + Math.random();
        callback(null, result);
    }, 1000);
}

const memoizedCalculation = memoize(expensiveCalculation);

// First call - will be cached
memoizedCalculation(5, 10, (error, result) => {
    console.log('Result 1:', result);
});

// Second call with same parameters - will use cache
setTimeout(() => {
    memoizedCalculation(5, 10, (error, result) => {
        console.log('Result 2 (cached):', result);
    });
}, 2000);
```

### Callback Queue System
```javascript
class CallbackQueue {
    constructor() {
        this.queue = [];
        this.running = false;
    }
    
    add(fn, ...args) {
        return new Promise((resolve, reject) => {
            this.queue.push({
                fn,
                args,
                resolve,
                reject
            });
            
            if (!this.running) {
                this.process();
            }
        });
    }
    
    async process() {
        this.running = true;
        
        while (this.queue.length > 0) {
            const { fn, args, resolve, reject } = this.queue.shift();
            
            try {
                // Convert callback-style to promise
                await new Promise((cbResolve, cbReject) => {
                    fn(...args, (error, result) => {
                        if (error) {
                            cbReject(error);
                        } else {
                            cbResolve(result);
                        }
                    });
                }).then(resolve).catch(reject);
                
                // Small delay between operations
                await new Promise(resolve => setTimeout(resolve, 100));
                
            } catch (error) {
                reject(error);
            }
        }
        
        this.running = false;
    }
}

// Usage
const queue = new CallbackQueue();

function asyncOperation(data, callback) {
    setTimeout(() => {
        if (data) {
            callback(null, `Processed: ${data}`);
        } else {
            callback(new Error('No data provided'));
        }
    }, Math.random() * 1000);
}

// Add operations to queue
queue.add(asyncOperation, 'Task 1')
    .then(result => console.log(result))
    .catch(error => console.error(error));

queue.add(asyncOperation, 'Task 2')
    .then(result => console.log(result))
    .catch(error => console.error(error));

queue.add(asyncOperation, 'Task 3')
    .then(result => console.log(result))
    .catch(error => console.error(error));
```

## Error Handling in Callbacks

### Standard Error-First Callback Pattern
```javascript
// Node.js style error-first callbacks
function readFile(filename, callback) {
    setTimeout(() => {
        if (filename === 'valid.txt') {
            callback(null, 'File contents here');
        } else {
            callback(new Error('File not found'));
        }
    }, 500);
}

// Proper error handling
readFile('valid.txt', (error, data) => {
    if (error) {
        console.error('Error reading file:', error.message);
        return;
    }
    
    console.log('File data:', data);
});

// Error propagation in nested callbacks
function processFile(filename, callback) {
    readFile(filename, (error, data) => {
        if (error) {
            return callback(error);
        }
        
        // Process the data
        try {
            const processed = data.toUpperCase();
            callback(null, processed);
        } catch (processingError) {
            callback(processingError);
        }
    });
}

processFile('valid.txt', (error, result) => {
    if (error) {
        console.error('Processing failed:', error.message);
    } else {
        console.log('Processed result:', result);
    }
});
```

### Callback Error Handling Utilities
```javascript
// Safe callback wrapper
function safeCallback(callback) {
    return function(error, result) {
        try {
            callback(error, result);
        } catch (callbackError) {
            console.error('Callback execution error:', callbackError);
        }
    };
}

// Timeout wrapper for callbacks
function withTimeout(fn, timeout, callback) {
    let completed = false;
    
    const timeoutId = setTimeout(() => {
        if (!completed) {
            completed = true;
            callback(new Error(`Operation timed out after ${timeout}ms`));
        }
    }, timeout);
    
    fn(safeCallback((error, result) => {
        if (!completed) {
            completed = true;
            clearTimeout(timeoutId);
            callback(error, result);
        }
    }));
}

// Usage
function slowOperation(callback) {
    setTimeout(() => {
        callback(null, 'Operation completed');
    }, 3000);
}

withTimeout(slowOperation, 2000, (error, result) => {
    if (error) {
        console.error('Error:', error.message); // "Operation timed out after 2000ms"
    } else {
        console.log('Result:', result);
    }
});
```

---

## Related Topics
- [[19 - Promises]]
- [[20 - Async Await]]
- [[02 - Functions]]
- [[13 - Higher Order Functions]]
- [[42 - Error Handling]]

---

*Next: [[19 - Promises]]*
