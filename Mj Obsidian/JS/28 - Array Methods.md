# Array Methods

## Overview
JavaScript arrays come with many built-in methods that provide powerful ways to manipulate, transform, and analyze array data. Understanding these methods is crucial for effective JavaScript programming.

## Mutating Methods (Change Original Array)

### push() and pop()
```javascript
const fruits = ['apple', 'banana'];

// push() - adds elements to end
const newLength = fruits.push('orange', 'grape');
console.log(fruits); // ['apple', 'banana', 'orange', 'grape']
console.log(newLength); // 4

// pop() - removes last element
const removed = fruits.pop();
console.log(removed); // 'grape'
console.log(fruits); // ['apple', 'banana', 'orange']
```

### unshift() and shift()
```javascript
const numbers = [2, 3, 4];

// unshift() - adds elements to beginning
numbers.unshift(0, 1);
console.log(numbers); // [0, 1, 2, 3, 4]

// shift() - removes first element
const first = numbers.shift();
console.log(first); // 0
console.log(numbers); // [1, 2, 3, 4]
```

### splice()
```javascript
const colors = ['red', 'green', 'blue', 'yellow'];

// Remove elements: splice(start, deleteCount)
const removed = colors.splice(1, 2);
console.log(removed); // ['green', 'blue']
console.log(colors); // ['red', 'yellow']

// Insert elements: splice(start, 0, ...items)
colors.splice(1, 0, 'purple', 'orange');
console.log(colors); // ['red', 'purple', 'orange', 'yellow']

// Replace elements: splice(start, deleteCount, ...items)
colors.splice(2, 1, 'pink', 'cyan');
console.log(colors); // ['red', 'purple', 'pink', 'cyan', 'yellow']
```

### sort()
```javascript
const numbers = [3, 1, 4, 1, 5, 9];

// Default sort (converts to strings)
numbers.sort();
console.log(numbers); // [1, 1, 3, 4, 5, 9]

// Custom sort function
const nums = [10, 5, 40, 25, 1000, 1];

// Ascending order
nums.sort((a, b) => a - b);
console.log(nums); // [1, 5, 10, 25, 40, 1000]

// Descending order
nums.sort((a, b) => b - a);
console.log(nums); // [1000, 40, 25, 10, 5, 1]

// Sort objects
const people = [
    { name: 'John', age: 30 },
    { name: 'Jane', age: 25 },
    { name: 'Bob', age: 35 }
];

people.sort((a, b) => a.age - b.age);
console.log(people); // Sorted by age ascending
```

### reverse()
```javascript
const letters = ['a', 'b', 'c', 'd'];
letters.reverse();
console.log(letters); // ['d', 'c', 'b', 'a']
```

### fill()
```javascript
const arr = new Array(5);
arr.fill(0);
console.log(arr); // [0, 0, 0, 0, 0]

// Fill with start and end positions
const nums = [1, 2, 3, 4, 5];
nums.fill(9, 1, 4); // fill with 9 from index 1 to 3
console.log(nums); // [1, 9, 9, 9, 5]
```

## Non-Mutating Methods (Return New Array/Value)

### map()
```javascript
const numbers = [1, 2, 3, 4, 5];

// Transform each element
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// Map with index
const withIndex = numbers.map((num, index) => `${index}: ${num}`);
console.log(withIndex); // ['0: 1', '1: 2', '2: 3', '3: 4', '4: 5']

// Map objects
const users = [
    { id: 1, name: 'John', age: 30 },
    { id: 2, name: 'Jane', age: 25 }
];

const userNames = users.map(user => user.name);
console.log(userNames); // ['John', 'Jane']

const userSummaries = users.map(user => ({
    id: user.id,
    summary: `${user.name} is ${user.age} years old`
}));
```

