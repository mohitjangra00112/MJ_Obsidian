# Loops

## Overview
Loops are fundamental control structures in JavaScript that allow you to execute code repeatedly based on certain conditions. JavaScript provides several types of loops, each suited for different scenarios.

## For Loop

### Traditional For Loop
```javascript
// Basic for loop syntax: for (initialization; condition; increment)
for (let i = 0; i < 5; i++) {
    console.log(`Iteration ${i}`);
}
// Output: Iteration 0, Iteration 1, Iteration 2, Iteration 3, Iteration 4

// Counting backwards
for (let i = 10; i >= 0; i--) {
    console.log(`Countdown: ${i}`);
}

// Step by 2
for (let i = 0; i < 10; i += 2) {
    console.log(`Even number: ${i}`);
}
// Output: 0, 2, 4, 6, 8

// Multiple variables
for (let i = 0, j = 10; i < 5; i++, j--) {
    console.log(`i: ${i}, j: ${j}`);
}

// Looping through arrays with traditional for loop
const fruits = ['apple', 'banana', 'orange', 'grape'];

for (let i = 0; i < fruits.length; i++) {
    console.log(`Index ${i}: ${fruits[i]}`);
}

// Reverse loop through array
for (let i = fruits.length - 1; i >= 0; i--) {
    console.log(`Reverse ${i}: ${fruits[i]}`);
}

// Nested for loops
for (let row = 1; row <= 3; row++) {
    for (let col = 1; col <= 3; col++) {
        console.log(`Row ${row}, Col ${col}`);
    }
}

// Creating multiplication table
for (let i = 1; i <= 5; i++) {
    let row = '';
    for (let j = 1; j <= 5; j++) {
        row += `${i * j}\t`;
    }
    console.log(row);
}
```

### For...In Loop
```javascript
// For...in loop iterates over enumerable properties
const person = {
    name: 'John',
    age: 30,
    city: 'New York',
    occupation: 'Developer'
};

// Iterate over object properties
for (let key in person) {
    console.log(`${key}: ${person[key]}`);
}
// Output: name: John, age: 30, city: New York, occupation: Developer

// Check if property is own property (not inherited)
for (let key in person) {
    if (person.hasOwnProperty(key)) {
        console.log(`Own property ${key}: ${person[key]}`);
    }
}

// For...in with arrays (not recommended - use for...of instead)
const colors = ['red', 'green', 'blue'];

for (let index in colors) {
    console.log(`Index ${index}: ${colors[index]}`);
}
// Note: index is a string, not a number

// For...in with array methods and properties
colors.customProperty = 'custom';

for (let key in colors) {
    console.log(`Key: ${key}, Value: ${colors[key]}`);
}
// This will also iterate over customProperty

// Safer way to iterate over arrays
for (let index in colors) {
    if (colors.hasOwnProperty(index) && !isNaN(index)) {
        console.log(`Array element ${index}: ${colors[index]}`);
    }
}

// Iterating over object with prototype chain
function Animal(name) {
    this.name = name;
}

Animal.prototype.species = 'Unknown';

const dog = new Animal('Buddy');
dog.breed = 'Golden Retriever';

for (let key in dog) {
    console.log(`${key}: ${dog[key]}`);
}
// This includes inherited property 'species'

// Only own properties
for (let key in dog) {
    if (dog.hasOwnProperty(key)) {
        console.log(`Own property ${key}: ${dog[key]}`);
    }
}
```

