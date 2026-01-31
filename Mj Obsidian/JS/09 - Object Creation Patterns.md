# Object Creation Patterns

## Overview
JavaScript offers multiple ways to create objects, each with its own advantages and use cases. Understanding these patterns helps you choose the best approach for different scenarios.

## Literal Object Pattern

### Basic Object Literals
```javascript
// Simple object literal
const person = {
    name: 'John',
    age: 30,
    email: 'john@example.com',
    
    greet: function() {
        return `Hello, I'm ${this.name}`;
    },
    
    // ES6 method shorthand
    getAge() {
        return this.age;
    },
    
    // Computed property names
    ['full' + 'Name']: 'John Doe'
};

console.log(person.greet()); // "Hello, I'm John"
console.log(person.fullName); // "John Doe"

// Property descriptor with Object.defineProperty
Object.defineProperty(person, 'id', {
    value: 123,
    writable: false,
    enumerable: false,
    configurable: false
});

console.log(person.id); // 123
// person.id = 456; // Won't change (writable: false)
console.log(Object.keys(person)); // Won't include 'id' (enumerable: false)

// Multiple properties with Object.defineProperties
Object.defineProperties(person, {
    createdAt: {
        value: new Date(),
        writable: false
    },
    isActive: {
        value: true,
        writable: true
    }
});

// ES6 enhanced object literals
const name = 'Alice';
const age = 25;

const enhancedPerson = {
    // Shorthand property names
    name,
    age,
    
    // Computed property names
    [name.toLowerCase()]: true,
    
    // Method definitions
    greet() {
        return `Hi, I'm ${this.name}`;
    },
    
    // Generator method
    *numbers() {
        yield 1;
        yield 2;
        yield 3;
    },
    
    // Async method
    async fetchData() {
        return new Promise(resolve => {
            setTimeout(() => resolve('data'), 1000);
        });
    }
};

console.log(enhancedPerson.alice); // true
console.log([...enhancedPerson.numbers()]); // [1, 2, 3]
```

### Nested Objects and Complex Structures
```javascript
// Complex object structure
const company = {
    name: 'Tech Corp',
    founded: 2010,
    
    address: {
        street: '123 Main St',
        city: 'San Francisco',
        country: 'USA',
        
        getFullAddress() {
            return `${this.street}, ${this.city}, ${this.country}`;
        }
    },
    
    employees: [
        {
            id: 1,
            name: 'John Doe',
            department: 'Engineering',
            salary: 100000,
            
            getAnnualSalary() {
                return this.salary * 12;
            }
        },
        {
            id: 2,
            name: 'Jane Smith',
            department: 'Marketing',
            salary: 80000,
            
            getAnnualSalary() {
                return this.salary * 12;
            }
        }
    ],
    
    departments: {
        engineering: {
            budget: 2000000,
            head: 'John Doe'
        },
        marketing: {
            budget: 500000,
            head: 'Jane Smith'
        }
    },
    
    // Methods
    getTotalEmployees() {
        return this.employees.length;
    },
    
    getEmployeeById(id) {
        return this.employees.find(emp => emp.id === id);
    },
    
    addEmployee(employee) {
        employee.id = this.employees.length + 1;
        this.employees.push(employee);
        return employee;
    },
    
    getTotalBudget() {
        return Object.values(this.departments)
            .reduce((total, dept) => total + dept.budget, 0);
    }
};

console.log(company.address.getFullAddress()); // "123 Main St, San Francisco, USA"
console.log(company.getTotalEmployees()); // 2
console.log(company.getTotalBudget()); // 2500000
```

## Factory Function Pattern

### Basic Factory Functions
```javascript
// Simple factory function
function createPerson(name, age, email) {
    return {
        name: name,
        age: age,
        email: email,
        
        greet() {
            return `Hello, I'm ${this.name}`;
        },
        
        getAge() {
            return this.age;
        },
        
        celebrateBirthday() {
            this.age++;
            return `Happy birthday! Now ${this.age} years old.`;
        }
    };
}

// Create instances
const person1 = createPerson('John', 30, 'john@example.com');
const person2 = createPerson('Jane', 25, 'jane@example.com');

