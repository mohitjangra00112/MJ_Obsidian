# Prototypes and Prototypal Inheritance

## Overview
JavaScript uses prototypal inheritance, where objects can inherit directly from other objects. Every object has an internal `[[Prototype]]` property that points to another object, creating a prototype chain.

## Understanding Prototypes

### The Prototype Property
```javascript
// Constructor function
function Person(name) {
    this.name = name;
}

// Adding method to prototype
Person.prototype.greet = function() {
    return `Hello, I'm ${this.name}`;
};

Person.prototype.species = "Homo sapiens";

const john = new Person("John");
console.log(john.greet()); // "Hello, I'm John"
console.log(john.species); // "Homo sapiens"

// john doesn't have these properties directly
console.log(john.hasOwnProperty("greet")); // false
console.log(john.hasOwnProperty("species")); // false
console.log(john.hasOwnProperty("name")); // true
```

### Prototype Chain
```javascript
function Animal(name) {
    this.name = name;
}

Animal.prototype.speak = function() {
    return `${this.name} makes a sound`;
};

function Dog(name, breed) {
    Animal.call(this, name); // Call parent constructor
    this.breed = breed;
}

// Set up inheritance
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

// Add Dog-specific method
Dog.prototype.bark = function() {
    return `${this.name} barks!`;
};

const buddy = new Dog("Buddy", "Golden Retriever");
console.log(buddy.speak()); // "Buddy makes a sound" (inherited)
console.log(buddy.bark()); // "Buddy barks!" (own method)
```

### Prototype Chain Lookup
```javascript
const obj = {
    name: "Object"
};

// Prototype chain: obj -> Object.prototype -> null
console.log(obj.toString()); // From Object.prototype
console.log(obj.hasOwnProperty("name")); // From Object.prototype

// Check prototype
console.log(Object.getPrototypeOf(obj) === Object.prototype); // true
console.log(Object.getPrototypeOf(Object.prototype)); // null
```

## Different Ways to Create Prototypal Inheritance

### 1. Constructor Functions
```javascript
function Vehicle(type) {
    this.type = type;
}

Vehicle.prototype.start = function() {
    return `${this.type} is starting`;
};

function Car(brand, model) {
    Vehicle.call(this, "car");
    this.brand = brand;
    this.model = model;
}

Car.prototype = Object.create(Vehicle.prototype);
Car.prototype.constructor = Car;

Car.prototype.drive = function() {
    return `Driving ${this.brand} ${this.model}`;
};

const myCar = new Car("Toyota", "Camry");
```

### 2. Object.create()
```javascript
const animal = {
    init(name) {
        this.name = name;
        return this;
    },
    
    speak() {
        return `${this.name} makes a sound`;
    }
};

const dog = Object.create(animal);
dog.bark = function() {
    return `${this.name} barks!`;
};

const myDog = Object.create(dog).init("Rex");
console.log(myDog.speak()); // "Rex makes a sound"
console.log(myDog.bark()); // "Rex barks!"
```

### 3. Factory Functions with Prototypes
```javascript
const animalMethods = {
    speak() {
        return `${this.name} makes a sound`;
    },
    
    eat(food) {
        return `${this.name} eats ${food}`;
    }
};

function createAnimal(name, species) {
    const animal = Object.create(animalMethods);
    animal.name = name;
    animal.species = species;
    return animal;
}

const lion = createAnimal("Simba", "Lion");
console.log(lion.speak()); // "Simba makes a sound"
```

### 4. ES6 Classes (Syntactic Sugar)
```javascript
class Animal {
    constructor(name) {
        this.name = name;
    }
    
    speak() {
        return `${this.name} makes a sound`;
    }
}

class Dog extends Animal {
    constructor(name, breed) {
        super(name); // Call parent constructor
        this.breed = breed;
    }
    
    bark() {
        return `${this.name} barks!`;
    }
}

