# Object Methods

## Overview
JavaScript provides numerous built-in methods for working with objects. These methods allow you to manipulate, analyze, and transform objects in various ways. Understanding these methods is crucial for effective object-oriented programming in JavaScript.

## Object Creation and Property Methods

### Object.create()
```javascript
// Create object with specific prototype
const personPrototype = {
    greet() {
        return `Hello, I'm ${this.name}`;
    },
    
    getFullName() {
        return `${this.firstName} ${this.lastName}`;
    }
};

// Create object using prototype
const person1 = Object.create(personPrototype);
person1.name = "John";
person1.firstName = "John";
person1.lastName = "Doe";

console.log(person1.greet()); // "Hello, I'm John"
console.log(person1.getFullName()); // "John Doe"

// Create object with properties
const person2 = Object.create(personPrototype, {
    name: {
        value: "Jane",
        writable: true,
        enumerable: true,
        configurable: true
    },
    age: {
        value: 30,
        writable: false,
        enumerable: true,
        configurable: false
    }
});

console.log(person2.name); // "Jane"
console.log(person2.age); // 30

// Create object with null prototype (no inherited methods)
const pureObject = Object.create(null);
pureObject.name = "Pure";
console.log(pureObject.toString); // undefined (no inherited methods)
```

### Object.assign()
```javascript
// Copy properties from one or more source objects to target
const target = { name: "John" };
const source1 = { age: 30, city: "New York" };
const source2 = { age: 25, country: "USA" }; // age will overwrite

const result = Object.assign(target, source1, source2);

console.log(result); // { name: "John", age: 25, city: "New York", country: "USA" }
console.log(target === result); // true (target is modified)

// Cloning objects (shallow copy)
const original = {
    name: "John",
    details: {
        age: 30,
        hobbies: ["reading", "coding"]
    }
};

const clone = Object.assign({}, original);
clone.name = "Jane"; // Won't affect original
clone.details.age = 25; // Will affect original (shallow copy)

console.log(original.name); // "John"
console.log(original.details.age); // 25 (affected by clone)

// Merging objects with default values
function createUser(userData) {
    const defaults = {
        role: "user",
        active: true,
        preferences: {
            theme: "light",
            notifications: true
        }
    };
    
    return Object.assign({}, defaults, userData);
}

const user1 = createUser({ name: "John", role: "admin" });
console.log(user1);
// { role: "admin", active: true, preferences: {...}, name: "John" }

// Converting string to object (with symbol keys)
const sym = Symbol('test');
const sourceWithSymbol = { [sym]: 'symbol value', normal: 'normal value' };
const targetObj = Object.assign({}, sourceWithSymbol);

console.log(targetObj[sym]); // 'symbol value'
console.log(targetObj.normal); // 'normal value'
```

### Object.defineProperty() and Object.defineProperties()
```javascript
const obj = {};

// Define single property with descriptor
Object.defineProperty(obj, 'name', {
    value: 'John',
    writable: true,      // Can be changed
    enumerable: true,    // Shows up in for...in loops
    configurable: true   // Can be deleted or reconfigured
});

// Define getter and setter
Object.defineProperty(obj, 'fullName', {
    get() {
        return `${this.firstName} ${this.lastName}`;
    },
    set(value) {
        [this.firstName, this.lastName] = value.split(' ');
    },
    enumerable: true,
    configurable: true
});

obj.firstName = "John";
obj.lastName = "Doe";
console.log(obj.fullName); // "John Doe"

obj.fullName = "Jane Smith";
console.log(obj.firstName); // "Jane"
console.log(obj.lastName); // "Smith"

// Define multiple properties
Object.defineProperties(obj, {
    age: {
        value: 30,
        writable: true,
        enumerable: true
    },
    email: {
        value: 'john@example.com',
        writable: false,
        enumerable: true
    },
    id: {
        value: 12345,
        writable: false,
        enumerable: false // Hidden from enumeration
    }
});

// Create computed properties
function createCounter(initialValue = 0) {
    const counter = {};
    let count = initialValue;
    
    Object.defineProperty(counter, 'value', {
        get() {
            return count;
        },
        set(newValue) {
            if (typeof newValue === 'number') {
                count = newValue;
            }
        }
    });
    
    Object.defineProperty(counter, 'increment', {
        value() {
            count++;
            return count;
        },
        writable: false,
        enumerable: false
    });
    
    Object.defineProperty(counter, 'decrement', {
        value() {
            count--;
            return count;
        },
        writable: false,
        enumerable: false
    });
    
    return counter;
}