console.log(person1.greet()); // "Hello, I'm John"
console.log(person2.celebrateBirthday()); // "Happy birthday! Now 26 years old."

// Factory with private variables (closure)
function createBankAccount(accountNumber, initialBalance) {
    // Private variables
    let balance = initialBalance || 0;
    let transactionHistory = [];
    
    // Private method
    function addTransaction(type, amount) {
        transactionHistory.push({
            type,
            amount,
            timestamp: new Date(),
            balance: balance
        });
    }
    
    // Return public interface
    return {
        // Public properties
        accountNumber: accountNumber,
        
        // Public methods
        deposit(amount) {
            if (amount <= 0) {
                throw new Error('Amount must be positive');
            }
            balance += amount;
            addTransaction('deposit', amount);
            return balance;
        },
        
        withdraw(amount) {
            if (amount <= 0) {
                throw new Error('Amount must be positive');
            }
            if (amount > balance) {
                throw new Error('Insufficient funds');
            }
            balance -= amount;
            addTransaction('withdrawal', amount);
            return balance;
        },
        
        getBalance() {
            return balance;
        },
        
        getTransactionHistory() {
            // Return copy to prevent external modification
            return transactionHistory.map(t => ({ ...t }));
        },
        
        transfer(targetAccount, amount) {
            if (amount > balance) {
                throw new Error('Insufficient funds for transfer');
            }
            this.withdraw(amount);
            targetAccount.deposit(amount);
            return `Transferred $${amount} to account ${targetAccount.accountNumber}`;
        }
    };
}

// Usage
const account1 = createBankAccount('12345', 1000);
const account2 = createBankAccount('67890', 500);

console.log(account1.deposit(200)); // 1200
console.log(account1.withdraw(100)); // 1100
console.log(account1.transfer(account2, 300)); // "Transferred $300 to account 67890"
console.log(account2.getBalance()); // 800

// Private variables are truly private
// console.log(account1.balance); // undefined
// account1.addTransaction('fake', 1000); // TypeError: not a function
```

### Advanced Factory Patterns
```javascript
// Factory with configuration
function createConfigurableTimer(config = {}) {
    const defaults = {
        interval: 1000,
        autoStart: false,
        maxTicks: Infinity,
        onTick: () => {},
        onComplete: () => {}
    };
    
    const settings = { ...defaults, ...config };
    let intervalId = null;
    let tickCount = 0;
    let isRunning = false;
    
    const timer = {
        start() {
            if (isRunning) return;
            
            isRunning = true;
            intervalId = setInterval(() => {
                tickCount++;
                settings.onTick(tickCount);
                
                if (tickCount >= settings.maxTicks) {
                    this.stop();
                    settings.onComplete();
                }
            }, settings.interval);
            
            return this;
        },
        
        stop() {
            if (intervalId) {
                clearInterval(intervalId);
                intervalId = null;
                isRunning = false;
            }
            return this;
        },
        
        reset() {
            this.stop();
            tickCount = 0;
            return this;
        },
        
        getTickCount() {
            return tickCount;
        },
        
        isRunning() {
            return isRunning;
        },
        
        configure(newConfig) {
            Object.assign(settings, newConfig);
            return this;
        }
    };
    
    if (settings.autoStart) {
        timer.start();
    }
    
    return timer;
}

// Usage
const countdown = createConfigurableTimer({
    interval: 1000,
    maxTicks: 5,
    onTick: (count) => console.log(`Countdown: ${6 - count}`),
    onComplete: () => console.log('Countdown finished!')
});

countdown.start();

