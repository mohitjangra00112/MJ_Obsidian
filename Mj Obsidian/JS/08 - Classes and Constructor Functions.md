# Classes and Constructor Functions

## Overview
JavaScript supports multiple ways to create objects and implement inheritance. ES6 introduced the `class` syntax as syntactic sugar over JavaScript's prototype-based inheritance, while constructor functions remain the traditional approach.

## Constructor Functions

### Basic Constructor Functions
```javascript
// Traditional constructor function
function Person(name, age, email) {
    // Properties
    this.name = name;
    this.age = age;
    this.email = email;
    
    // Method (not recommended - creates new function for each instance)
    this.greet = function() {
        return `Hello, I'm ${this.name}`;
    };
}

// Creating instances
const person1 = new Person('John', 30, 'john@example.com');
const person2 = new Person('Jane', 25, 'jane@example.com');

console.log(person1.name); // "John"
console.log(person1.greet()); // "Hello, I'm John"

// Check instance type
console.log(person1 instanceof Person); // true
console.log(person1.constructor === Person); // true

// Without 'new' keyword (problematic)
const person3 = Person('Bob', 35, 'bob@example.com'); // undefined
console.log(window.name); // "Bob" (in browser - pollutes global scope)
```

### Constructor Functions with Prototype Methods
```javascript
// Better approach - methods on prototype
function Vehicle(make, model, year) {
    this.make = make;
    this.model = model;
    this.year = year;
    this.isRunning = false;
}

// Add methods to prototype
Vehicle.prototype.start = function() {
    this.isRunning = true;
    return `${this.make} ${this.model} started`;
};

Vehicle.prototype.stop = function() {
    this.isRunning = false;
    return `${this.make} ${this.model} stopped`;
};

Vehicle.prototype.getInfo = function() {
    return `${this.year} ${this.make} ${this.model}`;
};

// Static method (not inherited by instances)
Vehicle.getVehicleCount = function() {
    return Vehicle.prototype.count || 0;
};

// Usage
const car1 = new Vehicle('Toyota', 'Camry', 2020);
const car2 = new Vehicle('Honda', 'Civic', 2019);

console.log(car1.start()); // "Toyota Camry started"
console.log(car1.getInfo()); // "2020 Toyota Camry"

// All instances share the same prototype methods
console.log(car1.start === car2.start); // true

// Inheritance with constructor functions
function Car(make, model, year, doors) {
    // Call parent constructor
    Vehicle.call(this, make, model, year);
    this.doors = doors;
    this.type = 'car';
}

// Set up inheritance
Car.prototype = Object.create(Vehicle.prototype);
Car.prototype.constructor = Car;

// Add car-specific methods
Car.prototype.openDoors = function() {
    return `Opening ${this.doors} doors`;
};

// Override parent method
Car.prototype.getInfo = function() {
    return `${Vehicle.prototype.getInfo.call(this)} (${this.doors} doors)`;
};

const myCar = new Car('BMW', 'X5', 2021, 4);
console.log(myCar.start()); // "BMW X5 started" (inherited)
console.log(myCar.openDoors()); // "Opening 4 doors" (own method)
console.log(myCar.getInfo()); // "2021 BMW X5 (4 doors)" (overridden)
```

### Advanced Constructor Patterns
```javascript
// Safe constructor pattern
function SafePerson(name, age) {
    // Ensure constructor is called with 'new'
    if (!(this instanceof SafePerson)) {
        return new SafePerson(name, age);
    }
    
    this.name = name;
    this.age = age;
}

// Works with or without 'new'
const safe1 = new SafePerson('John', 30);
const safe2 = SafePerson('Jane', 25); // Automatically uses 'new'

// Factory constructor pattern
function createUser(userData) {
    function User(name, email, role) {
        this.name = name;
        this.email = email;
        this.role = role || 'user';
        this.createdAt = new Date();
    }
    
    User.prototype.hasPermission = function(permission) {
        const permissions = {
            admin: ['read', 'write', 'delete'],
            moderator: ['read', 'write'],
            user: ['read']
        };
        
        return permissions[this.role]?.includes(permission) || false;
    };
    
    return new User(userData.name, userData.email, userData.role);
}