### For...Of Loop
```javascript
// For...of loop iterates over iterable objects (arrays, strings, Maps, Sets, etc.)
const numbers = [1, 2, 3, 4, 5];

// Basic for...of with arrays
for (let number of numbers) {
    console.log(number);
}

// For...of with strings
const message = 'Hello';
for (let char of message) {
    console.log(char);
}
// Output: H, e, l, l, o

// For...of with array destructuring
const coordinates = [[0, 0], [1, 2], [3, 4]];

for (let [x, y] of coordinates) {
    console.log(`x: ${x}, y: ${y}`);
}

// For...of with object entries
const user = { name: 'Alice', age: 25, email: 'alice@example.com' };

for (let [key, value] of Object.entries(user)) {
    console.log(`${key}: ${value}`);
}

// For...of with Sets
const uniqueNumbers = new Set([1, 2, 3, 2, 1]);

for (let number of uniqueNumbers) {
    console.log(number); // Output: 1, 2, 3 (duplicates removed)
}

// For...of with Maps
const userRoles = new Map([
    ['john', 'admin'],
    ['jane', 'user'],
    ['bob', 'moderator']
]);

for (let [username, role] of userRoles) {
    console.log(`${username}: ${role}`);
}

// For...of with Map methods
for (let key of userRoles.keys()) {
    console.log(`Username: ${key}`);
}

for (let value of userRoles.values()) {
    console.log(`Role: ${value}`);
}

// For...of with generators
function* numberGenerator() {
    yield 1;
    yield 2;
    yield 3;
}

for (let num of numberGenerator()) {
    console.log(num);
}

// For...of with array methods
const words = ['hello', 'world', 'javascript'];

for (let word of words.filter(w => w.length > 5)) {
    console.log(word); // Only 'javascript'
}
```

## While Loop

### Basic While Loop
```javascript
// Basic while loop
let count = 0;

while (count < 5) {
    console.log(`Count: ${count}`);
    count++;
}

// Reading array with while loop
const items = ['a', 'b', 'c', 'd'];
let index = 0;

while (index < items.length) {
    console.log(`Item ${index}: ${items[index]}`);
    index++;
}

// While loop with complex condition
let sum = 0;
let num = 1;

while (sum < 100) {
    sum += num;
    console.log(`Added ${num}, sum is now ${sum}`);
    num++;
}

// User input simulation with while loop
let userInput = '';
let attempts = 0;
const maxAttempts = 3;

while (userInput !== 'exit' && attempts < maxAttempts) {
    // Simulate getting user input
    userInput = ['continue', 'exit', 'maybe'][Math.floor(Math.random() * 3)];
    attempts++;
    console.log(`Attempt ${attempts}: User said "${userInput}"`);
}

// Infinite loop prevention
let safetyCounter = 0;
const maxIterations = 1000;

while (someCondition() && safetyCounter < maxIterations) {
    // Do something
    console.log('Working...');
    safetyCounter++;
    
    if (safetyCounter >= maxIterations) {
        console.warn('Loop safety limit reached');
    }
}

function someCondition() {
    return Math.random() > 0.1; // 90% chance to continue
}

// Processing queue with while loop
const taskQueue = ['task1', 'task2', 'task3', 'task4'];

while (taskQueue.length > 0) {
    const currentTask = taskQueue.shift();
    console.log(`Processing: ${currentTask}`);
    
    // Simulate task completion
    if (Math.random() > 0.8) {
        console.log(`${currentTask} failed, requeueing...`);
        taskQueue.push(currentTask);
    } else {
        console.log(`${currentTask} completed successfully`);
    }
}
```