const dog = new Dog("Max", "Labrador");
```

## Prototype Methods and Properties

### Object.getPrototypeOf() and Object.setPrototypeOf()
```javascript
const animal = { species: "unknown" };
const dog = { breed: "unknown" };

// Set prototype
Object.setPrototypeOf(dog, animal);

// Get prototype
console.log(Object.getPrototypeOf(dog) === animal); // true
console.log(dog.species); // "unknown" (inherited)

// Check if object is in prototype chain
console.log(animal.isPrototypeOf(dog)); // true
```

### __proto__ (Deprecated but widely supported)
```javascript
const animal = { type: "animal" };
const dog = { breed: "labrador" };

// Setting prototype (not recommended)
dog.__proto__ = animal;

console.log(dog.type); // "animal"
console.log(dog.__proto__ === animal); // true
```

### constructor Property
```javascript
function Person(name) {
    this.name = name;
}

const john = new Person("John");

console.log(john.constructor === Person); // true
console.log(john.constructor.name); // "Person"

// Can be used to create new instances
const jane = new john.constructor("Jane");
```

## Prototype Pollution Prevention

### Object.create(null)
```javascript
// Create object without prototype
const map = Object.create(null);
map.name = "Safe Map";

console.log(map.toString); // undefined
console.log(map.constructor); // undefined
console.log("constructor" in map); // false

// Safe for use as a dictionary
function createSafeMap() {
    return Object.create(null);
}
```

### Freezing Prototypes
```javascript
function SecureConstructor() {}

// Freeze the prototype
Object.freeze(SecureConstructor.prototype);

// Attempts to modify will fail
SecureConstructor.prototype.maliciousMethod = function() {
    // This won't be added
};

console.log(SecureConstructor.prototype.maliciousMethod); // undefined
```

## Advanced Prototype Patterns

### Mixin Pattern
```javascript
const CanFly = {
    fly() {
        return `${this.name} is flying`;
    }
};

const CanSwim = {
    swim() {
        return `${this.name} is swimming`;
    }
};

function Bird(name) {
    this.name = name;
}

function Duck(name) {
    Bird.call(this, name);
}

// Multiple inheritance through mixins
Duck.prototype = Object.create(Bird.prototype);
Object.assign(Duck.prototype, CanFly, CanSwim);

Duck.prototype.quack = function() {
    return `${this.name} quacks`;
};

const duck = new Duck("Donald");
console.log(duck.fly()); // "Donald is flying"
console.log(duck.swim()); // "Donald is swimming"
```

### Prototype Delegation Pattern
```javascript
const GameObject = {
    init(x, y) {
        this.x = x;
        this.y = y;
        return this;
    },
    
    move(dx, dy) {
        this.x += dx;
        this.y += dy;
    },
    
    getPosition() {
        return { x: this.x, y: this.y };
    }
};

const Player = Object.create(GameObject);
Player.attack = function() {
    return "Player attacks!";
};

const Enemy = Object.create(GameObject);
Enemy.patrol = function() {
    return "Enemy is patrolling";
};

const player1 = Object.create(Player).init(0, 0);
const enemy1 = Object.create(Enemy).init(10, 10);
```

### Method Borrowing
```javascript
const mathOperations = {
    add() {
        return Array.from(arguments).reduce((sum, num) => sum + num, 0);
    },
    
    multiply() {
        return Array.from(arguments).reduce((product, num) => product * num, 1);
    }
};

const calculator = {
    numbers: [1, 2, 3, 4, 5]
};

// Borrow methods
calculator.sum = function() {
    return mathOperations.add.apply(null, this.numbers);
};

calculator.product = function() {
    return mathOperations.multiply.apply(null, this.numbers);
};
```

## Prototype vs Class Comparison

### Performance Considerations
```javascript
// Constructor with prototype (memory efficient)
function Person(name) {
    this.name = name;
}

Person.prototype.greet = function() {
    return `Hello, I'm ${this.name}`;
};

// Constructor without prototype (memory inefficient)
function PersonBad(name) {
    this.name = name;
    this.greet = function() { // New function for each instance!
        return `Hello, I'm ${this.name}`;
    };
}