const admin = createUser({ name: 'Admin', email: 'admin@example.com', role: 'admin' });
console.log(admin.hasPermission('delete')); // true

// Mixin pattern with constructors
function Flyable() {}
Flyable.prototype.fly = function() {
    return `${this.name || 'Object'} is flying`;
};

function Swimmable() {}
Swimmable.prototype.swim = function() {
    return `${this.name || 'Object'} is swimming`;
};

function Duck(name) {
    this.name = name;
}

// Add multiple behaviors
Object.assign(Duck.prototype, Flyable.prototype, Swimmable.prototype);

Duck.prototype.quack = function() {
    return `${this.name} says quack!`;
};

const duck = new Duck('Donald');
console.log(duck.fly()); // "Donald is flying"
console.log(duck.swim()); // "Donald is swimming"
console.log(duck.quack()); // "Donald says quack!"
```

## ES6 Classes

### Basic Class Syntax
```javascript
// ES6 class syntax
class Person {
    // Constructor method
    constructor(name, age, email) {
        this.name = name;
        this.age = age;
        this.email = email;
    }
    
    // Instance methods
    greet() {
        return `Hello, I'm ${this.name}`;
    }
    
    getAge() {
        return this.age;
    }
    
    setAge(newAge) {
        if (newAge > 0 && newAge < 150) {
            this.age = newAge;
        } else {
            throw new Error('Invalid age');
        }
    }
    
    // Getter
    get info() {
        return `${this.name} (${this.age} years old)`;
    }
    
    // Setter
    set info(value) {
        const [name, age] = value.split(' (');
        this.name = name;
        this.age = parseInt(age);
    }
    
    // Static method
    static createAnonymous() {
        return new Person('Anonymous', 0, 'no-email@example.com');
    }
    
    // Static property (ES2022)
    static species = 'Homo sapiens';
}

// Usage
const person = new Person('Alice', 28, 'alice@example.com');
console.log(person.greet()); // "Hello, I'm Alice"
console.log(person.info); // "Alice (28 years old)" (getter)

person.info = 'Bob (35'; // Using setter
console.log(person.name); // "Bob"

// Static method usage
const anon = Person.createAnonymous();
console.log(Person.species); // "Homo sapiens"

// Class is just syntactic sugar over constructor functions
console.log(typeof Person); // "function"
console.log(person instanceof Person); // true
```

### Class Inheritance
```javascript
// Base class
class Animal {
    constructor(name, species) {
        this.name = name;
        this.species = species;
        this.isAlive = true;
    }
    
    speak() {
        return `${this.name} makes a sound`;
    }
    
    move() {
        return `${this.name} moves`;
    }
    
    static getKingdom() {
        return 'Animalia';
    }
}

// Derived class
class Dog extends Animal {
    constructor(name, breed, age) {
        // Call parent constructor
        super(name, 'Canis lupus');
        this.breed = breed;
        this.age = age;
    }
    
    // Override parent method
    speak() {
        return `${this.name} barks`;
    }
    
    // Call parent method
    move() {
        return super.move() + ' by running';
    }
    
    // New method
    wagTail() {
        return `${this.name} wags tail happily`;
    }
    
    // Static method
    static getDomesticationPeriod() {
        return '15,000 years ago';
    }
}

// Usage
const dog = new Dog('Buddy', 'Golden Retriever', 3);
console.log(dog.speak()); // "Buddy barks"
console.log(dog.move()); // "Buddy moves by running"
console.log(dog.wagTail()); // "Buddy wags tail happily"

// Inheritance chain
console.log(dog instanceof Dog); // true
console.log(dog instanceof Animal); // true
console.log(Object.getPrototypeOf(dog)); // Dog.prototype
console.log(Object.getPrototypeOf(Dog.prototype)); // Animal.prototype

