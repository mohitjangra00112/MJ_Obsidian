# Arrow Functions

## Overview
Arrow functions were introduced in ES6 and provide a more concise syntax for writing functions. They have different behavior regarding `this` binding compared to regular functions.

## Basic Syntax

### Single Parameter, Single Expression
```javascript
// Arrow function
const square = x => x * x;

// Equivalent regular function
const square = function(x) {
    return x * x;
};
```

### Multiple Parameters
```javascript
const add = (a, b) => a + b;
const multiply = (x, y) => x * y;

// With destructuring
const greet = ({name, age}) => `Hello ${name}, you are ${age}`;
```

### No Parameters
```javascript
const sayHello = () => "Hello!";
const random = () => Math.random();
```

### Multiple Statements
```javascript
const processData = (data) => {
    const cleaned = data.trim();
    const upper = cleaned.toUpperCase();
    return upper;
};

// With early return
const divide = (a, b) => {
    if (b === 0) {
        throw new Error("Division by zero");
    }
    return a / b;
};
```

## Parentheses Rules

### When Parentheses are Optional
```javascript
// Single parameter - parentheses optional
const double = x => x * 2;
const triple = (x) => x * 3; // Also valid

// No parameters - parentheses required
const getRandom = () => Math.random();
```

### When Parentheses are Required
```javascript
// Multiple parameters
const add = (a, b) => a + b;

// Default parameters
const greet = (name = "World") => `Hello ${name}`;

// Rest parameters
const sum = (...numbers) => numbers.reduce((a, b) => a + b, 0);

// Destructuring
const getFullName = ({first, last}) => `${first} ${last}`;
```

## Return Statement Behavior

### Implicit Return
```javascript
// Single expression - automatic return
const add = (a, b) => a + b;
const isEven = n => n % 2 === 0;

// Object literal - wrap in parentheses
const createUser = (name, age) => ({
    name: name,
    age: age,
    active: true
});

// Without parentheses, it's interpreted as block
const broken = (name) => { name: name }; // Syntax error
```

### Explicit Return
```javascript
const processArray = (arr) => {
    if (!Array.isArray(arr)) {
        return [];
    }
    
    return arr
        .filter(item => item != null)
        .map(item => item.toString());
};
```

## `this` Binding - Key Difference

### Arrow Functions Don't Have Their Own `this`
```javascript
const obj = {
    name: "John",
    
    // Regular function - has its own 'this'
    regularGreet: function() {
        console.log(`Hello, I'm ${this.name}`); // "Hello, I'm John"
    },
    
    // Arrow function - inherits 'this' from enclosing scope
    arrowGreet: () => {
        console.log(`Hello, I'm ${this.name}`); // "Hello, I'm undefined"
    }
};
```

### Practical Example - Event Handlers
```javascript
class Button {
    constructor(element) {
        this.element = element;
        this.clickCount = 0;
        
        // Wrong - 'this' will refer to the button element
        this.element.addEventListener('click', function() {
            this.clickCount++; // Error: this.clickCount is undefined
            console.log(this.clickCount);
        });
        
        // Correct - arrow function preserves 'this'
        this.element.addEventListener('click', () => {
            this.clickCount++; // Works correctly
            console.log(this.clickCount);
        });
    }
}
```

### Arrow Functions in Object Methods
```javascript
const person = {
    name: "John",
    hobbies: ["reading", "coding", "gaming"],
    
    // Don't use arrow function for object methods
    introduce: () => {
        // 'this' doesn't refer to person object
        console.log(`Hi, I'm ${this.name}`); // "Hi, I'm undefined"
    },
    
    // Use regular function for object methods
    introduce2: function() {
        console.log(`Hi, I'm ${this.name}`); // "Hi, I'm John"
        
        // Use arrow functions inside methods
        this.hobbies.forEach(hobby => {
            console.log(`${this.name} likes ${hobby}`);
        });
    }
};
```

## Common Use Cases

### Array Methods
```javascript
const numbers = [1, 2, 3, 4, 5];

// Map
const doubled = numbers.map(n => n * 2);
const squared = numbers.map(n => n * n);

// Filter
const evens = numbers.filter(n => n % 2 === 0);
const greaterThanThree = numbers.filter(n => n > 3);

// Reduce
const sum = numbers.reduce((acc, n) => acc + n, 0);
const product = numbers.reduce((acc, n) => acc * n, 1);

