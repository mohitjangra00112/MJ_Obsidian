am7# Strict Mode

## Overview
Strict mode is a way to opt into a restricted variant of JavaScript. It eliminates some JavaScript silent errors by changing them to throw errors, fixes mistakes that make it difficult for JavaScript engines to perform optimizations, and prohibits some syntax likely to be defined in future versions of ECMAScript.

## Enabling Strict Mode

### Global Strict Mode
```javascript
"use strict";

// All code in this file is in strict mode
var x = 10;
function myFunction() {
    // This function is also in strict mode
}
```

### Function-Level Strict Mode
```javascript
function strictFunction() {
    "use strict";
    // Only this function is in strict mode
}

function normalFunction() {
    // This function is not in strict mode
}
```

### Module Strict Mode
```javascript
// ES6 modules are automatically in strict mode
export function myFunction() {
    // Automatically in strict mode
}
```

## Changes in Strict Mode

### 1. Prevents Global Variable Creation
```javascript
"use strict";

// Non-strict mode: creates global variable
function createGlobal() {
    globalVar = "I'm global"; // Error in strict mode
}

// Correct way
function createVariable() {
    var localVar = "I'm local";
    // or
    let properVar = "I'm properly declared";
}
```

### 2. Throws Errors for Assignments to Non-Writable Properties
```javascript
"use strict";

// Non-writable property
const obj = {};
Object.defineProperty(obj, "readOnly", {
    value: "Can't change",
    writable: false
});

obj.readOnly = "New value"; // TypeError in strict mode

// Non-configurable property
delete obj.readOnly; // TypeError in strict mode

// Getter-only property
const obj2 = {
    get prop() {
        return this._prop;
    }
};

obj2.prop = "value"; // TypeError in strict mode
```

### 3. Throws Errors for Duplicate Parameter Names
```javascript
"use strict";

// Error in strict mode
function duplicateParams(a, b, a) { // SyntaxError
    return a + b;
}

// Correct
function uniqueParams(a, b, c) {
    return a + b + c;
}
```

### 4. Prohibits Octal Syntax
```javascript
"use strict";

var octal = 010; // SyntaxError in strict mode
var decimal = 8; // Use this instead

// Octal escape sequences also forbidden
var str = "\010"; // SyntaxError in strict mode
var str = "\x08"; // Use hex escape instead
```

### 5. Prohibits `with` Statement
```javascript
"use strict";

var obj = { a: 1, b: 2 };

// Error in strict mode
with (obj) { // SyntaxError
    console.log(a + b);
}

// Use explicit property access instead
console.log(obj.a + obj.b);
```

### 6. `eval` Creates Own Scope
```javascript
"use strict";

// Non-strict mode: eval can create variables in calling scope
eval("var evalVar = 'I exist';");
console.log(evalVar); // Error in strict mode

// In strict mode, eval has its own scope
eval("var strictEvalVar = 'I exist only in eval';");
// console.log(strictEvalVar); // ReferenceError
```

### 7. Restricts `delete` Operator
```javascript
"use strict";

var x = 1;
delete x; // SyntaxError in strict mode

function func() {}
delete func; // SyntaxError in strict mode

// delete only works on object properties
var obj = { prop: "value" };
delete obj.prop; // This works
```

### 8. `arguments` Object Behavior
```javascript
"use strict";

function strictArguments(a) {
    // In strict mode, arguments doesn't reflect parameter changes
    a = 42;
    console.log(arguments[0]); // Original value, not 42
}

function nonStrictArguments(a) {
    // Without strict mode, arguments reflects changes
    a = 42;
    console.log(arguments[0]); // 42
}
```

### 9. `this` Value in Functions
```javascript
"use strict";

function strictThis() {
    console.log(this); // undefined in strict mode
}

function nonStrictThis() {
    console.log(this); // global object (window) in non-strict mode
}

strictThis(); // undefined
nonStrictThis(); // Window object (in browser)

// Method calls still work normally
var obj = {
    method: function() {
        "use strict";
        console.log(this); // obj
    }
};
obj.method(); // obj
```

## Reserved Words in Strict Mode

### Additional Reserved Keywords
```javascript
"use strict";

// These are reserved and cannot be used as variable names
var implements = 1; // SyntaxError
var interface = 2; // SyntaxError
var let = 3; // SyntaxError (in older environments)
var package = 4; // SyntaxError
var private = 5; // SyntaxError
var protected = 6; // SyntaxError
var public = 7; // SyntaxError
var static = 8; // SyntaxError
var yield = 9; // SyntaxError
```

