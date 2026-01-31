# JSON (JavaScript Object Notation)

## Overview
JSON is a lightweight, text-based data interchange format. Despite its name suggesting a relationship with JavaScript, JSON is language-independent and widely used for data exchange between servers and web applications.

## JSON Syntax Rules

### Basic Structure
```javascript
// Valid JSON (as string)
const jsonString = `{
    "name": "John Doe",
    "age": 30,
    "isActive": true,
    "address": {
        "street": "123 Main St",
        "city": "New York",
        "zipCode": "10001"
    },
    "hobbies": ["reading", "coding", "hiking"],
    "spouse": null
}`;

// JSON supports these data types:
// - String (must use double quotes)
// - Number (integer or floating point)
// - Boolean (true or false)
// - null
// - Object (collection of key/value pairs)
// - Array (ordered list of values)
```

### JSON vs JavaScript Object
```javascript
// JavaScript Object
const jsObject = {
    name: 'John',           // Keys can be unquoted
    age: 30,
    sayHello() {            // Methods allowed
        return 'Hello';
    },
    createdAt: new Date(),  // Date objects allowed
    pattern: /\d+/          // Regular expressions allowed
};

// JSON (as string)
const jsonString = `{
    "name": "John",         // Keys must be quoted
    "age": 30
}`;                         // No methods, dates, regex, etc.

// JSON doesn't support:
// - Functions/methods
// - undefined
// - Date objects (must be strings)
// - Regular expressions
// - Comments
// - Trailing commas
```

## JSON Methods

### JSON.stringify()
```javascript
const person = {
    name: "John",
    age: 30,
    city: "New York",
    hobbies: ["reading", "coding"]
};

// Basic stringify
const jsonString = JSON.stringify(person);
console.log(jsonString);
// '{"name":"John","age":30,"city":"New York","hobbies":["reading","coding"]}'

// Pretty printing (with indentation)
const prettyJson = JSON.stringify(person, null, 2);
console.log(prettyJson);
/*
{
  "name": "John",
  "age": 30,
  "city": "New York",
  "hobbies": [
    "reading",
    "coding"
  ]
}
*/

// With tab indentation
const tabIndented = JSON.stringify(person, null, '\t');
```

### JSON.parse()
```javascript
const jsonString = '{"name":"John","age":30,"city":"New York"}';

// Basic parse
const person = JSON.parse(jsonString);
console.log(person.name); // "John"
console.log(typeof person); // "object"

// Parsing arrays
const jsonArray = '["apple", "banana", "orange"]';
const fruits = JSON.parse(jsonArray);
console.log(fruits[0]); // "apple"

// Parsing primitive values
console.log(JSON.parse('42')); // 42
console.log(JSON.parse('"hello"')); // "hello"
console.log(JSON.parse('true')); // true
console.log(JSON.parse('null')); // null
```

## Advanced JSON.stringify()

### Replacer Function
```javascript
const data = {
    name: "John",
    age: 30,
    password: "secret123",
    email: "john@example.com",
    internal: {
        id: 12345,
        role: "admin"
    }
};

// Filter out sensitive data
const sensitiveReplacer = (key, value) => {
    if (key === 'password') {
        return undefined; // Exclude from JSON
    }
    if (key === 'internal') {
        return '[REDACTED]'; // Replace with placeholder
    }
    return value;
};

const filteredJson = JSON.stringify(data, sensitiveReplacer, 2);
console.log(filteredJson);
/*
{
  "name": "John",
  "age": 30,
  "email": "john@example.com",
  "internal": "[REDACTED]"
}
*/

// Transform values
const transformReplacer = (key, value) => {
    if (typeof value === 'string') {
        return value.toUpperCase();
    }
    if (typeof value === 'number') {
        return value * 2;
    }
    return value;
};

const transformedJson = JSON.stringify(data, transformReplacer);
```

### Replacer Array
```javascript
const person = {
    name: "John",
    age: 30,
    email: "john@example.com",
    password: "secret",
    address: {
        street: "123 Main St",
        city: "New York"
    }
};

// Only include specific properties
const allowedFields = ['name', 'age', 'email', 'address', 'street', 'city'];
const filteredJson = JSON.stringify(person, allowedFields, 2);
console.log(filteredJson);
/*
{
  "name": "John",
  "age": 30,
  "email": "john@example.com",
  "address": {
    "street": "123 Main St",
    "city": "New York"
  }
}
*/
```

## Advanced JSON.parse()

### Reviver Function
```javascript
const jsonWithDates = `{
    "name": "John",
    "birthDate": "1990-05-15T00:00:00.000Z",
    "lastLogin": "2023-08-07T10:30:00.000Z",
    "metadata": {
        "createdAt": "2023-01-01T00:00:00.000Z"
    }
}`;

// Convert date strings to Date objects
const dateReviver = (key, value) => {
    if (typeof value === 'string' && /\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}/.test(value)) {
        return new Date(value);
    }
    return value;
};

const personWithDates = JSON.parse(jsonWithDates, dateReviver);
console.log(personWithDates.birthDate instanceof Date); // true

// Number transformation
const jsonWithNumbers = '{"price": "19.99", "quantity": "5", "name": "Product"}';

const numberReviver = (key, value) => {
    if (key === 'price' || key === 'quantity') {
        return parseFloat(value);
    }
    return value;
};

const product = JSON.parse(jsonWithNumbers, numberReviver);
console.log(typeof product.price); // "number"
```