// Factory with inheritance-like behavior
function createVehicle(type, options = {}) {
    const vehicle = {
        type: type,
        make: options.make || 'Unknown',
        model: options.model || 'Unknown',
        year: options.year || new Date().getFullYear(),
        isRunning: false,
        
        start() {
            this.isRunning = true;
            return `${this.make} ${this.model} started`;
        },
        
        stop() {
            this.isRunning = false;
            return `${this.make} ${this.model} stopped`;
        },
        
        getInfo() {
            return `${this.year} ${this.make} ${this.model}`;
        }
    };
    
    // Add type-specific methods
    if (type === 'car') {
        vehicle.doors = options.doors || 4;
        vehicle.honk = function() {
            return 'Beep beep!';
        };
    } else if (type === 'motorcycle') {
        vehicle.wheels = 2;
        vehicle.rev = function() {
            return 'Vroom vroom!';
        };
    } else if (type === 'truck') {
        vehicle.capacity = options.capacity || 1000;
        vehicle.loadCargo = function(weight) {
            if (weight > this.capacity) {
                return 'Cargo too heavy!';
            }
            return `Loaded ${weight}kg cargo`;
        };
    }
    
    return vehicle;
}

// Create different types of vehicles
const car = createVehicle('car', { make: 'Toyota', model: 'Camry', doors: 4 });
const bike = createVehicle('motorcycle', { make: 'Harley', model: 'Sportster' });
const truck = createVehicle('truck', { make: 'Ford', model: 'F-150', capacity: 1500 });

console.log(car.honk()); // "Beep beep!"
console.log(bike.rev()); // "Vroom vroom!"
console.log(truck.loadCargo(1000)); // "Loaded 1000kg cargo"
```

## Constructor Function Pattern

### Basic Constructor Functions
```javascript
// Traditional constructor function
function Person(name, age, email) {
    this.name = name;
    this.age = age;
    this.email = email;
    this.createdAt = new Date();
}

// Add methods to prototype
Person.prototype.greet = function() {
    return `Hello, I'm ${this.name}`;
};

Person.prototype.getAge = function() {
    return this.age;
};

Person.prototype.isAdult = function() {
    return this.age >= 18;
};

// Static method
Person.createAnonymous = function() {
    return new Person('Anonymous', 0, 'no-email@example.com');
};

// Usage
const john = new Person('John', 30, 'john@example.com');
const jane = new Person('Jane', 17, 'jane@example.com');

console.log(john.greet()); // "Hello, I'm John"
console.log(john.isAdult()); // true
console.log(jane.isAdult()); // false

// Check prototype chain
console.log(john instanceof Person); // true
console.log(john.constructor === Person); // true

// All instances share prototype methods
console.log(john.greet === jane.greet); // true
```

### Constructor with Private-like Properties
```javascript
// Constructor with closure for private data
function SecureUser(username, password) {
    // Private variables
    let _password = password;
    let _loginAttempts = 0;
    let _isLocked = false;
    
    // Public properties
    this.username = username;
    this.createdAt = new Date();
    this.lastLogin = null;
    
    // Privileged methods (have access to private variables)
    this.authenticate = function(inputPassword) {
        if (_isLocked) {
            throw new Error('Account is locked due to too many failed attempts');
        }
        
        if (_password === inputPassword) {
            _loginAttempts = 0;
            this.lastLogin = new Date();
            return true;
        } else {
            _loginAttempts++;
            if (_loginAttempts >= 3) {
                _isLocked = true;
            }
            return false;
        }
    };
    
    this.changePassword = function(oldPassword, newPassword) {
        if (!this.authenticate(oldPassword)) {
            throw new Error('Invalid current password');
        }
        
        if (newPassword.length < 8) {
            throw new Error('Password must be at least 8 characters');
        }
        
        _password = newPassword;
        return 'Password changed successfully';
    };
    
    this.getLoginAttempts = function() {
        return _loginAttempts;
    };
    
    this.isLocked = function() {
        return _isLocked;
    };
    
    this.unlock = function(adminKey) {
        if (adminKey === 'ADMIN_UNLOCK_KEY') {
            _isLocked = false;
            _loginAttempts = 0;
            return 'Account unlocked successfully';
        }
        throw new Error('Invalid admin key');
    };
}

// Add public methods to prototype
SecureUser.prototype.getInfo = function() {
    return {
        username: this.username,
        createdAt: this.createdAt,
        lastLogin: this.lastLogin,
        isLocked: this.isLocked()
    };
};

// Usage
const user = new SecureUser('alice', 'mypassword123');

