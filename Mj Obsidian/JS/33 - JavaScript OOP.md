# JavaScript OOP

## Overview
Object-Oriented Programming (OOP) in JavaScript provides powerful ways to organize and structure code using objects, classes, inheritance, and encapsulation. JavaScript supports both prototype-based and class-based OOP paradigms, offering flexibility in how you design your applications.

## ES6 Classes

### Basic Class Syntax
```javascript
// Basic class definition
class Person {
    // Constructor method
    constructor(name, age, email) {
        this.name = name;
        this.age = age;
        this.email = email;
        this.id = Math.random().toString(36).substr(2, 9);
    }
    
    // Instance methods
    introduce() {
        return `Hi, I'm ${this.name} and I'm ${this.age} years old.`;
    }
    
    celebrateBirthday() {
        this.age++;
        console.log(`${this.name} is now ${this.age} years old!`);
    }
    
    updateEmail(newEmail) {
        if (this.isValidEmail(newEmail)) {
            this.email = newEmail;
            return true;
        }
        return false;
    }
    
    // Private method (using # syntax - ES2022)
    isValidEmail(email) {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return emailRegex.test(email);
    }
    
    // Getter method
    get fullInfo() {
        return {
            name: this.name,
            age: this.age,
            email: this.email,
            id: this.id
        };
    }
    
    // Setter method
    set personAge(age) {
        if (age >= 0 && age <= 150) {
            this.age = age;
        } else {
            throw new Error('Age must be between 0 and 150');
        }
    }
    
    // Static method (belongs to class, not instance)
    static createRandomPerson() {
        const names = ['Alice', 'Bob', 'Charlie', 'Diana', 'Edward'];
        const randomName = names[Math.floor(Math.random() * names.length)];
        const randomAge = Math.floor(Math.random() * 50) + 18;
        const randomEmail = `${randomName.toLowerCase()}@example.com`;
        
        return new Person(randomName, randomAge, randomEmail);
    }
    
    static validateAge(age) {
        return typeof age === 'number' && age >= 0 && age <= 150;
    }
    
    // toString method override
    toString() {
        return `Person(${this.name}, ${this.age})`;
    }
    
    // JSON serialization
    toJSON() {
        return {
            name: this.name,
            age: this.age,
            email: this.email,
            id: this.id
        };
    }
}

// Using the class
const person1 = new Person('John Doe', 30, 'john@example.com');
console.log(person1.introduce()); // "Hi, I'm John Doe and I'm 30 years old."

person1.celebrateBirthday(); // "John Doe is now 31 years old!"

// Using getter and setter
console.log(person1.fullInfo);
person1.personAge = 35;

// Using static methods
const randomPerson = Person.createRandomPerson();
console.log(Person.validateAge(25)); // true

// Class instantiation check
console.log(person1 instanceof Person); // true
```

