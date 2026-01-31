# Deep Copy vs Shallow Copy

## Overview
Understanding the difference between shallow copy and deep copy is crucial in JavaScript programming. This concept affects how objects and arrays are copied, modified, and referenced, impacting data integrity and preventing unintended side effects in your applications.

## What is Copying?

### Reference vs Value
```javascript
// Primitive values are copied by value
let a = 5;
let b = a; // b gets a copy of a's value
a = 10;
console.log(a); // 10
console.log(b); // 5 (unchanged)

// Objects are copied by reference
let obj1 = { name: 'John', age: 30 };
let obj2 = obj1; // obj2 references the same object as obj1
obj1.age = 35;
console.log(obj1.age); // 35
console.log(obj2.age); // 35 (changed because they reference the same object)

// Arrays are also objects, so they're copied by reference
let arr1 = [1, 2, 3];
let arr2 = arr1; // arr2 references the same array as arr1
arr1.push(4);
console.log(arr1); // [1, 2, 3, 4]
console.log(arr2); // [1, 2, 3, 4] (changed)

// Function parameters and object references
function modifyObject(obj) {
    obj.modified = true;
    return obj;
}

let originalObj = { value: 42 };
let result = modifyObject(originalObj);
console.log(originalObj.modified); // true (original object was modified)
console.log(result === originalObj); // true (same reference)
```

## Shallow Copy

### Definition and Characteristics
A shallow copy creates a new object, but nested objects are still referenced from the original object.

```javascript
// Shallow copy examples
const original = {
    name: 'John',
    age: 30,
    address: {
        street: '123 Main St',
        city: 'New York',
        coordinates: {
            lat: 40.7128,
            lng: -74.0060
        }
    },
    hobbies: ['reading', 'swimming']
};

// Method 1: Object.assign()
const shallowCopy1 = Object.assign({}, original);

// Method 2: Spread operator (ES6+)
const shallowCopy2 = { ...original };

// Method 3: Array spread for arrays
const originalArray = [1, 2, [3, 4], { a: 5 }];
const shallowArrayCopy = [...originalArray];

// Test shallow copy behavior
shallowCopy1.name = 'Jane'; // Safe - primitive value
shallowCopy1.address.city = 'Boston'; // Dangerous - modifies original!

console.log(original.name); // 'John' (unchanged)
console.log(original.address.city); // 'Boston' (changed!)
console.log(shallowCopy2.address.city); // 'Boston' (all copies affected)

// Array shallow copy issues
shallowArrayCopy[0] = 10; // Safe - primitive
shallowArrayCopy[2][0] = 30; // Dangerous - modifies original array
shallowArrayCopy[3].a = 50; // Dangerous - modifies original object

console.log(originalArray); // [1, 2, [30, 4], { a: 50 }] - modified!
```

### Shallow Copy Methods

#### Object.assign()
```javascript
// Object.assign() creates shallow copy
const source = {
    name: 'John',
    details: { age: 30, city: 'NYC' },
    tags: ['developer', 'javascript']
};

const copy = Object.assign({}, source);

// Can also merge multiple objects
const additional = { role: 'senior' };
const merged = Object.assign({}, source, additional);

// Modify copy
copy.name = 'Jane'; // Safe
copy.details.age = 35; // Affects original!
copy.tags.push('react'); // Affects original!

console.log(source.details.age); // 35
console.log(source.tags); // ['developer', 'javascript', 'react']

// Object.assign() with arrays
const originalArr = [{ id: 1, name: 'Item 1' }, { id: 2, name: 'Item 2' }];
const assignedArr = Object.assign([], originalArr);

assignedArr[0].name = 'Modified Item'; // Affects original!
console.log(originalArr[0].name); // 'Modified Item'
```