## Custom Serialization

### toJSON() Method
```javascript
class Person {
    constructor(name, age, email) {
        this.name = name;
        this.age = age;
        this.email = email;
        this.password = 'secret'; // Private data
    }
    
    // Custom serialization
    toJSON() {
        return {
            name: this.name,
            age: this.age,
            email: this.email,
            // password is excluded
            type: 'Person'
        };
    }
}

const person = new Person("John", 30, "john@example.com");
const json = JSON.stringify(person);
console.log(json);
// '{"name":"John","age":30,"email":"john@example.com","type":"Person"}'

// Date example
class CustomDate extends Date {
    toJSON() {
        return {
            timestamp: this.getTime(),
            iso: this.toISOString(),
            formatted: this.toLocaleDateString()
        };
    }
}

const customDate = new CustomDate();
console.log(JSON.stringify(customDate));
```

### Handling Circular References
```javascript
// Object with circular reference
const obj1 = { name: "Object 1" };
const obj2 = { name: "Object 2", parent: obj1 };
obj1.child = obj2; // Circular reference

// This would throw an error:
// JSON.stringify(obj1); // TypeError: Converting circular structure to JSON

// Solution 1: Replacer function to handle circular references
const seen = new WeakSet();
const circularReplacer = (key, value) => {
    if (typeof value === "object" && value !== null) {
        if (seen.has(value)) {
            return "[Circular Reference]";
        }
        seen.add(value);
    }
    return value;
};

const jsonWithCircular = JSON.stringify(obj1, circularReplacer);
console.log(jsonWithCircular);

// Solution 2: Remove circular references
function removeCircular(obj, seen = new WeakSet()) {
    if (obj && typeof obj === 'object') {
        if (seen.has(obj)) {
            return {}; // or return null, or "[Circular]"
        }
        seen.add(obj);
        
        if (Array.isArray(obj)) {
            return obj.map(item => removeCircular(item, seen));
        } else {
            const result = {};
            for (const key in obj) {
                if (obj.hasOwnProperty(key)) {
                    result[key] = removeCircular(obj[key], seen);
                }
            }
            return result;
        }
    }
    return obj;
}

const cleanObj = removeCircular(obj1);
const cleanJson = JSON.stringify(cleanObj);
```

## Error Handling

### JSON.parse() Errors
```javascript
function safeJSONParse(jsonString, defaultValue = null) {
    try {
        return JSON.parse(jsonString);
    } catch (error) {
        console.error('JSON parsing failed:', error.message);
        return defaultValue;
    }
}

// Test cases
console.log(safeJSONParse('{"name": "John"}')); // {name: "John"}
console.log(safeJSONParse('invalid json')); // null
console.log(safeJSONParse('undefined', {})); // {}

// More detailed error handling
function parseJSONWithDetails(jsonString) {
    try {
        return {
            success: true,
            data: JSON.parse(jsonString),
            error: null
        };
    } catch (error) {
        return {
            success: false,
            data: null,
            error: {
                message: error.message,
                type: error.name,
                position: extractErrorPosition(error.message)
            }
        };
    }
}

function extractErrorPosition(errorMessage) {
    const match = errorMessage.match(/position (\d+)/);
    return match ? parseInt(match[1]) : null;
}
```

### JSON.stringify() Errors
```javascript
function safeJSONStringify(obj, replacer = null, space = null, defaultValue = '{}') {
    try {
        return JSON.stringify(obj, replacer, space);
    } catch (error) {
        console.error('JSON stringification failed:', error.message);
        
        if (error.message.includes('circular structure')) {
            // Handle circular references
            return JSON.stringify(obj, circularReplacer, space);
        }
        
        return defaultValue;
    }
}

// Test with problematic data
const problematicObj = {
    func: function() {}, // Functions are ignored
    undef: undefined,    // undefined is ignored
    symbol: Symbol('test') // Symbols are ignored
};

console.log(safeJSONStringify(problematicObj));
```

## Working with Large JSON Data

### Streaming JSON
```javascript
// For large JSON data, consider streaming approaches
async function processLargeJSON(jsonString) {
    // Parse in chunks if possible
    const chunkSize = 1000;
    
    if (jsonString.length < chunkSize) {
        return JSON.parse(jsonString);
    }
    
    // For arrays, we could process incrementally
    // This is a simplified example
    try {
        return JSON.parse(jsonString);
    } catch (error) {
        console.error('Large JSON parsing failed:', error.message);
        throw new Error('JSON too large or malformed');
    }
}

// Memory-efficient array processing
function processJSONArray(jsonArray, processor) {
    return jsonArray.map(processor);
}

// Example: Process users from large JSON
const largeUserArray = [
    {id: 1, name: "User1", data: "..."},
    {id: 2, name: "User2", data: "..."},
    // ... thousands more
];

const processedUsers = processJSONArray(largeUserArray, user => ({
    id: user.id,
    name: user.name.toUpperCase(),
    processed: true
}));
```