const counter = createCounter(5);
console.log(counter.value); // 5
counter.increment();
console.log(counter.value); // 6
counter.value = 10;
console.log(counter.value); // 10
```

## Property Inspection Methods

### Object.keys(), Object.values(), Object.entries()
```javascript
const person = {
    name: "John",
    age: 30,
    city: "New York",
    country: "USA"
};

// Get all enumerable property names
const keys = Object.keys(person);
console.log(keys); // ["name", "age", "city", "country"]

// Get all enumerable property values
const values = Object.values(person);
console.log(values); // ["John", 30, "New York", "USA"]

// Get all enumerable property [key, value] pairs
const entries = Object.entries(person);
console.log(entries);
// [["name", "John"], ["age", 30], ["city", "New York"], ["country", "USA"]]

// Practical examples
// Convert object to Map
const personMap = new Map(Object.entries(person));

// Filter object properties
const filteredEntries = Object.entries(person)
    .filter(([key, value]) => typeof value === 'string');
const stringProps = Object.fromEntries(filteredEntries);
console.log(stringProps); // { name: "John", city: "New York", country: "USA" }

// Transform object values
const upperCaseValues = Object.fromEntries(
    Object.entries(person).map(([key, value]) => [
        key,
        typeof value === 'string' ? value.toUpperCase() : value
    ])
);
console.log(upperCaseValues);
// { name: "JOHN", age: 30, city: "NEW YORK", country: "USA" }

// Count properties by type
function analyzeObject(obj) {
    const analysis = {};
    
    Object.entries(obj).forEach(([key, value]) => {
        const type = typeof value;
        analysis[type] = (analysis[type] || 0) + 1;
    });
    
    return analysis;
}

console.log(analyzeObject(person)); // { string: 3, number: 1 }
```

### Object.getOwnPropertyNames() and Object.getOwnPropertySymbols()
```javascript
const obj = {
    publicProp: 'public',
    [Symbol('hidden')]: 'symbol value'
};

// Define non-enumerable property
Object.defineProperty(obj, 'hiddenProp', {
    value: 'hidden',
    enumerable: false
});

// Get enumerable properties only
console.log(Object.keys(obj)); // ["publicProp"]

// Get all own properties (including non-enumerable)
console.log(Object.getOwnPropertyNames(obj)); // ["publicProp", "hiddenProp"]

// Get symbol properties
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(hidden)]

// Get ALL own properties (strings and symbols)
function getAllOwnProperties(obj) {
    const stringProps = Object.getOwnPropertyNames(obj);
    const symbolProps = Object.getOwnPropertySymbols(obj);
    return [...stringProps, ...symbolProps];
}

console.log(getAllOwnProperties(obj)); // ["publicProp", "hiddenProp", Symbol(hidden)]

// Practical example: Deep clone with all properties
function deepCloneWithAllProps(obj) {
    if (obj === null || typeof obj !== 'object') {
        return obj;
    }
    
    if (obj instanceof Date) return new Date(obj);
    if (obj instanceof Array) return obj.map(deepCloneWithAllProps);
    
    const clone = {};
    
    // Copy all string properties
    Object.getOwnPropertyNames(obj).forEach(key => {
        const descriptor = Object.getOwnPropertyDescriptor(obj, key);
        Object.defineProperty(clone, key, {
            ...descriptor,
            value: deepCloneWithAllProps(descriptor.value)
        });
    });
    
    // Copy all symbol properties
    Object.getOwnPropertySymbols(obj).forEach(sym => {
        const descriptor = Object.getOwnPropertyDescriptor(obj, sym);
        Object.defineProperty(clone, sym, {
            ...descriptor,
            value: deepCloneWithAllProps(descriptor.value)
        });
    });
    
    return clone;
}
```

### Object.getOwnPropertyDescriptor() and Object.getOwnPropertyDescriptors()
```javascript
const obj = {
    name: "John"
};

