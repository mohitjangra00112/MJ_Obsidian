Here you go, Mohit — your **FS module notes fully converted into clean, perfect Obsidian format**.

You can directly paste this into your Obsidian vault.

---

# 📁 **FS Module in Node.js — Complete Notes**

---

## ## What is the `fs` Module?

- `fs` stands for **File System**.
    
- Allows Node.js to:
    
    - Create files
        
    - Read files
        
    - Write/Update files
        
    - Delete files
        
    - Work with directories
        
- It is a **core module** → no installation needed.
    

---

## ## Importing `fs`

### **CommonJS**

```js
const fs = require("fs");
```

### **ES Module**

```js
import fs from "fs";
```

---

## ## Types of FS Methods

### **1. Synchronous (Blocking)**

- Ends with `Sync`
    
- Stops the program until operation completes
    
- Example: `fs.readFileSync()`
    

### **2. Asynchronous (Non-Blocking)**

- Uses callbacks or promises
    
- Recommended
    
- Example: `fs.readFile()`
    

---

## # Reading Files

### **Async (Recommended)**

```js
fs.readFile("file.txt", "utf-8", (err, data) => {
  if (err) return console.log(err);
  console.log(data);
});
```

### **Sync**

```js
const data = fs.readFileSync("file.txt", "utf-8");
console.log(data);
```

---

## # Writing Files

### **Async**

```js
fs.writeFile("notes.txt", "Hello Mohit!", err => {
  if (err) throw err;
  console.log("File created!");
});
```

### **Sync**

```js
fs.writeFileSync("notes.txt", "Hello Mohit!");
```

---

## # Appending Data

### **Async**

```js
fs.appendFile("notes.txt", "\nThis is new data.", err => {
  if (err) throw err;
  console.log("Data added!");
});
```

### **Sync**

```js
fs.appendFileSync("notes.txt", "\nMore data.");
```

---

## # Deleting Files

### **Async**

```js
fs.unlink("notes.txt", err => {
  if (err) throw err;
  console.log("File deleted");
});
```

### **Sync**

```js
fs.unlinkSync("notes.txt");
```

---

## # Renaming Files

### **Async**

```js
fs.rename("old.txt", "new.txt", err => {
  if (err) throw err;
  console.log("Renamed!");
});
```

### **Sync**

```js
fs.renameSync("old.txt", "new.txt");
```

---

## # Creating Folders

### **Async**

```js
fs.mkdir("myfolder", err => {
  if (err) throw err;
  console.log("Folder created");
});
```

### **Sync**

```js
fs.mkdirSync("myfolder");
```

---

## # Removing Folders

### **Async**

```js
fs.rmdir("myfolder", err => {
  if (err) throw err;
  console.log("Folder removed");
});
```

### **Sync**

```js
fs.rmdirSync("myfolder");
```

> ⚠ Folder must be empty.  
> For non-empty folders, use `fs.rm()`.

---

## # Reading Directory Contents

```js
fs.readdir("myfolder", (err, files) => {
  if (err) throw err;
  console.log(files);
});
```

---

## # Check if File/Folder Exists

```js
if (fs.existsSync("test.txt")) {
  console.log("File exists!");
}
```

---

## # FS with Promises (`fs.promises`)

```js
import { promises as fs } from "fs";

async function read() {
  const data = await fs.readFile("file.txt", "utf-8");
  console.log(data);
}

read();
```

---

## # Working with JSON Files

### Write JSON

```js
fs.writeFileSync(
  "data.json",
  JSON.stringify({ name: "Mohit", age: 21 })
);
```

### Read JSON

```js
const json = JSON.parse(
  fs.readFileSync("data.json", "utf-8")
);
console.log(json.name);
```

---

## # File Stats (Metadata)

```js
fs.stat("file.txt", (err, stats) => {
  if (err) throw err;
  console.log(stats.isFile());
  console.log(stats.isDirectory());
  console.log(stats.size);
});
```

---

## # Summary Table

|Operation|Async Method|Sync Method|
|---|---|---|
|Read File|`readFile`|`readFileSync`|
|Write File|`writeFile`|`writeFileSync`|
|Append File|`appendFile`|`appendFileSync`|
|Delete File|`unlink`|`unlinkSync`|
|Rename File|`rename`|`renameSync`|
|Create Folder|`mkdir`|`mkdirSync`|
|Remove Folder|`rmdir`|`rmdirSync`|
|Check Existence|—|`existsSync`|

---

## ## Final Notes

- Prefer **async** methods for performance.
    
- `fs` is essential for backend work with Node.js.
    
- Used in:
    
    - File-based servers
        
    - Logging systems
        
    - Config loading
        
    - CLI tools
        
    - Databases like LokiJS
        

---

If you want, I can also convert:

📌 `path` module notes  
📌 `http` module notes  
📌 Streams notes  
📌 Complete Node.js notes (full Obsidian-ready)

Just tell me, Mohit!