#### Spread Operator
```javascript
// Spread operator for objects
const person = {
    firstName: 'John',
    lastName: 'Doe',
    contact: {
        email: 'john@example.com',
        phone: '555-1234'
    },
    preferences: ['dark-mode', 'notifications']
};

const personCopy = { ...person };

// Safe modifications
personCopy.firstName = 'Jane';
personCopy.newField = 'added';

// Unsafe modifications (affects original)
personCopy.contact.email = 'jane@example.com';
personCopy.preferences.push('auto-save');

console.log(person.contact.email); // 'jane@example.com' (changed!)
console.log(person.preferences); // ['dark-mode', 'notifications', 'auto-save']

// Spread operator for arrays
const numbers = [1, [2, 3], { value: 4 }];
const numbersCopy = [...numbers];

numbersCopy[0] = 10; // Safe
numbersCopy[1][0] = 20; // Affects original!
numbersCopy[2].value = 40; // Affects original!

console.log(numbers); // [1, [20, 3], { value: 40 }]

// Combining spread with modifications
const user = { id: 1, name: 'John', settings: { theme: 'light' } };
const updatedUser = {
    ...user,
    name: 'John Doe', // Override
    lastLogin: new Date(), // Add new field
    settings: {
        ...user.settings, // Shallow copy of nested object
        theme: 'dark' // Override nested field
    }
};

console.log(user.settings.theme); // 'light' (unchanged)
console.log(updatedUser.settings.theme); // 'dark'
```

#### Array Methods for Shallow Copy
```javascript
// Array.from()
const original = [1, { a: 2 }, [3, 4]];
const copy1 = Array.from(original);

// slice()
const copy2 = original.slice();

// concat()
const copy3 = [].concat(original);

// Spread operator
const copy4 = [...original];

// All create shallow copies
copy1[1].a = 20; // Affects all copies and original
console.log(original[1].a); // 20

// Array destructuring with rest
const [first, second, ...rest] = original;
const reconstructed = [first, second, ...rest]; // Still shallow copy

// Array.slice() with modification
const largeArray = [
    { id: 1, data: [1, 2, 3] },
    { id: 2, data: [4, 5, 6] },
    { id: 3, data: [7, 8, 9] }
];

const partialCopy = largeArray.slice(0, 2); // Copy first 2 elements
partialCopy[0].data.push(10); // Affects original!

console.log(largeArray[0].data); // [1, 2, 3, 10]
```

## Deep Copy

### Definition and Characteristics
A deep copy creates a completely independent copy of an object, including all nested objects and arrays.

```javascript
// Deep copy example
const originalComplex = {
    id: 1,
    name: 'John',
    address: {
        street: '123 Main St',
        city: 'NYC',
        coordinates: {
            lat: 40.7128,
            lng: -74.0060
        }
    },
    hobbies: ['reading', 'swimming'],
    friends: [
        { name: 'Alice', age: 25 },
        { name: 'Bob', age: 30 }
    ],
    metadata: {
        created: new Date(),
        tags: ['employee', 'senior']
    }
};

// After deep copy, modifications won't affect original
const deepCopy = createDeepCopy(originalComplex);
deepCopy.address.city = 'Boston';
deepCopy.friends[0].name = 'Alicia';
deepCopy.hobbies.push('cycling');

console.log(originalComplex.address.city); // 'NYC' (unchanged)
console.log(originalComplex.friends[0].name); // 'Alice' (unchanged)
console.log(originalComplex.hobbies.length); // 2 (unchanged)
```

### Deep Copy Methods

#### JSON.parse(JSON.stringify()) - Simple but Limited
```javascript
// JSON method - quick but has limitations
const original = {
    name: 'John',
    age: 30,
    address: {
        street: '123 Main St',
        city: 'NYC'
    },
    hobbies: ['reading', 'swimming']
};

const deepCopy = JSON.parse(JSON.stringify(original));

// Test independence
deepCopy.address.city = 'Boston';
deepCopy.hobbies.push('cycling');

console.log(original.address.city); // 'NYC' (unchanged)
console.log(original.hobbies); // ['reading', 'swimming'] (unchanged)

// Limitations of JSON method
const problematicObject = {
    date: new Date(),
    regex: /hello/g,
    fn: function() { return 'hello'; },
    undef: undefined,
    sym: Symbol('test'),
    circular: null
};

// Create circular reference
problematicObject.circular = problematicObject;

try {
    // This will fail due to circular reference
    const copy = JSON.parse(JSON.stringify(problematicObject));
} catch (error) {
    console.error('JSON method failed:', error.message);
}

// What gets lost/converted:
const testObj = {
    date: new Date('2023-01-01'),
    regex: /test/gi,
    fn: () => 'hello',
    undef: undefined,
    nullVal: null,
    num: 42,
    bool: true,
    str: 'test'
};

const jsonCopy = JSON.parse(JSON.stringify(testObj));
console.log(jsonCopy);
// Result: {
//   date: "2023-01-01T00:00:00.000Z", // String, not Date
//   nullVal: null,
//   num: 42,
//   bool: true,
//   str: "test"
//   // regex, fn, undef are missing!
// }
```