console.log(user.authenticate('wrongpassword')); // false
console.log(user.authenticate('wrongpassword')); // false
console.log(user.authenticate('wrongpassword')); // false (account now locked)

try {
    user.authenticate('mypassword123');
} catch (error) {
    console.log(error.message); // "Account is locked..."
}

console.log(user.unlock('ADMIN_UNLOCK_KEY')); // "Account unlocked successfully"
console.log(user.authenticate('mypassword123')); // true

// Private variables are truly private
// console.log(user._password); // undefined
```

## Object.create() Pattern

### Basic Object.create()
```javascript
// Create object with specific prototype
const personPrototype = {
    greet() {
        return `Hello, I'm ${this.name}`;
    },
    
    getAge() {
        return this.age;
    },
    
    celebrateBirthday() {
        this.age++;
        return `Happy ${this.age}th birthday!`;
    }
};

// Create object with personPrototype as prototype
const john = Object.create(personPrototype);
john.name = 'John';
john.age = 30;

console.log(john.greet()); // "Hello, I'm John"
console.log(john.celebrateBirthday()); // "Happy 31th birthday!"

// Check prototype chain
console.log(Object.getPrototypeOf(john) === personPrototype); // true
console.log(john.hasOwnProperty('name')); // true
console.log(john.hasOwnProperty('greet')); // false (inherited)

// Create with property descriptors
const jane = Object.create(personPrototype, {
    name: {
        value: 'Jane',
        writable: true,
        enumerable: true,
        configurable: true
    },
    age: {
        value: 25,
        writable: true,
        enumerable: true,
        configurable: true
    },
    id: {
        value: Math.random().toString(36),
        writable: false,
        enumerable: false,
        configurable: false
    }
});

console.log(jane.greet()); // "Hello, I'm Jane"
console.log(Object.keys(jane)); // ['name', 'age'] (id is not enumerable)
```

### Prototype Inheritance Chain
```javascript
// Base prototype
const animalPrototype = {
    eat() {
        return `${this.name} is eating`;
    },
    
    sleep() {
        return `${this.name} is sleeping`;
    },
    
    makeSound() {
        return `${this.name} makes a sound`;
    }
};

// Dog prototype inheriting from animal
const dogPrototype = Object.create(animalPrototype);
dogPrototype.bark = function() {
    return `${this.name} barks: Woof!`;
};

dogPrototype.makeSound = function() {
    return this.bark(); // Override parent method
};

dogPrototype.fetch = function() {
    return `${this.name} fetches the ball`;
};

// Cat prototype inheriting from animal
const catPrototype = Object.create(animalPrototype);
catPrototype.meow = function() {
    return `${this.name} meows: Meow!`;
};

catPrototype.makeSound = function() {
    return this.meow(); // Override parent method
};

catPrototype.climb = function() {
    return `${this.name} climbs the tree`;
};

// Create specific animals
const dog = Object.create(dogPrototype);
dog.name = 'Buddy';
dog.breed = 'Golden Retriever';

const cat = Object.create(catPrototype);
cat.name = 'Whiskers';
cat.color = 'Orange';

console.log(dog.eat()); // "Buddy is eating" (inherited from animal)
console.log(dog.bark()); // "Buddy barks: Woof!" (own method)
console.log(dog.makeSound()); // "Buddy barks: Woof!" (overridden)

console.log(cat.sleep()); // "Whiskers is sleeping" (inherited from animal)
console.log(cat.meow()); // "Whiskers meows: Meow!" (own method)
console.log(cat.makeSound()); // "Whiskers meows: Meow!" (overridden)

// Check inheritance chain
console.log(Object.getPrototypeOf(dog) === dogPrototype); // true
console.log(Object.getPrototypeOf(dogPrototype) === animalPrototype); // true
console.log(Object.getPrototypeOf(animalPrototype) === Object.prototype); // true
```

### Mixin Pattern with Object.create()
```javascript
// Mixin objects
const flyable = {
    fly() {
        return `${this.name} is flying through the sky`;
    },
    
    land() {
        return `${this.name} has landed`;
    }
};

const swimmable = {
    swim() {
        return `${this.name} is swimming in the water`;
    },
    
    dive() {
        return `${this.name} dives underwater`;
    }
};