// Static methods
console.log(Animal.getKingdom()); // "Animalia"
console.log(Dog.getDomesticationPeriod()); // "15,000 years ago"
```

### Advanced Class Features
```javascript
// Private fields and methods (ES2022)
class BankAccount {
    // Private fields
    #balance = 0;
    #accountNumber;
    #pin;
    
    constructor(accountNumber, pin, initialBalance = 0) {
        this.#accountNumber = accountNumber;
        this.#pin = pin;
        this.#balance = initialBalance;
        this.transactions = [];
    }
    
    // Private method
    #validatePin(pin) {
        return this.#pin === pin;
    }
    
    // Private method
    #addTransaction(type, amount) {
        this.transactions.push({
            type,
            amount,
            timestamp: new Date(),
            balance: this.#balance
        });
    }
    
    // Public methods
    deposit(amount, pin) {
        if (!this.#validatePin(pin)) {
            throw new Error('Invalid PIN');
        }
        
        if (amount <= 0) {
            throw new Error('Amount must be positive');
        }
        
        this.#balance += amount;
        this.#addTransaction('deposit', amount);
        return this.#balance;
    }
    
    withdraw(amount, pin) {
        if (!this.#validatePin(pin)) {
            throw new Error('Invalid PIN');
        }
        
        if (amount <= 0) {
            throw new Error('Amount must be positive');
        }
        
        if (amount > this.#balance) {
            throw new Error('Insufficient funds');
        }
        
        this.#balance -= amount;
        this.#addTransaction('withdrawal', amount);
        return this.#balance;
    }
    
    getBalance(pin) {
        if (!this.#validatePin(pin)) {
            throw new Error('Invalid PIN');
        }
        return this.#balance;
    }
    
    // Static private field
    static #interestRate = 0.02;
    
    static getInterestRate() {
        return BankAccount.#interestRate;
    }
}

// Usage
const account = new BankAccount('12345', '1234', 1000);
console.log(account.deposit(500, '1234')); // 1500
console.log(account.withdraw(200, '1234')); // 1300
console.log(account.getBalance('1234')); // 1300

// Private fields are not accessible
// console.log(account.#balance); // SyntaxError
// account.#validatePin('1234'); // SyntaxError

// Public field declarations (ES2022)
class ModernClass {
    // Public fields
    publicField = 'public value';
    anotherField;
    
    // Private fields
    #privateField = 'private value';
    
    constructor(value) {
        this.anotherField = value;
    }
    
    getPrivateField() {
        return this.#privateField;
    }
    
    // Static fields
    static staticField = 'static value';
    static #staticPrivateField = 'static private value';
    
    static getStaticPrivateField() {
        return ModernClass.#staticPrivateField;
    }
}

const modern = new ModernClass('constructor value');
console.log(modern.publicField); // "public value"
console.log(modern.anotherField); // "constructor value"
console.log(ModernClass.staticField); // "static value"
```

### Mixins with Classes
```javascript
// Mixin functions
const Timestamped = (BaseClass) => class extends BaseClass {
    constructor(...args) {
        super(...args);
        this.createdAt = new Date();
        this.updatedAt = new Date();
    }
    
    touch() {
        this.updatedAt = new Date();
    }
    
    getAge() {
        return Date.now() - this.createdAt.getTime();
    }
};

const Validated = (BaseClass) => class extends BaseClass {
    constructor(...args) {
        super(...args);
        this.errors = [];
    }
    
    validate() {
        this.errors = [];
        
        // Override in subclasses
        this._performValidation();
        
        return this.errors.length === 0;
    }
    
    _performValidation() {
        // To be implemented by subclasses
    }
    
    addError(field, message) {
        this.errors.push({ field, message });
    }
    
    getErrors() {
        return [...this.errors];
    }
};

// Base class
class BaseModel {
    constructor(data = {}) {
        Object.assign(this, data);
    }
    
    toJSON() {
        return { ...this };
    }
}