Object.defineProperty(obj, 'age', {
    value: 30,
    writable: false,
    enumerable: true,
    configurable: false
});

// Get descriptor for single property
const nameDescriptor = Object.getOwnPropertyDescriptor(obj, 'name');
console.log(nameDescriptor);
// { value: "John", writable: true, enumerable: true, configurable: true }

const ageDescriptor = Object.getOwnPropertyDescriptor(obj, 'age');
console.log(ageDescriptor);
// { value: 30, writable: false, enumerable: true, configurable: false }

// Get descriptors for all properties
const allDescriptors = Object.getOwnPropertyDescriptors(obj);
console.log(allDescriptors);
/*
{
  name: { value: "John", writable: true, enumerable: true, configurable: true },
  age: { value: 30, writable: false, enumerable: true, configurable: false }
}
*/

// Practical example: Perfect clone preserving descriptors
function perfectClone(obj) {
    return Object.create(
        Object.getPrototypeOf(obj),
        Object.getOwnPropertyDescriptors(obj)
    );
}

const original = {};
Object.defineProperty(original, 'readOnly', {
    value: 'cannot change',
    writable: false
});

const cloned = perfectClone(original);
console.log(Object.getOwnPropertyDescriptor(cloned, 'readOnly'));
// { value: "cannot change", writable: false, enumerable: false, configurable: false }

// Property analysis utility
function analyzeProperty(obj, propName) {
    const descriptor = Object.getOwnPropertyDescriptor(obj, propName);
    
    if (!descriptor) {
        return { exists: false };
    }
    
    return {
        exists: true,
        type: descriptor.hasOwnProperty('value') ? 'data' : 'accessor',
        value: descriptor.value,
        getter: descriptor.get,
        setter: descriptor.set,
        writable: descriptor.writable,
        enumerable: descriptor.enumerable,
        configurable: descriptor.configurable
    };
}

console.log(analyzeProperty(obj, 'name'));
console.log(analyzeProperty(obj, 'nonexistent'));
```

## Object State Control Methods

### Object.freeze(), Object.seal(), Object.preventExtensions()
```javascript
const obj = {
    name: "John",
    age: 30,
    details: {
        email: "john@example.com"
    }
};

// Object.freeze() - makes object immutable
const frozenObj = Object.freeze({ ...obj });

// Cannot modify, add, or delete properties
frozenObj.name = "Jane"; // Silently fails (strict mode: throws error)
frozenObj.newProp = "new"; // Silently fails
delete frozenObj.age; // Silently fails

console.log(frozenObj.name); // "John" (unchanged)
console.log(Object.isFrozen(frozenObj)); // true

// Object.seal() - prevents adding/deleting properties but allows modification
const sealedObj = Object.seal({ ...obj });

sealedObj.name = "Jane"; // Works
sealedObj.newProp = "new"; // Silently fails
delete sealedObj.age; // Silently fails

console.log(sealedObj.name); // "Jane" (changed)
console.log(Object.isSealed(sealedObj)); // true

// Object.preventExtensions() - prevents adding new properties
const nonExtensibleObj = Object.preventExtensions({ ...obj });

nonExtensibleObj.name = "Jane"; // Works
nonExtensibleObj.newProp = "new"; // Silently fails
delete nonExtensibleObj.age; // Works

console.log(nonExtensibleObj.name); // "Jane"
console.log(nonExtensibleObj.age); // undefined (deleted)
console.log(Object.isExtensible(nonExtensibleObj)); // false

// Deep freeze utility
function deepFreeze(obj) {
    // Get property names
    Object.getOwnPropertyNames(obj).forEach(prop => {
        const value = obj[prop];
        
        // Freeze nested objects
        if (value && typeof value === 'object') {
            deepFreeze(value);
        }
    });
    
    return Object.freeze(obj);
}

const deepFrozenObj = deepFreeze({
    user: {
        name: "John",
        details: {
            age: 30
        }
    }
});

// None of these will work
deepFrozenObj.user.name = "Jane";
deepFrozenObj.user.details.age = 25;

console.log(deepFrozenObj.user.name); // "John"
console.log(deepFrozenObj.user.details.age); // 30

