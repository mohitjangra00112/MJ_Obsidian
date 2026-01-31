# Scope and Hoisting

## Overview
Understanding scope and hoisting is crucial for writing predictable JavaScript code. Scope determines variable accessibility, while hoisting affects when variables and functions become available.

## Types of Scope

### 1. Global Scope
```javascript
// Global variables
var globalVar = "I'm global";
let globalLet = "I'm also global";
const globalConst = "Me too";

function testGlobal() {
    console.log(globalVar); // Accessible
    console.log(globalLet); // Accessible
    console.log(globalConst); // Accessible
}

// In browser, var creates window property
console.log(window.globalVar); // "I'm global"
console.log(window.globalLet); // undefined
```

### 2. Function Scope
```javascript
function functionScope() {
    var functionVar = "I'm function-scoped";
    let functionLet = "I'm also function-scoped";
    const functionConst = "Me too";
    
    if (true) {
        var stillFunctionScoped = "Still function-scoped";
        let blockScoped = "I'm block-scoped";
        const alsoBlockScoped = "Me too";
    }
    
    console.log(stillFunctionScoped); // Works - var is function-scoped
    // console.log(blockScoped); // Error - let is block-scoped
}

// console.log(functionVar); // Error - not accessible outside function
```

### 3. Block Scope (ES6+)
```javascript
if (true) {
    var varInBlock = "I'm var in block";
    let letInBlock = "I'm let in block";
    const constInBlock = "I'm const in block";
}

console.log(varInBlock); // "I'm var in block" - var ignores block scope
// console.log(letInBlock); // Error - let respects block scope
// console.log(constInBlock); // Error - const respects block scope

// Loop example
for (var i = 0; i < 3; i++) {
    // var i is function/global scoped
}
console.log(i); // 3 - i is still accessible

for (let j = 0; j < 3; j++) {
    // let j is block scoped
}
// console.log(j); // Error - j is not accessible
```

### 4. Module Scope
```javascript
// In a module file
let moduleVar = "I'm module-scoped";

export function getModuleVar() {
    return moduleVar; // Accessible within module
}

// moduleVar is not accessible from outside the module
// unless explicitly exported
```

## Lexical Scope
```javascript
function outerFunction(x) {
    function innerFunction(y) {
        function deepFunction(z) {
            // Can access x, y, z
            console.log(x + y + z);
        }
        deepFunction(3);
    }
    innerFunction(2);
}

outerFunction(1); // 6

// Closure example
function createCounter() {
    let count = 0;
    
    return function() {
        return ++count; // Has access to outer count
    };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
```

## Hoisting

### Variable Hoisting

#### `var` Hoisting
```javascript
// What you write:
console.log(myVar); // undefined (not error)
var myVar = 5;
console.log(myVar); // 5

// How JavaScript interprets it:
var myVar; // Declaration hoisted
console.log(myVar); // undefined
myVar = 5; // Assignment stays in place
console.log(myVar); // 5
```

#### `let` and `const` Hoisting
```javascript
// Temporal Dead Zone
console.log(myLet); // ReferenceError: Cannot access before initialization
let myLet = 5;

console.log(myConst); // ReferenceError: Cannot access before initialization
const myConst = 10;

// What happens:
// let myLet; // Hoisted but in "temporal dead zone"
// console.log(myLet); // Error - in temporal dead zone
// myLet = 5; // Now accessible
```

### Function Hoisting

#### Function Declarations
```javascript
// Function declarations are fully hoisted
sayHello(); // "Hello!" - works before declaration

function sayHello() {
    console.log("Hello!");
}

// Complex example
function test() {
    if (false) {
        function nevercalled() {
            console.log("Never executed");
        }
    }
    
    // nevercalled(); // Error in strict mode, undefined in non-strict
}
```

#### Function Expressions
```javascript
// Function expressions are not hoisted
sayGoodbye(); // TypeError: sayGoodbye is not a function

var sayGoodbye = function() {
    console.log("Goodbye!");
};

// Equivalent to:
var sayGoodbye; // undefined
sayGoodbye(); // Error: undefined is not a function
sayGoodbye = function() {
    console.log("Goodbye!");
};
```

#### Arrow Functions
```javascript
// Arrow functions are not hoisted
arrowFunc(); // TypeError: arrowFunc is not a function

var arrowFunc = () => {
    console.log("Arrow function");
};
```

