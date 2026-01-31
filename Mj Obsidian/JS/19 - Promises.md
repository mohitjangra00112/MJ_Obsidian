# Promises

## Overview
Promises represent the eventual completion (or failure) of an asynchronous operation. They provide a cleaner alternative to callbacks for handling asynchronous code and help avoid "callback hell."

## Promise States
A Promise can be in one of three states:
- **Pending**: Initial state, neither fulfilled nor rejected
- **Fulfilled**: Operation completed successfully
- **Rejected**: Operation failed

```javascript
// Creating a simple promise
const myPromise = new Promise((resolve, reject) => {
    const success = true;
    
    setTimeout(() => {
        if (success) {
            resolve("Operation successful!"); // Fulfill the promise
        } else {
            reject("Operation failed!"); // Reject the promise
        }
    }, 1000);
});

console.log(myPromise); // Promise { <pending> }
```

## Creating Promises

### Basic Promise Creation
```javascript
// Promise constructor takes an executor function
const promise = new Promise((resolve, reject) => {
    // Asynchronous operation
    const randomNumber = Math.random();
    
    if (randomNumber > 0.5) {
        resolve(randomNumber); // Success
    } else {
        reject(new Error("Number too small")); // Failure
    }
});
```

### Immediate Resolution
```javascript
// Immediately resolved promise
const resolvedPromise = Promise.resolve("Success!");

// Immediately rejected promise
const rejectedPromise = Promise.reject("Error occurred");

// Converting value to promise
const valuePromise = Promise.resolve(42);
```

### Promisifying Callbacks
```javascript
// Converting callback-based function to promise
function readFilePromise(filename) {
    return new Promise((resolve, reject) => {
        fs.readFile(filename, 'utf8', (err, data) => {
            if (err) {
                reject(err);
            } else {
                resolve(data);
            }
        });
    });
}

// Using util.promisify (Node.js)
const util = require('util');
const fs = require('fs');
const readFileAsync = util.promisify(fs.readFile);
```

## Promise Methods

### .then()
```javascript
const promise = new Promise((resolve) => {
    setTimeout(() => resolve("Hello"), 1000);
});

promise
    .then(result => {
        console.log(result); // "Hello"
        return result + " World";
    })
    .then(result => {
        console.log(result); // "Hello World"
        return result + "!";
    })
    .then(result => {
        console.log(result); // "Hello World!"
    });
```

### .catch()
```javascript
const promise = new Promise((resolve, reject) => {
    setTimeout(() => reject(new Error("Something went wrong")), 1000);
});

promise
    .then(result => {
        console.log(result);
    })
    .catch(error => {
        console.error("Error:", error.message); // "Error: Something went wrong"
    });
```

### .finally()
```javascript
const promise = new Promise((resolve, reject) => {
    const success = Math.random() > 0.5;
    setTimeout(() => {
        if (success) {
            resolve("Success!");
        } else {
            reject(new Error("Failed!"));
        }
    }, 1000);
});

promise
    .then(result => console.log("Success:", result))
    .catch(error => console.error("Error:", error.message))
    .finally(() => console.log("Cleanup code here")); // Always runs
```

## Promise Chaining

### Basic Chaining
```javascript
function fetchUser(id) {
    return Promise.resolve({ id, name: `User${id}` });
}

function fetchUserPosts(userId) {
    return Promise.resolve([
        { id: 1, title: "Post 1", userId },
        { id: 2, title: "Post 2", userId }
    ]);
}

function fetchPostComments(postId) {
    return Promise.resolve([
        { id: 1, text: "Great post!", postId },
        { id: 2, text: "Thanks for sharing", postId }
    ]);
}

// Chaining promises
fetchUser(1)
    .then(user => {
        console.log("User:", user);
        return fetchUserPosts(user.id);
    })
    .then(posts => {
        console.log("Posts:", posts);
        return fetchPostComments(posts[0].id);
    })
    .then(comments => {
        console.log("Comments:", comments);
    })
    .catch(error => {
        console.error("Error in chain:", error);
    });
```

### Returning Promises in .then()
```javascript
function asyncOperation1() {
    return new Promise(resolve => {
        setTimeout(() => resolve("Result 1"), 1000);
    });
}

function asyncOperation2(data) {
    return new Promise(resolve => {
        setTimeout(() => resolve(data + " -> Result 2"), 1000);
    });
}

asyncOperation1()
    .then(result1 => {
        console.log(result1); // "Result 1"
        // Returning a promise - next .then() waits for it
        return asyncOperation2(result1);
    })
    .then(result2 => {
        console.log(result2); // "Result 1 -> Result 2"
    });
```

## Promise Static Methods