### Do...While Loop
```javascript
// Do...while loop - executes at least once
let password;

do {
    // Simulate password input
    password = Math.random().toString(36).substring(7);
    console.log(`Trying password: ${password}`);
} while (password !== 'secret' && password.length < 6);

console.log('Password accepted!');

// Menu system with do...while
let choice;

do {
    // Display menu
    console.log('\n--- Menu ---');
    console.log('1. Option A');
    console.log('2. Option B');
    console.log('3. Exit');
    
    // Simulate user choice
    choice = Math.floor(Math.random() * 4) + 1;
    console.log(`User selected: ${choice}`);
    
    switch (choice) {
        case 1:
            console.log('Executing Option A');
            break;
        case 2:
            console.log('Executing Option B');
            break;
        case 3:
            console.log('Exiting...');
            break;
        default:
            console.log('Invalid choice, try again');
    }
} while (choice !== 3);

// Validation with do...while
function validateEmail(email) {
    return email.includes('@') && email.includes('.');
}

let email;
let isValid;

do {
    // Simulate email input
    const testEmails = ['invalid', 'test@', 'user@example.com'];
    email = testEmails[Math.floor(Math.random() * testEmails.length)];
    isValid = validateEmail(email);
    
    console.log(`Testing email: ${email} - ${isValid ? 'Valid' : 'Invalid'}`);
    
    if (!isValid) {
        console.log('Please enter a valid email address');
    }
} while (!isValid);

console.log(`Email ${email} accepted!`);

// Game loop example
let gameRunning = true;
let playerScore = 0;

do {
    // Simulate game round
    const roundScore = Math.floor(Math.random() * 100);
    playerScore += roundScore;
    
    console.log(`Round score: ${roundScore}, Total: ${playerScore}`);
    
    // Random chance to end game
    gameRunning = Math.random() > 0.3;
    
    if (!gameRunning) {
        console.log('Game Over!');
    }
    
} while (gameRunning && playerScore < 500);

console.log(`Final score: ${playerScore}`);
```

## Advanced Loop Patterns

### Nested Loops
```javascript
// Matrix operations with nested loops
const matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

// Print matrix
for (let row = 0; row < matrix.length; row++) {
    let rowString = '';
    for (let col = 0; col < matrix[row].length; col++) {
        rowString += matrix[row][col] + ' ';
    }
    console.log(rowString);
}

// Find element in matrix
function findInMatrix(matrix, target) {
    for (let row = 0; row < matrix.length; row++) {
        for (let col = 0; col < matrix[row].length; col++) {
            if (matrix[row][col] === target) {
                return { row, col };
            }
        }
    }
    return null;
}

console.log(findInMatrix(matrix, 5)); // { row: 1, col: 1 }

// Generate patterns with nested loops
// Right triangle pattern
for (let i = 1; i <= 5; i++) {
    let pattern = '';
    for (let j = 1; j <= i; j++) {
        pattern += '* ';
    }
    console.log(pattern);
}

// Diamond pattern
const size = 5;
// Upper half
for (let i = 1; i <= size; i++) {
    let spaces = ' '.repeat(size - i);
    let stars = '* '.repeat(i);
    console.log(spaces + stars);
}
// Lower half
for (let i = size - 1; i >= 1; i--) {
    let spaces = ' '.repeat(size - i);
    let stars = '* '.repeat(i);
    console.log(spaces + stars);
}

// Nested object traversal
const nestedData = {
    users: {
        admins: ['admin1', 'admin2'],
        moderators: ['mod1', 'mod2'],
        members: ['user1', 'user2', 'user3']
    },
    settings: {
        theme: 'dark',
        notifications: true
    }
};

function traverseObject(obj, depth = 0) {
    const indent = '  '.repeat(depth);
    
    for (let key in obj) {
        if (obj.hasOwnProperty(key)) {
            console.log(`${indent}${key}:`);
            
            if (typeof obj[key] === 'object' && !Array.isArray(obj[key])) {
                traverseObject(obj[key], depth + 1);
            } else if (Array.isArray(obj[key])) {
                for (let i = 0; i < obj[key].length; i++) {
                    console.log(`${indent}  [${i}]: ${obj[key][i]}`);
                }
            } else {
                console.log(`${indent}  ${obj[key]}`);
            }
        }
    }
}

traverseObject(nestedData);
```