const walkable = {
    walk() {
        return `${this.name} is walking on land`;
    },
    
    run() {
        return `${this.name} is running fast`;
    }
};

// Create base animal prototype
const baseAnimal = {
    name: '',
    species: '',
    
    introduce() {
        return `I am ${this.name}, a ${this.species}`;
    }
};

// Create duck (can fly, swim, and walk)
const duckPrototype = Object.create(baseAnimal);
Object.assign(duckPrototype, flyable, swimmable, walkable);

duckPrototype.quack = function() {
    return `${this.name} says: Quack quack!`;
};

// Create fish (can only swim)
const fishPrototype = Object.create(baseAnimal);
Object.assign(fishPrototype, swimmable);

fishPrototype.bubble = function() {
    return `${this.name} blows bubbles`;
};

// Create bird (can fly and walk)
const birdPrototype = Object.create(baseAnimal);
Object.assign(birdPrototype, flyable, walkable);

birdPrototype.chirp = function() {
    return `${this.name} chirps melodiously`;
};

// Create instances
const duck = Object.create(duckPrototype);
duck.name = 'Donald';
duck.species = 'Duck';

const fish = Object.create(fishPrototype);
fish.name = 'Nemo';
fish.species = 'Clownfish';

const bird = Object.create(birdPrototype);
bird.name = 'Tweety';
bird.species = 'Canary';

// Test abilities
console.log(duck.introduce()); // "I am Donald, a Duck"
console.log(duck.fly()); // "Donald is flying through the sky"
console.log(duck.swim()); // "Donald is swimming in the water"
console.log(duck.walk()); // "Donald is walking on land"
console.log(duck.quack()); // "Donald says: Quack quack!"

console.log(fish.swim()); // "Nemo is swimming in the water"
console.log(fish.dive()); // "Nemo dives underwater"
// console.log(fish.fly()); // TypeError: fish.fly is not a function

console.log(bird.fly()); // "Tweety is flying through the sky"
console.log(bird.walk()); // "Tweety is walking on land"
// console.log(bird.swim()); // TypeError: bird.swim is not a function
```

## Module Pattern

### Immediately Invoked Function Expression (IIFE) Module
```javascript
// Basic module pattern
const PersonModule = (function() {
    // Private variables and functions
    let personCount = 0;
    const persons = [];
    
    function generateId() {
        return Math.random().toString(36).substr(2, 9);
    }
    
    function validateData(data) {
        if (!data.name || typeof data.name !== 'string') {
            throw new Error('Name is required and must be a string');
        }
        if (!data.age || typeof data.age !== 'number' || data.age < 0) {
            throw new Error('Age is required and must be a positive number');
        }
    }
    
    // Public API
    return {
        create(data) {
            validateData(data);
            
            const person = {
                id: generateId(),
                name: data.name,
                age: data.age,
                email: data.email || '',
                createdAt: new Date(),
                
                greet() {
                    return `Hello, I'm ${this.name}`;
                },
                
                getAge() {
                    return this.age;
                }
            };
            
            persons.push(person);
            personCount++;
            
            return person;
        },
        
        findById(id) {
            return persons.find(person => person.id === id);
        },
        
        findByName(name) {
            return persons.filter(person => 
                person.name.toLowerCase().includes(name.toLowerCase())
            );
        },
        
        getAll() {
            return [...persons]; // Return copy to prevent external modification
        },
        
        getCount() {
            return personCount;
        },
        
        remove(id) {
            const index = persons.findIndex(person => person.id === id);
            if (index !== -1) {
                const removed = persons.splice(index, 1)[0];
                personCount--;
                return removed;
            }
            return null;
        },
        
        clear() {
            persons.length = 0;
            personCount = 0;
        }
    };
})();

// Usage
const john = PersonModule.create({ name: 'John', age: 30, email: 'john@example.com' });
const jane = PersonModule.create({ name: 'Jane', age: 25 });

console.log(john.greet()); // "Hello, I'm John"
console.log(PersonModule.getCount()); // 2
console.log(PersonModule.findByName('j')); // [john, jane]