### filter()
```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Filter even numbers
const evens = numbers.filter(num => num % 2 === 0);
console.log(evens); // [2, 4, 6, 8, 10]

// Filter objects
const products = [
    { name: 'Laptop', price: 1000, inStock: true },
    { name: 'Phone', price: 500, inStock: false },
    { name: 'Tablet', price: 300, inStock: true }
];

const availableProducts = products.filter(product => product.inStock);
const expensiveProducts = products.filter(product => product.price > 400);

// Complex filtering
const cheapAvailableProducts = products.filter(product => 
    product.inStock && product.price < 600
);
```

### reduce()
```javascript
const numbers = [1, 2, 3, 4, 5];

// Sum all numbers
const sum = numbers.reduce((accumulator, current) => accumulator + current, 0);
console.log(sum); // 15

// Find maximum
const max = numbers.reduce((max, current) => current > max ? current : max);

// Count occurrences
const fruits = ['apple', 'banana', 'apple', 'orange', 'banana', 'apple'];
const fruitCount = fruits.reduce((count, fruit) => {
    count[fruit] = (count[fruit] || 0) + 1;
    return count;
}, {});
console.log(fruitCount); // { apple: 3, banana: 2, orange: 1 }

// Group objects
const people = [
    { name: 'John', department: 'IT' },
    { name: 'Jane', department: 'HR' },
    { name: 'Bob', department: 'IT' },
    { name: 'Alice', department: 'Finance' }
];

const groupedByDept = people.reduce((groups, person) => {
    const dept = person.department;
    if (!groups[dept]) groups[dept] = [];
    groups[dept].push(person);
    return groups;
}, {});

// Flatten nested arrays
const nested = [[1, 2], [3, 4], [5, 6]];
const flattened = nested.reduce((flat, arr) => flat.concat(arr), []);
console.log(flattened); // [1, 2, 3, 4, 5, 6]
```

### forEach()
```javascript
const fruits = ['apple', 'banana', 'orange'];

// Basic iteration
fruits.forEach(fruit => console.log(fruit));

// With index
fruits.forEach((fruit, index) => {
    console.log(`${index + 1}: ${fruit}`);
});

// Cannot break out of forEach (use for...of instead)
fruits.forEach(fruit => {
    if (fruit === 'banana') {
        return; // Only skips current iteration, doesn't break loop
    }
    console.log(fruit);
});
```

## Search and Test Methods

### find() and findIndex()
```javascript
const users = [
    { id: 1, name: 'John', active: true },
    { id: 2, name: 'Jane', active: false },
    { id: 3, name: 'Bob', active: true }
];

// Find first matching element
const activeUser = users.find(user => user.active);
console.log(activeUser); // { id: 1, name: 'John', active: true }

// Find index of first matching element
const inactiveUserIndex = users.findIndex(user => !user.active);
console.log(inactiveUserIndex); // 1

// Returns undefined/-1 if not found
const admin = users.find(user => user.role === 'admin');
console.log(admin); // undefined
```

### indexOf() and lastIndexOf()
```javascript
const numbers = [1, 2, 3, 2, 4, 2];

// Find first occurrence
const firstIndex = numbers.indexOf(2);
console.log(firstIndex); // 1

// Find last occurrence
const lastIndex = numbers.lastIndexOf(2);
console.log(lastIndex); // 5

// Check if element exists
const hasElement = numbers.indexOf(5) !== -1;
console.log(hasElement); // false

// Modern way to check existence
const exists = numbers.includes(3);
console.log(exists); // true
```

### includes()
```javascript
const fruits = ['apple', 'banana', 'orange'];

console.log(fruits.includes('apple')); // true
console.log(fruits.includes('grape')); // false

// Case sensitive
console.log(fruits.includes('Apple')); // false

// With start position
const numbers = [1, 2, 3, 4, 5];
console.log(numbers.includes(3, 2)); // true (search from index 2)
console.log(numbers.includes(2, 2)); // false (search from index 2)
```

