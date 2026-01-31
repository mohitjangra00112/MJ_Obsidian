# Call, Apply, and Bind

## Overview
`call`, `apply`, and `bind` are methods available on all JavaScript functions that allow you to explicitly set the value of `this` and call functions with a specific context.

## Understanding `this` Context
```javascript
const person = {
    name: "John",
    greet: function() {
        return `Hello, I'm ${this.name}`;
    }
};

console.log(person.greet()); // "Hello, I'm John"

// When method is assigned to variable, 'this' is lost
const greetFunction = person.greet;
console.log(greetFunction()); // "Hello, I'm undefined"
```

## Function.prototype.call()

### Basic Usage
```javascript
function greet() {
    return `Hello, I'm ${this.name}`;
}

const person1 = { name: "John" };
const person2 = { name: "Jane" };

// Call function with specific 'this'
console.log(greet.call(person1)); // "Hello, I'm John"
console.log(greet.call(person2)); // "Hello, I'm Jane"
```

### With Arguments
```javascript
function introduce(age, city) {
    return `I'm ${this.name}, ${age} years old from ${city}`;
}

const person = { name: "John" };

// Pass arguments after the context
console.log(introduce.call(person, 30, "New York"));
// "I'm John, 30 years old from New York"
```

### Practical Examples
```javascript
// Borrowing array methods for array-like objects
function sum() {
    // arguments is array-like but not an array
    const numbers = Array.prototype.slice.call(arguments);
    return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3, 4)); // 10

// Modern alternative with spread
function modernSum(...args) {
    return args.reduce((total, num) => total + num, 0);
}

// Finding max in array using Math.max
const numbers = [1, 5, 3, 9, 2];
const max = Math.max.call(null, ...numbers); // or Math.max(...numbers)
```

## Function.prototype.apply()

### Basic Usage
```javascript
function greet(greeting, punctuation) {
    return `${greeting}, I'm ${this.name}${punctuation}`;
}

const person = { name: "John" };

// Apply takes array of arguments
console.log(greet.apply(person, ["Hello", "!"]));
// "Hello, I'm John!"
```

### Array vs Individual Arguments
```javascript
function multiply(a, b, c) {
    return a * b * c;
}

const context = {};
const numbers = [2, 3, 4];

// call - individual arguments
console.log(multiply.call(context, 2, 3, 4)); // 24

// apply - array of arguments
console.log(multiply.apply(context, numbers)); // 24
console.log(multiply.apply(context, [2, 3, 4])); // 24
```

### Practical Examples
```javascript
// Finding min/max in array
const numbers = [1, 5, 3, 9, 2];

const max = Math.max.apply(null, numbers);
const min = Math.min.apply(null, numbers);

console.log(max); // 9
console.log(min); // 1

// Flattening arrays (before flat() method)
const arrays = [[1, 2], [3, 4], [5, 6]];
const flattened = [].concat.apply([], arrays);
console.log(flattened); // [1, 2, 3, 4, 5, 6]

// Modern way
const modernFlattened = arrays.flat();
```

## Function.prototype.bind()

### Basic Usage
```javascript
function greet() {
    return `Hello, I'm ${this.name}`;
}

const person = { name: "John" };

// bind returns a new function with fixed 'this'
const boundGreet = greet.bind(person);
console.log(boundGreet()); // "Hello, I'm John"

// Original function unchanged
console.log(greet()); // "Hello, I'm undefined"
```

### Partial Application with bind
```javascript
function multiply(a, b, c) {
    return a * b * c;
}

// Bind with partial arguments
const multiplyByTwo = multiply.bind(null, 2);
console.log(multiplyByTwo(3, 4)); // 24 (2 * 3 * 4)

const multiplyBy2And3 = multiply.bind(null, 2, 3);
console.log(multiplyBy2And3(4)); // 24 (2 * 3 * 4)
```

### Event Handlers
```javascript
class Counter {
    constructor() {
        this.count = 0;
    }
    
