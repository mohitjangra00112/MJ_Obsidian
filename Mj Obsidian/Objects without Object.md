Okay — you’re asking about **creating objects without directly writing an object literal** (`{}`) or using `new Object()` — basically **"object without object"**.

There are several ways to do this in JavaScript, and each is a little sneaky.

---

## 1️⃣ Using a Constructor Function

Instead of writing `{}`, you define a function and use `new`.

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
}

const p1 = new Person("Alice", 25);
console.log(p1); // { name: 'Alice', age: 25 }
```

Here, we never wrote `{}` directly — `new` secretly creates an object behind the scenes.

---

## 2️⃣ Using `Object.create()`

This creates a new object with a specific prototype.

```javascript
const proto = {
  greet() {
    console.log(`Hello, my name is ${this.name}`);
  }
};

const p2 = Object.create(proto);
p2.name = "Bob";
p2.greet(); // Hello, my name is Bob
```

No `{}` in sight — the object is created from the prototype.

---

## 3️⃣ Using a Class (Still Creates an Object)

Even though we’re not writing `{}`, the `new` keyword still makes one.

```javascript
class Animal {
  constructor(type) {
    this.type = type;
  }
}

const a1 = new Animal("Dog");
console.log(a1); // Animal { type: 'Dog' }
```

---

## 4️⃣ Using JSON Parsing

If you have a JSON string, `JSON.parse` creates an object.

```javascript
const jsonString = '{"city": "Delhi", "country": "India"}';
const location = JSON.parse(jsonString);
console.log(location); // { city: 'Delhi', country: 'India' }
```

---

## 5️⃣ Using Functions That Return Objects

You can create an object without `new` or `{}` by returning it from a function.

```javascript
function makeUser(name) {
  return { name };
}

const u1 = makeUser("Charlie");
console.log(u1); // { name: 'Charlie' }
```

---

💡 **The trick**:  
No matter which method you use, under the hood JavaScript is _always_ creating an object structure in memory — you’re just choosing a different syntax to do it.

---

If you want, I can show you **ways to create an object in JS without using `{}` OR `new` at all**, using only built-in functions. That’s even more fun.