## JSON Validation

### Schema Validation
```javascript
// Simple JSON schema validation
function validateUserJSON(data) {
    const errors = [];
    
    if (typeof data !== 'object' || data === null) {
        errors.push('Data must be an object');
        return errors;
    }
    
    // Required fields
    const requiredFields = ['name', 'email', 'age'];
    requiredFields.forEach(field => {
        if (!(field in data)) {
            errors.push(`Missing required field: ${field}`);
        }
    });
    
    // Type validation
    if ('name' in data && typeof data.name !== 'string') {
        errors.push('Name must be a string');
    }
    
    if ('age' in data && (typeof data.age !== 'number' || data.age < 0)) {
        errors.push('Age must be a positive number');
    }
    
    if ('email' in data && typeof data.email !== 'string') {
        errors.push('Email must be a string');
    }
    
    return errors;
}

// Usage
const userData = {
    name: "John",
    email: "john@example.com",
    age: 30
};

const validationErrors = validateUserJSON(userData);
if (validationErrors.length > 0) {
    console.error('Validation failed:', validationErrors);
} else {
    console.log('Data is valid');
}
```

## Practical Examples

### API Data Processing
```javascript
class APIDataProcessor {
    static processUserData(jsonResponse) {
        try {
            const data = typeof jsonResponse === 'string' 
                ? JSON.parse(jsonResponse) 
                : jsonResponse;
            
            return {
                id: data.id,
                name: data.name || 'Unknown',
                email: data.email || '',
                age: parseInt(data.age) || 0,
                lastLogin: data.lastLogin ? new Date(data.lastLogin) : null,
                preferences: data.preferences || {},
                isActive: Boolean(data.isActive)
            };
        } catch (error) {
            throw new Error(`Failed to process user data: ${error.message}`);
        }
    }
    
    static serializeUserData(userData) {
        return JSON.stringify({
            id: userData.id,
            name: userData.name,
            email: userData.email,
            age: userData.age,
            lastLogin: userData.lastLogin?.toISOString(),
            preferences: userData.preferences,
            isActive: userData.isActive
        }, null, 2);
    }
}

// Usage
const rawUserData = `{
    "id": "123",
    "name": "John Doe",
    "email": "john@example.com",
    "age": "30",
    "lastLogin": "2023-08-07T10:30:00.000Z",
    "isActive": "true"
}`;

const processedUser = APIDataProcessor.processUserData(rawUserData);
const serializedUser = APIDataProcessor.serializeUserData(processedUser);
```

### Configuration Management
```javascript
class ConfigManager {
    constructor(defaultConfig = {}) {
        this.defaultConfig = defaultConfig;
        this.config = { ...defaultConfig };
    }
    
    loadFromJSON(jsonString) {
        try {
            const loadedConfig = JSON.parse(jsonString);
            this.config = { ...this.defaultConfig, ...loadedConfig };
            return true;
        } catch (error) {
            console.error('Failed to load config:', error.message);
            return false;
        }
    }
    
    saveToJSON() {
        return JSON.stringify(this.config, null, 2);
    }
    
    get(key) {
        return this.config[key];
    }
    
    set(key, value) {
        this.config[key] = value;
    }
    
    reset() {
        this.config = { ...this.defaultConfig };
    }
}

// Usage
const configManager = new ConfigManager({
    theme: 'light',
    language: 'en',
    autoSave: true
});

const configJSON = '{"theme": "dark", "language": "es"}';
configManager.loadFromJSON(configJSON);

console.log(configManager.get('theme')); // "dark"
console.log(configManager.saveToJSON());
```

### Local Storage with JSON
```javascript
class JSONStorage {
    static save(key, data) {
        try {
            const jsonString = JSON.stringify(data);
            localStorage.setItem(key, jsonString);
            return true;
        } catch (error) {
            console.error('Failed to save to localStorage:', error.message);
            return false;
        }
    }
    
    static load(key, defaultValue = null) {
        try {
            const jsonString = localStorage.getItem(key);
            return jsonString ? JSON.parse(jsonString) : defaultValue;
        } catch (error) {
            console.error('Failed to load from localStorage:', error.message);
            return defaultValue;
        }
    }
    
    static remove(key) {
        localStorage.removeItem(key);
    }
    
    static clear() {
        localStorage.clear();
    }
}

// Usage
const userData = {
    name: "John",
    preferences: { theme: "dark" },
    lastLogin: new Date()
};

JSONStorage.save('user', userData);
const loadedUser = JSONStorage.load('user', {});
```

---

## Related Topics
- [[01 - Variables and Data Types]]
- [[06 - Objects]]
- [[24 - Local Storage]]
- [[22 - Fetch API]]
- [[42 - Error Handling]]

---

*Next: [[24 - Local Storage]]*