### Loop Control Statements
```javascript
// Break statement - exits the loop immediately
const numbers = [1, 3, 5, 8, 9, 12, 15];

// Find first even number
for (let i = 0; i < numbers.length; i++) {
    if (numbers[i] % 2 === 0) {
        console.log(`First even number: ${numbers[i]} at index ${i}`);
        break;
    }
}

// Break in nested loops
outerLoop: for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
        if (i === 1 && j === 1) {
            console.log(`Breaking outer loop at i=${i}, j=${j}`);
            break outerLoop; // Breaks the outer loop
        }
        console.log(`i=${i}, j=${j}`);
    }
}

// Continue statement - skips current iteration
const values = [1, -2, 3, -4, 5, -6, 7];

// Process only positive numbers
for (let value of values) {
    if (value < 0) {
        continue; // Skip negative numbers
    }
    console.log(`Processing positive number: ${value}`);
}

// Continue with label
outerProcess: for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
        if (j === 1) {
            continue outerProcess; // Continue outer loop
        }
        console.log(`Processing i=${i}, j=${j}`);
    }
}

// Complex break/continue logic
const data = [
    { type: 'user', status: 'active', id: 1 },
    { type: 'admin', status: 'inactive', id: 2 },
    { type: 'user', status: 'active', id: 3 },
    { type: 'moderator', status: 'active', id: 4 },
    { type: 'user', status: 'banned', id: 5 }
];

// Process only active users, stop at first banned user
for (let item of data) {
    if (item.status === 'banned') {
        console.log(`Stopping at banned user: ${item.id}`);
        break;
    }
    
    if (item.type !== 'user') {
        continue; // Skip non-users
    }
    
    if (item.status !== 'active') {
        continue; // Skip inactive users
    }
    
    console.log(`Processing active user: ${item.id}`);
}
```

### Performance Optimization in Loops
```javascript
// Cache array length to avoid repeated property access
const largeArray = new Array(1000000).fill(0).map((_, i) => i);

// Inefficient - accesses .length property every iteration
console.time('Uncached Length');
for (let i = 0; i < largeArray.length; i++) {
    // Some operation
    largeArray[i] = largeArray[i] * 2;
}
console.timeEnd('Uncached Length');

// Efficient - caches length
console.time('Cached Length');
for (let i = 0, len = largeArray.length; i < len; i++) {
    largeArray[i] = largeArray[i] * 2;
}
console.timeEnd('Cached Length');

// Use for...of for simple iterations (often faster)
console.time('For...of');
for (let value of largeArray) {
    // Read-only operations
    Math.sqrt(value);
}
console.timeEnd('For...of');

// Use forEach for functional style
console.time('forEach');
largeArray.forEach(value => {
    Math.sqrt(value);
});
console.timeEnd('forEach');

// Avoid creating functions inside loops
const processors = [];

// Inefficient - creates new function each iteration
for (let i = 0; i < 1000; i++) {
    processors.push(function(data) {
        return data * i; // This won't work as expected anyway
    });
}

// Efficient - reuse function
function createProcessor(multiplier) {
    return function(data) {
        return data * multiplier;
    };
}

for (let i = 0; i < 1000; i++) {
    processors.push(createProcessor(i));
}

// Early exit optimizations
function findUserById(users, targetId) {
    for (let user of users) {
        if (user.id === targetId) {
            return user; // Exit immediately when found
        }
    }
    return null;
}

// Batch processing for large datasets
function processBatch(data, batchSize = 100) {
    for (let i = 0; i < data.length; i += batchSize) {
        const batch = data.slice(i, i + batchSize);
        
        // Process batch
        console.log(`Processing batch ${Math.floor(i / batchSize) + 1}`);
        
        // Yield control periodically for large datasets
        if (i % (batchSize * 10) === 0) {
            setTimeout(() => {
                // Continue processing
            }, 0);
        }
    }
}
```