### some() and every()
```javascript
const numbers = [1, 2, 3, 4, 5];

// some() - at least one element passes test
const hasEven = numbers.some(num => num % 2 === 0);
console.log(hasEven); // true

const hasLarge = numbers.some(num => num > 10);
console.log(hasLarge); // false

// every() - all elements pass test
const allPositive = numbers.every(num => num > 0);
console.log(allPositive); // true

const allEven = numbers.every(num => num % 2 === 0);
console.log(allEven); // false

// Practical examples
const users = [
    { name: 'John', age: 25 },
    { name: 'Jane', age: 30 },
    { name: 'Bob', age: 17 }
];

const hasMinor = users.some(user => user.age < 18);
const allAdults = users.every(user => user.age >= 18);
```

## Transformation Methods

### concat()
```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const arr3 = [7, 8, 9];

// Concatenate arrays
const combined = arr1.concat(arr2, arr3);
console.log(combined); // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// Modern alternative with spread
const modernCombined = [...arr1, ...arr2, ...arr3];

// Concat with individual elements
const withElements = arr1.concat(10, 11, arr2);
console.log(withElements); // [1, 2, 3, 10, 11, 4, 5, 6]
```

### slice()
```javascript
const animals = ['cat', 'dog', 'elephant', 'fish', 'giraffe'];

// Extract portion (start, end)
const subset = animals.slice(1, 4);
console.log(subset); // ['dog', 'elephant', 'fish']

// From index to end
const fromIndex = animals.slice(2);
console.log(fromIndex); // ['elephant', 'fish', 'giraffe']

// Last n elements
const lastTwo = animals.slice(-2);
console.log(lastTwo); // ['fish', 'giraffe']

// Shallow copy
const copy = animals.slice();
```

### join()
```javascript
const words = ['Hello', 'World', 'JavaScript'];

// Default separator (comma)
const defaultJoin = words.join();
console.log(defaultJoin); // "Hello,World,JavaScript"

// Custom separator
const spaceJoin = words.join(' ');
console.log(spaceJoin); // "Hello World JavaScript"

const dashJoin = words.join('-');
console.log(dashJoin); // "Hello-World-JavaScript"

// Empty separator
const noSeparator = words.join('');
console.log(noSeparator); // "HelloWorldJavaScript"
```

## ES6+ Methods

### flat() and flatMap()
```javascript
// flat() - flattens nested arrays
const nested = [1, [2, 3], [4, [5, 6]]];

const flattened = nested.flat();
console.log(flattened); // [1, 2, 3, 4, [5, 6]]

const deepFlattened = nested.flat(2);
console.log(deepFlattened); // [1, 2, 3, 4, 5, 6]

// Flatten all levels
const reallyNested = [1, [2, [3, [4, [5]]]]];
const allFlat = reallyNested.flat(Infinity);
console.log(allFlat); // [1, 2, 3, 4, 5]

// flatMap() - map + flat
const sentences = ["Hello world", "How are you"];
const words = sentences.flatMap(sentence => sentence.split(' '));
console.log(words); // ['Hello', 'world', 'How', 'are', 'you']

// Equivalent to:
const wordsManual = sentences.map(sentence => sentence.split(' ')).flat();
```

### from() and of()
```javascript
// Array.from() - create array from iterable or array-like
const str = "hello";
const chars = Array.from(str);
console.log(chars); // ['h', 'e', 'l', 'l', 'o']

// With mapping function
const doubled = Array.from([1, 2, 3], x => x * 2);
console.log(doubled); // [2, 4, 6]

// Create array with length and fill
const zeros = Array.from({ length: 5 }, () => 0);
console.log(zeros); // [0, 0, 0, 0, 0]

const sequence = Array.from({ length: 5 }, (_, i) => i + 1);
console.log(sequence); // [1, 2, 3, 4, 5]

// Array.of() - create array from arguments
const arr = Array.of(1, 2, 3);
console.log(arr); // [1, 2, 3]

// Difference from Array constructor
console.log(Array(3)); // [empty × 3]
console.log(Array.of(3)); // [3]
```

## Method Chaining

