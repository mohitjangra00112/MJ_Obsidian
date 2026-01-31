# Currying

## Overview
Currying is a functional programming technique that transforms a function with multiple arguments into a sequence of functions, each taking a single argument. It's named after mathematician Haskell Curry.

## Basic Currying Example
```javascript
// Regular function
function add(a, b, c) {
    return a + b + c;
}

// Curried version
function curriedAdd(a) {
    return function(b) {
        return function(c) {
            return a + b + c;
        };
    };
}

// Usage
console.log(add(1, 2, 3)); // 6
console.log(curriedAdd(1)(2)(3)); // 6

// Partial application
const addOne = curriedAdd(1);
const addOneAndTwo = addOne(2);
console.log(addOneAndTwo(3)); // 6
```

## Arrow Function Currying
```javascript
// More concise with arrow functions
const multiply = a => b => c => a * b * c;

// Usage
console.log(multiply(2)(3)(4)); // 24

// Partial applications
const double = multiply(2);
const doubleAndTriple = double(3);
console.log(doubleAndTriple(5)); // 30

// Can also be written as:
const multiplyBy2 = multiply(2);
const multiplyBy2And3 = multiplyBy2(3);
const result = multiplyBy2And3(4); // 24
```

## Manual Currying Transformation
```javascript
// Transform regular function to curried
function curry(fn) {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        } else {
            return function(...args2) {
                return curried.apply(this, args.concat(args2));
            };
        }
    };
}

// Original function
function sum(a, b, c, d) {
    return a + b + c + d;
}

// Curry it
const curriedSum = curry(sum);

// All these work:
console.log(curriedSum(1)(2)(3)(4)); // 10
console.log(curriedSum(1, 2)(3, 4)); // 10
console.log(curriedSum(1)(2, 3, 4)); // 10
console.log(curriedSum(1, 2, 3, 4)); // 10
```

## Practical Examples

### Configuration Functions
```javascript
const createApiCall = baseUrl => endpoint => method => data => {
    const url = `${baseUrl}/${endpoint}`;
    
    const options = {
        method: method,
        headers: {
            'Content-Type': 'application/json'
        }
    };
    
    if (data && method !== 'GET') {
        options.body = JSON.stringify(data);
    }
    
    return fetch(url, options);
};

// Create specialized functions
const apiCall = createApiCall('https://api.example.com');
const usersApi = apiCall('users');
const getUsers = usersApi('GET');
const postUser = usersApi('POST');

// Usage
getUsers(); // GET https://api.example.com/users
postUser({ name: 'John', email: 'john@example.com' });
```

### Validation Functions
```javascript
const validate = predicate => errorMessage => value => {
    if (predicate(value)) {
        return { isValid: true, value };
    } else {
        return { isValid: false, error: errorMessage };
    }
};

// Create specific validators
const isNotEmpty = validate(val => val.length > 0)('Field cannot be empty');
const isEmail = validate(val => /\S+@\S+\.\S+/.test(val))('Invalid email format');
const isMinLength = min => validate(val => val.length >= min)(`Minimum length is ${min}`);

// Usage
console.log(isNotEmpty('hello')); // { isValid: true, value: 'hello' }
console.log(isEmail('test@example.com')); // { isValid: true, value: 'test@example.com' }
console.log(isMinLength(8)('password')); // { isValid: true, value: 'password' }
console.log(isMinLength(8)('short')); // { isValid: false, error: 'Minimum length is 8' }
```

### Mathematical Operations
```javascript
// Mathematical operations
const mathOp = operation => a => b => operation(a, b);

const add = mathOp((a, b) => a + b);
const subtract = mathOp((a, b) => a - b);
const multiply = mathOp((a, b) => a * b);
const divide = mathOp((a, b) => a / b);

// Create specialized functions
const addTen = add(10);
const multiplyByThree = multiply(3);

console.log(addTen(5)); // 15
console.log(multiplyByThree(4)); // 12

// Chain operations
const calculate = value => 
    multiplyByThree(addTen(value));

console.log(calculate(5)); // 45 (5 + 10 = 15, 15 * 3 = 45)
```

