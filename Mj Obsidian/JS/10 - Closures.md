# Closures

## Overview
A closure is the combination of a function and the lexical environment within which that function was declared. Closures give you access to an outer function's scope from an inner function.

## Basic Closure Example
```javascript
function outerFunction(x) {
    // Outer function's variable
    
    function innerFunction(y) {
        // Inner function has access to outer function's variable
        console.log(x + y);
    }
    
    return innerFunction;
}

const addFive = outerFunction(5);
addFive(3); // 8 - the inner function remembers x = 5
```

## How Closures Work

### Lexical Scoping
```javascript
function init() {
    var name = "Mozilla"; // Local variable created by init
    
    function displayName() {
        // Inner function, a closure
        console.log(name); // Uses variable declared in parent function
    }
    
    displayName();
}

init(); // "Mozilla"
```

### Persistent Local Variables
```javascript
function makeCounter() {
    let count = 0;
    
    return function() {
        count++;
        return count;
    };
}

const counter1 = makeCounter();
const counter2 = makeCounter();

console.log(counter1()); // 1
console.log(counter1()); // 2
console.log(counter2()); // 1 (independent counter)
console.log(counter1()); // 3
```

## Practical Examples

### Data Privacy
```javascript
function createBankAccount(initialBalance) {
    let balance = initialBalance;
    
    return {
        deposit: function(amount) {
            if (amount > 0) {
                balance += amount;
                return balance;
            }
            throw new Error("Amount must be positive");
        },
        
        withdraw: function(amount) {
            if (amount > 0 && amount <= balance) {
                balance -= amount;
                return balance;
            }
            throw new Error("Invalid withdrawal amount");
        },
        
        getBalance: function() {
            return balance;
        }
        
        // balance is not directly accessible from outside
    };
}

const account = createBankAccount(100);
console.log(account.getBalance()); // 100
account.deposit(50);
console.log(account.getBalance()); // 150
// console.log(account.balance); // undefined - private!
```

### Function Factories
```javascript
function createMultiplier(multiplier) {
    return function(x) {
        return x * multiplier;
    };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);
const quadruple = createMultiplier(4);

console.log(double(5)); // 10
console.log(triple(5)); // 15
console.log(quadruple(5)); // 20
```

### Module Pattern
```javascript
const mathModule = (function() {
    // Private variables
    let calculations = 0;
    
    // Private function
    function logCalculation(operation, a, b, result) {
        calculations++;
        console.log(`${operation}: ${a} and ${b} = ${result} (calc #${calculations})`);
    }
    
    // Public API
    return {
        add: function(a, b) {
            const result = a + b;
            logCalculation("Addition", a, b, result);
            return result;
        },
        
        subtract: function(a, b) {
            const result = a - b;
            logCalculation("Subtraction", a, b, result);
            return result;
        },
        
        getCalculationCount: function() {
            return calculations;
        }
    };
})();

mathModule.add(5, 3); // Addition: 5 and 3 = 8 (calc #1)
mathModule.subtract(10, 4); // Subtraction: 10 and 4 = 6 (calc #2)
console.log(mathModule.getCalculationCount()); // 2
```

## Event Handlers and Closures

### Classic Problem
```javascript
// Problem: all buttons alert "3"
for (var i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // 3, 3, 3
    }, 100);
}

// Solution 1: Use let (block scope)
for (let i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // 0, 1, 2
    }, 100);
}

// Solution 2: IIFE closure
for (var i = 0; i < 3; i++) {
    (function(index) {
        setTimeout(function() {
            console.log(index); // 0, 1, 2
        }, 100);
    })(i);
}

// Solution 3: bind
for (var i = 0; i < 3; i++) {
    setTimeout(function(index) {
        console.log(index); // 0, 1, 2
    }.bind(null, i), 100);
}
```

### Event Listeners
```javascript
function attachListeners() {
    const buttons = document.querySelectorAll('button');
    
    for (let i = 0; i < buttons.length; i++) {
        buttons[i].addEventListener('click', function() {
            console.log(`Button ${i} clicked`); // Each button remembers its i
        });
    }
}

// Better approach with closure
function createClickHandler(buttonIndex) {
    return function() {
        console.log(`Button ${buttonIndex} clicked`);
    };
}

function attachListenersBetter() {
    const buttons = document.querySelectorAll('button');
    
    for (let i = 0; i < buttons.length; i++) {
        buttons[i].addEventListener('click', createClickHandler(i));
    }
}
```

## Advanced Closure Patterns

### Partial Application
```javascript
function multiply(a, b, c) {
    return a * b * c;
}

function partial(fn, ...argsToApply) {
    return function(...restArgs) {
        return fn(...argsToApply, ...restArgs);
    };
}

const multiplyByTwo = partial(multiply, 2);
const multiplyByTwoAndThree = partial(multiply, 2, 3);

console.log(multiplyByTwo(3, 4)); // 2 * 3 * 4 = 24
console.log(multiplyByTwoAndThree(5)); // 2 * 3 * 5 = 30
```

### Memoization
```javascript
function memoize(fn) {
    const cache = {};
    
    return function(...args) {
        const key = JSON.stringify(args);
        
        if (key in cache) {
            console.log('Cache hit!');
            return cache[key];
        }
        
        console.log('Computing result...');
        const result = fn.apply(this, args);
        cache[key] = result;
        return result;
    };
}