    increment() {
        this.count++;
        console.log(`Count: ${this.count}`);
    }
    
    setupButton(button) {
        // Without bind, 'this' would refer to the button
        button.addEventListener('click', this.increment.bind(this));
        
        // Alternative approaches:
        // button.addEventListener('click', () => this.increment());
        // button.addEventListener('click', this.increment.bind(this));
    }
}

const counter = new Counter();
const button = document.querySelector('#myButton');
counter.setupButton(button);
```

## Comparison: call vs apply vs bind

### Summary Table
| Method | Arguments | Returns | Execution |
|--------|-----------|---------|-----------|
| call | Individual arguments | Function result | Immediate |
| apply | Array of arguments | Function result | Immediate |
| bind | Individual arguments | New function | Deferred |

### Code Comparison
```javascript
function test(a, b, c) {
    console.log(`this.name: ${this.name}, args: ${a}, ${b}, ${c}`);
    return `${this.name}: ${a + b + c}`;
}

const obj = { name: "TestObject" };

// call - executes immediately with individual args
const result1 = test.call(obj, 1, 2, 3);
console.log(result1); // "TestObject: 6"

// apply - executes immediately with array of args
const result2 = test.apply(obj, [1, 2, 3]);
console.log(result2); // "TestObject: 6"

// bind - returns new function
const boundTest = test.bind(obj, 1, 2, 3);
const result3 = boundTest(); // Execute later
console.log(result3); // "TestObject: 6"

// bind with partial application
const partialBound = test.bind(obj, 1, 2);
const result4 = partialBound(3);
console.log(result4); // "TestObject: 6"
```

## Advanced Use Cases

### Method Borrowing
```javascript
// Borrowing slice method for NodeList
const divs = document.querySelectorAll('div');
const divsArray = Array.prototype.slice.call(divs);

// Borrowing push method
const arrayLike = { 0: 'a', 1: 'b', length: 2 };
Array.prototype.push.call(arrayLike, 'c');
console.log(arrayLike); // { 0: 'a', 1: 'b', 2: 'c', length: 3 }

// Modern alternatives
const divsArray2 = Array.from(divs);
const divsArray3 = [...divs];
```

### Function Composition
```javascript
function pipe(...fns) {
    return function(value) {
        return fns.reduce((acc, fn) => fn.call(this, acc), value);
    };
}

const add = (x) => x + 1;
const multiply = (x) => x * 2;
const square = (x) => x * x;

const composed = pipe(add, multiply, square);
console.log(composed(3)); // ((3 + 1) * 2)² = 64
```

### Validation Chains
```javascript
const validators = {
    required: function(value) {
        if (!value) throw new Error(`${this.field} is required`);
        return value;
    },
    
    minLength: function(min) {
        return function(value) {
            if (value.length < min) {
                throw new Error(`${this.field} must be at least ${min} characters`);
            }
            return value;
        };
    },
    
    email: function(value) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailRegex.test(value)) {
            throw new Error(`${this.field} must be a valid email`);
        }
        return value;
    }
};

function validate(field, value, ...validatorFns) {
    const context = { field };
    
    try {
        return validatorFns.reduce((val, validator) => {
            if (typeof validator === 'function') {
                return validator.call(context, val);
            } else {
                return validator.call(context)(val);
            }
        }, value);
    } catch (error) {
        return { error: error.message, field };
    }
}

// Usage
const result = validate(
    'email',
    'test@example.com',
    validators.required,
    validators.minLength(5),
    validators.email
);
```

### Decorator Pattern
```javascript
function logger(fn) {
    return function(...args) {
        console.log(`Calling ${fn.name} with args:`, args);
        const result = fn.apply(this, args);
        console.log(`${fn.name} returned:`, result);
        return result;
    };
}