## Advanced Currying Techniques

### Logging and Debugging
```javascript
const log = level => message => data => {
    const timestamp = new Date().toISOString();
    console.log(`[${timestamp}] ${level.toUpperCase()}: ${message}`, data || '');
    return data; // Return data for chaining
};

// Create specific loggers
const logInfo = log('info');
const logError = log('error');
const logDebug = log('debug');

// Create specific message loggers
const logUserAction = logInfo('User action');
const logApiError = logError('API Error');

// Usage
logUserAction('Button clicked');
logApiError({ status: 500, message: 'Server error' });

// Chainable logging
const processUser = user => {
    return logInfo('Processing user')(user)
        && validateUser(user)
        && saveUser(user);
};
```

### Event Handling
```javascript
const addEventListener = eventType => element => handler => {
    element.addEventListener(eventType, handler);
    return element; // For chaining
};

// Create specific event handlers
const onClick = addEventListener('click');
const onMouseOver = addEventListener('mouseover');
const onSubmit = addEventListener('submit');

// Usage
const button = document.querySelector('#myButton');
const form = document.querySelector('#myForm');

onClick(button)(() => console.log('Button clicked'));
onSubmit(form)(handleFormSubmit);

// Multiple events on same element
const setupButton = element => {
    onClick(element)(() => console.log('Clicked'));
    onMouseOver(element)(() => console.log('Hovered'));
    return element;
};
```

### Data Transformation
```javascript
const transform = transformer => data => transformer(data);
const filter = predicate => array => array.filter(predicate);
const map = fn => array => array.map(fn);
const reduce = (fn, initial) => array => array.reduce(fn, initial);

// Create specific transformations
const filterActive = filter(user => user.active);
const mapToNames = map(user => user.name);
const sumAges = reduce((sum, user) => sum + user.age, 0);

// Data
const users = [
    { name: 'John', age: 30, active: true },
    { name: 'Jane', age: 25, active: false },
    { name: 'Bob', age: 35, active: true }
];

// Chain transformations
const activeUserNames = users
    |> filterActive
    |> mapToNames;

// Or using function composition
const pipe = (...fns) => value => fns.reduce((acc, fn) => fn(acc), value);

const processUsers = pipe(
    filterActive,
    mapToNames
);

console.log(processUsers(users)); // ['John', 'Bob']
```

## Partial Application vs Currying

### Partial Application
```javascript
// Partial application - fixing some arguments
function partial(fn, ...argsToApply) {
    return function(...restArgs) {
        return fn(...argsToApply, ...restArgs);
    };
}

function greet(greeting, name, punctuation) {
    return `${greeting}, ${name}${punctuation}`;
}

// Partial application
const sayHello = partial(greet, 'Hello');
const sayHelloJohn = partial(greet, 'Hello', 'John');

console.log(sayHello('World', '!')); // "Hello, World!"
console.log(sayHelloJohn('!')); // "Hello, John!"
```

### True Currying
```javascript
// True currying - always returns a function until all args are provided
const curriedGreet = greeting => name => punctuation => 
    `${greeting}, ${name}${punctuation}`;

// Must be called with one argument at a time
const hello = curriedGreet('Hello');
const helloJohn = hello('John');
const result = helloJohn('!'); // "Hello, John!"

// Or all at once
const result2 = curriedGreet('Hi')('Jane')('?'); // "Hi, Jane?"
```

## Performance Considerations

### Memoization with Currying
```javascript
const memoize = fn => {
    const cache = new Map();
    
    return function(...args) {
        const key = JSON.stringify(args);
        
        if (cache.has(key)) {
            return cache.get(key);
        }
        
        const result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
};

// Expensive calculation
const expensiveCalculation = memoize((a, b, c) => {
    console.log('Calculating...');
    return a * b * c + Math.random(); // Simulated expensive operation
});

// Curried version with memoization
const curriedExpensive = curry(expensiveCalculation);

const multiplyBy2 = curriedExpensive(2);
const multiplyBy2And3 = multiplyBy2(3);

console.log(multiplyBy2And3(4)); // Calculating... (some result)
console.log(multiplyBy2And3(4)); // (same result, from cache)
```

