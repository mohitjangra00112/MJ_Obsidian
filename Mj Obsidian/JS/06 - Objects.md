# Objects

## Overview
Objects are fundamental data structures in JavaScript that store collections of key-value pairs. They are reference types and form the basis of JavaScript's prototype-based programming model.

## Object Creation Methods

### 1. Object Literal
```javascript
// Basic object literal
const person = {
    name: "John",
    age: 30,
    city: "New York"
};

// With methods
const calculator = {
    add: function(a, b) {
        return a + b;
    },
    
    // ES6 method shorthand
    subtract(a, b) {
        return a - b;
    },
    
    // Arrow function (be careful with 'this')
    multiply: (a, b) => a * b
};
```

### 2. Constructor Function
```javascript
function Person(name, age) {
    this.name = name;
    this.age = age;
    this.greet = function() {
        return `Hello, I'm ${this.name}`;
    };
}

const john = new Person("John", 30);
const jane = new Person("Jane", 25);
```

### 3. Object.create()
```javascript
// Create object with specific prototype
const personPrototype = {
    greet: function() {
        return `Hello, I'm ${this.name}`;
    }
};

const person = Object.create(personPrototype);
person.name = "John";
person.age = 30;

// Create object with null prototype (no inherited properties)
const pureObject = Object.create(null);
pureObject.name = "Test";
console.log(pureObject.toString); // undefined
```

### 4. Class Syntax (ES6)
```javascript
class Person {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    
    greet() {
        return `Hello, I'm ${this.name}`;
    }
    
    get info() {
        return `${this.name} is ${this.age} years old`;
    }
}

const person = new Person("John", 30);
```

### 5. Factory Function
```javascript
function createPerson(name, age) {
    return {
        name: name,
        age: age,
        greet() {
            return `Hello, I'm ${this.name}`;
        }
    };
}

const person = createPerson("John", 30);
```

## Property Access

### Dot Notation
```javascript
const person = { name: "John", age: 30 };

// Reading
console.log(person.name); // "John"

// Writing
person.age = 31;
person.city = "Boston"; // Add new property
```

### Bracket Notation
```javascript
const person = { name: "John", age: 30 };

// Reading
console.log(person["name"]); // "John"

// Dynamic property access
const prop = "age";
console.log(person[prop]); // 30

// Properties with special characters
const obj = {
    "first-name": "John",
    "2ndProperty": "value",
    "property with spaces": "test"
};

console.log(obj["first-name"]);
console.log(obj["2ndProperty"]);
console.log(obj["property with spaces"]);
```

### Computed Property Names (ES6)
```javascript
const prefix = "user";
const id = 123;

const user = {
    [prefix + "Id"]: id, // userId: 123
    [`${prefix}Name`]: "John", // userName: "John"
    [prefix.toUpperCase()]: "data" // USER: "data"
};
```

## Property Descriptors

### Defining Properties
```javascript
const obj = {};

// Simple property
Object.defineProperty(obj, "name", {
    value: "John",
    writable: true,
    enumerable: true,
    configurable: true
});

// Read-only property
Object.defineProperty(obj, "id", {
    value: 123,
    writable: false,
    enumerable: true,
    configurable: false
});

// Non-enumerable property (won't show in for...in)
Object.defineProperty(obj, "secret", {
    value: "hidden",
    writable: true,
    enumerable: false,
    configurable: true
});
```

### Getters and Setters
```javascript
const person = {
    firstName: "John",
    lastName: "Doe",
    
    // Getter
    get fullName() {
        return `${this.firstName} ${this.lastName}`;
    },
    
    // Setter
    set fullName(value) {
        [this.firstName, this.lastName] = value.split(" ");
    }
};

console.log(person.fullName); // "John Doe"
person.fullName = "Jane Smith";
console.log(person.firstName); // "Jane"