// Private variables are not accessible
// console.log(PersonModule.personCount); // undefined
// PersonModule.generateId(); // TypeError: not a function
```

### Revealing Module Pattern
```javascript
// Revealing module pattern
const CalculatorModule = (function() {
    // Private variables
    let result = 0;
    let history = [];
    
    // Private methods
    function addToHistory(operation, operand, result) {
        history.push({
            operation,
            operand,
            result,
            timestamp: new Date()
        });
    }
    
    function validateNumber(num) {
        if (typeof num !== 'number' || isNaN(num)) {
            throw new Error('Invalid number provided');
        }
    }
    
    // Public methods
    function add(num) {
        validateNumber(num);
        result += num;
        addToHistory('add', num, result);
        return result;
    }
    
    function subtract(num) {
        validateNumber(num);
        result -= num;
        addToHistory('subtract', num, result);
        return result;
    }
    
    function multiply(num) {
        validateNumber(num);
        result *= num;
        addToHistory('multiply', num, result);
        return result;
    }
    
    function divide(num) {
        validateNumber(num);
        if (num === 0) {
            throw new Error('Division by zero is not allowed');
        }
        result /= num;
        addToHistory('divide', num, result);
        return result;
    }
    
    function getResult() {
        return result;
    }
    
    function clear() {
        result = 0;
        history = [];
        return result;
    }
    
    function getHistory() {
        return [...history];
    }
    
    function setValue(num) {
        validateNumber(num);
        result = num;
        addToHistory('setValue', num, result);
        return result;
    }
    
    // Reveal public methods
    return {
        add: add,
        subtract: subtract,
        multiply: multiply,
        divide: divide,
        result: getResult,
        clear: clear,
        history: getHistory,
        set: setValue
    };
})();

// Usage
console.log(CalculatorModule.set(10)); // 10
console.log(CalculatorModule.add(5)); // 15
console.log(CalculatorModule.multiply(2)); // 30
console.log(CalculatorModule.subtract(10)); // 20
console.log(CalculatorModule.divide(4)); // 5
console.log(CalculatorModule.result()); // 5
console.log(CalculatorModule.history()); // Array of operations
```

## Prototype Pattern

### Object.setPrototypeOf() and Prototype Manipulation
```javascript
// Base prototype object
const vehiclePrototype = {
    start() {
        this.isRunning = true;
        return `${this.make} ${this.model} started`;
    },
    
    stop() {
        this.isRunning = false;
        return `${this.make} ${this.model} stopped`;
    },
    
    getInfo() {
        return `${this.year} ${this.make} ${this.model}`;
    }
};

// Create object and set prototype
const car = {
    make: 'Toyota',
    model: 'Camry',
    year: 2020,
    doors: 4,
    isRunning: false
};

// Set prototype after object creation
Object.setPrototypeOf(car, vehiclePrototype);

console.log(car.start()); // "Toyota Camry started"
console.log(car.getInfo()); // "2020 Toyota Camry"

// Create another prototype that extends vehicle
const carPrototype = Object.create(vehiclePrototype);
carPrototype.honk = function() {
    return 'Beep beep!';
};

carPrototype.openDoors = function() {
    return `Opening ${this.doors} doors`;
};

// Create car with car prototype
const sportsCar = Object.create(carPrototype);
sportsCar.make = 'Ferrari';
sportsCar.model = 'F488';
sportsCar.year = 2021;
sportsCar.doors = 2;
sportsCar.topSpeed = 205;

console.log(sportsCar.start()); // "Ferrari F488 started"
console.log(sportsCar.honk()); // "Beep beep!"
console.log(sportsCar.openDoors()); // "Opening 2 doors"

// Dynamic prototype modification
vehiclePrototype.getAge = function() {
    return new Date().getFullYear() - this.year;
};

// All objects with vehiclePrototype now have getAge method
console.log(car.getAge()); // Current year - 2020
console.log(sportsCar.getAge()); // Current year - 2021
```

### Prototype Chain Navigation
```javascript
// Create a complex prototype chain
const livingBeing = {
    isAlive: true,
    breathe() {
        return `${this.name || 'Being'} is breathing`;
    }
};