// Immutability checker
function checkImmutability(obj) {
    return {
        frozen: Object.isFrozen(obj),
        sealed: Object.isSealed(obj),
        extensible: Object.isExtensible(obj),
        level: Object.isFrozen(obj) ? 'frozen' : 
               Object.isSealed(obj) ? 'sealed' : 
               Object.isExtensible(obj) ? 'mutable' : 'non-extensible'
    };
}

console.log(checkImmutability(frozenObj)); // { frozen: true, sealed: true, extensible: false, level: 'frozen' }
```

## Object Comparison and Utility Methods

### Object.is()
```javascript
// Object.is() provides same-value equality comparison
console.log(Object.is(25, 25)); // true
console.log(Object.is('hello', 'hello')); // true
console.log(Object.is(NaN, NaN)); // true (different from === comparison)
console.log(Object.is(0, -0)); // false (different from === comparison)
console.log(Object.is(null, null)); // true
console.log(Object.is(undefined, undefined)); // true

// Comparison with === operator
console.log(NaN === NaN); // false
console.log(Object.is(NaN, NaN)); // true

console.log(0 === -0); // true
console.log(Object.is(0, -0)); // false

// Object comparison (reference equality)
const obj1 = { name: "John" };
const obj2 = { name: "John" };
const obj3 = obj1;

console.log(Object.is(obj1, obj2)); // false (different objects)
console.log(Object.is(obj1, obj3)); // true (same reference)

// Practical utility for deep equality checking
function deepEqual(a, b) {
    if (Object.is(a, b)) {
        return true;
    }
    
    if (a === null || b === null || typeof a !== 'object' || typeof b !== 'object') {
        return false;
    }
    
    const keysA = Object.keys(a);
    const keysB = Object.keys(b);
    
    if (keysA.length !== keysB.length) {
        return false;
    }
    
    for (const key of keysA) {
        if (!keysB.includes(key) || !deepEqual(a[key], b[key])) {
            return false;
        }
    }
    
    return true;
}

console.log(deepEqual({ a: 1, b: 2 }, { a: 1, b: 2 })); // true
console.log(deepEqual({ a: 1, b: 2 }, { a: 1, b: 3 })); // false
```

### Object.hasOwnProperty() vs Object.prototype.hasOwnProperty.call()
```javascript
const obj = {
    name: "John",
    age: 30
};

// Standard hasOwnProperty
console.log(obj.hasOwnProperty('name')); // true
console.log(obj.hasOwnProperty('toString')); // false (inherited)

// Problem: hasOwnProperty can be overridden
const problematicObj = {
    name: "John",
    hasOwnProperty: function() {
        return false; // Always returns false
    }
};

console.log(problematicObj.hasOwnProperty('name')); // false (wrong!)

// Solution: Use call method
console.log(Object.prototype.hasOwnProperty.call(problematicObj, 'name')); // true

// Modern alternative: Object.hasOwn() (ES2022)
if (Object.hasOwn) {
    console.log(Object.hasOwn(problematicObj, 'name')); // true
}

// Utility function for safe property checking
function hasOwnProperty(obj, prop) {
    return Object.prototype.hasOwnProperty.call(obj, prop);
}

// Or use the newer Object.hasOwn if available
const safeHasOwnProperty = Object.hasOwn || hasOwnProperty;

console.log(safeHasOwnProperty(problematicObj, 'name')); // true
console.log(safeHasOwnProperty(problematicObj, 'toString')); // false
```

## Object Transformation Methods

### Object.fromEntries()
```javascript
// Convert array of key-value pairs to object
const entries = [
    ['name', 'John'],
    ['age', 30],
    ['city', 'New York']
];

const obj = Object.fromEntries(entries);
console.log(obj); // { name: "John", age: 30, city: "New York" }

// Convert Map to object
const map = new Map([
    ['name', 'John'],
    ['age', 30]
]);

const objFromMap = Object.fromEntries(map);
console.log(objFromMap); // { name: "John", age: 30 }

// Practical examples
// Transform object values
const prices = { apple: 1.50, banana: 0.75, orange: 2.00 };

const discountedPrices = Object.fromEntries(
    Object.entries(prices).map(([fruit, price]) => [fruit, price * 0.9])
);
console.log(discountedPrices); // { apple: 1.35, banana: 0.675, orange: 1.8 }