// Apply mixins
class User extends Validated(Timestamped(BaseModel)) {
    constructor(name, email) {
        super();
        this.name = name;
        this.email = email;
    }
    
    _performValidation() {
        if (!this.name || this.name.length < 2) {
            this.addError('name', 'Name must be at least 2 characters');
        }
        
        if (!this.email || !this.email.includes('@')) {
            this.addError('email', 'Invalid email format');
        }
    }
}

// Usage
const user = new User('John', 'john@example.com');
console.log(user.validate()); // true
console.log(user.getAge()); // milliseconds since creation

const invalidUser = new User('', 'invalid-email');
console.log(invalidUser.validate()); // false
console.log(invalidUser.getErrors()); 
// [{ field: 'name', message: '...' }, { field: 'email', message: '...' }]

// Multiple inheritance simulation
const Flyable = {
    fly() {
        return `${this.name || 'Object'} is flying`;
    }
};

const Swimmable = {
    swim() {
        return `${this.name || 'Object'} is swimming`;
    }
};

class SuperDuck extends User {
    constructor(name) {
        super(name, `${name.toLowerCase()}@duck.com`);
    }
}

// Add behaviors
Object.assign(SuperDuck.prototype, Flyable, Swimmable);

const superDuck = new SuperDuck('Donald');
console.log(superDuck.fly()); // "Donald is flying"
console.log(superDuck.swim()); // "Donald is swimming"
```

## Constructor Functions vs Classes Comparison

### Syntax Comparison
```javascript
// Constructor Function Approach
function PersonConstructor(name, age) {
    this.name = name;
    this.age = age;
}

PersonConstructor.prototype.greet = function() {
    return `Hello, I'm ${this.name}`;
};

PersonConstructor.prototype.getAge = function() {
    return this.age;
};

PersonConstructor.createAnonymous = function() {
    return new PersonConstructor('Anonymous', 0);
};

// Class Approach
class PersonClass {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    
    greet() {
        return `Hello, I'm ${this.name}`;
    }
    
    getAge() {
        return this.age;
    }
    
    static createAnonymous() {
        return new PersonClass('Anonymous', 0);
    }
}

// Both approaches create similar objects
const person1 = new PersonConstructor('John', 30);
const person2 = new PersonClass('Jane', 25);

console.log(person1.greet()); // "Hello, I'm John"
console.log(person2.greet()); // "Hello, I'm Jane"

// Inheritance comparison
// Constructor function inheritance
function StudentConstructor(name, age, school) {
    PersonConstructor.call(this, name, age);
    this.school = school;
}

StudentConstructor.prototype = Object.create(PersonConstructor.prototype);
StudentConstructor.prototype.constructor = StudentConstructor;

StudentConstructor.prototype.study = function() {
    return `${this.name} is studying`;
};

// Class inheritance
class StudentClass extends PersonClass {
    constructor(name, age, school) {
        super(name, age);
        this.school = school;
    }
    
    study() {
        return `${this.name} is studying`;
    }
}

const student1 = new StudentConstructor('Bob', 20, 'MIT');
const student2 = new StudentClass('Alice', 19, 'Harvard');

console.log(student1.study()); // "Bob is studying"
console.log(student2.study()); // "Alice is studying"
```

### Key Differences
```javascript
// 1. Hoisting behavior
console.log(typeof PersonConstructor); // "function" (hoisted)
// console.log(typeof PersonClass); // ReferenceError (not hoisted)

function PersonConstructor() {}
class PersonClass {}

// 2. Strict mode
// Classes automatically run in strict mode
class StrictClass {
    constructor() {
        console.log(this); // undefined if called without 'new'
    }
}

// Constructor functions don't enforce strict mode
function LooseConstructor() {
    console.log(this); // global object if called without 'new'
}

// 3. Enumerable methods
const constructorInstance = new PersonConstructor('John', 30);
const classInstance = new PersonClass('Jane', 25);

