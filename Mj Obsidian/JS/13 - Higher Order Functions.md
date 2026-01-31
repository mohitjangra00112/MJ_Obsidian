# Higher Order Functions

## Overview
Higher Order Functions are functions that either take one or more functions as arguments, return a function, or both. They are a fundamental concept in functional programming and are extensively used in JavaScript for array manipulation, event handling, and creating reusable, composable code.

## Understanding Higher Order Functions

### Functions as First-Class Citizens
```javascript
// Functions are first-class citizens in JavaScript
// They can be stored in variables
const greet = function(name) {
    return `Hello, ${name}!`;
};

// They can be stored in arrays
const functions = [
    function add(a, b) { return a + b; },
    function subtract(a, b) { return a - b; },
    function multiply(a, b) { return a * b; }
];

// They can be stored as object properties
const calculator = {
    add: function(a, b) { return a + b; },
    subtract: function(a, b) { return a - b; },
    operation: null,
    
    setOperation: function(fn) {
        this.operation = fn;
    },
    
    calculate: function(a, b) {
        return this.operation ? this.operation(a, b) : 0;
    }
};

// They can be passed as arguments
function executeOperation(operation, a, b) {
    return operation(a, b);
}

console.log(executeOperation(functions[0], 5, 3)); // 8 (add)
console.log(executeOperation(functions[1], 5, 3)); // 2 (subtract)

// They can be returned from other functions
function createMultiplier(factor) {
    return function(number) {
        return number * factor;
    };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

### Basic Higher Order Functions
```javascript
// Function that takes another function as argument
function processArray(array, processor) {
    const result = [];
    for (let i = 0; i < array.length; i++) {
        result.push(processor(array[i], i, array));
    }
    return result;
}

// Processor functions
function square(x) {
    return x * x;
}

function addIndex(value, index) {
    return value + index;
}

function isEven(x) {
    return x % 2 === 0;
}

const numbers = [1, 2, 3, 4, 5];

console.log(processArray(numbers, square)); // [1, 4, 9, 16, 25]
console.log(processArray(numbers, addIndex)); // [1, 3, 5, 7, 9]
console.log(processArray(numbers, isEven)); // [false, true, false, true, false]

// Function that returns another function
function createValidator(rule) {
    return function(value) {
        return rule(value);
    };
}

function createLogger(prefix) {
    return function(message) {
        console.log(`[${prefix}] ${message}`);
    };
}

// Usage
const isPositive = createValidator(x => x > 0);
const isString = createValidator(x => typeof x === 'string');

console.log(isPositive(5)); // true
console.log(isPositive(-3)); // false
console.log(isString('hello')); // true
console.log(isString(123)); // false

const errorLogger = createLogger('ERROR');
const infoLogger = createLogger('INFO');

errorLogger('Something went wrong'); // [ERROR] Something went wrong
infoLogger('Process completed'); // [INFO] Process completed
```

## Array Higher Order Methods

### map() - Transform Elements
```javascript
// Basic map usage
const numbers = [1, 2, 3, 4, 5];

// Square each number
const squared = numbers.map(x => x * x);
console.log(squared); // [1, 4, 9, 16, 25]

// Convert to strings
const strings = numbers.map(x => x.toString());
console.log(strings); // ['1', '2', '3', '4', '5']

// Working with objects
const users = [
    { id: 1, name: 'John', age: 30 },
    { id: 2, name: 'Jane', age: 25 },
    { id: 3, name: 'Bob', age: 35 }
];

// Extract specific properties
const names = users.map(user => user.name);
console.log(names); // ['John', 'Jane', 'Bob']

// Transform objects
const userSummaries = users.map(user => ({
    id: user.id,
    summary: `${user.name} (${user.age} years old)`
}));
console.log(userSummaries);

// Using index and array parameters
const numbersWithIndex = numbers.map((value, index, array) => ({
    value,
    index,
    isLast: index === array.length - 1
}));
console.log(numbersWithIndex);

// Chaining with other methods
const processedNumbers = numbers
    .map(x => x * 2)           // Double each number
    .map(x => x + 1)           // Add 1 to each
    .map(x => `Number: ${x}`); // Convert to string format

console.log(processedNumbers); // ['Number: 3', 'Number: 5', 'Number: 7', 'Number: 9', 'Number: 11']