### Functional Alternatives to Loops
```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Traditional loop vs functional approach
// Sum all numbers
let sum = 0;
for (let num of numbers) {
    sum += num;
}
console.log('Loop sum:', sum);

// Functional approach
const functionalSum = numbers.reduce((acc, num) => acc + num, 0);
console.log('Functional sum:', functionalSum);

// Filter and transform
const evenSquares = [];
for (let num of numbers) {
    if (num % 2 === 0) {
        evenSquares.push(num * num);
    }
}
console.log('Loop even squares:', evenSquares);

// Functional approach
const functionalEvenSquares = numbers
    .filter(num => num % 2 === 0)
    .map(num => num * num);
console.log('Functional even squares:', functionalEvenSquares);

// Find element
let foundItem = null;
for (let num of numbers) {
    if (num > 5) {
        foundItem = num;
        break;
    }
}
console.log('Loop found:', foundItem);

// Functional approach
const functionalFound = numbers.find(num => num > 5);
console.log('Functional found:', functionalFound);

// Check if all/some elements meet condition
let allPositive = true;
for (let num of numbers) {
    if (num <= 0) {
        allPositive = false;
        break;
    }
}
console.log('Loop all positive:', allPositive);

// Functional approach
const functionalAllPositive = numbers.every(num => num > 0);
const someGreaterThanFive = numbers.some(num => num > 5);
console.log('Functional all positive:', functionalAllPositive);
console.log('Some greater than 5:', someGreaterThanFive);

// Complex data transformations
const users = [
    { name: 'John', age: 25, active: true },
    { name: 'Jane', age: 30, active: false },
    { name: 'Bob', age: 35, active: true }
];

// Traditional loop approach
const activeUserNames = [];
for (let user of users) {
    if (user.active && user.age >= 30) {
        activeUserNames.push(user.name.toUpperCase());
    }
}

// Functional approach
const functionalActiveUserNames = users
    .filter(user => user.active && user.age >= 30)
    .map(user => user.name.toUpperCase());

console.log('Active users 30+:', functionalActiveUserNames);

// Group by age range
const ageGroups = { young: [], middle: [], senior: [] };
for (let user of users) {
    if (user.age < 30) {
        ageGroups.young.push(user);
    } else if (user.age < 60) {
        ageGroups.middle.push(user);
    } else {
        ageGroups.senior.push(user);
    }
}

// Functional grouping
const functionalAgeGroups = users.reduce((groups, user) => {
    const group = user.age < 30 ? 'young' : user.age < 60 ? 'middle' : 'senior';
    groups[group].push(user);
    return groups;
}, { young: [], middle: [], senior: [] });

console.log('Age groups:', functionalAgeGroups);
```

### Async Loops
```javascript
// Sequential async processing
async function processItemsSequentially(items) {
    const results = [];
    
    for (let item of items) {
        try {
            const result = await processItem(item);
            results.push(result);
            console.log(`Processed ${item}: ${result}`);
        } catch (error) {
            console.error(`Error processing ${item}:`, error);
        }
    }
    
    return results;
}

// Parallel async processing
async function processItemsInParallel(items) {
    const promises = items.map(item => processItem(item));
    
    try {
        const results = await Promise.all(promises);
        return results;
    } catch (error) {
        console.error('Error in parallel processing:', error);
        throw error;
    }
}

// Controlled concurrency
async function processItemsWithConcurrency(items, concurrency = 3) {
    const results = [];
    
    for (let i = 0; i < items.length; i += concurrency) {
        const batch = items.slice(i, i + concurrency);
        const batchPromises = batch.map(item => processItem(item));
        
        try {
            const batchResults = await Promise.all(batchPromises);
            results.push(...batchResults);
            console.log(`Completed batch ${Math.floor(i / concurrency) + 1}`);
        } catch (error) {
            console.error(`Error in batch ${Math.floor(i / concurrency) + 1}:`, error);
        }
    }
    
    return results;
}

async function processItem(item) {
    // Simulate async operation
    return new Promise(resolve => {
        setTimeout(() => {
            resolve(`Processed: ${item}`);
        }, Math.random() * 1000);
    });
}

// Usage examples
(async () => {
    const items = ['item1', 'item2', 'item3', 'item4', 'item5'];
    
    console.log('Sequential processing:');
    await processItemsSequentially(items);
    
    console.log('\nParallel processing:');
    await processItemsInParallel(items);
    
    console.log('\nConcurrency-controlled processing:');
    await processItemsWithConcurrency(items, 2);
})();
```

---

## Related Topics
- [[28 - Array Methods]]
- [[02 - Functions]]
- [[13 - Higher Order Functions]]
- [[19 - Promises]]
- [[20 - Async Await]]

---

*Next: [[32 - jQuery]]*