// Filter object properties
const user = {
    name: "John",
    age: 30,
    password: "secret123",
    email: "john@example.com"
};

const publicUser = Object.fromEntries(
    Object.entries(user).filter(([key]) => key !== 'password')
);
console.log(publicUser); // { name: "John", age: 30, email: "john@example.com" }

// Invert object (swap keys and values)
function invertObject(obj) {
    return Object.fromEntries(
        Object.entries(obj).map(([key, value]) => [value, key])
    );
}

const colorCodes = { red: '#FF0000', green: '#00FF00', blue: '#0000FF' };
const codeColors = invertObject(colorCodes);
console.log(codeColors); // { '#FF0000': 'red', '#00FF00': 'green', '#0000FF': 'blue' }

// Group array items by property
function groupBy(array, keyFn) {
    return Object.fromEntries(
        array.reduce((groups, item) => {
            const key = keyFn(item);
            groups[key] = groups[key] || [];
            groups[key].push(item);
            return groups;
        }, {})
    );
}

const people = [
    { name: 'John', department: 'IT' },
    { name: 'Jane', department: 'HR' },
    { name: 'Bob', department: 'IT' }
];

const groupedByDept = groupBy(people, person => person.department);
console.log(groupedByDept);
// { IT: [{name: 'John', department: 'IT'}, {name: 'Bob', department: 'IT'}], HR: [{name: 'Jane', department: 'HR'}] }
```

## Prototype Methods

### Object.getPrototypeOf() and Object.setPrototypeOf()
```javascript
// Get prototype of an object
const arr = [1, 2, 3];
console.log(Object.getPrototypeOf(arr) === Array.prototype); // true

const obj = {};
console.log(Object.getPrototypeOf(obj) === Object.prototype); // true

// Create custom prototype
const animalPrototype = {
    speak() {
        return `${this.name} makes a sound`;
    },
    
    eat(food) {
        return `${this.name} eats ${food}`;
    }
};

const dog = { name: "Buddy", breed: "Golden Retriever" };

// Set prototype
Object.setPrototypeOf(dog, animalPrototype);

console.log(dog.speak()); // "Buddy makes a sound"
console.log(dog.eat("treats")); // "Buddy eats treats"

// Check prototype chain
console.log(Object.getPrototypeOf(dog) === animalPrototype); // true

// Safer way to set prototype during object creation
const cat = Object.create(animalPrototype);
cat.name = "Whiskers";
cat.breed = "Persian";

console.log(cat.speak()); // "Whiskers makes a sound"

// Prototype chain inspection
function getPrototypeChain(obj) {
    const chain = [];
    let current = obj;
    
    while (current !== null) {
        chain.push(current);
        current = Object.getPrototypeOf(current);
    }
    
    return chain;
}

console.log(getPrototypeChain(dog).length); // Number of objects in prototype chain

// Check if object is in prototype chain
function isInPrototypeChain(obj, prototype) {
    let current = Object.getPrototypeOf(obj);
    
    while (current !== null) {
        if (current === prototype) {
            return true;
        }
        current = Object.getPrototypeOf(current);
    }
    
    return false;
}

console.log(isInPrototypeChain(dog, animalPrototype)); // true
console.log(isInPrototypeChain(dog, Array.prototype)); // false
```

### Object.isPrototypeOf()
```javascript
// Check if object exists in another object's prototype chain
const vehiclePrototype = {
    start() {
        return `${this.type} is starting`;
    }
};

const carPrototype = Object.create(vehiclePrototype);
carPrototype.drive = function() {
    return `${this.type} is driving`;
};

const car = Object.create(carPrototype);
car.type = "Car";

// Check prototype relationships
console.log(vehiclePrototype.isPrototypeOf(car)); // true
console.log(carPrototype.isPrototypeOf(car)); // true
console.log(Object.prototype.isPrototypeOf(car)); // true

// Compare with instanceof
console.log(car instanceof Object); // true
// vehiclePrototype.isPrototypeOf(car) is similar to car instanceof VehicleConstructor

// Practical example: Type checking with prototypes
const shapes = {
    circle: {
        area() { return Math.PI * this.radius ** 2; },
        perimeter() { return 2 * Math.PI * this.radius; }
    },
    
    rectangle: {
        area() { return this.width * this.height; },
        perimeter() { return 2 * (this.width + this.height); }
    }
};