### Combining Multiple Methods
```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Chain multiple operations
const result = numbers
    .filter(num => num % 2 === 0)  // [2, 4, 6, 8, 10]
    .map(num => num * num)         // [4, 16, 36, 64, 100]
    .reduce((sum, num) => sum + num, 0); // 220

console.log(result); // 220

// More complex example
const users = [
    { name: 'John', age: 25, active: true, scores: [85, 92, 78] },
    { name: 'Jane', age: 30, active: false, scores: [91, 87, 95] },
    { name: 'Bob', age: 22, active: true, scores: [76, 84, 88] },
    { name: 'Alice', age: 28, active: true, scores: [93, 89, 96] }
];

const activeUserAverages = users
    .filter(user => user.active)
    .map(user => ({
        name: user.name,
        average: user.scores.reduce((sum, score) => sum + score, 0) / user.scores.length
    }))
    .sort((a, b) => b.average - a.average);

console.log(activeUserAverages);
```

## Performance Considerations

### Choosing the Right Method
```javascript
const largeArray = Array.from({ length: 1000000 }, (_, i) => i);

// For finding first match, use find() not filter()
console.time('find');
const found = largeArray.find(num => num === 500000);
console.timeEnd('find');

console.time('filter');
const filtered = largeArray.filter(num => num === 500000)[0];
console.timeEnd('filter'); // Much slower

// For existence check, use includes() not indexOf()
console.time('includes');
const exists = largeArray.includes(500000);
console.timeEnd('includes');

console.time('indexOf');
const indexExists = largeArray.indexOf(500000) !== -1;
console.timeEnd('indexOf'); // Usually slower
```

### Memory Efficiency
```javascript
// Avoid creating unnecessary intermediate arrays
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Less efficient - creates multiple intermediate arrays
const inefficient = numbers
    .map(x => x * 2)
    .map(x => x + 1)
    .filter(x => x > 10);

// More efficient - single pass
const efficient = numbers
    .reduce((acc, x) => {
        const doubled = x * 2;
        const incremented = doubled + 1;
        if (incremented > 10) {
            acc.push(incremented);
        }
        return acc;
    }, []);

// Or use a for loop for maximum efficiency
const mostEfficient = [];
for (const x of numbers) {
    const result = x * 2 + 1;
    if (result > 10) {
        mostEfficient.push(result);
    }
}
```

## Common Patterns

### Remove Duplicates
```javascript
const numbers = [1, 2, 2, 3, 3, 3, 4, 5, 5];

// Using Set
const unique = [...new Set(numbers)];
console.log(unique); // [1, 2, 3, 4, 5]

// Using filter and indexOf
const uniqueFilter = numbers.filter((num, index) => numbers.indexOf(num) === index);

// For objects (by property)
const users = [
    { id: 1, name: 'John' },
    { id: 2, name: 'Jane' },
    { id: 1, name: 'John' }, // duplicate
    { id: 3, name: 'Bob' }
];

const uniqueUsers = users.filter((user, index, arr) => 
    arr.findIndex(u => u.id === user.id) === index
);
```

### Shuffle Array
```javascript
function shuffleArray(array) {
    const shuffled = [...array]; // Don't mutate original
    
    for (let i = shuffled.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
    }
    
    return shuffled;
}

const cards = ['A', 'K', 'Q', 'J', '10', '9'];
const shuffled = shuffleArray(cards);
```

### Chunk Array
```javascript
function chunkArray(array, size) {
    const chunks = [];
    for (let i = 0; i < array.length; i += size) {
        chunks.push(array.slice(i, i + size));
    }
    return chunks;
}

const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const chunked = chunkArray(numbers, 3);
console.log(chunked); // [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10]]
```

### Partition Array
```javascript
function partition(array, predicate) {
    return array.reduce(
        (acc, item) => {
            acc[predicate(item) ? 0 : 1].push(item);
            return acc;
        },
        [[], []]
    );
}

const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const [evens, odds] = partition(numbers, num => num % 2 === 0);
console.log(evens); // [2, 4, 6, 8, 10]
console.log(odds);  // [1, 3, 5, 7, 9]
```

---

## Related Topics
- [[29 - Object Methods]]
- [[31 - Loops]]
- [[30 - Deep Copy vs Shallow Copy]]
- [[40 - Destructuring]]

---

*Next: [[29 - Object Methods]]*
