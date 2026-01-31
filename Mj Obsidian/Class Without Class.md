Got it — you want to understand **JavaScript classes but without using the `class` keyword**.

That means we’ll talk about **how classes work under the hood** in JavaScript, because before ES6 introduced `class`, people used **constructor functions** and **prototypes** to achieve the same thing.

---

## 1️⃣ Why “without class” makes sense

In JavaScript, **classes are just syntactic sugar** over its existing prototype-based inheritance.  
The `class` keyword doesn't create a _real_ class like in Java or C++; it just makes it look nicer.

---

## 2️⃣ The Old Way — Constructor Functions

Before `class` was introduced in ES6, we used a **function** to act like a blueprint:

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
}

// Adding methods using prototype
Person.prototype.sayHello = function () {
  console.log(`Hi, I'm ${this.name} and I'm ${this.age} years old.`);
};

// Create objects
const p1 = new Person("Alice", 25);
const p2 = new Person("Bob", 30);

p1.sayHello(); // Hi, I'm Alice and I'm 25 years old.
p2.sayHello(); // Hi, I'm Bob and I'm 30 years old.
```

**Explanation:**

- `function Person(...) { ... }` → Works like a constructor.
    
- `this.name` → Each object gets its own property.
    
- `Person.prototype.sayHello` → Shared method for all objects created with `new Person(...)`.
    
- `new` → Creates a new object, links it to the function’s prototype, and sets `this`.
    

---

## 3️⃣ Prototypes in Action

Every function in JS has a `.prototype` object.  
When you call a method, JS first checks if the property exists **on the object** itself, and if not, it looks **up the prototype chain**.

```javascript
console.log(p1.__proto__ === Person.prototype); // true
console.log(Person.prototype.constructor === Person); // true
```

---

## 4️⃣ Inheritance Without `class`

You can make one “class” inherit from another using prototypes:

```javascript
function Student(name, age, grade) {
  Person.call(this, name, age); // Call parent constructor
  // Note -> In call first parameter 
  this.grade = grade;
}

// Inherit prototype methods
Student.prototype = Object.create(Person.prototype);

// Reset constructor reference
Student.prototype.constructor = Student;

// Add new method
Student.prototype.study = function () {
  console.log(`${this.name} is studying in grade ${this.grade}.`);
};

const s1 = new Student("Charlie", 20, "A");
s1.sayHello(); // Hi, I'm Charlie and I'm 20 years old.
s1.study();    // Charlie is studying in grade A.
```

---

✅ **Key takeaway**:  
A “class” in JavaScript is just:

1. A constructor function.
    
2. Methods attached to `.prototype`.
    
3. Optional inheritance via `Object.create`.
    

If you understand this, the ES6 `class` keyword will feel much less magical — it just hides this pattern.

---

If you want, I can show you a **side-by-side comparison** of `class` vs old-style constructor + prototype so you see they’re exactly the same under the hood.