#### Custom Deep Copy Implementation
```javascript
// Comprehensive deep copy function
function deepCopy(obj, visited = new WeakMap()) {
    // Handle null and non-objects
    if (obj === null || typeof obj !== 'object') {
        return obj;
    }
    
    // Handle circular references
    if (visited.has(obj)) {
        return visited.get(obj);
    }
    
    // Handle Date objects
    if (obj instanceof Date) {
        return new Date(obj.getTime());
    }
    
    // Handle RegExp objects
    if (obj instanceof RegExp) {
        return new RegExp(obj);
    }
    
    // Handle Arrays
    if (Array.isArray(obj)) {
        const arrCopy = [];
        visited.set(obj, arrCopy);
        for (let i = 0; i < obj.length; i++) {
            arrCopy[i] = deepCopy(obj[i], visited);
        }
        return arrCopy;
    }
    
    // Handle Maps
    if (obj instanceof Map) {
        const mapCopy = new Map();
        visited.set(obj, mapCopy);
        for (const [key, value] of obj) {
            mapCopy.set(deepCopy(key, visited), deepCopy(value, visited));
        }
        return mapCopy;
    }
    
    // Handle Sets
    if (obj instanceof Set) {
        const setCopy = new Set();
        visited.set(obj, setCopy);
        for (const value of obj) {
            setCopy.add(deepCopy(value, visited));
        }
        return setCopy;
    }
    
    // Handle plain objects
    const objCopy = {};
    visited.set(obj, objCopy);
    
    for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
            objCopy[key] = deepCopy(obj[key], visited);
        }
    }
    
    return objCopy;
}

// Test comprehensive deep copy
const complexObj = {
    str: 'hello',
    num: 42,
    bool: true,
    nullVal: null,
    undef: undefined,
    date: new Date(),
    regex: /test/gi,
    arr: [1, 2, { nested: 'value' }],
    map: new Map([['key1', 'value1'], ['key2', { nested: true }]]),
    set: new Set([1, 2, 3, { unique: 'value' }]),
    nested: {
        deep: {
            deeper: {
                value: 'found me!'
            }
        }
    }
};

// Add circular reference
complexObj.self = complexObj;

const copied = deepCopy(complexObj);

// Test modifications
copied.str = 'modified';
copied.arr[2].nested = 'changed';
copied.nested.deep.deeper.value = 'modified deep';
copied.map.get('key2').nested = false;

console.log(complexObj.str); // 'hello' (unchanged)
console.log(complexObj.arr[2].nested); // 'value' (unchanged)
console.log(complexObj.nested.deep.deeper.value); // 'found me!' (unchanged)
console.log(complexObj.map.get('key2').nested); // true (unchanged)
```

#### Using Lodash for Deep Copy
```javascript
// Lodash provides robust deep copy functionality
// npm install lodash

const _ = require('lodash'); // Node.js
// or
// import _ from 'lodash'; // ES6 modules
// or
// <script src="https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js"></script>

const original = {
    name: 'John',
    age: 30,
    address: {
        street: '123 Main St',
        city: 'NYC',
        coordinates: { lat: 40.7128, lng: -74.0060 }
    },
    hobbies: ['reading', 'swimming'],
    metadata: {
        created: new Date(),
        updated: new Date()
    }
};

// Lodash deep clone
const lodashCopy = _.cloneDeep(original);

// Test independence
lodashCopy.address.coordinates.lat = 41.0000;
lodashCopy.hobbies.push('cycling');
lodashCopy.metadata.created.setFullYear(2020);

console.log(original.address.coordinates.lat); // 40.7128 (unchanged)
console.log(original.hobbies.length); // 2 (unchanged)
console.log(original.metadata.created.getFullYear()); // Current year (unchanged)

// Lodash handles complex cases
const complexLodashTest = {
    func: function() { return 'hello'; },
    regex: /test/gi,
    date: new Date(),
    map: new Map([['key', 'value']]),
    set: new Set([1, 2, 3])
};

const lodashComplexCopy = _.cloneDeep(complexLodashTest);
console.log(typeof lodashComplexCopy.func); // 'function'
console.log(lodashComplexCopy.regex instanceof RegExp); // true
console.log(lodashComplexCopy.date instanceof Date); // true
```