// Complex transformations
const products = [
    { name: 'Laptop', price: 1000, category: 'Electronics', inStock: true },
    { name: 'Book', price: 20, category: 'Education', inStock: false },
    { name: 'Phone', price: 800, category: 'Electronics', inStock: true }
];

const productCards = products.map(product => ({
    ...product,
    priceWithTax: product.price * 1.1,
    displayName: product.name.toUpperCase(),
    availability: product.inStock ? 'Available' : 'Out of Stock',
    isExpensive: product.price > 500
}));

console.log(productCards);
```

### filter() - Select Elements
```javascript
// Basic filter usage
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Get even numbers
const evenNumbers = numbers.filter(x => x % 2 === 0);
console.log(evenNumbers); // [2, 4, 6, 8, 10]

// Get numbers greater than 5
const largeNumbers = numbers.filter(x => x > 5);
console.log(largeNumbers); // [6, 7, 8, 9, 10]

// Working with objects
const users = [
    { id: 1, name: 'John', age: 30, active: true, role: 'admin' },
    { id: 2, name: 'Jane', age: 25, active: false, role: 'user' },
    { id: 3, name: 'Bob', age: 35, active: true, role: 'user' },
    { id: 4, name: 'Alice', age: 28, active: true, role: 'admin' }
];

// Filter active users
const activeUsers = users.filter(user => user.active);
console.log(activeUsers);

// Filter admin users
const adminUsers = users.filter(user => user.role === 'admin');
console.log(adminUsers);

// Multiple conditions
const activeAdmins = users.filter(user => user.active && user.role === 'admin');
console.log(activeAdmins);

// Complex filtering
const eligibleUsers = users.filter(user => {
    return user.active && 
           user.age >= 25 && 
           user.age <= 35 &&
           user.name.length > 3;
});
console.log(eligibleUsers);

// Using index parameter
const firstHalf = users.filter((user, index, array) => index < array.length / 2);
console.log(firstHalf);

// Filter with external data
const allowedRoles = ['admin', 'moderator'];
const privilegedUsers = users.filter(user => allowedRoles.includes(user.role));
console.log(privilegedUsers);

// Remove duplicates using filter
const numbersWithDuplicates = [1, 2, 2, 3, 3, 3, 4, 5, 5];
const uniqueNumbers = numbersWithDuplicates.filter((value, index, array) => 
    array.indexOf(value) === index
);
console.log(uniqueNumbers); // [1, 2, 3, 4, 5]

// Filter falsy values
const mixedArray = [1, null, 2, undefined, 3, '', 4, 0, 5, false, 6];
const truthyValues = mixedArray.filter(Boolean);
console.log(truthyValues); // [1, 2, 3, 4, 5, 6]
```

### reduce() - Accumulate Values
```javascript
// Basic reduce usage
const numbers = [1, 2, 3, 4, 5];

// Sum all numbers
const sum = numbers.reduce((accumulator, current) => accumulator + current, 0);
console.log(sum); // 15

// Product of all numbers
const product = numbers.reduce((acc, curr) => acc * curr, 1);
console.log(product); // 120

// Find maximum value
const max = numbers.reduce((acc, curr) => curr > acc ? curr : acc);
console.log(max); // 5

// Find minimum value
const min = numbers.reduce((acc, curr) => curr < acc ? curr : acc);
console.log(min); // 1

// Working with objects
const purchases = [
    { item: 'Book', price: 20, quantity: 2 },
    { item: 'Pen', price: 5, quantity: 5 },
    { item: 'Notebook', price: 15, quantity: 3 }
];

// Calculate total cost
const totalCost = purchases.reduce((total, purchase) => {
    return total + (purchase.price * purchase.quantity);
}, 0);
console.log(totalCost); // 110

// Group by category
const products = [
    { name: 'Laptop', category: 'Electronics', price: 1000 },
    { name: 'Book', category: 'Education', price: 20 },
    { name: 'Phone', category: 'Electronics', price: 800 },
    { name: 'Notebook', category: 'Education', price: 15 }
];

const groupedByCategory = products.reduce((groups, product) => {
    const category = product.category;
    if (!groups[category]) {
        groups[category] = [];
    }
    groups[category].push(product);
    return groups;
}, {});
console.log(groupedByCategory);