const animal = Object.create(livingBeing);
animal.move = function() {
    return `${this.name || 'Animal'} is moving`;
};
animal.eat = function() {
    return `${this.name || 'Animal'} is eating`;
};

const mammal = Object.create(animal);
mammal.feedMilk = function() {
    return `${this.name || 'Mammal'} feeds milk to offspring`;
};
mammal.hasHair = true;

const human = Object.create(mammal);
human.think = function() {
    return `${this.name || 'Human'} is thinking`;
};
human.speak = function() {
    return `${this.name || 'Human'} is speaking`;
};

// Create a specific human instance
const person = Object.create(human);
person.name = 'Alice';
person.age = 30;

// Test the prototype chain
console.log(person.breathe()); // "Alice is breathing" (from livingBeing)
console.log(person.move()); // "Alice is moving" (from animal)
console.log(person.feedMilk()); // "Alice feeds milk to offspring" (from mammal)
console.log(person.think()); // "Alice is thinking" (from human)

// Check prototype chain
function getPrototypeChain(obj) {
    const chain = [];
    let current = obj;
    
    while (current) {
        chain.push(current);
        current = Object.getPrototypeOf(current);
    }
    
    return chain;
}

const chain = getPrototypeChain(person);
console.log('Prototype chain length:', chain.length);
console.log('Chain:', chain.map(obj => obj.constructor?.name || 'Object'));

// Navigate the chain
console.log(Object.getPrototypeOf(person) === human); // true
console.log(Object.getPrototypeOf(human) === mammal); // true
console.log(Object.getPrototypeOf(mammal) === animal); // true
console.log(Object.getPrototypeOf(animal) === livingBeing); // true
console.log(Object.getPrototypeOf(livingBeing) === Object.prototype); // true
console.log(Object.getPrototypeOf(Object.prototype) === null); // true

// Check for properties in chain
console.log('hasHair' in person); // true (inherited from mammal)
console.log(person.hasOwnProperty('hasHair')); // false
console.log(person.hasOwnProperty('name')); // true
```

## Best Practices and Performance Considerations

### Choosing the Right Pattern
```javascript
// Performance comparison helper
function measurePerformance(fn, iterations = 100000) {
    const start = performance.now();
    for (let i = 0; i < iterations; i++) {
        fn();
    }
    const end = performance.now();
    return end - start;
}

// Object literal (fastest for simple objects)
const literalTime = measurePerformance(() => {
    const obj = {
        name: 'Test',
        value: 42,
        method() { return this.value; }
    };
});

// Factory function (slower but flexible)
function createObject() {
    return {
        name: 'Test',
        value: 42,
        method() { return this.value; }
    };
}
const factoryTime = measurePerformance(createObject);

// Constructor function (good performance, prototypal inheritance)
function ObjectConstructor() {
    this.name = 'Test';
    this.value = 42;
}
ObjectConstructor.prototype.method = function() { return this.value; };
const constructorTime = measurePerformance(() => new ObjectConstructor());

// Object.create (flexible but slower)
const prototype = {
    method() { return this.value; }
};
const createTime = measurePerformance(() => {
    const obj = Object.create(prototype);
    obj.name = 'Test';
    obj.value = 42;
    return obj;
});

console.log('Performance comparison (lower is better):');
console.log('Object literal:', literalTime.toFixed(2), 'ms');
console.log('Factory function:', factoryTime.toFixed(2), 'ms');
console.log('Constructor function:', constructorTime.toFixed(2), 'ms');
console.log('Object.create:', createTime.toFixed(2), 'ms');

// Memory usage considerations
function createObjectsComparison() {
    const objects1 = [];
    const objects2 = [];
    
    // Bad: Each object has its own method (more memory)
    for (let i = 0; i < 1000; i++) {
        objects1.push({
            id: i,
            getValue() { return this.id; }
        });
    }
    
    // Good: Shared method via prototype (less memory)
    function MyObject(id) {
        this.id = id;
    }
    MyObject.prototype.getValue = function() { return this.id; };
    
    for (let i = 0; i < 1000; i++) {
        objects2.push(new MyObject(i));
    }
    
    console.log('Objects with individual methods:', objects1.length);
    console.log('Objects with shared prototype method:', objects2.length);
    console.log('Method comparison:', objects1[0].getValue === objects1[1].getValue); // false
    console.log('Prototype method comparison:', objects2[0].getValue === objects2[1].getValue); // true
}