#### Structured Clone Algorithm (Modern Browsers)
```javascript
// Modern browsers support structuredClone (2022+)
if (typeof structuredClone !== 'undefined') {
    const original = {
        name: 'John',
        date: new Date(),
        regex: /test/gi,
        array: [1, { nested: true }],
        map: new Map([['key', 'value']]),
        set: new Set([1, 2, 3]),
        typedArray: new Uint8Array([1, 2, 3])
    };
    
    const cloned = structuredClone(original);
    
    // Test independence
    cloned.array[1].nested = false;
    cloned.map.set('key', 'modified');
    
    console.log(original.array[1].nested); // true (unchanged)
    console.log(original.map.get('key')); // 'value' (unchanged)
    
    // Limitations: functions are not cloneable
    try {
        const withFunction = { fn: () => 'hello' };
        structuredClone(withFunction); // Throws error
    } catch (error) {
        console.error('structuredClone cannot clone functions:', error.message);
    }
}

// Polyfill check and fallback
function robustDeepCopy(obj) {
    if (typeof structuredClone !== 'undefined') {
        try {
            return structuredClone(obj);
        } catch (error) {
            console.warn('structuredClone failed, falling back to custom implementation');
        }
    }
    
    return deepCopy(obj);
}
```

## Performance Considerations

### Benchmarking Copy Methods
```javascript
// Performance comparison function
function benchmarkCopyMethods(obj, iterations = 10000) {
    console.log(`Benchmarking with ${iterations} iterations`);
    
    // JSON method
    console.time('JSON.parse(JSON.stringify())');
    for (let i = 0; i < iterations; i++) {
        JSON.parse(JSON.stringify(obj));
    }
    console.timeEnd('JSON.parse(JSON.stringify())');
    
    // Custom deep copy
    console.time('Custom deepCopy');
    for (let i = 0; i < iterations; i++) {
        deepCopy(obj);
    }
    console.timeEnd('Custom deepCopy');
    
    // Shallow copy (spread)
    console.time('Shallow copy (spread)');
    for (let i = 0; i < iterations; i++) {
        { ...obj };
    }
    console.timeEnd('Shallow copy (spread)');
    
    // Shallow copy (Object.assign)
    console.time('Object.assign');
    for (let i = 0; i < iterations; i++) {
        Object.assign({}, obj);
    }
    console.timeEnd('Object.assign');
    
    // structuredClone (if available)
    if (typeof structuredClone !== 'undefined') {
        console.time('structuredClone');
        for (let i = 0; i < iterations; i++) {
            structuredClone(obj);
        }
        console.timeEnd('structuredClone');
    }
}

// Test objects of different complexity
const simpleObj = { a: 1, b: 'test', c: true };
const complexObj = {
    users: Array(100).fill().map((_, i) => ({
        id: i,
        name: `User ${i}`,
        details: { age: 20 + i, active: i % 2 === 0 }
    })),
    metadata: { created: new Date(), version: '1.0' }
};

console.log('Simple object:');
benchmarkCopyMethods(simpleObj);

console.log('\nComplex object:');
benchmarkCopyMethods(complexObj, 1000); // Fewer iterations for complex object
```

