Alright — let’s break down **`super`** in JavaScript class inheritance.

---

## 1️⃣ What `super` Means

In ES6 classes, **`super`** is used to call methods or constructors from the **parent class**.

There are **two main uses**:

1. **Inside a constructor** → to call the parent class's constructor.
    
2. **Inside a method** → to call the parent class's method.
    

---

## 2️⃣ `super` in Constructor

When a child class has a constructor, it **must** call `super()` **before using `this`** — because `super()` sets up the `this` context.

```javascript
class Parent {
  constructor(name) {
    this.name = name;
  }
}

class Child extends Parent {
  constructor(name, age) {
    super(name); // Calls Parent's constructor
    this.age = age;
  }
}

const c1 = new Child("Alice", 20);
console.log(c1); // Child { name: 'Alice', age: 20 }
```

⚠ If you forget `super()` before using `this` in a subclass constructor → **ReferenceError**.

---

## 3️⃣ `super` in Methods

You can use `super.methodName()` to call a parent’s method from a child’s method.

```javascript
class Parent {
  greet() {
    console.log("Hello from Parent");
  }
}

class Child extends Parent {
  greet() {
    super.greet(); // Call Parent's greet()
    console.log("Hello from Child");
  }
}

const c2 = new Child();
c2.greet();
// Output:
// Hello from Parent
// Hello from Child
```

---

## 4️⃣ `super` in Static Methods

In a **static method**, `super` refers to the **parent class itself**.

```javascript
class Parent {
  static describe() {
    console.log("I am the Parent class");
  }
}

class Child extends Parent {
  static describe() {
    super.describe(); // Call static method of Parent
    console.log("I am the Child class");
  }
}

Child.describe();
// I am the Parent class
// I am the Child class
```

---

## 5️⃣ Key Rules to Remember

- In **subclass constructors**, `super()` must be called before `this`.
    
- In methods, `super.methodName()` calls the parent version.
    
- Works with **instance methods** and **static methods**.
    
- `super` doesn’t just call — it also sets the correct `this` binding to the current instance.
    

---

Got it — you’re asking if **`super`** can be used for more than one property in the constructor.

Short answer:  
✅ Yes, you can pass **multiple arguments** to `super()`, and the parent constructor can handle them to set multiple properties.

---

## Example — Passing Multiple Properties

```javascript
class Parent {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
}

class Child extends Parent {
  constructor(name, age, grade) {
    super(name, age); // pass multiple arguments to parent
    this.grade = grade;
  }
}

const c1 = new Child("Alice", 20, "A");
console.log(c1); 
// Child { name: 'Alice', age: 20, grade: 'A' }
```

---

## How It Works

- `super(name, age)` calls the **Parent** constructor.
    
- The **Parent** constructor sets `this.name` and `this.age`.
    
- Then the **Child** constructor adds `this.grade`.
    

---

## With More Than Two Properties

If the parent constructor has more parameters, you just pass them all in `super()`:

```javascript
class Parent {
  constructor(name, age, city) {
    this.name = name;
    this.age = age;
    this.city = city;
  }
}

class Child extends Parent {
  constructor(name, age, city, grade) {
    super(name, age, city); // more than one property
    this.grade = grade;
  }
}

const c2 = new Child("Bob", 22, "Delhi", "B+");
console.log(c2);
// Child { name: 'Bob', age: 22, city: 'Delhi', grade: 'B+' }
```

---

💡 **Tip:**

- `super()` can take _any number of arguments_.
    
- You just need to make sure the parent constructor accepts and uses them.
    

---

If you want, I can show you **how `super()` internally works without classes** so you see why it can take multiple arguments. That makes this concept crystal clear.