// Count occurrences
const fruits = ['apple', 'banana', 'apple', 'orange', 'banana', 'apple'];
const fruitCount = fruits.reduce((count, fruit) => {
    count[fruit] = (count[fruit] || 0) + 1;
    return count;
}, {});
console.log(fruitCount); // { apple: 3, banana: 2, orange: 1 }

// Flatten arrays
const nestedArrays = [[1, 2], [3, 4], [5, 6]];
const flattened = nestedArrays.reduce((acc, curr) => acc.concat(curr), []);
console.log(flattened); // [1, 2, 3, 4, 5, 6]

// Create object from array
const users = [
    { id: 1, name: 'John' },
    { id: 2, name: 'Jane' },
    { id: 3, name: 'Bob' }
];

const usersById = users.reduce((acc, user) => {
    acc[user.id] = user;
    return acc;
}, {});
console.log(usersById);

// Complex data processing
const sales = [
    { salesperson: 'John', amount: 1000, month: 'January' },
    { salesperson: 'Jane', amount: 1500, month: 'January' },
    { salesperson: 'John', amount: 1200, month: 'February' },
    { salesperson: 'Bob', amount: 800, month: 'January' },
    { salesperson: 'Jane', amount: 1300, month: 'February' }
];

const salesSummary = sales.reduce((summary, sale) => {
    // Group by salesperson
    if (!summary.bySalesperson[sale.salesperson]) {
        summary.bySalesperson[sale.salesperson] = 0;
    }
    summary.bySalesperson[sale.salesperson] += sale.amount;
    
    // Group by month
    if (!summary.byMonth[sale.month]) {
        summary.byMonth[sale.month] = 0;
    }
    summary.byMonth[sale.month] += sale.amount;
    
    // Update totals
    summary.total += sale.amount;
    summary.count++;
    
    return summary;
}, {
    bySalesperson: {},
    byMonth: {},
    total: 0,
    count: 0
});

console.log(salesSummary);
```

### forEach() - Iterate Elements
```javascript
// Basic forEach usage
const numbers = [1, 2, 3, 4, 5];

// Print each number
numbers.forEach(number => {
    console.log(number);
});

// With index
numbers.forEach((number, index) => {
    console.log(`Index ${index}: ${number}`);
});

// Working with objects
const users = [
    { name: 'John', email: 'john@example.com' },
    { name: 'Jane', email: 'jane@example.com' },
    { name: 'Bob', email: 'bob@example.com' }
];

// Send email to each user (simulation)
users.forEach(user => {
    console.log(`Sending email to ${user.email}`);
    // sendEmail(user.email); // Would call actual email function
});

// Modify DOM elements (in browser environment)
const elements = document.querySelectorAll('.item');
elements.forEach((element, index) => {
    element.textContent = `Item ${index + 1}`;
    element.addEventListener('click', () => {
        console.log(`Clicked item ${index + 1}`);
    });
});

// Side effects with forEach
const products = [
    { name: 'Laptop', price: 1000 },
    { name: 'Phone', price: 800 },
    { name: 'Tablet', price: 500 }
];

let totalValue = 0;
products.forEach(product => {
    totalValue += product.price;
    console.log(`Added ${product.name}: $${product.price}`);
});
console.log(`Total value: $${totalValue}`);

// Note: forEach doesn't return anything, unlike map
const result = numbers.forEach(x => x * 2);
console.log(result); // undefined

// Use map for transformations instead
const doubled = numbers.map(x => x * 2);
console.log(doubled); // [2, 4, 6, 8, 10]
```

### find() and findIndex()
```javascript
// find() - returns first matching element
const numbers = [1, 3, 5, 8, 9, 12, 15];

// Find first even number
const firstEven = numbers.find(x => x % 2 === 0);
console.log(firstEven); // 8

// Find first number greater than 10
const firstLarge = numbers.find(x => x > 10);
console.log(firstLarge); // 12

// Working with objects
const users = [
    { id: 1, name: 'John', age: 30, active: true },
    { id: 2, name: 'Jane', age: 25, active: false },
    { id: 3, name: 'Bob', age: 35, active: true }
];

// Find user by id
const user = users.find(u => u.id === 2);
console.log(user); // { id: 2, name: 'Jane', age: 25, active: false }

// Find first active user
const activeUser = users.find(u => u.active);
console.log(activeUser); // { id: 1, name: 'John', age: 30, active: true }

// Find user by name
const johnUser = users.find(u => u.name.toLowerCase() === 'john');
console.log(johnUser);