### Memory-Efficient Deep Copy
```javascript
// Memory-efficient deep copy for large objects
function efficientDeepCopy(obj, maxDepth = 10, currentDepth = 0) {
    if (currentDepth >= maxDepth) {
        console.warn('Maximum depth reached, returning shallow copy');
        return obj;
    }
    
    if (obj === null || typeof obj !== 'object') {
        return obj;
    }
    
    if (obj instanceof Date) {
        return new Date(obj.getTime());
    }
    
    if (Array.isArray(obj)) {
        return obj.map(item => efficientDeepCopy(item, maxDepth, currentDepth + 1));
    }
    
    const result = {};
    for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
            result[key] = efficientDeepCopy(obj[key], maxDepth, currentDepth + 1);
        }
    }
    
    return result;
}

// Lazy deep copy - only copy when accessed
function createLazyDeepCopy(obj) {
    const cache = new Map();
    
    return new Proxy(obj, {
        get(target, prop) {
            if (!cache.has(prop)) {
                const value = target[prop];
                if (value && typeof value === 'object') {
                    cache.set(prop, deepCopy(value));
                } else {
                    cache.set(prop, value);
                }
            }
            return cache.get(prop);
        },
        
        set(target, prop, value) {
            cache.set(prop, value);
            return true;
        }
    });
}

// Usage
const largeObject = {
    data: Array(1000).fill().map((_, i) => ({ id: i, value: Math.random() })),
    config: { theme: 'dark', language: 'en' }
};

const lazyCopy = createLazyDeepCopy(largeObject);
// Only 'config' is deep copied when accessed
console.log(lazyCopy.config.theme); // Triggers deep copy of config only
```

## Practical Use Cases

### State Management in React
```javascript
// Immutable state updates in React
function useImmutableState(initialState) {
    const [state, setState] = React.useState(initialState);
    
    const updateState = (updates) => {
        setState(prevState => {
            // Deep merge updates with previous state
            return deepMerge(prevState, updates);
        });
    };
    
    const updateNestedState = (path, value) => {
        setState(prevState => {
            const newState = deepCopy(prevState);
            setNestedValue(newState, path, value);
            return newState;
        });
    };
    
    return [state, updateState, updateNestedState];
}

function deepMerge(target, source) {
    const result = deepCopy(target);
    
    for (const key in source) {
        if (source.hasOwnProperty(key)) {
            if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
                result[key] = deepMerge(result[key] || {}, source[key]);
            } else {
                result[key] = source[key];
            }
        }
    }
    
    return result;
}

function setNestedValue(obj, path, value) {
    const keys = path.split('.');
    let current = obj;
    
    for (let i = 0; i < keys.length - 1; i++) {
        const key = keys[i];
        if (!current[key] || typeof current[key] !== 'object') {
            current[key] = {};
        }
        current = current[key];
    }
    
    current[keys[keys.length - 1]] = value;
}

// Example usage in React component
function UserProfile() {
    const [userState, updateUserState, updateNestedUserState] = useImmutableState({
        personal: {
            name: 'John Doe',
            email: 'john@example.com'
        },
        preferences: {
            theme: 'light',
            notifications: true
        },
        history: []
    });
    
    const handleNameChange = (newName) => {
        updateNestedUserState('personal.name', newName);
    };
    
    const handleThemeToggle = () => {
        const newTheme = userState.preferences.theme === 'light' ? 'dark' : 'light';
        updateNestedUserState('preferences.theme', newTheme);
    };
    
    const addHistoryItem = (item) => {
        updateUserState({
            history: [...userState.history, item]
        });
    };
    
    return (
        <div>
            <h1>{userState.personal.name}</h1>
            <button onClick={() => handleNameChange('Jane Doe')}>
                Change Name
            </button>
            <button onClick={handleThemeToggle}>
                Toggle Theme ({userState.preferences.theme})
            </button>
        </div>
    );
}
```

### API Response Caching
```javascript
// API cache with deep copy to prevent mutation
class APICache {
    constructor() {
        this.cache = new Map();
        this.timestamps = new Map();
        this.ttl = 5 * 60 * 1000; // 5 minutes TTL
    }
    
    set(key, data) {
        // Store deep copy to prevent external mutations
        this.cache.set(key, deepCopy(data));
        this.timestamps.set(key, Date.now());
    }
    
    get(key) {
        if (!this.cache.has(key)) {
            return null;
        }
        
        // Check if data has expired
        const timestamp = this.timestamps.get(key);
        if (Date.now() - timestamp > this.ttl) {
            this.cache.delete(key);
            this.timestamps.delete(key);
            return null;
        }
        
        // Return deep copy to prevent external mutations
        return deepCopy(this.cache.get(key));
    }
    
    has(key) {
        return this.cache.has(key) && 
               Date.now() - this.timestamps.get(key) <= this.ttl;
    }
    
    clear() {
        this.cache.clear();
        this.timestamps.clear();
    }
}

// Usage with API calls
const apiCache = new APICache();

async function fetchUserData(userId) {
    const cacheKey = `user_${userId}`;
    
    // Check cache first
    if (apiCache.has(cacheKey)) {
        console.log('Returning cached data');
        return apiCache.get(cacheKey);
    }
    
    // Fetch from API
    try {
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        
        // Cache the response
        apiCache.set(cacheKey, userData);
        
        // Return copy to prevent mutations
        return deepCopy(userData);
    } catch (error) {
        console.error('Failed to fetch user data:', error);
        throw error;
    }
}

// Safe to modify returned data without affecting cache
const user = await fetchUserData(123);
user.name = 'Modified Name'; // Won't affect cached data
```