function createShape(type, properties) {
    if (!shapes[type]) {
        throw new Error(`Unknown shape type: ${type}`);
    }
    
    const shape = Object.create(shapes[type]);
    return Object.assign(shape, properties, { type });
}

const circle = createShape('circle', { radius: 5 });
const rectangle = createShape('rectangle', { width: 4, height: 6 });

console.log(shapes.circle.isPrototypeOf(circle)); // true
console.log(shapes.rectangle.isPrototypeOf(rectangle)); // true
console.log(shapes.circle.isPrototypeOf(rectangle)); // false

console.log(circle.area()); // 78.54
console.log(rectangle.area()); // 24
```

## Practical Object Utility Functions

### Object Merging and Cloning
```javascript
// Deep merge objects
function deepMerge(target, ...sources) {
    if (!sources.length) return target;
    const source = sources.shift();
    
    if (isObject(target) && isObject(source)) {
        for (const key in source) {
            if (isObject(source[key])) {
                if (!target[key]) Object.assign(target, { [key]: {} });
                deepMerge(target[key], source[key]);
            } else {
                Object.assign(target, { [key]: source[key] });
            }
        }
    }
    
    return deepMerge(target, ...sources);
}

function isObject(item) {
    return item && typeof item === 'object' && !Array.isArray(item);
}

// Usage
const config1 = {
    api: {
        url: 'https://api.example.com',
        timeout: 5000
    },
    features: {
        logging: true
    }
};

const config2 = {
    api: {
        timeout: 10000,
        retries: 3
    },
    features: {
        caching: true
    }
};

const mergedConfig = deepMerge({}, config1, config2);
console.log(mergedConfig);
/*
{
  api: { url: 'https://api.example.com', timeout: 10000, retries: 3 },
  features: { logging: true, caching: true }
}
*/

// Object path utilities
function getNestedValue(obj, path, defaultValue = undefined) {
    const keys = path.split('.');
    let current = obj;
    
    for (const key of keys) {
        if (current === null || current === undefined || !(key in current)) {
            return defaultValue;
        }
        current = current[key];
    }
    
    return current;
}

function setNestedValue(obj, path, value) {
    const keys = path.split('.');
    const lastKey = keys.pop();
    let current = obj;
    
    for (const key of keys) {
        if (!(key in current) || typeof current[key] !== 'object') {
            current[key] = {};
        }
        current = current[key];
    }
    
    current[lastKey] = value;
    return obj;
}

// Usage
const data = {
    user: {
        profile: {
            name: 'John'
        }
    }
};

console.log(getNestedValue(data, 'user.profile.name')); // 'John'
console.log(getNestedValue(data, 'user.settings.theme', 'light')); // 'light'

setNestedValue(data, 'user.settings.theme', 'dark');
console.log(data.user.settings.theme); // 'dark'

// Object transformation utilities
function mapObject(obj, mapFn) {
    return Object.fromEntries(
        Object.entries(obj).map(([key, value]) => mapFn(key, value))
    );
}

function filterObject(obj, predicate) {
    return Object.fromEntries(
        Object.entries(obj).filter(([key, value]) => predicate(key, value))
    );
}

function reduceObject(obj, reduceFn, initialValue) {
    return Object.entries(obj).reduce(
        (acc, [key, value]) => reduceFn(acc, key, value),
        initialValue
    );
}

// Usage examples
const numbers = { a: 1, b: 2, c: 3, d: 4 };

// Double all values
const doubled = mapObject(numbers, (key, value) => [key, value * 2]);
console.log(doubled); // { a: 2, b: 4, c: 6, d: 8 }

// Filter even values
const evens = filterObject(numbers, (key, value) => value % 2 === 0);
console.log(evens); // { b: 2, d: 4 }

// Sum all values
const sum = reduceObject(numbers, (acc, key, value) => acc + value, 0);
console.log(sum); // 10
```

---

## Related Topics
- [[06 - Objects]]
- [[07 - Prototypes and Prototypal Inheritance]]
- [[27 - JSON]]
- [[28 - Array Methods]]
- [[43 - Browser APIs]]

---

*Next: [[30 - Deep Copy vs Shallow Copy]]*