## Variable Shadowing
```javascript
let name = "Global";

function outer() {
    let name = "Outer";
    console.log(name); // "Outer"
    
    function inner() {
        let name = "Inner";
        console.log(name); // "Inner"
    }
    
    inner();
    console.log(name); // "Outer"
}

outer();
console.log(name); // "Global"
```

## Common Gotchas

### 1. Loop Variable Problem
```javascript
// Problem with var
for (var i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // 3, 3, 3 (not 0, 1, 2)
    }, 100);
}

// Solution 1: Use let
for (let i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // 0, 1, 2
    }, 100);
}

// Solution 2: IIFE with var
for (var i = 0; i < 3; i++) {
    (function(index) {
        setTimeout(function() {
            console.log(index); // 0, 1, 2
        }, 100);
    })(i);
}
```

### 2. Variable Declaration Order
```javascript
// Confusing hoisting behavior
var a = 1;

function test() {
    console.log(a); // undefined (not 1!)
    var a = 2;
    console.log(a); // 2
}

// Equivalent to:
function test() {
    var a; // Hoisted declaration shadows global
    console.log(a); // undefined
    a = 2;
    console.log(a); // 2
}
```

### 3. Function vs Variable Hoisting Priority
```javascript
console.log(foo); // [Function: foo]

var foo = "variable";
function foo() {
    return "function";
}

console.log(foo); // "variable"

// Equivalent to:
function foo() { // Function declaration hoisted first
    return "function";
}
var foo; // Variable declaration hoisted (but doesn't override function)
console.log(foo); // [Function: foo]
foo = "variable"; // Assignment
console.log(foo); // "variable"
```

## Best Practices

### 1. Always Declare Variables Before Use
```javascript
// Good
let userName;
const API_URL = "https://api.example.com";
var legacyVar = "only if needed";

// Use variables after declaration
```

### 2. Use `let` and `const` Instead of `var`
```javascript
// Prefer
const config = { theme: "dark" };
let counter = 0;

// Over
var config = { theme: "dark" };
var counter = 0;
```

### 3. Declare Functions Before Use
```javascript
// Good practice even though hoisting allows it
function calculateTotal(items) {
    return items.reduce((sum, item) => sum + item.price, 0);
}

const total = calculateTotal(shoppingCart);
```

### 4. Understand Block Scope
```javascript
function processItems(items) {
    const results = [];
    
    for (let i = 0; i < items.length; i++) {
        const item = items[i]; // Block-scoped
        
        if (item.valid) {
            const processed = processItem(item); // Block-scoped
            results.push(processed);
        }
        // processed is not accessible here
    }
    // i and item are not accessible here
    
    return results;
}
```

## Scope Chain Resolution
```javascript
let global = "global";

function level1() {
    let level1Var = "level1";
    
    function level2() {
        let level2Var = "level2";
        
        function level3() {
            let level3Var = "level3";
            
            // Scope chain: level3 -> level2 -> level1 -> global
            console.log(level3Var); // Found in level3
            console.log(level2Var); // Found in level2
            console.log(level1Var); // Found in level1
            console.log(global);    // Found in global
        }
        
        level3();
    }
    
    level2();
}

level1();
```

## Module Pattern and Scope
```javascript
const myModule = (function() {
    // Private variables (not accessible outside)
    let privateVar = "I'm private";
    let privateCounter = 0;
    
    // Private function
    function privateFunction() {
        console.log("Private function called");
    }
    
    // Public API
    return {
        publicMethod: function() {
            privateCounter++;
            return `Called ${privateCounter} times`;
        },
        
        getPrivateVar: function() {
            return privateVar;
        },
        
        setPrivateVar: function(value) {
            privateVar = value;
        }
    };
})();

// Usage
console.log(myModule.publicMethod()); // "Called 1 times"
console.log(myModule.getPrivateVar()); // "I'm private"
// console.log(myModule.privateVar); // undefined
```

## ES6 Modules and Scope
```javascript
// math.js
let counter = 0; // Module-scoped

export function add(a, b) {
    counter++;
    return a + b;
}

export function getCallCount() {
    return counter;
}

// main.js
import { add, getCallCount } from './math.js';

console.log(add(2, 3)); // 5
console.log(getCallCount()); // 1
// console.log(counter); // Error: counter is not exported
```

---

## Related Topics
- [[01 - Variables and Data Types]]
- [[02 - Functions]]
- [[05 - Strict Mode]]
- [[10 - Closures]]
- [[39 - Modules]]

---

*Next: [[05 - Strict Mode]]*