// findIndex() - returns index of first matching element
const firstEvenIndex = numbers.findIndex(x => x % 2 === 0);
console.log(firstEvenIndex); // 3 (index of number 8)

const activeUserIndex = users.findIndex(u => u.active);
console.log(activeUserIndex); // 0

// If no match found
const youngUser = users.find(u => u.age < 20);
console.log(youngUser); // undefined

const youngUserIndex = users.findIndex(u => u.age < 20);
console.log(youngUserIndex); // -1

// Complex find operations
const products = [
    { id: 1, name: 'Laptop', category: 'Electronics', inStock: true, price: 1000 },
    { id: 2, name: 'Book', category: 'Education', inStock: false, price: 20 },
    { id: 3, name: 'Phone', category: 'Electronics', inStock: true, price: 800 }
];

// Find available electronic product under $900
const affordableElectronics = products.find(product => 
    product.category === 'Electronics' && 
    product.inStock && 
    product.price < 900
);
console.log(affordableElectronics);
```

### some() and every()
```javascript
// some() - returns true if at least one element matches
const numbers = [1, 3, 5, 7, 9];

// Check if some numbers are even
const hasEvenNumbers = numbers.some(x => x % 2 === 0);
console.log(hasEvenNumbers); // false

const mixedNumbers = [1, 3, 4, 7, 9];
const hasSomeEven = mixedNumbers.some(x => x % 2 === 0);
console.log(hasSomeEven); // true

// every() - returns true if all elements match
const allOdd = numbers.every(x => x % 2 !== 0);
console.log(allOdd); // true

const allPositive = numbers.every(x => x > 0);
console.log(allPositive); // true

// Working with objects
const users = [
    { name: 'John', age: 30, verified: true },
    { name: 'Jane', age: 25, verified: true },
    { name: 'Bob', age: 35, verified: false }
];

// Check if some users are verified
const someVerified = users.some(user => user.verified);
console.log(someVerified); // true

// Check if all users are verified
const allVerified = users.every(user => user.verified);
console.log(allVerified); // false

// Check if all users are adults
const allAdults = users.every(user => user.age >= 18);
console.log(allAdults); // true

// Complex conditions
const products = [
    { name: 'Laptop', price: 1000, inStock: true, rating: 4.5 },
    { name: 'Phone', price: 800, inStock: true, rating: 4.8 },
    { name: 'Tablet', price: 500, inStock: false, rating: 4.2 }
];

// Check if some products are expensive (>$700) and in stock
const hasExpensiveInStock = products.some(product => 
    product.price > 700 && product.inStock
);
console.log(hasExpensiveInStock); // true

// Check if all products have good ratings (>4.0)
const allWellRated = products.every(product => product.rating > 4.0);
console.log(allWellRated); // true

// Validation using every()
function validateUserData(users) {
    const validationRules = [
        user => user.name && user.name.length > 0,
        user => user.age && user.age > 0,
        user => user.email && user.email.includes('@')
    ];
    
    return users.every(user => 
        validationRules.every(rule => rule(user))
    );
}

const validUsers = [
    { name: 'John', age: 30, email: 'john@example.com' },
    { name: 'Jane', age: 25, email: 'jane@example.com' }
];

const invalidUsers = [
    { name: 'John', age: 30, email: 'john@example.com' },
    { name: '', age: 25, email: 'jane@example.com' } // Invalid name
];

console.log(validateUserData(validUsers)); // true
console.log(validateUserData(invalidUsers)); // false
```

## Custom Higher Order Functions

### Creating Reusable Higher Order Functions
```javascript
// Function that creates other functions
function createValidator(validationFn, errorMessage) {
    return function(value) {
        if (validationFn(value)) {
            return { valid: true, value };
        } else {
            return { valid: false, error: errorMessage };
        }
    };
}

// Create specific validators
const isRequired = createValidator(
    value => value !== null && value !== undefined && value !== '',
    'This field is required'
);

const isEmail = createValidator(
    value => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
    'Please enter a valid email address'
);

const isMinLength = (minLength) => createValidator(
    value => value && value.length >= minLength,
    `Must be at least ${minLength} characters long`
);