createObjectsComparison();
```

### Pattern Selection Guidelines
```javascript
// Guidelines for choosing object creation patterns

const PatternGuidelines = {
    // Use object literals when:
    objectLiteral: {
        useCases: [
            'Creating simple, one-off objects',
            'Configuration objects',
            'Data transfer objects',
            'Small objects without complex behavior'
        ],
        
        example() {
            const config = {
                apiUrl: 'https://api.example.com',
                timeout: 5000,
                retries: 3
            };
            return config;
        }
    },
    
    // Use factory functions when:
    factoryFunction: {
        useCases: [
            'Need private variables (closure)',
            'Conditional object creation',
            'Want to avoid "new" keyword',
            'Complex initialization logic'
        ],
        
        example() {
            function createUser(type, data) {
                const user = {
                    ...data,
                    type: type,
                    createdAt: new Date()
                };
                
                if (type === 'admin') {
                    user.permissions = ['read', 'write', 'delete'];
                } else {
                    user.permissions = ['read'];
                }
                
                return user;
            }
            
            return createUser('admin', { name: 'Alice' });
        }
    },
    
    // Use constructor functions when:
    constructorFunction: {
        useCases: [
            'Need instanceof checks',
            'Want prototypal inheritance',
            'Creating many similar objects',
            'Framework/library compatibility'
        ],
        
        example() {
            function User(name, role) {
                this.name = name;
                this.role = role;
                this.createdAt = new Date();
            }
            
            User.prototype.hasPermission = function(permission) {
                return this.role === 'admin' || permission === 'read';
            };
            
            return new User('Bob', 'user');
        }
    },
    
    // Use Object.create when:
    objectCreate: {
        useCases: [
            'Fine-grained prototype control',
            'Implementing inheritance manually',
            'Need specific prototype chains',
            'Educational/experimental purposes'
        ],
        
        example() {
            const userPrototype = {
                greet() { return `Hello, I'm ${this.name}`; }
            };
            
            const user = Object.create(userPrototype);
            user.name = 'Charlie';
            
            return user;
        }
    },
    
    // Use ES6 classes when:
    es6Classes: {
        useCases: [
            'Modern JavaScript projects',
            'Clear inheritance hierarchies',
            'Need private fields/methods',
            'Team familiar with class syntax'
        ],
        
        example() {
            class User {
                constructor(name, role) {
                    this.name = name;
                    this.role = role;
                    this.createdAt = new Date();
                }
                
                hasPermission(permission) {
                    return this.role === 'admin' || permission === 'read';
                }
            }
            
            return new User('David', 'admin');
        }
    },
    
    // Use module pattern when:
    modulePattern: {
        useCases: [
            'Creating namespaces',
            'Encapsulating related functionality',
            'Need private state/methods',
            'Building libraries/utilities'
        ],
        
        example() {
            const UserManager = (function() {
                const users = [];
                
                return {
                    addUser(user) {
                        users.push(user);
                    },
                    getUsers() {
                        return [...users];
                    },
                    getUserCount() {
                        return users.length;
                    }
                };
            })();
            
            return UserManager;
        }
    }
};

// Demonstrate each pattern
console.log('Pattern Examples:');
Object.keys(PatternGuidelines).forEach(pattern => {
    console.log(`\n${pattern}:`);
    console.log('Use cases:', PatternGuidelines[pattern].useCases);
    console.log('Example result:', PatternGuidelines[pattern].example());
});
```

---

## Related Topics
- [[06 - Objects]]
- [[07 - Prototypes and Prototypal Inheritance]]
- [[08 - Classes and Constructor Functions]]
- [[35 - JavaScript OOP]]
- [[38 - ES6+ Features]]

---

*Next: [[10 - Closures]]*