// Using Object.defineProperty for getters/setters
const obj = {};
Object.defineProperty(obj, "temperature", {
    get() {
        return this._celsius;
    },
    set(value) {
        this._celsius = value;
        this._fahrenheit = value * 9/5 + 32;
    }
});
```

## Object Methods

### Object.keys(), Object.values(), Object.entries()
```javascript
const person = {
    name: "John",
    age: 30,
    city: "New York"
};

// Get all keys
const keys = Object.keys(person);
console.log(keys); // ["name", "age", "city"]

// Get all values
const values = Object.values(person);
console.log(values); // ["John", 30, "New York"]

// Get key-value pairs
const entries = Object.entries(person);
console.log(entries); // [["name", "John"], ["age", 30], ["city", "New York"]]

// Convert back to object
const reconstructed = Object.fromEntries(entries);
```

### Object.assign()
```javascript
const target = { a: 1 };
const source1 = { b: 2 };
const source2 = { c: 3, a: 4 }; // 'a' will override

const result = Object.assign(target, source1, source2);
console.log(result); // { a: 4, b: 2, c: 3 }
console.log(target); // target is modified!

// Shallow copy using Object.assign
const original = { a: 1, b: { c: 2 } };
const copy = Object.assign({}, original);
// or
const copy2 = { ...original }; // Spread operator (ES6)
```

### Object.freeze(), Object.seal(), Object.preventExtensions()
```javascript
const obj = { name: "John", age: 30 };

// Freeze - no modifications allowed
Object.freeze(obj);
obj.name = "Jane"; // Ignored (or error in strict mode)
obj.city = "Boston"; // Ignored
delete obj.age; // Ignored

// Seal - can modify existing properties but can't add/remove
const obj2 = { name: "John", age: 30 };
Object.seal(obj2);
obj2.name = "Jane"; // Works
obj2.city = "Boston"; // Ignored
delete obj2.age; // Ignored

// Prevent extensions - can't add new properties
const obj3 = { name: "John", age: 30 };
Object.preventExtensions(obj3);
obj3.name = "Jane"; // Works
obj3.city = "Boston"; // Ignored
delete obj3.age; // Works

// Check status
console.log(Object.isFrozen(obj)); // true
console.log(Object.isSealed(obj2)); // true
console.log(Object.isExtensible(obj3)); // false
```

### Object.hasOwnProperty() and 'in' operator
```javascript
const person = { name: "John", age: 30 };

// Check own properties
console.log(person.hasOwnProperty("name")); // true
console.log(person.hasOwnProperty("toString")); // false

// Check all properties (including inherited)
console.log("name" in person); // true
console.log("toString" in person); // true

// Modern way (Object.hasOwn in newer environments)
console.log(Object.hasOwn(person, "name")); // true
```

## Property Enumeration

### for...in Loop
```javascript
const person = {
    name: "John",
    age: 30,
    city: "New York"
};

// Iterates over enumerable properties (including inherited)
for (const key in person) {
    if (person.hasOwnProperty(key)) {
        console.log(`${key}: ${person[key]}`);
    }
}
```

### Object.getOwnPropertyNames()
```javascript
const obj = { a: 1, b: 2 };
Object.defineProperty(obj, "hidden", {
    value: "secret",
    enumerable: false
});

console.log(Object.keys(obj)); // ["a", "b"]
console.log(Object.getOwnPropertyNames(obj)); // ["a", "b", "hidden"]
```

## Destructuring (ES6)

### Basic Destructuring
```javascript
const person = { name: "John", age: 30, city: "New York" };

// Extract properties
const { name, age } = person;
console.log(name); // "John"
console.log(age); // 30

// Rename variables
const { name: fullName, age: years } = person;
console.log(fullName); // "John"

// Default values
const { name, height = 180 } = person;
console.log(height); // 180 (default)
```

### Nested Destructuring
```javascript
const user = {
    id: 1,
    profile: {
        name: "John",
        contact: {
            email: "john@example.com",
            phone: "123-456-7890"
        }
    }
};

const {
    profile: {
        name,
        contact: { email }
    }
} = user;