// Constructor function methods are enumerable by default
PersonConstructor.prototype.testMethod = function() {};
console.log(Object.getOwnPropertyDescriptor(PersonConstructor.prototype, 'testMethod').enumerable); // true

// Class methods are non-enumerable by default
console.log(Object.getOwnPropertyDescriptor(PersonClass.prototype, 'greet').enumerable); // false

// 4. Constructor invocation
// Constructor functions can be called without 'new' (problematic)
const wrong = PersonConstructor('Wrong', 30); // undefined, pollutes global scope

// Classes throw error when called without 'new'
// const error = PersonClass('Error', 25); // TypeError

// 5. Subclassing built-ins
// Classes can extend built-ins more easily
class CustomArray extends Array {
    first() {
        return this[0];
    }
    
    last() {
        return this[this.length - 1];
    }
}

const arr = new CustomArray(1, 2, 3, 4, 5);
console.log(arr.first()); // 1
console.log(arr.last()); // 5
console.log(arr.length); // 5
```

## Best Practices and Patterns

### When to Use Which
```javascript
// Use constructor functions when:
// 1. Working with legacy code
// 2. Need function hoisting
// 3. Want more explicit prototype manipulation

// Use classes when:
// 1. Writing new code
// 2. Want cleaner, more readable syntax
// 3. Need private fields/methods
// 4. Extending built-in objects

// Factory function alternative (functional approach)
function createPerson(name, age) {
    // Private variables (closure)
    let _id = Math.random().toString(36);
    
    return {
        name,
        age,
        
        // Public methods
        greet() {
            return `Hello, I'm ${this.name}`;
        },
        
        getId() {
            return _id;
        },
        
        // Computed property
        get info() {
            return `${this.name} (${this.age})`;
        }
    };
}

const person = createPerson('John', 30);
console.log(person.greet()); // "Hello, I'm John"
console.log(person.getId()); // random ID
// console.log(person._id); // undefined (truly private)

// Object.create pattern
const PersonPrototype = {
    init(name, age) {
        this.name = name;
        this.age = age;
        return this;
    },
    
    greet() {
        return `Hello, I'm ${this.name}`;
    }
};

const person3 = Object.create(PersonPrototype).init('Jane', 25);
console.log(person3.greet()); // "Hello, I'm Jane"
```

### Error Handling and Validation
```javascript
class ValidationError extends Error {
    constructor(message, field) {
        super(message);
        this.name = 'ValidationError';
        this.field = field;
    }
}

class ValidatedUser {
    constructor(userData) {
        this.validate(userData);
        Object.assign(this, userData);
    }
    
    validate(data) {
        const errors = [];
        
        if (!data.name || typeof data.name !== 'string') {
            errors.push(new ValidationError('Name is required and must be string', 'name'));
        }
        
        if (!data.email || !this.isValidEmail(data.email)) {
            errors.push(new ValidationError('Valid email is required', 'email'));
        }
        
        if (data.age !== undefined && (typeof data.age !== 'number' || data.age < 0)) {
            errors.push(new ValidationError('Age must be positive number', 'age'));
        }
        
        if (errors.length > 0) {
            const error = new Error('Validation failed');
            error.validationErrors = errors;
            throw error;
        }
    }
    
    isValidEmail(email) {
        return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
    }
}

// Usage with error handling
try {
    const user = new ValidatedUser({
        name: 'John',
        email: 'john@example.com',
        age: 30
    });
    console.log('User created successfully:', user);
} catch (error) {
    if (error.validationErrors) {
        console.error('Validation errors:');
        error.validationErrors.forEach(err => {
            console.error(`- ${err.field}: ${err.message}`);
        });
    } else {
        console.error('Unexpected error:', error.message);
    }
}
```

---

## Related Topics
- [[06 - Objects]]
- [[07 - Prototypes and Prototypal Inheritance]]
- [[09 - Object Creation Patterns]]
- [[35 - JavaScript OOP]]
- [[38 - ES6+ Features]]

---

*Next: [[09 - Object Creation Patterns]]*
