# Node.js Fundamentals

**Navigation**: [[00-Main-Index]] | Next: [[02-Modules-and-Exports]]

---

## 🎯 What is Node.js?

Node.js is a **JavaScript runtime** built on Chrome's V8 JavaScript engine. It allows you to run JavaScript on the server-side.

### Key Features
- **Non-blocking I/O**: Asynchronous operations
- **Event-driven**: Event loop architecture
- **Cross-platform**: Windows, macOS, Linux
- **NPM**: Vast package ecosystem

---

## 🚀 Getting Started

### Installation
```bash
# Check if Node.js is installed
node --version
npm --version

# Install Node.js from nodejs.org
# Recommended: Use LTS version
```

### Your First Node.js Script
```javascript
// hello.js
console.log('Hello, Node.js!');
console.log('Current directory:', __dirname);
console.log('Current file:', __filename);
```

```bash
# Run the script
node hello.js
```

---

## 🌍 Global Objects

### `__dirname` and `__filename`
```javascript
console.log('Directory:', __dirname);
console.log('File:', __filename);
```

### `process` Object
```javascript
// Process information
console.log('Node version:', process.version);
console.log('Platform:', process.platform);
console.log('Arguments:', process.argv);

// Environment variables
console.log('NODE_ENV:', process.env.NODE_ENV);

// Exit process
process.exit(0); // 0 = success, 1 = error
```

### `global` Object
```javascript
// Global scope (similar to window in browser)
global.myGlobalVar = 'Hello World';
console.log(myGlobalVar); // Accessible everywhere
```

---

## 📦 Built-in Modules

### File System (`fs`)
```javascript
const fs = require('fs');

// Synchronous read
const data = fs.readFileSync('file.txt', 'utf8');
console.log(data);

// Asynchronous read
fs.readFile('file.txt', 'utf8', (err, data) => {
    if (err) throw err;
    console.log(data);
});
```

### Path Module
```javascript
const path = require('path');

console.log(path.join('/users', 'mohit', 'documents'));
console.log(path.extname('file.txt')); // .txt
console.log(path.basename('/path/to/file.txt')); // file.txt
```

### OS Module
```javascript
const os = require('os');

console.log('Platform:', os.platform());
console.log('CPU Architecture:', os.arch());
console.log('Free Memory:', os.freemem());
console.log('Total Memory:', os.totalmem());
```

---

## 🔄 Event-Driven Architecture

### EventEmitter
```javascript
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}
const myEmitter = new MyEmitter();

// Listen for event
myEmitter.on('event', () => {
    console.log('An event occurred!');
});

// Emit event
myEmitter.emit('event');
```

### Custom Events
```javascript
const EventEmitter = require('events');

class Logger extends EventEmitter {
    log(message) {
        console.log(message);
        this.emit('messageLogged', { id: 1, url: 'http://mysite.com' });
    }
}

const logger = new Logger();

logger.on('messageLogged', (arg) => {
    console.log('Listener called:', arg);
});

logger.log('Hello World');
```

---

## 📝 Best Practices

1. **Use const for requires**
   ```javascript
   const fs = require('fs');
   ```

2. **Handle errors properly**
   ```javascript
   fs.readFile('file.txt', (err, data) => {
       if (err) {
           console.error('Error:', err);
           return;
       }
       console.log(data);
   });
   ```

3. **Use path.join for file paths**
   ```javascript
   const filePath = path.join(__dirname, 'data', 'file.txt');
   ```

---

## 🔗 Related Topics
- [[02-Modules-and-Exports]] - Learn about module system
- [[03-Event-Loop-and-Threading]] - Understanding async nature
- [[04-HTTP-Server]] - Creating web servers

---

*Back to [[00-Main-Index]]*