// Create many instances
const people = [];
for (let i = 0; i < 1000; i++) {
    people.push(new Person(`Person${i}`));
}
// Only one greet function in memory (on prototype)

const peopleBad = [];
for (let i = 0; i < 1000; i++) {
    peopleBad.push(new PersonBad(`Person${i}`));
}
// 1000 greet functions in memory!
```

## Common Prototype Gotchas

### 1. Modifying Built-in Prototypes
```javascript
// Generally avoid this!
Array.prototype.shuffle = function() {
    for (let i = this.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [this[i], this[j]] = [this[j], this[i]];
    }
    return this;
};

// Can cause conflicts with other libraries
// Use utility functions instead
function shuffle(array) {
    const result = [...array];
    for (let i = result.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [result[i], result[j]] = [result[j], result[i]];
    }
    return result;
}
```

### 2. Shared References in Prototypes
```javascript
function Person(name) {
    this.name = name;
}

// Dangerous: shared array reference
Person.prototype.hobbies = [];

const john = new Person("John");
const jane = new Person("Jane");

john.hobbies.push("reading");
console.log(jane.hobbies); // ["reading"] - shared!

// Better approach
Person.prototype.addHobby = function(hobby) {
    if (!this.hobbies) {
        this.hobbies = [];
    }
    this.hobbies.push(hobby);
};
```

### 3. Losing Constructor Reference
```javascript
function Animal() {}
function Dog() {}

Dog.prototype = Object.create(Animal.prototype);
// Forgot to restore constructor!

const dog = new Dog();
console.log(dog.constructor === Dog); // false
console.log(dog.constructor === Animal); // true

// Fix it
Dog.prototype.constructor = Dog;
```

## Modern Prototype Utilities

### Reflection Methods
```javascript
const obj = { name: "John" };

// Check if property exists in prototype chain
console.log(Reflect.has(obj, "toString")); // true

// Get property descriptor
console.log(Reflect.getOwnPropertyDescriptor(obj, "name"));

// Get prototype
console.log(Reflect.getPrototypeOf(obj) === Object.prototype); // true

// Set prototype
const proto = { greet() { return "Hello"; } };
Reflect.setPrototypeOf(obj, proto);
```

### Proxy for Prototype Interception
```javascript
const prototypeHandler = {
    get(target, property) {
        console.log(`Accessing ${property}`);
        return Reflect.get(target, property);
    }
};

function ProxiedConstructor(name) {
    this.name = name;
}

ProxiedConstructor.prototype = new Proxy({
    greet() {
        return `Hello, I'm ${this.name}`;
    }
}, prototypeHandler);

const obj = new ProxiedConstructor("John");
obj.greet(); // Logs: "Accessing greet"
```

## Best Practices

### 1. Prefer Composition over Inheritance
```javascript
// Instead of complex inheritance hierarchies
const behaviors = {
    canFly: {
        fly() { return `${this.name} flies`; }
    },
    canSwim: {
        swim() { return `${this.name} swims`; }
    }
};

function createDuck(name) {
    return Object.assign(
        { name },
        behaviors.canFly,
        behaviors.canSwim
    );
}
```

### 2. Use Object.create() for Clear Inheritance
```javascript
// Clear and explicit
const parent = { parentMethod() {} };
const child = Object.create(parent);
child.childMethod = function() {};
```

### 3. Always Check hasOwnProperty in Loops
```javascript
const obj = { a: 1, b: 2 };

for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
        console.log(`${key}: ${obj[key]}`);
    }
}

// Or use Object.keys()
Object.keys(obj).forEach(key => {
    console.log(`${key}: ${obj[key]}`);
});
```

---

## Related Topics
- [[06 - Objects]]
- [[08 - Classes and Constructor Functions]]
- [[09 - Object Creation Patterns]]
- [[35 - JavaScript OOP]]
- [[36 - Inheritance Patterns]]

---

*Next: [[08 - Classes and Constructor Functions]]*