### Configuration Management
```javascript
// Configuration manager with deep copy protection
class ConfigurationManager {
    constructor(defaultConfig = {}) {
        this.defaultConfig = deepCopy(defaultConfig);
        this.userConfig = {};
        this.mergedConfig = null;
        this.configHistory = [];
    }
    
    setUserConfig(config) {
        // Store previous config for rollback
        this.configHistory.push(deepCopy(this.userConfig));
        
        // Set new user config (deep copy to prevent external mutations)
        this.userConfig = deepCopy(config);
        
        // Invalidate merged config cache
        this.mergedConfig = null;
    }
    
    updateUserConfig(path, value) {
        // Store current state for rollback
        this.configHistory.push(deepCopy(this.userConfig));
        
        // Update specific path
        const newConfig = deepCopy(this.userConfig);
        setNestedValue(newConfig, path, value);
        this.userConfig = newConfig;
        
        // Invalidate merged config cache
        this.mergedConfig = null;
    }
    
    getConfig() {
        if (!this.mergedConfig) {
            this.mergedConfig = deepMerge(this.defaultConfig, this.userConfig);
        }
        
        // Return deep copy to prevent external mutations
        return deepCopy(this.mergedConfig);
    }
    
    rollback() {
        if (this.configHistory.length > 0) {
            this.userConfig = this.configHistory.pop();
            this.mergedConfig = null;
            return true;
        }
        return false;
    }
    
    reset() {
        this.userConfig = {};
        this.mergedConfig = null;
        this.configHistory = [];
    }
    
    exportConfig() {
        return {
            default: deepCopy(this.defaultConfig),
            user: deepCopy(this.userConfig),
            merged: this.getConfig()
        };
    }
}

// Usage
const configManager = new ConfigurationManager({
    app: {
        name: 'MyApp',
        version: '1.0.0',
        features: {
            darkMode: false,
            notifications: true
        }
    },
    api: {
        baseUrl: 'https://api.example.com',
        timeout: 5000
    }
});

// User customizations
configManager.updateUserConfig('app.features.darkMode', true);
configManager.updateUserConfig('api.timeout', 10000);

// Get final configuration (safe to modify)
const config = configManager.getConfig();
console.log(config.app.features.darkMode); // true
console.log(config.api.timeout); // 10000

// Modifications won't affect internal config
config.app.name = 'Modified Name';
console.log(configManager.getConfig().app.name); // 'MyApp' (unchanged)

// Rollback capability
configManager.rollback(); // Reverts timeout change
console.log(configManager.getConfig().api.timeout); // 5000
```

## Best Practices

