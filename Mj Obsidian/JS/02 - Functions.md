# Functions

## Overview
Functions are fundamental building blocks in JavaScript. They are first-class objects, meaning they can be stored in variables, passed as arguments, and returned from other functions.

## Function Declaration
```javascript
function greet(name) {
    return `Hello, ${name}!`;
}

console.log(greet("John")); // "Hello, John!"

// Function declarations are hoisted
sayHello(); // Works! Prints "Hello!"

function sayHello() {
    console.log("Hello!");
}
```

## Function Expression
```javascript
const greet = function(name) {
    return `Hello, ${name}!`;
};

// Anonymous function expression
const add = function(a, b) {
    return a + b;
};

// Named function expression (useful for debugging)
const factorial = function fact(n) {
    if (n <= 1) return 1;
    return n * fact(n - 1);
};
```

## Function Parameters

### Default Parameters (ES6)
```javascript
function greet(name = "World", punctuation = "!") {
    return `Hello, ${name}${punctuation}`;
}

console.log(greet()); // "Hello, World!"
console.log(greet("John")); // "Hello, John!"
console.log(greet("John", "?")); // "Hello, John?"
```

### Rest Parameters
```javascript
function sum(...numbers) {
    return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3, 4)); // 10
console.log(sum(1, 2)); // 3

// Rest parameter must be last
function example(first, second, ...rest) {
    console.log(first); // 1
    console.log(second); // 2
    console.log(rest); // [3, 4, 5]
}
example(1, 2, 3, 4, 5);
```

### Arguments Object (Legacy)
```javascript
function oldWay() {
    console.log(arguments[0]); // First argument
    console.log(arguments.length); // Number of arguments
    
    // Convert to array
    const args = Array.prototype.slice.call(arguments);
    return args;
}

console.log(oldWay(1, 2, 3)); // [1, 2, 3]
```

## Function Scope and Context

### `this` Keyword
```javascript
const person = {
    name: "John",
    greet: function() {
        return `Hello, I'm ${this.name}`;
    },
    
    delayedGreet: function() {
        setTimeout(function() {
            console.log(this.name); // undefined (wrong context)
        }, 1000);
        
        setTimeout(() => {
            console.log(this.name); // "John" (arrow function preserves context)
        }, 1000);
    }
};
```
#### 1. **Normal function (`function() { ... }`)**

- **Has its own `this`.**
    
- When used as a callback in `setTimeout`, `this` **does not refer to the surrounding object** (`person`).
    
- Instead, `this` is set to the global object (`window` in browsers) or `undefined` (in strict mode).
    
- So `this.name` is `undefined`.
    

#### 2. **Arrow function (`() => { ... }`)**

- **Does NOT have its own `this`.**
    
- Instead, it **inherits `this` from the enclosing lexical scope** (where the function is defined).

### Function vs Method
```javascript
// Function - this refers to global object (window in browser)
function globalFunction() {
    console.log(this); // Window object (in browser)
}

// Method - this refers to the object
const obj = {
    method: function() {
        console.log(this); // obj
    }
};
```

## IIFE (Immediately Invoked Function Expression)
```javascript
// Basic IIFE
(function() {
    console.log("IIFE executed!");
})();

// IIFE with parameters
(function(name) {
    console.log(`Hello, ${name}!`);
})("John");

// Arrow function IIFE
(() => {
    console.log("Arrow IIFE!");
})();

// Practical use - avoiding global pollution
const myModule = (function() {
    let privateVar = "I'm private";
    
    return {
        publicMethod: function() {
            return privateVar;
        }
    };
})();
```

## Function as First-Class Objects

### Storing in Variables
```javascript
const myFunction = function() {
    return "Hello!";
};

const functions = [
    function() { return "First"; },
    function() { return "Second"; }
];
```

### Passing as Arguments
```javascript
function executeFunction(fn, value) {
    return fn(value);
}

function double(x) {
    return x * 2;
}

function square(x) {
    return x * x;
}