## Benefits of Strict Mode

### 1. Catches Common Coding Mistakes
```javascript
"use strict";

// Typos in variable names
function calculateTotal() {
    var total = 0;
    // ... calculations ...
    totla = 100; // TypeError: assignment to undeclared variable
}

// Prevents accidental globals
function processData() {
    data = []; // Error instead of creating global
    return data;
}
```

### 2. Prevents Security Issues
```javascript
"use strict";

// Prevents access to caller and arguments
function secureFunction() {
    // These throw errors in strict mode
    // arguments.caller; // TypeError
    // arguments.callee; // TypeError
}
```

### 3. Better Performance
```javascript
"use strict";

// JavaScript engines can optimize strict mode code better
function optimizedFunction() {
    var x = 1;
    var y = 2;
    return x + y; // Can be optimized more aggressively
}
```

## Practical Examples

### Form Validation
```javascript
"use strict";

function validateForm(formData) {
    // Strict mode catches typos
    if (!formData.emial) { // Would create global 'emial' without strict mode
        throw new Error("Email is required");
    }
    
    // Proper validation
    if (!formData.email) {
        throw new Error("Email is required");
    }
}
```

### Constructor Functions
```javascript
"use strict";

function Person(name, age) {
    // Without 'new', this is undefined in strict mode
    if (this === undefined) {
        throw new Error("Person must be called with 'new'");
    }
    
    this.name = name;
    this.age = age;
}

// Error: Cannot set property 'name' of undefined
var person = Person("John", 25); // Error in strict mode

// Correct usage
var person = new Person("John", 25); // Works
```

### Object Operations
```javascript
"use strict";

const config = Object.freeze({
    apiUrl: "https://api.example.com",
    timeout: 5000
});

// Error in strict mode
config.apiUrl = "https://malicious.com"; // TypeError

// Proper way to modify config
const newConfig = {
    ...config,
    apiUrl: "https://new-api.example.com"
};
```

## Best Practices

### 1. Always Use Strict Mode
```javascript
// At the top of files
"use strict";

// Or in modules (automatic strict mode)
export function myFunction() {
    // Automatically strict
}
```

### 2. Migrate Gradually
```javascript
// Enable strict mode function by function
function newFeature() {
    "use strict";
    // New code in strict mode
}

function legacyFeature() {
    // Legacy code remains non-strict during migration
}
```

### 3. Use Linting Tools
```javascript
// ESLint configuration
{
    "rules": {
        "strict": ["error", "global"]
    }
}
```

### 4. Handle Browser Compatibility
```javascript
// Feature detection for strict mode
function isStrictMode() {
    return (function() {
        "use strict";
        return this === undefined;
    })();
}

if (isStrictMode()) {
    console.log("Strict mode is supported");
} else {
    console.log("Falling back to non-strict behavior");
}
```

## Common Migration Issues

### 1. Global Variable Dependencies
```javascript
// Problem code
function problematicFunction() {
    "use strict";
    // This was creating a global variable
    result = processData(); // Error now
}

// Solution
function fixedFunction() {
    "use strict";
    var result = processData(); // Explicitly declare
    return result;
}
```

### 2. Arguments Object Usage
```javascript
// Problem code
function oldStyle(a, b) {
    "use strict";
    a = 10;
    return arguments[0]; // Still original value in strict mode
}

// Solution
function newStyle(a, b) {
    "use strict";
    var originalA = a;
    a = 10;
    return a; // Use parameter directly
}
```

### 3. Function Context Issues
```javascript
// Problem code
var obj = {
    method: function() {
        "use strict";
        setTimeout(function() {
            this.doSomething(); // 'this' is undefined
        }, 1000);
    }
};

// Solution
var obj = {
    method: function() {
        "use strict";
        var self = this;
        setTimeout(function() {
            self.doSomething(); // Use saved reference
        }, 1000);
        
        // Or use arrow functions
        setTimeout(() => {
            this.doSomething(); // Arrow function preserves 'this'
        }, 1000);
    }
};
```

---

## Related Topics
- [[01 - Variables and Data Types]]
- [[04 - Scope and Hoisting]]
- [[42 - Error Handling]]
- [[48 - Code Quality]]

---

*Next: [[06 - Objects]]*