### Advanced Class Features
```javascript
// Class with private fields and methods (ES2022)
class BankAccount {
    // Private fields
    #balance = 0;
    #accountNumber;
    #transactions = [];
    
    // Static private field
    static #nextAccountNumber = 1000;
    
    constructor(ownerName, initialDeposit = 0) {
        this.ownerName = ownerName;
        this.#accountNumber = BankAccount.#nextAccountNumber++;
        this.createdAt = new Date();
        
        if (initialDeposit > 0) {
            this.#balance = initialDeposit;
            this.#recordTransaction('deposit', initialDeposit, 'Initial deposit');
        }
    }
    
    // Private method
    #recordTransaction(type, amount, description) {
        this.#transactions.push({
            id: Date.now(),
            type,
            amount,
            description,
            timestamp: new Date(),
            balanceAfter: this.#balance
        });
    }
    
    // Private method for validation
    #validateAmount(amount) {
        if (typeof amount !== 'number' || amount <= 0) {
            throw new Error('Amount must be a positive number');
        }
    }
    
    // Public methods
    deposit(amount, description = 'Deposit') {
        this.#validateAmount(amount);
        this.#balance += amount;
        this.#recordTransaction('deposit', amount, description);
        return this.#balance;
    }
    
    withdraw(amount, description = 'Withdrawal') {
        this.#validateAmount(amount);
        
        if (amount > this.#balance) {
            throw new Error('Insufficient funds');
        }
        
        this.#balance -= amount;
        this.#recordTransaction('withdrawal', amount, description);
        return this.#balance;
    }
    
    transfer(amount, targetAccount, description = 'Transfer') {
        if (!(targetAccount instanceof BankAccount)) {
            throw new Error('Target must be a BankAccount instance');
        }
        
        this.withdraw(amount, `${description} to ${targetAccount.ownerName}`);
        targetAccount.deposit(amount, `${description} from ${this.ownerName}`);
        
        return {
            fromBalance: this.#balance,
            toBalance: targetAccount.getBalance()
        };
    }
    
    // Getter for balance (controlled access to private field)
    getBalance() {
        return this.#balance;
    }
    
    // Getter for account number
    getAccountNumber() {
        return this.#accountNumber;
    }
    
    // Get transaction history
    getTransactions(limit = null) {
        const transactions = [...this.#transactions]; // Return copy to prevent mutation
        return limit ? transactions.slice(-limit) : transactions;
    }
    
    // Calculate interest
    calculateInterest(rate = 0.02) {
        const interest = this.#balance * rate;
        return interest;
    }
    
    applyInterest(rate = 0.02) {
        const interest = this.calculateInterest(rate);
        if (interest > 0) {
            this.deposit(interest, `Interest applied at ${(rate * 100).toFixed(2)}%`);
        }
        return interest;
    }
    
    // Account summary
    getSummary() {
        return {
            ownerName: this.ownerName,
            accountNumber: this.#accountNumber,
            balance: this.#balance,
            transactionCount: this.#transactions.length,
            createdAt: this.createdAt,
            lastTransaction: this.#transactions[this.#transactions.length - 1] || null
        };
    }
    
    // Static method to create account with validation
    static createAccount(ownerName, initialDeposit = 0) {
        if (!ownerName || typeof ownerName !== 'string') {
            throw new Error('Owner name is required and must be a string');
        }
        
        if (initialDeposit < 0) {
            throw new Error('Initial deposit cannot be negative');
        }
        
        return new BankAccount(ownerName, initialDeposit);
    }
    
    toString() {
        return `BankAccount(${this.ownerName}, $${this.#balance.toFixed(2)})`;
    }
}

// Usage examples
const account1 = BankAccount.createAccount('Alice Johnson', 1000);
const account2 = BankAccount.createAccount('Bob Smith', 500);

console.log(account1.getSummary());

// Perform transactions
account1.deposit(200, 'Salary deposit');
account1.withdraw(50, 'ATM withdrawal');
account1.transfer(100, account2, 'Payment to Bob');

// Apply interest
account1.applyInterest(0.05); // 5% interest

console.log('Account 1 Balance:', account1.getBalance());
console.log('Account 2 Balance:', account2.getBalance());
console.log('Recent transactions:', account1.getTransactions(3));