// Usage
console.log(isRequired('test')); // { valid: true, value: 'test' }
console.log(isRequired('')); // { valid: false, error: 'This field is required' }
console.log(isEmail('user@example.com')); // { valid: true, value: 'user@example.com' }
console.log(isMinLength(8)('password123')); // { valid: true, value: 'password123' }

// Function composition
function compose(...functions) {
    return function(value) {
        return functions.reduceRight((acc, fn) => fn(acc), value);
    };
}

// Helper functions
const addOne = x => x + 1;
const double = x => x * 2;
const square = x => x * x;

// Compose functions
const addOneDoubleSquare = compose(square, double, addOne);
console.log(addOneDoubleSquare(3)); // ((3 + 1) * 2)² = 64

// Pipeline function (left to right composition)
function pipe(...functions) {
    return function(value) {
        return functions.reduce((acc, fn) => fn(acc), value);
    };
}

const squareDoubleAddOne = pipe(square, double, addOne);
console.log(squareDoubleAddOne(3)); // (3² * 2) + 1 = 19

// Curry function creator
function curry(fn) {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        } else {
            return function(...nextArgs) {
                return curried.apply(this, args.concat(nextArgs));
            };
        }
    };
}

// Create curried functions
const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);

console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6
console.log(curriedAdd(1)(2, 3)); // 6