console.log(executeFunction(double, 5)); // 10
console.log(executeFunction(square, 5)); // 25
```

### Returning from Functions
```javascript
function createMultiplier(multiplier) {
    return function(number) {
        return number * multiplier;
    };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

## Recursive Functions
```javascript
// Factorial
function factorial(n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

// Fibonacci
function fibonacci(n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

// With memoization for better performance
function fibonacciMemo() {
    const cache = {};
    
    return function fib(n) {
        if (n in cache) return cache[n];
        if (n <= 1) return n;
        
        cache[n] = fib(n - 1) + fib(n - 2);
        return cache[n];
    };
}

const fastFib = fibonacciMemo();
```

## Generator Functions (ES6)
```javascript
function* numberGenerator() {
    let i = 0;
    while (true) {
        yield i++;
    }
}

const gen = numberGenerator();
console.log(gen.next().value); // 0
console.log(gen.next().value); // 1
console.log(gen.next().value); // 2

// Finite generator
function* range(start, end) {
    for (let i = start; i < end; i++) {
        yield i;
    }
}

for (const num of range(1, 5)) {
    console.log(num); // 1, 2, 3, 4
}
```

## Callback Functions
```javascript
// Basic callback
function processData(data, callback) {
    const result = data.toUpperCase();
    callback(result);
}

processData("hello", function(result) {
    console.log(result); // "HELLO"
});

// Array methods with callbacks
const numbers = [1, 2, 3, 4, 5];

const doubled = numbers.map(function(num) {
    return num * 2;
});

const evens = numbers.filter(function(num) {
    return num % 2 === 0;
});
```

## Function Methods

### call()
```javascript
function greet() {
    return `Hello, ${this.name}!`;
}

const person = { name: "John" };

console.log(greet.call(person)); // "Hello, John!"

// With arguments
function introduce(age, city) {
    return `I'm ${this.name}, ${age} years old from ${city}`;
}

console.log(introduce.call(person, 25, "New York"));
```

### apply() 
```javascript
function sum(a, b, c) {
    return a + b + c;
}

const numbers = [1, 2, 3];
console.log(sum.apply(null, numbers)); // 6

// Finding max in array
const nums = [1, 5, 3, 9, 2];
console.log(Math.max.apply(null, nums)); // 9
```

### bind()
`bind()` is a method that creates a **new function** with a fixed `this` value and optionally preset arguments
```javascript
const person = {
    name: "John",
    greet: function() {
        return `Hello, ${this.name}!`;
    }
};

const greetFunction = person.greet.bind(person);
console.log(greetFunction()); // "Hello, John!"

// Partial application
function multiply(a, b) {
    return a * b;
}

const double = multiply.bind(null, 2);
console.log(double(5)); // 10
```

## Function Properties and Methods

### length Property
```javascript
function example(a, b, c) {}
console.log(example.length); // 3 (number of parameters)

function withDefault(a, b = 5) {}
console.log(withDefault.length); // 1 (defaults don't count)

function withRest(a, ...rest) {}
console.log(withRest.length); // 1 (rest doesn't count)
```

### name Property
```javascript
function namedFunction() {}
console.log(namedFunction.name); // "namedFunction"

const anonymous = function() {};
console.log(anonymous.name); // "anonymous"

const arrow = () => {};
console.log(arrow.name); // "arrow"
```

## Performance Considerations

### Function Creation in Loops (Avoid)
```javascript
// Bad - creates new function each iteration
const buttons = document.querySelectorAll('button');
for (let i = 0; i < buttons.length; i++) {
    buttons[i].onclick = function() {
        console.log(`Button ${i} clicked`);
    };
}

// Good - reuse function
function handleClick(event) {
    console.log('Button clicked');
}

for (let i = 0; i < buttons.length; i++) {
    buttons[i].onclick = handleClick;
}
```

### Memoization
```javascript
function memoize(fn) {
    const cache = {};
    return function(...args) {
        const key = JSON.stringify(args);
        if (key in cache) {
            return cache[key];
        }
        const result = fn.apply(this, args);
        cache[key] = result;
        return result;
    };
}

const expensiveFunction = memoize(function(n) {
    console.log('Computing...');
    return n * n;
});

console.log(expensiveFunction(5)); // Computing... 25
console.log(expensiveFunction(5)); // 25 (from cache)
```

## Best Practices

### 1. Use Descriptive Names
```javascript
// Good
function calculateTotalPrice(items, taxRate) {
    return items.reduce((total, item) => total + item.price, 0) * (1 + taxRate);
}

// Bad
function calc(arr, rate) {
    return arr.reduce((t, i) => t + i.price, 0) * (1 + rate);
}
```

### 2. Keep Functions Small
```javascript
// Good - single responsibility
function validateEmail(email) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return emailRegex.test(email);
}

function validatePassword(password) {
    return password.length >= 8;
}

function validateUser(user) {
    return validateEmail(user.email) && validatePassword(user.password);
}
```

### 3. Avoid Deep Nesting
```javascript
// Bad
function processUser(user) {
    if (user) {
        if (user.isActive) {
            if (user.hasPermission) {
                // do something
            }
        }
    }
}

// Good
function processUser(user) {
    if (!user || !user.isActive || !user.hasPermission) {
        return;
    }
    // do something
}
```

---

## Related Topics
- [[03 - Arrow Functions]]
- [[10 - Closures]]
- [[11 - Currying]]
- [[12 - Call Apply Bind]]
- [[13 - Higher Order Functions]]

---

*Next: [[03 - Arrow Functions]]*