### When to Use Shallow vs Deep Copy
```javascript
// Guidelines for choosing copy method

// Use shallow copy when:
// 1. Object has only primitive values
const userPreferences = {
    theme: 'dark',
    language: 'en',
    fontSize: 14,
    autoSave: true
};
const preferencesCopy = { ...userPreferences }; // Shallow copy is sufficient

// 2. You only need to modify top-level properties
const apiResponse = {
    data: [...], // Don't need to modify array contents
    status: 200,
    message: 'Success'
};
const modifiedResponse = {
    ...apiResponse,
    status: 201, // Only changing top-level property
    processedAt: new Date()
};

// Use deep copy when:
// 1. Object has nested objects/arrays that will be modified
const complexUserData = {
    profile: { name: 'John', settings: { theme: 'dark' } },
    posts: [{ id: 1, content: 'Hello' }]
};
const userDataCopy = deepCopy(complexUserData); // Need deep copy
userDataCopy.posts[0].content = 'Modified'; // Safe modification

// 2. Storing data in cache or state management
function storeInCache(key, data) {
    // Always deep copy to prevent external mutations
    cache.set(key, deepCopy(data));
}

// 3. Creating immutable updates
function updateUserProfile(user, updates) {
    return deepMerge(user, updates); // Deep merge for nested updates
}

// Performance vs Safety trade-offs
class DataManager {
    constructor() {
        this.data = {};
        this.performanceMode = false;
    }
    
    setPerformanceMode(enabled) {
        this.performanceMode = enabled;
    }
    
    getData(key) {
        const data = this.data[key];
        if (!data) return null;
        
        // Use shallow copy in performance mode, deep copy in safety mode
        return this.performanceMode ? { ...data } : deepCopy(data);
    }
    
    setData(key, value) {
        // Always store deep copy to prevent mutations
        this.data[key] = deepCopy(value);
    }
}
```

### Common Pitfalls and Solutions
```javascript
// Pitfall 1: Assuming spread operator does deep copy
const problematicCode = {
    user: {
        name: 'John',
        preferences: { theme: 'light' }
    }
};

const copy = { ...problematicCode }; // Shallow copy!
copy.user.preferences.theme = 'dark'; // Modifies original!

// Solution: Deep copy nested objects
const safeCopy = {
    ...problematicCode,
    user: {
        ...problematicCode.user,
        preferences: { ...problematicCode.user.preferences }
    }
};

// Pitfall 2: JSON method losing data types
const dataWithTypes = {
    date: new Date(),
    fn: () => 'hello',
    regex: /test/,
    undefined: undefined
};

const jsonCopy = JSON.parse(JSON.stringify(dataWithTypes));
// Lost: function, undefined, regex becomes object, date becomes string

// Solution: Use proper deep copy for complex types
const properCopy = deepCopy(dataWithTypes);

// Pitfall 3: Circular references breaking JSON method
const circular = { a: 1 };
circular.self = circular;

try {
    JSON.parse(JSON.stringify(circular)); // Throws error
} catch (error) {
    console.error('Circular reference error');
}

// Solution: Use deep copy with circular reference handling
const circularCopy = deepCopy(circular); // Works correctly

// Pitfall 4: Modifying arrays in loops
const items = [
    { id: 1, selected: false },
    { id: 2, selected: false }
];

// Wrong way - modifies original
const wrongCopy = [...items];
wrongCopy.forEach(item => item.selected = true); // Modifies original items!

// Right way - deep copy array elements
const rightCopy = items.map(item => ({ ...item }));
rightCopy.forEach(item => item.selected = true); // Safe

// Pitfall 5: Sharing state between components
class ComponentA {
    constructor(sharedState) {
        this.state = sharedState; // Dangerous - shared reference
    }
    
    updateState(newData) {
        Object.assign(this.state, newData); // Affects all components!
    }
}

class ComponentB {
    constructor(sharedState) {
        this.state = sharedState; // Same reference as ComponentA
    }
}

// Solution: Each component gets its own copy
class SafeComponentA {
    constructor(initialState) {
        this.state = deepCopy(initialState); // Own copy
    }
    
    updateState(newData) {
        this.state = { ...this.state, ...newData }; // Only affects this component
    }
}

// Pitfall 6: Assuming immutability in functional programming
const users = [{ id: 1, name: 'John' }];

// Wrong - modifies original array
const addUser = (users, newUser) => {
    users.push(newUser); // Mutates original!
    return users;
};

// Right - returns new array
const addUserImmutable = (users, newUser) => {
    return [...users, newUser]; // New array, original unchanged
};

// For nested modifications
const updateUserName = (users, userId, newName) => {
    return users.map(user => 
        user.id === userId 
            ? { ...user, name: newName } // New user object
            : user // Keep original reference for unchanged users
    );
};
```

---

## Related Topics
- [[15 - Objects]]
- [[16 - Arrays]]
- [[42 - Error Handling]]
- [[47 - Performance Optimization]]

---

*Next: [[31 - jQuery Basics]]*
