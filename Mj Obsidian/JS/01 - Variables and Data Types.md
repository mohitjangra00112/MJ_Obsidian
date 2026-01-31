# Variables and Data Types

## Overview
JavaScript has different ways to declare variables and various data types. Understanding these is fundamental to JavaScript programming.

## Variable Declaration Keywords

### `var`
```javascript
var name = "John";
var age = 25;
var isStudent; // undefined

// Function-scoped
function example() {
    var x = 1;
    if (true) {
        var x = 2; // Same variable
        console.log(x); // 2
    }
    console.log(x); // 2
}
```

**Characteristics:**
- Function-scoped or globally-scoped
- Can be redeclared
- Hoisted and initialized with `undefined`
- Creates property on global object

### `let`
```javascript
let name = "John";
let age = 25;

// Block-scoped
function example() {
    let x = 1;
    if (true) {
        let x = 2; // Different variable
        console.log(x); // 2
    }
    console.log(x); // 1
}
```

**Characteristics:**
- Block-scoped
- Cannot be redeclared in same scope
- Hoisted but not initialized (Temporal Dead Zone)
- Does not create property on global object

### `const`
```javascript
const PI = 3.14159;
const user = { name: "John", age: 25 };

// const PI = 3.14; // Error: Cannot redeclare
// PI = 3.15; // Error: Cannot reassign

// But object properties can be modified
user.age = 26; // This is allowed
user.city = "New York"; // This is allowed
```

**Characteristics:**
- Block-scoped
- Must be initialized at declaration
- Cannot be reassigned
- Object/array contents can still be modified

## Data Types

### Primitive Types

#### 1. Number
```javascript
let integer = 42;
let float = 3.14;
let negative = -7;
let infinity = Infinity;
let notANumber = NaN;

// Special numbers
console.log(Number.MAX_VALUE); // Largest number
console.log(Number.MIN_VALUE); // Smallest positive number
console.log(Number.POSITIVE_INFINITY);
console.log(Number.NEGATIVE_INFINITY);
```

#### 2. String
```javascript
let singleQuote = 'Hello';
let doubleQuote = "World";
let template = `Hello ${name}!`;

// String methods
let text = "JavaScript";
console.log(text.length); // 10
console.log(text.toUpperCase()); // JAVASCRIPT
console.log(text.charAt(0)); // J
console.log(text.substring(0, 4)); // Java
```

#### 3. Boolean
```javascript
let isTrue = true;
let isFalse = false;

// Falsy values
console.log(Boolean(false)); // false
console.log(Boolean(0)); // false
console.log(Boolean("")); // false
console.log(Boolean(null)); // false
console.log(Boolean(undefined)); // false
console.log(Boolean(NaN)); // false
```

#### 4. Undefined
```javascript
let undefinedVar;
console.log(undefinedVar); // undefined

function test(param) {
    console.log(param); // undefined if not passed
}
```

#### 5. Null
```javascript
let nullVar = null;
console.log(typeof null); // "object" (known quirk)
console.log(null === undefined); // false
console.log(null == undefined); // true
```

#### 6. Symbol (ES6)
```javascript
let sym1 = Symbol();
let sym2 = Symbol("description");
let sym3 = Symbol("description");

console.log(sym2 === sym3); // false - each symbol is unique
console.log(sym2.toString()); // Symbol(description)
```

#### 7. BigInt (ES2020)
```javascript
let bigNumber = 123456789012345678901234567890n;
let anotherBig = BigInt("123456789012345678901234567890");

console.log(bigNumber + 1n); // Works with BigInt
// console.log(bigNumber + 1); // Error: Cannot mix BigInt and number
```

### Reference Types

#### Objects
```javascript
let person = {
    name: "John",
    age: 30,
    greet: function() {
        return `Hello, I'm ${this.name}`;
    }
};

let numbers = [1, 2, 3, 4, 5];
let date = new Date();
let regex = /pattern/gi;
```

## Type Checking

### `typeof` Operator
```javascript
console.log(typeof 42); // "number"
console.log(typeof "hello"); // "string"
console.log(typeof true); // "boolean"
console.log(typeof undefined); // "undefined"
console.log(typeof null); // "object" (quirk)
console.log(typeof {}); // "object"
console.log(typeof []); // "object"
console.log(typeof function(){}); // "function"
console.log(typeof Symbol()); // "symbol"
console.log(typeof 123n); // "bigint"
```

### Better Type Checking
```javascript
// Check for array
Array.isArray([1, 2, 3]); // true

// Check for null
value === null

// Check for object (not null, not array)
function isObject(value) {
    return value !== null && 
           typeof value === 'object' && 
           !Array.isArray(value);
}

// Check for primitive
function isPrimitive(value) {
    return value !== Object(value);
}
```

## Type Conversion

### Implicit Conversion (Coercion)
```javascript
// String coercion
console.log("5" + 3); // "53"
console.log("5" - 3); // 2
console.log("5" * 3); // 15

// Boolean coercion
console.log(!""); // true
console.log(!0); // true
console.log(!![]); // true
```

### Explicit Conversion
```javascript
// To String
String(123); // "123"
(123).toString(); // "123"
123 + ""; // "123"

// To Number
Number("123"); // 123
parseInt("123px"); // 123
parseFloat("123.45px"); // 123.45
+"123"; // 123

// To Boolean
Boolean(1); // true
!!1; // true
```

## Variable Best Practices

### 1. Use `const` by default
```javascript
const API_URL = "https://api.example.com";
const users = [];
const config = { theme: "dark" };
```

### 2. Use `let` when reassignment needed
```javascript
let counter = 0;
let message = "Loading...";

for (let i = 0; i < 10; i++) {
    counter += i;
}
```

### 3. Avoid `var` in modern JavaScript
```javascript
// Prefer this:
const name = "John";
let age = 25;

// Over this:
var name = "John";
var age = 25;
```

### 4. Meaningful variable names
```javascript
// Good
const userAge = 25;
const isLoggedIn = true;
const calculateTotalPrice = () => {};

// Bad
const a = 25;
const flag = true;
const calc = () => {};
```

## Common Gotchas

### 1. Variable Hoisting with `var`
```javascript
console.log(x); // undefined (not error)
var x = 5;

// Equivalent to:
var x;
console.log(x); // undefined
x = 5;
```

### 2. Temporal Dead Zone with `let`/`const`
```javascript
console.log(y); // ReferenceError
let y = 5;
```

### 3. `typeof null`
```javascript
console.log(typeof null); // "object" (historical bug)
console.log(null === undefined); // false
console.log(null == undefined); // true
```

### 4. Object vs Primitive Comparison
```javascript
let a = [1, 2, 3];
let b = [1, 2, 3];
console.log(a === b); // false (different references)

let x = 5;
let y = 5;
console.log(x === y); // true (same value)
```

---

## Related Topics
- [[04 - Scope and Hoisting]]
- [[06 - Objects]]
- [[27 - JSON]]
- [[42 - Error Handling]]

---

*Next: [[02 - Functions]]*