### Promise.all()
```javascript
const promise1 = Promise.resolve(1);
const promise2 = new Promise(resolve => setTimeout(() => resolve(2), 1000));
const promise3 = Promise.resolve(3);

// Wait for all promises to resolve
Promise.all([promise1, promise2, promise3])
    .then(results => {
        console.log(results); // [1, 2, 3]
    })
    .catch(error => {
        // If any promise rejects, this catch will run
        console.error("One or more promises failed:", error);
    });

// Practical example: parallel API calls
async function fetchAllData() {
    try {
        const [users, posts, comments] = await Promise.all([
            fetch('/api/users').then(r => r.json()),
            fetch('/api/posts').then(r => r.json()),
            fetch('/api/comments').then(r => r.json())
        ]);
        
        return { users, posts, comments };
    } catch (error) {
        throw new Error(`Failed to fetch data: ${error.message}`);
    }
}
```

### Promise.allSettled()
```javascript
const promises = [
    Promise.resolve("Success 1"),
    Promise.reject("Error 1"),
    Promise.resolve("Success 2"),
    Promise.reject("Error 2")
];

// Wait for all promises to settle (resolve or reject)
Promise.allSettled(promises)
    .then(results => {
        results.forEach((result, index) => {
            if (result.status === 'fulfilled') {
                console.log(`Promise ${index}: ${result.value}`);
            } else {
                console.log(`Promise ${index}: ${result.reason}`);
            }
        });
        
        // Results:
        // Promise 0: Success 1
        // Promise 1: Error 1
        // Promise 2: Success 2
        // Promise 3: Error 2
    });
```

### Promise.race()
```javascript
const slowPromise = new Promise(resolve => 
    setTimeout(() => resolve("Slow"), 2000)
);

const fastPromise = new Promise(resolve => 
    setTimeout(() => resolve("Fast"), 1000)
);

// First promise to settle wins
Promise.race([slowPromise, fastPromise])
    .then(result => {
        console.log(result); // "Fast"
    });

// Timeout implementation
function withTimeout(promise, timeoutMs) {
    const timeoutPromise = new Promise((_, reject) => {
        setTimeout(() => reject(new Error("Timeout")), timeoutMs);
    });
    
    return Promise.race([promise, timeoutPromise]);
}

// Usage
withTimeout(fetch('/api/data'), 5000)
    .then(response => response.json())
    .catch(error => {
        if (error.message === "Timeout") {
            console.log("Request timed out");
        } else {
            console.log("Request failed:", error);
        }
    });
```

### Promise.any() (ES2021)
```javascript
const promises = [
    Promise.reject("Error 1"),
    Promise.reject("Error 2"),
    Promise.resolve("Success!"),
    Promise.resolve("Another success")
];

// First promise to fulfill (not just settle)
Promise.any(promises)
    .then(result => {
        console.log(result); // "Success!"
    })
    .catch(error => {
        // AggregateError if all promises reject
        console.log("All promises rejected:", error.errors);
    });
```

## Error Handling

### Basic Error Handling
```javascript
function riskyOperation() {
    return new Promise((resolve, reject) => {
        const random = Math.random();
        
        setTimeout(() => {
            if (random > 0.5) {
                resolve("Success!");
            } else {
                reject(new Error("Random failure"));
            }
        }, 1000);
    });
}

riskyOperation()
    .then(result => {
        console.log("Result:", result);
    })
    .catch(error => {
        console.error("Caught error:", error.message);
    });
```

### Error Recovery
```javascript
function unreliableService() {
    return Promise.reject(new Error("Service unavailable"));
}

function fallbackService() {
    return Promise.resolve("Fallback data");
}

unreliableService()
    .catch(error => {
        console.log("Primary service failed, using fallback");
        return fallbackService(); // Recovery
    })
    .then(result => {
        console.log("Final result:", result); // "Fallback data"
    });
```

### Error Propagation
```javascript
Promise.resolve("start")
    .then(value => {
        console.log("1:", value);
        throw new Error("Something went wrong");
    })
    .then(value => {
        // This won't run because previous .then() threw
        console.log("2:", value);
        return value + " -> step 2";
    })
    .catch(error => {
        console.log("Caught:", error.message);
        return "recovered"; // Recover from error
    })
    .then(value => {
        // This runs because .catch() returned a value
        console.log("3:", value); // "recovered"
    });
```

## Real-World Examples

### API Calls with Error Handling
```javascript
class APIClient {
    constructor(baseURL) {
        this.baseURL = baseURL;
    }
    
    request(endpoint, options = {}) {
        const url = `${this.baseURL}${endpoint}`;
        
        return fetch(url, {
            headers: {
                'Content-Type': 'application/json',
                ...options.headers
            },
            ...options
        })
        .then(response => {
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}: ${response.statusText}`);
            }
            return response.json();
        })
        .catch(error => {
            console.error(`API request failed: ${error.message}`);
            throw error;
        });
    }
    
    getUser(id) {
        return this.request(`/users/${id}`)
            .then(user => {
                console.log("User fetched:", user);
                return user;
            });
    }
    
    createUser(userData) {
        return this.request('/users', {
            method: 'POST',
            body: JSON.stringify(userData)
        })
        .then(user => {
            console.log("User created:", user);
            return user;
        });
    }
}

const api = new APIClient('https://api.example.com');

api.getUser(1)
    .then(user => {
        return api.createUser({
            name: "New User",
            email: "newuser@example.com"
        });
    })
    .catch(error => {
        console.error("Operation failed:", error.message);
    });