console.log(name); // "John"
console.log(email); // "john@example.com"
```

### Rest Properties
```javascript
const person = { name: "John", age: 30, city: "New York", country: "USA" };

const { name, ...rest } = person;
console.log(name); // "John"
console.log(rest); // { age: 30, city: "New York", country: "USA" }
```

## Prototype and Inheritance

### Prototype Chain
```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.greet = function() {
    return `Hello, I'm ${this.name}`;
};

const john = new Person("John");
console.log(john.greet()); // "Hello, I'm John"

// Check prototype
console.log(john.__proto__ === Person.prototype); // true
console.log(Object.getPrototypeOf(john) === Person.prototype); // true
```

### Prototype Delegation
```javascript
const parent = {
    greet() {
        return `Hello from ${this.name}`;
    }
};

const child = Object.create(parent);
child.name = "Child";

console.log(child.greet()); // "Hello from Child"
console.log(child.hasOwnProperty("greet")); // false
console.log("greet" in child); // true
```

## Advanced Object Patterns

### Module Pattern
```javascript
const userModule = (function() {
    let users = [];
    let currentId = 1;
    
    return {
        addUser(name, email) {
            const user = {
                id: currentId++,
                name,
                email,
                createdAt: new Date()
            };
            users.push(user);
            return user;
        },
        
        getUser(id) {
            return users.find(user => user.id === id);
        },
        
        getAllUsers() {
            return [...users]; // Return copy
        }
    };
})();
```

### Observer Pattern
```javascript
function EventEmitter() {
    this.events = {};
}

EventEmitter.prototype = {
    on(event, listener) {
        if (!this.events[event]) {
            this.events[event] = [];
        }
        this.events[event].push(listener);
    },
    
    emit(event, ...args) {
        if (this.events[event]) {
            this.events[event].forEach(listener => {
                listener.apply(this, args);
            });
        }
    },
    
    off(event, listenerToRemove) {
        if (this.events[event]) {
            this.events[event] = this.events[event].filter(
                listener => listener !== listenerToRemove
            );
        }
    }
};
```

### Proxy Objects (ES6)
```javascript
const person = {
    name: "John",
    age: 30
};

const personProxy = new Proxy(person, {
    get(target, property) {
        console.log(`Getting ${property}`);
        return target[property];
    },
    
    set(target, property, value) {
        console.log(`Setting ${property} to ${value}`);
        if (property === "age" && value < 0) {
            throw new Error("Age cannot be negative");
        }
        target[property] = value;
        return true;
    }
});

personProxy.name; // Logs: "Getting name"
personProxy.age = 25; // Logs: "Setting age to 25"
```

## Common Object Patterns

### Configuration Objects
```javascript
function createConnection(options = {}) {
    const defaults = {
        host: "localhost",
        port: 3000,
        timeout: 5000,
        retries: 3
    };
    
    const config = { ...defaults, ...options };
    
    return {
        connect() {
            console.log(`Connecting to ${config.host}:${config.port}`);
        },
        
        disconnect() {
            console.log("Disconnecting...");
        }
    };
}

const connection = createConnection({
    host: "api.example.com",
    port: 443
});
```

### Builder Pattern
```javascript
class QueryBuilder {
    constructor() {
        this.query = {};
    }
    
    select(fields) {
        this.query.select = fields;
        return this;
    }
    
    from(table) {
        this.query.from = table;
        return this;
    }
    
    where(condition) {
        this.query.where = condition;
        return this;
    }
    
    build() {
        return { ...this.query };
    }
}

const query = new QueryBuilder()
    .select(["name", "email"])
    .from("users")
    .where("active = true")
    .build();
```

---

## Related Topics
- [[07 - Prototypes and Prototypal Inheritance]]
- [[08 - Classes and Constructor Functions]]
- [[09 - Object Creation Patterns]]
- [[27 - JSON]]
- [[29 - Object Methods]]

---

*Next: [[07 - Prototypes and Prototypal Inheritance]]*