// Private fields are truly private
try {
    console.log(account1.#balance); // SyntaxError: Private field '#balance' must be declared in an enclosing class
} catch (error) {
    console.log('Cannot access private fields directly');
}
```

## Class Inheritance

### Basic Inheritance
```javascript
// Base class
class Animal {
    constructor(name, species, age) {
        this.name = name;
        this.species = species;
        this.age = age;
        this.isAlive = true;
    }
    
    makeSound() {
        return 'Some generic animal sound';
    }
    
    move() {
        return `${this.name} is moving`;
    }
    
    eat(food) {
        return `${this.name} is eating ${food}`;
    }
    
    sleep() {
        return `${this.name} is sleeping`;
    }
    
    getInfo() {
        return {
            name: this.name,
            species: this.species,
            age: this.age,
            isAlive: this.isAlive
        };
    }
    
    toString() {
        return `${this.species}(${this.name}, ${this.age} years old)`;
    }
}

// Derived class - Dog
class Dog extends Animal {
    constructor(name, breed, age, isVaccinated = false) {
        super(name, 'Dog', age); // Call parent constructor
        this.breed = breed;
        this.isVaccinated = isVaccinated;
        this.tricks = [];
    }
    
    // Override parent method
    makeSound() {
        return 'Woof! Woof!';
    }
    
    // Override parent method with additional functionality
    move() {
        return `${super.move()} by running on four legs`;
    }
    
    // Dog-specific methods
    bark() {
        return this.makeSound();
    }
    
    wagTail() {
        return `${this.name} is wagging their tail happily!`;
    }
    
    learnTrick(trick) {
        if (!this.tricks.includes(trick)) {
            this.tricks.push(trick);
            return `${this.name} learned the trick: ${trick}`;
        }
        return `${this.name} already knows this trick`;
    }
    
    performTrick(trick) {
        if (this.tricks.includes(trick)) {
            return `${this.name} performs: ${trick}`;
        }
        return `${this.name} doesn't know the trick: ${trick}`;
    }
    
    vaccinate() {
        this.isVaccinated = true;
        return `${this.name} has been vaccinated`;
    }
    
    // Override getInfo to include dog-specific information
    getInfo() {
        return {
            ...super.getInfo(), // Get parent info
            breed: this.breed,
            isVaccinated: this.isVaccinated,
            tricks: [...this.tricks] // Return copy of tricks array
        };
    }
}

// Derived class - Cat
class Cat extends Animal {
    constructor(name, breed, age, isIndoor = true) {
        super(name, 'Cat', age);
        this.breed = breed;
        this.isIndoor = isIndoor;
        this.livesLeft = 9;
    }
    
    makeSound() {
        return 'Meow!';
    }
    
    move() {
        return `${super.move()} gracefully and silently`;
    }
    
    purr() {
        return `${this.name} is purring contentedly`;
    }
    
    scratch(target = 'furniture') {
        return `${this.name} is scratching the ${target}`;
    }
    
    climb() {
        if (this.isIndoor) {
            return `${this.name} climbs the cat tree`;
        } else {
            return `${this.name} climbs a tree outside`;
        }
    }
    
    loseLife() {
        if (this.livesLeft > 0) {
            this.livesLeft--;
            return `${this.name} lost a life. ${this.livesLeft} lives remaining.`;
        } else {
            this.isAlive = false;
            return `${this.name} has no more lives left`;
        }
    }
    
    getInfo() {
        return {
            ...super.getInfo(),
            breed: this.breed,
            isIndoor: this.isIndoor,
            livesLeft: this.livesLeft
        };
    }
}

// Further inheritance - Service Dog
class ServiceDog extends Dog {
    constructor(name, breed, age, serviceType, handler) {
        super(name, breed, age, true); // Service dogs are always vaccinated
        this.serviceType = serviceType; // 'guide', 'hearing', 'mobility', 'medical', etc.
        this.handler = handler;
        this.isWorking = false;
        this.certificationDate = new Date();
    }
    
    startWork() {
        this.isWorking = true;
        return `${this.name} is now working as a ${this.serviceType} dog`;
    }
    
    stopWork() {
        this.isWorking = false;
        return `${this.name} is now off duty`;
    }
    
    performService() {
        if (!this.isWorking) {
            return `${this.name} is not currently working`;
        }
        
        switch (this.serviceType) {
            case 'guide':
                return `${this.name} is guiding ${this.handler} safely`;
            case 'hearing':
                return `${this.name} is alerting ${this.handler} to sounds`;
            case 'mobility':
                return `${this.name} is providing mobility assistance to ${this.handler}`;
            case 'medical':
                return `${this.name} is monitoring ${this.handler}'s medical condition`;
            default:
                return `${this.name} is performing ${this.serviceType} service`;
        }
    }
    
    // Override bark method - service dogs are trained to bark only when necessary
    bark() {
        if (this.isWorking) {
            return `${this.name} gives a controlled alert bark`;
        } else {
            return super.bark();
        }
    }
    
    getInfo() {
        return {
            ...super.getInfo(),
            serviceType: this.serviceType,
            handler: this.handler,
            isWorking: this.isWorking,
            certificationDate: this.certificationDate
        };
    }
}

// Usage examples
const genericAnimal = new Animal('Unknown', 'Generic', 5);
const dog = new Dog('Buddy', 'Golden Retriever', 3);
const cat = new Cat('Whiskers', 'Persian', 2, true);
const serviceDog = new ServiceDog('Max', 'German Shepherd', 4, 'guide', 'John Smith');

// Test inheritance and polymorphism
const animals = [genericAnimal, dog, cat, serviceDog];

animals.forEach(animal => {
    console.log(animal.toString());
    console.log(animal.makeSound());
    console.log(animal.move());
    console.log('---');
});

// Test method overriding and specific functionality
console.log(dog.learnTrick('sit'));
console.log(dog.learnTrick('fetch'));
console.log(dog.performTrick('sit'));

console.log(cat.purr());
console.log(cat.climb());
console.log(cat.loseLife());

console.log(serviceDog.startWork());
console.log(serviceDog.performService());
console.log(serviceDog.bark());

// Test instanceof
console.log(dog instanceof Dog); // true
console.log(dog instanceof Animal); // true
console.log(serviceDog instanceof ServiceDog); // true
console.log(serviceDog instanceof Dog); // true
console.log(serviceDog instanceof Animal); // true
console.log(cat instanceof Dog); // false
```

### Abstract Classes and Methods
```javascript
// Abstract base class (simulated in JavaScript)
class Shape {
    constructor(color = 'black') {
        if (this.constructor === Shape) {
            throw new Error('Cannot instantiate abstract class Shape directly');
        }
        this.color = color;
        this.created = new Date();
    }
    
    // Abstract method - must be implemented by subclasses
    calculateArea() {
        throw new Error('calculateArea() method must be implemented by subclass');
    }
    
    calculatePerimeter() {
        throw new Error('calculatePerimeter() method must be implemented by subclass');
    }
    
    // Concrete method - can be used by all subclasses
    getColor() {
        return this.color;
    }
    
    setColor(color) {
        this.color = color;
    }
    
    getAge() {
        return Date.now() - this.created.getTime();
    }
    
    // Template method pattern
    describe() {
        return `This is a ${this.color} ${this.constructor.name.toLowerCase()} with area ${this.calculateArea().toFixed(2)} and perimeter ${this.calculatePerimeter().toFixed(2)}`;
    }
    
    // Static method to create shape factory
    static createShape(type, ...args) {
        switch (type.toLowerCase()) {
            case 'circle':
                return new Circle(...args);
            case 'rectangle':
                return new Rectangle(...args);
            case 'triangle':
                return new Triangle(...args);
            default:
                throw new Error(`Unknown shape type: ${type}`);
        }
    }
}

// Concrete implementation - Circle
class Circle extends Shape {
    constructor(radius, color = 'black') {
        super(color);
        this.radius = radius;
    }
    
    calculateArea() {
        return Math.PI * this.radius * this.radius;
    }
    
    calculatePerimeter() {
        return 2 * Math.PI * this.radius;
    }
    
    getDiameter() {
        return 2 * this.radius;
    }
    
    // Method specific to Circle
    isPointInside(x, y, centerX = 0, centerY = 0) {
        const distance = Math.sqrt((x - centerX) ** 2 + (y - centerY) ** 2);
        return distance <= this.radius;
    }
}

// Concrete implementation - Rectangle
class Rectangle extends Shape {
    constructor(width, height, color = 'black') {
        super(color);
        this.width = width;
        this.height = height;
    }
    
    calculateArea() {
        return this.width * this.height;
    }
    
    calculatePerimeter() {
        return 2 * (this.width + this.height);
    }
    
    // Method specific to Rectangle
    isSquare() {
        return this.width === this.height;
    }
    
    getDiagonal() {
        return Math.sqrt(this.width ** 2 + this.height ** 2);
    }
}

// Concrete implementation - Triangle
class Triangle extends Shape {
    constructor(side1, side2, side3, color = 'black') {
        super(color);
        this.side1 = side1;
        this.side2 = side2;
        this.side3 = side3;
        
        // Validate triangle inequality
        if (!this.isValidTriangle()) {
            throw new Error('Invalid triangle: sides do not satisfy triangle inequality');
        }
    }
    
    isValidTriangle() {
        return (this.side1 + this.side2 > this.side3) &&
               (this.side1 + this.side3 > this.side2) &&
               (this.side2 + this.side3 > this.side1);
    }
    
    calculatePerimeter() {
        return this.side1 + this.side2 + this.side3;
    }
    
    calculateArea() {
        // Using Heron's formula
        const s = this.calculatePerimeter() / 2;
        return Math.sqrt(s * (s - this.side1) * (s - this.side2) * (s - this.side3));
    }
    
    // Method specific to Triangle
    getTriangleType() {
        if (this.side1 === this.side2 && this.side2 === this.side3) {
            return 'equilateral';
        } else if (this.side1 === this.side2 || this.side2 === this.side3 || this.side1 === this.side3) {
            return 'isosceles';
        } else {
            return 'scalene';
        }
    }
}

// Usage examples
try {
    // This will throw an error
    const abstractShape = new Shape('red');
} catch (error) {
    console.log(error.message); // "Cannot instantiate abstract class Shape directly"
}

// Create concrete shapes
const circle = new Circle(5, 'blue');
const rectangle = new Rectangle(4, 6, 'red');
const triangle = new Triangle(3, 4, 5, 'green');

// Test polymorphism
const shapes = [circle, rectangle, triangle];

shapes.forEach(shape => {
    console.log(shape.describe());
    console.log(`Type: ${shape.constructor.name}`);
    console.log(`Is square: ${shape.isSquare ? shape.isSquare() : 'N/A'}`);
    console.log('---');
});

// Use factory method
const factoryCircle = Shape.createShape('circle', 3, 'yellow');
const factoryRectangle = Shape.createShape('rectangle', 2, 8, 'purple');

console.log(factoryCircle.describe());
console.log(factoryRectangle.describe());
```

## Prototype-based OOP

### Understanding Prototypes
```javascript
// Constructor function (pre-ES6 way)
function Vehicle(make, model, year) {
    this.make = make;
    this.model = model;
    this.year = year;
    this.isRunning = false;
}

// Adding methods to prototype
Vehicle.prototype.start = function() {
    this.isRunning = true;
    return `${this.make} ${this.model} is starting`;
};

Vehicle.prototype.stop = function() {
    this.isRunning = false;
    return `${this.make} ${this.model} has stopped`;
};

Vehicle.prototype.getInfo = function() {
    return `${this.year} ${this.make} ${this.model}`;
};

Vehicle.prototype.getAge = function() {
    return new Date().getFullYear() - this.year;
};

// Static method on constructor
Vehicle.createVehicle = function(make, model, year) {
    return new Vehicle(make, model, year);
};

// Car constructor inheriting from Vehicle
function Car(make, model, year, doors, fuelType) {
    Vehicle.call(this, make, model, year); // Call parent constructor
    this.doors = doors;
    this.fuelType = fuelType;
    this.gear = 'P'; // Park
}

// Set up inheritance
Car.prototype = Object.create(Vehicle.prototype);
Car.prototype.constructor = Car;

// Add Car-specific methods
Car.prototype.honk = function() {
    return 'Beep beep!';
};

Car.prototype.changeGear = function(newGear) {
    const validGears = ['P', 'R', 'N', 'D', '1', '2', '3'];
    if (validGears.includes(newGear)) {
        this.gear = newGear;
        return `Changed gear to ${newGear}`;
    }
    return 'Invalid gear';
};

// Override parent method
Car.prototype.start = function() {
    if (this.gear === 'P') {
        this.isRunning = true;
        return `${this.make} ${this.model} engine started. Ready to drive.`;
    }
    return 'Cannot start: gear must be in Park';
};

// Motorcycle constructor
function Motorcycle(make, model, year, engineSize) {
    Vehicle.call(this, make, model, year);
    this.engineSize = engineSize;
    this.wheelieMode = false;
}

Motorcycle.prototype = Object.create(Vehicle.prototype);
Motorcycle.prototype.constructor = Motorcycle;

Motorcycle.prototype.wheelie = function() {
    if (this.isRunning) {
        this.wheelieMode = !this.wheelieMode;
        return this.wheelieMode ? 'Pulling a wheelie!' : 'Back on two wheels';
    }
    return 'Cannot do wheelie: engine not running';
};

// Usage
const car = new Car('Toyota', 'Camry', 2020, 4, 'gasoline');
const motorcycle = new Motorcycle('Harley-Davidson', 'Street 750', 2019, 750);

console.log(car.getInfo()); // "2020 Toyota Camry"
console.log(car.start()); // "Toyota Camry engine started. Ready to drive."
console.log(car.honk()); // "Beep beep!"

console.log(motorcycle.start()); // "Harley-Davidson Street 750 is starting"
console.log(motorcycle.wheelie()); // "Pulling a wheelie!"

// Check prototype chain
console.log(car instanceof Car); // true
console.log(car instanceof Vehicle); // true
console.log(motorcycle instanceof Motorcycle); // true
console.log(motorcycle instanceof Vehicle); // true

// Prototype manipulation
Vehicle.prototype.isOld = function() {
    return this.getAge() > 10;
};

// Now all vehicles have this method
console.log(car.isOld()); // false
console.log(motorcycle.isOld()); // false
```

### Advanced Prototype Patterns
```javascript
// Mixin pattern using prototypes
const Flyable = {
    fly() {
        return `${this.name || 'Object'} is flying`;
    },
    
    land() {
        return `${this.name || 'Object'} has landed`;
    },
    
    altitude: 0,
    
    changeAltitude(newAltitude) {
        this.altitude = newAltitude;
        return `Altitude changed to ${newAltitude} feet`;
    }
};

const Swimmable = {
    swim() {
        return `${this.name || 'Object'} is swimming`;
    },
    
    dive(depth) {
        return `${this.name || 'Object'} dives to ${depth} meters`;
    },
    
    surface() {
        return `${this.name || 'Object'} surfaces`;
    }
};

// Function to apply mixins
function mixin(target, ...sources) {
    sources.forEach(source => {
        Object.getOwnPropertyNames(source).forEach(name => {
            if (name !== 'constructor') {
                target[name] = source[name];
            }
        });
    });
    return target;
}

// Duck class with mixins
function Duck(name) {
    this.name = name;
    this.species = 'Duck';
}

Duck.prototype.quack = function() {
    return `${this.name} says quack!`;
};

// Apply mixins to Duck prototype
mixin(Duck.prototype, Flyable, Swimmable);

// Airplane class with Flyable mixin
function Airplane(model, airline) {
    this.model = model;
    this.airline = airline;
    this.name = `${airline} ${model}`;
}

mixin(Airplane.prototype, Flyable);

// Fish class with Swimmable mixin
function Fish(species, name) {
    this.species = species;
    this.name = name;
}

mixin(Fish.prototype, Swimmable);

// Usage
const duck = new Duck('Donald');
console.log(duck.quack()); // "Donald says quack!"
console.log(duck.fly()); // "Donald is flying"
console.log(duck.swim()); // "Donald is swimming"

const airplane = new Airplane('Boeing 747', 'American Airlines');
console.log(airplane.fly()); // "American Airlines Boeing 747 is flying"
console.log(airplane.changeAltitude(35000)); // "Altitude changed to 35000 feet"

const fish = new Fish('Salmon', 'Sammy');
console.log(fish.swim()); // "Sammy is swimming"
console.log(fish.dive(20)); // "Sammy dives to 20 meters"

// Prototype chain inspection
function inspectPrototypeChain(obj) {
    const chain = [];
    let current = obj;
    
    while (current) {
        chain.push(current.constructor.name || 'Object');
        current = Object.getPrototypeOf(current);
    }
    
    return chain;
}

console.log('Duck prototype chain:', inspectPrototypeChain(duck));
console.log('Airplane prototype chain:', inspectPrototypeChain(airplane));

// Dynamic prototype modification
Vehicle.prototype.horn = function() {
    return this.honk ? this.honk() : 'Generic horn sound';
};

// All existing and future Vehicle instances now have horn method
console.log(car.horn()); // "Beep beep!" (uses car's honk method)
console.log(motorcycle.horn()); // "Generic horn sound"
```

## Design Patterns with OOP

### Singleton Pattern
```javascript
class DatabaseConnection {
    constructor() {
        if (DatabaseConnection.instance) {
            return DatabaseConnection.instance;
        }
        
        this.host = 'localhost';
        this.port = 5432;
        this.database = 'myapp';
        this.connected = false;
        this.connectionTime = null;
        
        DatabaseConnection.instance = this;
        return this;
    }
    
    connect() {
        if (!this.connected) {
            this.connected = true;
            this.connectionTime = new Date();
            console.log(`Connected to ${this.database} at ${this.host}:${this.port}`);
        }
        return this.connected;
    }
    
    disconnect() {
        if (this.connected) {
            this.connected = false;
            this.connectionTime = null;
            console.log('Database disconnected');
        }
    }
    
    query(sql) {
        if (!this.connected) {
            throw new Error('Database not connected');
        }
        return `Executing: ${sql}`;
    }
    
    getConnectionInfo() {
        return {
            host: this.host,
            port: this.port,
            database: this.database,
            connected: this.connected,
            connectionTime: this.connectionTime
        };
    }
    
    static getInstance() {
        if (!DatabaseConnection.instance) {
            DatabaseConnection.instance = new DatabaseConnection();
        }
        return DatabaseConnection.instance;
    }
}

// Usage - both variables reference the same instance
const db1 = new DatabaseConnection();
const db2 = new DatabaseConnection();
const db3 = DatabaseConnection.getInstance();

console.log(db1 === db2); // true
console.log(db2 === db3); // true

db1.connect();
console.log(db2.getConnectionInfo()); // Shows connected state
```

### Factory Pattern
```javascript
// Product classes
class Smartphone {
    constructor(brand, model, specs) {
        this.brand = brand;
        this.model = model;
        this.specs = specs;
        this.type = 'smartphone';
    }
    
    getInfo() {
        return `${this.brand} ${this.model} - ${this.specs.screen}, ${this.specs.storage}GB`;
    }
    
    makeCall(number) {
        return `Calling ${number} from ${this.brand} ${this.model}`;
    }
}

class Laptop {
    constructor(brand, model, specs) {
        this.brand = brand;
        this.model = model;
        this.specs = specs;
        this.type = 'laptop';
    }
    
    getInfo() {
        return `${this.brand} ${this.model} - ${this.specs.processor}, ${this.specs.ram}GB RAM`;
    }
    
    runProgram(program) {
        return `Running ${program} on ${this.brand} ${this.model}`;
    }
}

class Tablet {
    constructor(brand, model, specs) {
        this.brand = brand;
        this.model = model;
        this.specs = specs;
        this.type = 'tablet';
    }
    
    getInfo() {
        return `${this.brand} ${this.model} - ${this.specs.screen}, ${this.specs.storage}GB`;
    }
    
    drawWithStylus() {
        return `Drawing on ${this.brand} ${this.model}`;
    }
}

// Factory class
class DeviceFactory {
    static createDevice(type, brand, model, specs) {
        switch (type.toLowerCase()) {
            case 'smartphone':
                return new Smartphone(brand, model, specs);
            case 'laptop':
                return new Laptop(brand, model, specs);
            case 'tablet':
                return new Tablet(brand, model, specs);
            default:
                throw new Error(`Unknown device type: ${type}`);
        }
    }
    
    static createSmartphone(brand, model, screen, storage, camera) {
        return new Smartphone(brand, model, { screen, storage, camera });
    }
    
    static createLaptop(brand, model, processor, ram, storage) {
        return new Laptop(brand, model, { processor, ram, storage });
    }
    
    static createTablet(brand, model, screen, storage, hasPen) {
        return new Tablet(brand, model, { screen, storage, hasPen });
    }
}

// Usage
const devices = [
    DeviceFactory.createSmartphone('Apple', 'iPhone 14', '6.1"', 128, '12MP'),
    DeviceFactory.createLaptop('Dell', 'XPS 13', 'Intel i7', 16, 512),
    DeviceFactory.createTablet('Samsung', 'Galaxy Tab S8', '11"', 256, true),
    DeviceFactory.createDevice('smartphone', 'Google', 'Pixel 7', {
        screen: '6.3"',
        storage: 128,
        camera: '50MP'
    })
];

devices.forEach(device => {
    console.log(device.getInfo());
    console.log(`Type: ${device.type}`);
    console.log('---');
});
```

### Observer Pattern
```javascript
class EventEmitter {
    constructor() {
        this.events = {};
    }
    
    on(event, listener) {
        if (!this.events[event]) {
            this.events[event] = [];
        }
        this.events[event].push(listener);
        return this; // For chaining
    }
    
    off(event, listenerToRemove) {
        if (!this.events[event]) return this;
        
        this.events[event] = this.events[event].filter(
            listener => listener !== listenerToRemove
        );
        return this;
    }
    
    emit(event, ...args) {
        if (!this.events[event]) return this;
        
        this.events[event].forEach(listener => {
            try {
                listener.apply(this, args);
            } catch (error) {
                console.error(`Error in event listener for ${event}:`, error);
            }
        });
        return this;
    }
    
    once(event, listener) {
        const onceWrapper = (...args) => {
            listener.apply(this, args);
            this.off(event, onceWrapper);
        };
        
        this.on(event, onceWrapper);
        return this;
    }
    
    removeAllListeners(event) {
        if (event) {
            delete this.events[event];
        } else {
            this.events = {};
        }
        return this;
    }
    
    listenerCount(event) {
        return this.events[event] ? this.events[event].length : 0;
    }
}

// Subject class using EventEmitter
class NewsAgency extends EventEmitter {
    constructor(name) {
        super();
        this.name = name;
        this.articles = [];
    }
    
    publishArticle(article) {
        this.articles.push({
            ...article,
            id: Date.now(),
            publishedAt: new Date()
        });
        
        this.emit('articlePublished', article);
        this.emit('newsUpdate', {
            type: 'article',
            data: article,
            agency: this.name
        });
    }
    
    breakingNews(news) {
        const breakingNewsItem = {
            ...news,
            id: Date.now(),
            priority: 'breaking',
            publishedAt: new Date()
        };
        
        this.articles.push(breakingNewsItem);
        this.emit('breakingNews', breakingNewsItem);
        this.emit('newsUpdate', {
            type: 'breaking',
            data: breakingNewsItem,
            agency: this.name
        });
    }
    
    getArticles() {
        return [...this.articles];
    }
}

// Observer classes
class NewsChannel {
    constructor(name) {
        this.name = name;
        this.subscribers = 0;
    }
    
    subscribeToAgency(agency) {
        agency.on('articlePublished', this.handleArticle.bind(this));
        agency.on('breakingNews', this.handleBreakingNews.bind(this));
        console.log(`${this.name} subscribed to ${agency.name}`);
    }
    
    handleArticle(article) {
        console.log(`${this.name} received article: ${article.title}`);
        this.broadcast(`Regular news: ${article.title}`);
    }
    
    handleBreakingNews(news) {
        console.log(`${this.name} received BREAKING NEWS: ${news.title}`);
        this.broadcast(`🚨 BREAKING: ${news.title}`);
    }
    
    broadcast(message) {
        console.log(`📺 ${this.name} broadcasting: ${message}`);
    }
}

class MobileApp {
    constructor(name) {
        this.name = name;
        this.users = [];
        this.notifications = [];
    }
    
    subscribeToAgency(agency) {
        agency.on('breakingNews', this.sendPushNotification.bind(this));
        agency.on('newsUpdate', this.updateFeed.bind(this));
        console.log(`${this.name} app connected to ${agency.name}`);
    }
    
    sendPushNotification(news) {
        const notification = {
            title: 'Breaking News',
            body: news.title,
            timestamp: new Date()
        };
        
        this.notifications.push(notification);
        console.log(`📱 ${this.name} sent push notification: ${news.title}`);
    }
    
    updateFeed(update) {
        console.log(`📱 ${this.name} updated feed with ${update.type} news from ${update.agency}`);
    }
}

// Usage
const cnn = new NewsAgency('CNN');
const bbc = new NewsAgency('BBC');

const newsChannel1 = new NewsChannel('Channel 1');
const newsChannel2 = new NewsChannel('Channel 2');
const mobileApp = new MobileApp('NewsApp');

// Subscribe observers to subjects
newsChannel1.subscribeToAgency(cnn);
newsChannel2.subscribeToAgency(cnn);
newsChannel2.subscribeToAgency(bbc);
mobileApp.subscribeToAgency(cnn);
mobileApp.subscribeToAgency(bbc);

// Publish news
cnn.publishArticle({
    title: 'Regular News Article',
    content: 'This is a regular news article...',
    category: 'politics'
});

cnn.breakingNews({
    title: 'Major Event Happening',
    content: 'This is breaking news...',
    category: 'breaking'
});

bbc.breakingNews({
    title: 'International Development',
    content: 'Important international news...',
    category: 'international'
});

// One-time subscription
const temporaryObserver = (article) => {
    console.log('Temporary observer received:', article.title);
};

cnn.once('articlePublished', temporaryObserver);

cnn.publishArticle({
    title: 'Another Article',
    content: 'This will trigger the temporary observer only once',
    category: 'technology'
});

cnn.publishArticle({
    title: 'Yet Another Article',
    content: 'This will not trigger the temporary observer',
    category: 'sports'
});
```

---

## Related Topics
- [[15 - Objects]]
- [[37 - ES6+ Features]]
- [[38 - Modules]]
- [[32 - Frontend Frameworks]]
- [[42 - Error Handling]]

---

*Next: [[34 - Regular Expressions]]*