function timer(fn) {
    return function(...args) {
        const start = performance.now();
        const result = fn.apply(this, args);
        const end = performance.now();
        console.log(`${fn.name} took ${end - start} milliseconds`);
        return result;
    };
}

// Original function
function expensiveCalculation(n) {
    let result = 0;
    for (let i = 0; i < n; i++) {
        result += i;
    }
    return result;
}

// Decorated function
const decoratedCalc = timer(logger(expensiveCalculation));
decoratedCalc(1000000);
```

## Arrow Functions and Context

### Arrow Functions Don't Have Their Own `this`
```javascript
const obj = {
    name: "MyObject",
    
    regularMethod: function() {
        console.log(this.name); // "MyObject"
        
        // Regular function loses context
        setTimeout(function() {
            console.log(this.name); // undefined
        }, 100);
        
        // Arrow function preserves context
        setTimeout(() => {
            console.log(this.name); // "MyObject"
        }, 100);
    },
    
    arrowMethod: () => {
        // Arrow function doesn't have its own 'this'
        console.log(this.name); // undefined (global this)
    }
};

// call/apply/bind don't work with arrow functions
const arrowFunc = () => console.log(this.name);
const boundArrow = arrowFunc.bind({ name: "Test" });
boundArrow(); // Still uses lexical 'this', not bound context
```

## Performance Considerations

### Binding in Render Methods (React Example)
```javascript
class MyComponent {
    constructor() {
        this.state = { count: 0 };
        
        // Pre-bind methods (better performance)
        this.handleClick = this.handleClick.bind(this);
    }
    
    handleClick() {
        this.setState({ count: this.state.count + 1 });
    }
    
    render() {
        // Avoid binding in render (creates new function each time)
        return (
            <button onClick={this.handleClick}>
                Count: {this.state.count}
            </button>
        );
    }
}

// Alternative: arrow function properties
class MyComponent2 {
    state = { count: 0 };
    
    handleClick = () => {
        this.setState({ count: this.state.count + 1 });
    }
    
    render() {
        return (
            <button onClick={this.handleClick}>
                Count: {this.state.count}
            </button>
        );
    }
}
```

### Memory Considerations
```javascript
// Memory efficient - reuse bound function
class EventHandler {
    constructor() {
        this.boundHandler = this.handler.bind(this);
    }
    
    handler(event) {
        console.log('Event handled:', event.type);
    }
    
    attachToElements(elements) {
        elements.forEach(element => {
            element.addEventListener('click', this.boundHandler);
        });
    }
}

// Memory inefficient - creates new function each time
class BadEventHandler {
    handler(event) {
        console.log('Event handled:', event.type);
    }
    
    attachToElements(elements) {
        elements.forEach(element => {
            element.addEventListener('click', this.handler.bind(this));
        });
    }
}
```

## Polyfills and Implementation

### Simple bind Polyfill
```javascript
if (!Function.prototype.bind) {
    Function.prototype.bind = function(thisArg, ...args) {
        const fn = this;
        
        return function(...innerArgs) {
            return fn.apply(thisArg, args.concat(innerArgs));
        };
    };
}
```

### Understanding Implementation
```javascript
// Manual implementation of call
Function.prototype.myCall = function(context, ...args) {
    context = context || globalThis;
    const fnSymbol = Symbol();
    context[fnSymbol] = this;
    const result = context[fnSymbol](...args);
    delete context[fnSymbol];
    return result;
};

// Manual implementation of apply
Function.prototype.myApply = function(context, args = []) {
    return this.myCall(context, ...args);
};

// Manual implementation of bind
Function.prototype.myBind = function(context, ...args) {
    const fn = this;
    return function(...innerArgs) {
        return fn.myApply(context, args.concat(innerArgs));
    };
};
```

---

## Related Topics
- [[02 - Functions]]
- [[03 - Arrow Functions]]
- [[10 - Closures]]
- [[11 - Currying]]
- [[35 - JavaScript OOP]]

---

*Next: [[13 - Higher Order Functions]]*