// Find
const firstEven = numbers.find(n => n % 2 === 0);
const hasLargeNumber = numbers.some(n => n > 10);
```

### Promise Chains
```javascript
fetch('/api/users')
    .then(response => response.json())
    .then(users => users.filter(user => user.active))
    .then(activeUsers => activeUsers.map(user => user.name))
    .then(names => console.log(names))
    .catch(error => console.error(error));
```

### Async/Await
```javascript
const fetchUserData = async (userId) => {
    try {
        const response = await fetch(`/api/users/${userId}`);
        const user = await response.json();
        return user;
    } catch (error) {
        throw new Error(`Failed to fetch user: ${error.message}`);
    }
};
```

## Limitations and Restrictions

### 1. Cannot Use as Constructors
```javascript
// Regular function - can be constructor
function Person(name) {
    this.name = name;
}
const john = new Person("John"); // Works

// Arrow function - cannot be constructor
const PersonArrow = (name) => {
    this.name = name;
};
// const jane = new PersonArrow("Jane"); // TypeError
```

### 2. No `arguments` Object
```javascript
// Regular function has 'arguments'
function regularFunc() {
    console.log(arguments[0]); // First argument
}

// Arrow function doesn't have 'arguments'
const arrowFunc = () => {
    console.log(arguments); // ReferenceError
};

// Use rest parameters instead
const arrowWithRest = (...args) => {
    console.log(args[0]); // First argument
};
```

### 3. Cannot Use `yield`
```javascript
// Regular generator function
function* regularGenerator() {
    yield 1;
    yield 2;
}

// Arrow functions cannot be generators
// const arrowGenerator = * () => { // Syntax Error
//     yield 1;
// };
```

## Best Practices

### 1. Use for Short Functions
```javascript
// Good - concise and readable
const users = data.map(item => item.user);
const adults = people.filter(person => person.age >= 18);

// Okay, but regular function might be clearer for complex logic
const processComplexData = (data) => {
    // Many lines of complex logic
    // Consider using regular function
};
```

### 2. Avoid for Object Methods
```javascript
// Don't do this
const obj = {
    name: "John",
    greet: () => console.log(`Hello ${this.name}`) // 'this' is wrong
};

// Do this instead
const obj = {
    name: "John",
    greet() {
        console.log(`Hello ${this.name}`);
    }
};
```

### 3. Use When You Need to Preserve `this`
```javascript
class Timer {
    constructor() {
        this.seconds = 0;
        
        // Arrow function preserves 'this'
        setInterval(() => {
            this.seconds++;
            console.log(this.seconds);
        }, 1000);
        
        // Regular function would need .bind(this)
        // setInterval(function() {
        //     this.seconds++;
        // }.bind(this), 1000);
    }
}
```

### 4. Readability Considerations
```javascript
// Sometimes regular functions are clearer
const calculateTax = function(income, rate, deductions) {
    const taxableIncome = income - deductions;
    if (taxableIncome <= 0) return 0;
    return taxableIncome * rate;
};

// Arrow function for simple operations
const tax = (income, rate) => income * rate;
```

## Arrow Functions vs Regular Functions Comparison

| Feature | Arrow Function | Regular Function |
|---------|----------------|------------------|
| Syntax | Concise | Verbose |
| `this` binding | Lexical | Dynamic |
| `arguments` object | No | Yes |
| Constructor | No | Yes |
| Hoisting | No | Yes |
| Generator | No | Yes |
| Method definition | Not recommended | Recommended |

## Advanced Examples

### Currying with Arrow Functions
```javascript
const add = a => b => a + b;
const addFive = add(5);
console.log(addFive(3)); // 8

// Multi-level currying
const multiply = a => b => c => a * b * c;
console.log(multiply(2)(3)(4)); // 24
```

### Higher-Order Functions
```javascript
const withLogging = fn => (...args) => {
    console.log('Function called with:', args);
    const result = fn(...args);
    console.log('Function returned:', result);
    return result;
};

const add = (a, b) => a + b;
const loggedAdd = withLogging(add);
loggedAdd(2, 3); // Logs arguments and result
```

### Conditional Execution
```javascript
const processUser = user => 
    user.isActive 
        ? {...user, lastSeen: new Date()}
        : {...user, status: 'inactive'};

// Chain operations
const pipeline = data => 
    data
        .filter(item => item.valid)
        .map(item => ({...item, processed: true}))
        .sort((a, b) => a.priority - b.priority);
```

---

## Related Topics
- [[02 - Functions]]
- [[04 - Scope and Hoisting]]
- [[10 - Closures]]
- [[11 - Currying]]
- [[35 - JavaScript OOP]]

---

*Next: [[04 - Scope and Hoisting]]*