// Memoization higher order function
function memoize(fn) {
    const cache = new Map();
    
    return function(...args) {
        const key = JSON.stringify(args);
        
        if (cache.has(key)) {
            console.log('Cache hit for:', key);
            return cache.get(key);
        }
        
        console.log('Computing for:', key);
        const result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
}

// Expensive function example
function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

const memoizedFibonacci = memoize(fibonacci);

console.log(memoizedFibonacci(10)); // Will compute
console.log(memoizedFibonacci(10)); // Will use cache
```

### Throttle and Debounce Functions
```javascript
// Throttle function - limits function calls to once per specified time
function throttle(func, delay) {
    let lastCall = 0;
    
    return function(...args) {
        const now = Date.now();
        
        if (now - lastCall >= delay) {
            lastCall = now;
            return func.apply(this, args);
        }
    };
}

// Debounce function - delays function call until after specified time of inactivity
function debounce(func, delay) {
    let timeoutId;
    
    return function(...args) {
        clearTimeout(timeoutId);
        
        timeoutId = setTimeout(() => {
            func.apply(this, args);
        }, delay);
    };
}

// Example usage (in browser environment)
const handleScroll = throttle(() => {
    console.log('Scroll event handled');
}, 100);

const handleSearch = debounce((query) => {
    console.log('Searching for:', query);
    // Perform search API call
}, 300);

// window.addEventListener('scroll', handleScroll);
// searchInput.addEventListener('input', (e) => handleSearch(e.target.value));

// Advanced throttle with leading/trailing options
function advancedThrottle(func, delay, options = {}) {
    let timeoutId;
    let lastCall = 0;
    const { leading = true, trailing = true } = options;
    
    return function(...args) {
        const now = Date.now();
        
        if (!lastCall && !leading) {
            lastCall = now;
        }
        
        const remaining = delay - (now - lastCall);
        
        if (remaining <= 0 || remaining > delay) {
            if (timeoutId) {
                clearTimeout(timeoutId);
                timeoutId = null;
            }
            
            lastCall = now;
            func.apply(this, args);
        } else if (!timeoutId && trailing) {
            timeoutId = setTimeout(() => {
                lastCall = leading ? 0 : Date.now();
                timeoutId = null;
                func.apply(this, args);
            }, remaining);
        }
    };
}

// Once function - ensures function is called only once
function once(func) {
    let called = false;
    let result;
    
    return function(...args) {
        if (!called) {
            called = true;
            result = func.apply(this, args);
        }
        return result;
    };
}

const initializeApp = once(() => {
    console.log('App initialized');
    return 'initialization complete';
});

console.log(initializeApp()); // "App initialized", returns "initialization complete"
console.log(initializeApp()); // Returns "initialization complete" (no log)
```

### Functional Programming Utilities
```javascript
// Partial application
function partial(func, ...partialArgs) {
    return function(...remainingArgs) {
        return func(...partialArgs, ...remainingArgs);
    };
}

// Example usage
function greetUser(greeting, name, punctuation) {
    return `${greeting}, ${name}${punctuation}`;
}

const sayHello = partial(greetUser, 'Hello');
const sayHelloExcited = partial(greetUser, 'Hello', undefined, '!');

console.log(sayHello('John', '.')); // "Hello, John."
console.log(sayHelloExcited('Jane')); // "Hello, Jane!"

// Function that creates array methods
function createArrayMethod(operation) {
    return function(array, ...args) {
        return array[operation](...args);
    };
}

const mapArray = createArrayMethod('map');
const filterArray = createArrayMethod('filter');
const reduceArray = createArrayMethod('reduce');

const numbers = [1, 2, 3, 4, 5];
console.log(mapArray(numbers, x => x * 2)); // [2, 4, 6, 8, 10]
console.log(filterArray(numbers, x => x > 3)); // [4, 5]
console.log(reduceArray(numbers, (a, b) => a + b, 0)); // 15

// Function pipeline for data processing
function createPipeline(...operations) {
    return function(data) {
        return operations.reduce((result, operation) => {
            if (typeof operation === 'function') {
                return operation(result);
            } else if (operation.type && operation.args) {
                switch (operation.type) {
                    case 'map':
                        return result.map(operation.args);
                    case 'filter':
                        return result.filter(operation.args);
                    case 'reduce':
                        return result.reduce(operation.args.fn, operation.args.initial);
                    default:
                        return result;
                }
            }
            return result;
        }, data);
    };
}

// Create data processing pipeline
const processNumbers = createPipeline(
    { type: 'filter', args: x => x > 2 },
    { type: 'map', args: x => x * 2 },
    { type: 'reduce', args: { fn: (a, b) => a + b, initial: 0 } }
);

console.log(processNumbers([1, 2, 3, 4, 5])); // ((3+4+5) * 2) = 24

// Conditional execution
function when(predicate, fn) {
    return function(value) {
        return predicate(value) ? fn(value) : value;
    };
}

function unless(predicate, fn) {
    return function(value) {
        return !predicate(value) ? fn(value) : value;
    };
}

const processPositive = when(x => x > 0, x => x * 2);
const processNegative = unless(x => x >= 0, x => Math.abs(x));

console.log(processPositive(5)); // 10
console.log(processPositive(-3)); // -3
console.log(processNegative(-5)); // 5
console.log(processNegative(3)); // 3

// Error handling wrapper
function tryCatch(fn, errorHandler = console.error) {
    return function(...args) {
        try {
            return fn.apply(this, args);
        } catch (error) {
            return errorHandler(error);
        }
    };
}

const safeJSONParse = tryCatch(
    JSON.parse,
    (error) => ({ error: 'Invalid JSON', message: error.message })
);

console.log(safeJSONParse('{"valid": true}')); // { valid: true }
console.log(safeJSONParse('invalid json')); // { error: 'Invalid JSON', message: '...' }
```

## Practical Applications

### Event System with Higher Order Functions
```javascript
// Event emitter using higher order functions
function createEventEmitter() {
    const events = {};
    
    return {
        on(eventName, callback) {
            if (!events[eventName]) {
                events[eventName] = [];
            }
            events[eventName].push(callback);
            
            // Return unsubscribe function
            return () => {
                const index = events[eventName].indexOf(callback);
                if (index > -1) {
                    events[eventName].splice(index, 1);
                }
            };
        },
        
        off(eventName, callback) {
            if (events[eventName]) {
                const index = events[eventName].indexOf(callback);
                if (index > -1) {
                    events[eventName].splice(index, 1);
                }
            }
        },
        
        emit(eventName, ...args) {
            if (events[eventName]) {
                events[eventName].forEach(callback => {
                    callback(...args);
                });
            }
        },
        
        once(eventName, callback) {
            const unsubscribe = this.on(eventName, (...args) => {
                callback(...args);
                unsubscribe();
            });
            return unsubscribe;
        }
    };
}

// Usage
const emitter = createEventEmitter();

const unsubscribe = emitter.on('user:login', (user) => {
    console.log(`User ${user.name} logged in`);
});

emitter.once('app:ready', () => {
    console.log('Application is ready');
});

emitter.emit('user:login', { name: 'John' }); // "User John logged in"
emitter.emit('app:ready'); // "Application is ready"
emitter.emit('app:ready'); // No output (once only)

unsubscribe(); // Remove login listener
emitter.emit('user:login', { name: 'Jane' }); // No output
```

### State Management with Higher Order Functions
```javascript
// Simple state management
function createStore(initialState = {}) {
    let state = { ...initialState };
    const listeners = [];
    
    function subscribe(listener) {
        listeners.push(listener);
        
        // Return unsubscribe function
        return () => {
            const index = listeners.indexOf(listener);
            if (index > -1) {
                listeners.splice(index, 1);
            }
        };
    }
    
    function getState() {
        return { ...state };
    }
    
    function setState(updates) {
        const prevState = { ...state };
        state = { ...state, ...updates };
        
        // Notify all listeners
        listeners.forEach(listener => {
            listener(state, prevState);
        });
    }
    
    function dispatch(action) {
        if (typeof action === 'function') {
            action(getState, setState, dispatch);
        } else {
            setState(action);
        }
    }
    
    return {
        subscribe,
        getState,
        setState,
        dispatch
    };
}

// Create store instance
const store = createStore({ count: 0, user: null });

// Subscribe to state changes
const unsubscribe = store.subscribe((newState, prevState) => {
    console.log('State changed:', { from: prevState, to: newState });
});

// Action creators (higher order functions)
const increment = (amount = 1) => (getState, setState) => {
    const currentState = getState();
    setState({ count: currentState.count + amount });
};

const setUser = (user) => ({ user });

const asyncLogin = (username) => (getState, setState, dispatch) => {
    console.log('Logging in...');
    
    // Simulate async operation
    setTimeout(() => {
        dispatch(setUser({ name: username, id: Date.now() }));
        console.log('Login complete');
    }, 1000);
};

// Dispatch actions
store.dispatch(increment(5));
store.dispatch(setUser({ name: 'John', id: 1 }));
store.dispatch(asyncLogin('Jane'));

console.log('Current state:', store.getState());
```

### Form Validation with Higher Order Functions
```javascript
// Validation framework using higher order functions
function createValidator() {
    const rules = {};
    
    // Rule creators
    const required = (message = 'This field is required') => (value) => {
        return value !== null && value !== undefined && value !== '' 
            ? null 
            : message;
    };
    
    const minLength = (min, message) => (value) => {
        return value && value.length >= min 
            ? null 
            : message || `Must be at least ${min} characters`;
    };
    
    const email = (message = 'Invalid email format') => (value) => {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return emailRegex.test(value) ? null : message;
    };
    
    const pattern = (regex, message) => (value) => {
        return regex.test(value) ? null : message;
    };
    
    const custom = (validatorFn, message) => (value) => {
        return validatorFn(value) ? null : message;
    };
    
    // Combine validators
    const combine = (...validators) => (value) => {
        for (const validator of validators) {
            const error = validator(value);
            if (error) return error;
        }
        return null;
    };
    
    // Field validation
    const field = (name, ...validators) => {
        rules[name] = combine(...validators);
        return { field, validate };
    };
    
    // Validate all fields
    const validate = (data) => {
        const errors = {};
        let isValid = true;
        
        Object.keys(rules).forEach(fieldName => {
            const error = rules[fieldName](data[fieldName]);
            if (error) {
                errors[fieldName] = error;
                isValid = false;
            }
        });
        
        return { isValid, errors };
    };
    
    return {
        field,
        validate,
        rules: { required, minLength, email, pattern, custom, combine }
    };
}

// Usage
const validator = createValidator();
const { rules } = validator;

// Define validation rules
validator
    .field('name', rules.required(), rules.minLength(2))
    .field('email', rules.required(), rules.email())
    .field('password', 
        rules.required(),
        rules.minLength(8),
        rules.pattern(/[A-Z]/, 'Must contain uppercase letter'),
        rules.pattern(/[0-9]/, 'Must contain number')
    )
    .field('age', 
        rules.required(),
        rules.custom(value => value >= 18, 'Must be 18 or older')
    );

// Test validation
const testData1 = {
    name: 'John',
    email: 'john@example.com',
    password: 'Password123',
    age: 25
};

const testData2 = {
    name: '',
    email: 'invalid-email',
    password: 'weak',
    age: 16
};

console.log('Valid data:', validator.validate(testData1));
console.log('Invalid data:', validator.validate(testData2));
```

---

## Related Topics
- [[02 - Functions]]
- [[03 - Arrow Functions]]
- [[11 - Currying]]
- [[28 - Array Methods]]
- [[35 - JavaScript OOP]]

---

*Next: [[14 - Events]]*