function expensiveFunction(n) {
    // Simulate expensive computation
    let result = 0;
    for (let i = 0; i < n; i++) {
        result += i;
    }
    return result;
}

const memoizedFunction = memoize(expensiveFunction);

console.log(memoizedFunction(1000)); // Computing result... 499500
console.log(memoizedFunction(1000)); // Cache hit! 499500
```

### Throttling and Debouncing
```javascript
// Throttle - limit function calls
function throttle(func, delay) {
    let timeoutId;
    let lastExecTime = 0;
    
    return function(...args) {
        const currentTime = Date.now();
        
        if (currentTime - lastExecTime > delay) {
            func.apply(this, args);
            lastExecTime = currentTime;
        }
    };
}

// Debounce - delay function execution
function debounce(func, delay) {
    let timeoutId;
    
    return function(...args) {
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => {
            func.apply(this, args);
        }, delay);
    };
}

// Usage
const throttledResize = throttle(function() {
    console.log('Window resized (throttled)');
}, 100);

const debouncedSearch = debounce(function(query) {
    console.log(`Searching for: ${query}`);
}, 300);

window.addEventListener('resize', throttledResize);
```

## Closure with Arrow Functions
```javascript
// Arrow functions capture 'this' from enclosing scope
const obj = {
    name: "MyObject",
    
    regularMethod: function() {
        // Regular function - can use closure to capture 'this'
        const self = this;
        
        setTimeout(function() {
            console.log(self.name); // "MyObject"
        }, 1000);
        
        // Or use arrow function (automatically captures 'this')
        setTimeout(() => {
            console.log(this.name); // "MyObject"
        }, 1000);
    },
    
    arrowMethod: () => {
        // Arrow function doesn't have its own 'this'
        console.log(this.name); // undefined (this refers to global)
    }
};
```

## Memory Management and Closures

### Memory Leaks
```javascript
// Potential memory leak
function problematicClosure() {
    const largeArray = new Array(1000000).fill('data');
    
    return function() {
        // Even if we don't use largeArray, it stays in memory
        // because the closure has access to the entire scope
        console.log('Hello');
    };
}

// Better approach - only keep what you need
function improvedClosure() {
    const largeArray = new Array(1000000).fill('data');
    const importantValue = largeArray[0];
    
    return function() {
        // Only importantValue is kept in closure
        console.log(importantValue);
    };
}
```

### Cleaning Up Closures
```javascript
function createTimer() {
    let count = 0;
    let intervalId;
    
    return {
        start: function() {
            intervalId = setInterval(() => {
                count++;
                console.log(count);
            }, 1000);
        },
        
        stop: function() {
            clearInterval(intervalId);
        },
        
        getCount: function() {
            return count;
        }
    };
}

const timer = createTimer();
timer.start();
// ... later
timer.stop(); // Important to clean up!
```

## Common Closure Gotchas

### 1. Closures in Loops
```javascript
// Wrong way
var funcs = [];
for (var i = 0; i < 3; i++) {
    funcs[i] = function() {
        console.log(i); // All will log 3
    };
}

// Right way
var funcs = [];
for (let i = 0; i < 3; i++) {
    funcs[i] = function() {
        console.log(i); // Each logs its own i
    };
}
```

### 2. Accidental Closures
```javascript
function createFunctions() {
    const functions = [];
    const sharedObject = { value: 0 };
    
    for (let i = 0; i < 3; i++) {
        functions.push(function() {
            // All functions share the same object reference!
            sharedObject.value++;
            return sharedObject.value;
        });
    }
    
    return functions;
}

const funcs = createFunctions();
console.log(funcs[0]()); // 1
console.log(funcs[1]()); // 2 (modified the same object)
console.log(funcs[2]()); // 3
```

### 3. Performance Considerations
```javascript
// Inefficient - creates new closure each time
function createHandler(element) {
    return function(event) {
        // Large closure capturing entire scope
        element.style.backgroundColor = 'red';
    };
}

// More efficient - reuse closure
const changeColor = (function() {
    return function(event) {
        this.style.backgroundColor = 'red';
    };
})();

// Usage
elements.forEach(element => {
    element.addEventListener('click', changeColor);
});
```

## Testing Closures
```javascript
function createTests() {
    let testCount = 0;
    const results = [];
    
    return {
        test: function(name, fn) {
            testCount++;
            try {
                fn();
                results.push({ name, status: 'PASS' });
                console.log(`✓ ${name}`);
            } catch (error) {
                results.push({ name, status: 'FAIL', error });
                console.log(`✗ ${name}: ${error.message}`);
            }
        },
        
        getResults: function() {
            return [...results]; // Return copy
        },
        
        getTestCount: function() {
            return testCount;
        }
    };
}

const testSuite = createTests();

testSuite.test('Addition works', () => {
    if (2 + 2 !== 4) throw new Error('Math is broken');
});

testSuite.test('Strings concatenate', () => {
    if ('hello' + ' world' !== 'hello world') throw new Error('String concat failed');
});

console.log(testSuite.getResults());
```

---

## Related Topics
- [[02 - Functions]]
- [[03 - Arrow Functions]]
- [[04 - Scope and Hoisting]]
- [[11 - Currying]]
- [[13 - Higher Order Functions]]

---

*Next: [[11 - Currying]]*