### Avoiding Excessive Function Creation
```javascript
// Inefficient - creates new functions every time
function inefficientCurry(a) {
    return function(b) {
        return function(c) {
            return a + b + c;
        };
    };
}

// More efficient - reuse functions when possible
const efficientCurry = (() => {
    const cache = new Map();
    
    return function(a) {
        if (cache.has(a)) {
            return cache.get(a);
        }
        
        const fn = function(b) {
            return function(c) {
                return a + b + c;
            };
        };
        
        cache.set(a, fn);
        return fn;
    };
})();
```

## Real-World Use Cases

### Redux Action Creators
```javascript
// Redux-style action creators
const createAction = type => payload => ({
    type,
    payload
});

// Specific action creators
const setUser = createAction('SET_USER');
const setError = createAction('SET_ERROR');
const setLoading = createAction('SET_LOADING');

// Usage
dispatch(setUser({ id: 1, name: 'John' }));
dispatch(setError('Something went wrong'));
dispatch(setLoading(true));
```

### Database Queries
```javascript
const query = table => operation => conditions => {
    let sql = '';
    
    switch (operation) {
        case 'SELECT':
            sql = `SELECT * FROM ${table}`;
            break;
        case 'DELETE':
            sql = `DELETE FROM ${table}`;
            break;
        case 'UPDATE':
            sql = `UPDATE ${table} SET`;
            break;
    }
    
    if (conditions) {
        sql += ` WHERE ${conditions}`;
    }
    
    return sql;
};

// Create table-specific queries
const usersQuery = query('users');
const ordersQuery = query('orders');

// Create operation-specific queries
const selectUsers = usersQuery('SELECT');
const deleteUsers = usersQuery('DELETE');

// Generate SQL
console.log(selectUsers('active = 1')); // "SELECT * FROM users WHERE active = 1"
console.log(deleteUsers('id = 5')); // "DELETE FROM users WHERE id = 5"
```

### CSS-in-JS Styling
```javascript
const style = property => value => element => {
    element.style[property] = value;
    return element; // For chaining
};

// Create specific styling functions
const setColor = style('color');
const setBackground = style('backgroundColor');
const setFontSize = style('fontSize');

// Create color-specific functions
const makeRed = setColor('red');
const makeBlue = setColor('blue');
const makeLarge = setFontSize('24px');

// Usage
const element = document.querySelector('#myElement');
makeRed(element);
makeLarge(element);

// Chaining
const styleElement = pipe(
    makeRed,
    makeLarge,
    setBackground('yellow')
);

styleElement(element);
```

## Testing Curried Functions
```javascript
// Test utilities for curried functions
const test = description => fn => {
    try {
        fn();
        console.log(`✓ ${description}`);
    } catch (error) {
        console.log(`✗ ${description}: ${error.message}`);
    }
};

const assertEquals = expected => actual => {
    if (expected !== actual) {
        throw new Error(`Expected ${expected}, got ${actual}`);
    }
};

// Testing curried functions
const testAdd = test('Curried add function');
const assertAddResult = assertEquals(6);

testAdd(() => {
    const add = a => b => c => a + b + c;
    assertAddResult(add(1)(2)(3));
});

testAdd(() => {
    const add = curry((a, b, c) => a + b + c);
    assertAddResult(add(1)(2)(3));
    assertAddResult(add(1, 2)(3));
    assertAddResult(add(1)(2, 3));
});
```

---

## Related Topics
- [[02 - Functions]]
- [[10 - Closures]]
- [[12 - Call Apply Bind]]
- [[13 - Higher Order Functions]]

---

*Next: [[12 - Call Apply Bind]]*