```

### Sequential vs Parallel Execution
```javascript
// Sequential execution (one after another)
function sequential() {
    console.time('Sequential');
    
    return fetch('/api/data1')
        .then(r => r.json())
        .then(data1 => {
            console.log('Data 1:', data1);
            return fetch('/api/data2');
        })
        .then(r => r.json())
        .then(data2 => {
            console.log('Data 2:', data2);
            return fetch('/api/data3');
        })
        .then(r => r.json())
        .then(data3 => {
            console.log('Data 3:', data3);
            console.timeEnd('Sequential');
            return [data1, data2, data3]; // Problem: data1, data2 not in scope
        });
}

// Parallel execution (all at once)
function parallel() {
    console.time('Parallel');
    
    const promises = [
        fetch('/api/data1').then(r => r.json()),
        fetch('/api/data2').then(r => r.json()),
        fetch('/api/data3').then(r => r.json())
    ];
    
    return Promise.all(promises)
        .then(([data1, data2, data3]) => {
            console.log('All data:', { data1, data2, data3 });
            console.timeEnd('Parallel');
            return { data1, data2, data3 };
        });
}

// Mixed approach: some sequential, some parallel
function mixed() {
    // First, get user info
    return fetch('/api/user/1')
        .then(r => r.json())
        .then(user => {
            // Then get user's posts and comments in parallel
            return Promise.all([
                Promise.resolve(user), // Pass user data through
                fetch(`/api/users/${user.id}/posts`).then(r => r.json()),
                fetch(`/api/users/${user.id}/comments`).then(r => r.json())
            ]);
        })
        .then(([user, posts, comments]) => {
            return { user, posts, comments };
        });
}
```

### Retry Logic
```javascript
function retry(promiseFactory, maxRetries = 3, delay = 1000) {
    return new Promise((resolve, reject) => {
        let attempts = 0;
        
        function attempt() {
            attempts++;
            
            promiseFactory()
                .then(resolve)
                .catch(error => {
                    if (attempts >= maxRetries) {
                        reject(new Error(`Failed after ${maxRetries} attempts: ${error.message}`));
                    } else {
                        console.log(`Attempt ${attempts} failed, retrying in ${delay}ms...`);
                        setTimeout(attempt, delay);
                    }
                });
        }
        
        attempt();
    });
}

// Usage
function unreliableAPI() {
    return fetch('/api/unreliable')
        .then(response => {
            if (!response.ok) {
                throw new Error(`HTTP ${response.status}`);
            }
            return response.json();
        });
}

retry(() => unreliableAPI(), 3, 1000)
    .then(data => console.log("Success:", data))
    .catch(error => console.error("Final failure:", error.message));
```

### Progress Tracking
```javascript
function trackProgress(promises) {
    let completed = 0;
    const total = promises.length;
    
    const trackablePromises = promises.map((promise, index) => {
        return promise
            .then(result => {
                completed++;
                console.log(`Progress: ${completed}/${total} (${Math.round(completed/total*100)}%)`);
                return { index, result, status: 'fulfilled' };
            })
            .catch(error => {
                completed++;
                console.log(`Progress: ${completed}/${total} (${Math.round(completed/total*100)}%)`);
                return { index, error, status: 'rejected' };
            });
    });
    
    return Promise.all(trackablePromises);
}

// Usage
const tasks = [
    new Promise(resolve => setTimeout(() => resolve("Task 1"), 1000)),
    new Promise(resolve => setTimeout(() => resolve("Task 2"), 2000)),
    new Promise((_, reject) => setTimeout(() => reject("Task 3 failed"), 1500)),
    new Promise(resolve => setTimeout(() => resolve("Task 4"), 3000))
];

trackProgress(tasks)
    .then(results => {
        console.log("All tasks completed:", results);
    });
```

## Best Practices

### 1. Always Handle Errors
```javascript
// Bad - unhandled promise rejection
fetch('/api/data')
    .then(response => response.json())
    .then(data => console.log(data));

// Good - error handling
fetch('/api/data')
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error('Error:', error));
```

### 2. Avoid Promise Constructor Antipattern
```javascript
// Bad - unnecessary promise wrapping
function badAsync() {
    return new Promise((resolve, reject) => {
        fetch('/api/data')
            .then(response => response.json())
            .then(data => resolve(data))
            .catch(error => reject(error));
    });
}

// Good - return the promise directly
function goodAsync() {
    return fetch('/api/data')
        .then(response => response.json());
}
```

### 3. Don't Forget to Return
```javascript
// Bad - broken chain
somePromise
    .then(data => {
        anotherPromise(data); // Missing return!
    })
    .then(result => {
        console.log(result); // undefined
    });

// Good - proper chaining
somePromise
    .then(data => {
        return anotherPromise(data);
    })
    .then(result => {
        console.log(result); // Correct result
    });
```

---

## Related Topics
- [[18 - Callbacks]]
- [[20 - Async Await]]
- [[21 - XMLHttpRequest]]
- [[22 - Fetch API]]
- [[42 - Error Handling]]

---

*Next: [[20 - Async Await]]*
