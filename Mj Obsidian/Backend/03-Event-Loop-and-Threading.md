# Event Loop and Threading

**Navigation**: [[02-Modules-and-Exports]] | [[00-Main-Index]] | Next: [[04-HTTP-Server]]

---

## 🎯 Understanding Node.js Event Loop

The **Event Loop** is the heart of Node.js's non-blocking I/O operations. It's what makes Node.js highly efficient for I/O-intensive applications.

---

## 🔄 Event Loop Phases

### Six Main Phases
1. **Timer Phase** - executes setTimeout() and setInterval()
2. **Pending Callbacks** - executes I/O callbacks deferred to the next loop iteration
3. **Idle, Prepare** - internal use only
4. **Poll Phase** - fetch new I/O events; execute I/O related callbacks
5. **Check Phase** - setImmediate() callbacks are invoked here
6. **Close Callbacks** - close callbacks (e.g., socket.on('close', ...))

### Visual Representation
```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           poll            │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

---

## ⏰ Timer Functions

### setTimeout vs setImmediate vs process.nextTick
```javascript
console.log('Start');

// Timer phase
setTimeout(() => console.log('Timer 1'), 0);
setTimeout(() => console.log('Timer 2'), 0);

// Check phase
setImmediate(() => console.log('Immediate 1'));
setImmediate(() => console.log('Immediate 2'));

// Next tick queue (highest priority)
process.nextTick(() => console.log('Next Tick 1'));
process.nextTick(() => console.log('Next Tick 2'));

// Microtask queue
Promise.resolve().then(() => console.log('Promise 1'));
Promise.resolve().then(() => console.log('Promise 2'));

console.log('End');

// Output:
// Start
// End
// Next Tick 1
// Next Tick 2
// Promise 1
// Promise 2
// Timer 1
// Timer 2
// Immediate 1
// Immediate 2
```

---

## 🎭 Execution Order Priority

### Priority Queue (Highest to Lowest)
1. **process.nextTick()** - Highest priority
2. **Microtasks** - Promise.then(), queueMicrotask()
3. **Timer callbacks** - setTimeout(), setInterval()
4. **I/O callbacks**
5. **setImmediate()**

### Example
```javascript
const fs = require('fs');

console.log('=== Start ===');

// Microtask
Promise.resolve().then(() => console.log('Promise'));

// Timer
setTimeout(() => console.log('Timer'), 0);

// Immediate
setImmediate(() => console.log('Immediate'));

// Next tick (highest priority)
process.nextTick(() => console.log('Next Tick'));

// I/O operation
fs.readFile(__filename, () => {
    console.log('File read');
    
    setTimeout(() => console.log('Timer in I/O'), 0);
    setImmediate(() => console.log('Immediate in I/O'));
    process.nextTick(() => console.log('Next Tick in I/O'));
});

console.log('=== End ===');
```

---

## 🧵 Single Thread vs Multi-Threading

### Single-Threaded Nature
```javascript
// Node.js main thread is single-threaded
console.log('Main thread ID:', process.pid);

// This blocks the entire event loop
function blockingOperation() {
    const start = Date.now();
    while (Date.now() - start < 5000) {
        // Blocking for 5 seconds
    }
    console.log('Blocking operation complete');
}

// Don't do this in production!
// blockingOperation();
```

### Non-Blocking I/O
```javascript
const fs = require('fs');

console.log('Start reading file...');

// Non-blocking file read
fs.readFile('large-file.txt', (err, data) => {
    if (err) throw err;
    console.log('File read complete');
});

console.log('This executes immediately');

// Output:
// Start reading file...
// This executes immediately
// File read complete (after file is read)
```

---

## 🔧 Worker Threads (Multi-threading)

### Basic Worker Thread
```javascript
// main.js
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');

if (isMainThread) {
    // Main thread
    console.log('Main thread');
    
    const worker = new Worker(__filename, {
        workerData: { num: 42 }
    });
    
    worker.on('message', (result) => {
        console.log('Result from worker:', result);
    });
    
    worker.on('error', (error) => {
        console.error('Worker error:', error);
    });
    
    worker.on('exit', (code) => {
        if (code !== 0) {
            console.error('Worker stopped with exit code', code);
        }
    });
} else {
    // Worker thread
    console.log('Worker thread');
    
    const result = workerData.num * 2;
    parentPort.postMessage(result);
}
```

### CPU-Intensive Task with Workers
```javascript
// cpu-intensive.js
const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');

function fibonacci(n) {
    if (n < 2) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

if (isMainThread) {
    function runWorker(workerData) {
        return new Promise((resolve, reject) => {
            const worker = new Worker(__filename, { workerData });
            worker.on('message', resolve);
            worker.on('error', reject);
            worker.on('exit', (code) => {
                if (code !== 0) {
                    reject(new Error(`Worker stopped with exit code ${code}`));
                }
            });
        });
    }
    
    async function main() {
        const start = Date.now();
        
        // Run multiple workers in parallel
        const promises = [35, 36, 37, 38].map(num => 
            runWorker({ num })
        );
        
        const results = await Promise.all(promises);
        console.log('Results:', results);
        console.log('Time taken:', Date.now() - start, 'ms');
    }
    
    main().catch(console.error);
} else {
    // Worker thread
    const result = fibonacci(workerData.num);
    parentPort.postMessage(result);
}
```

---

## 🌊 Asynchronous Patterns

### Callback Pattern
```javascript
const fs = require('fs');

function readFileCallback(filename, callback) {
    fs.readFile(filename, 'utf8', (err, data) => {
        if (err) {
            return callback(err, null);
        }
        callback(null, data);
    });
}

readFileCallback('file.txt', (err, data) => {
    if (err) {
        console.error('Error:', err);
        return;
    }
    console.log('File content:', data);
});
```

### Promise Pattern
```javascript
const fs = require('fs').promises;

function readFilePromise(filename) {
    return fs.readFile(filename, 'utf8');
}

readFilePromise('file.txt')
    .then(data => console.log('File content:', data))
    .catch(err => console.error('Error:', err));
```

### Async/Await Pattern
```javascript
const fs = require('fs').promises;

async function readFileAsync(filename) {
    try {
        const data = await fs.readFile(filename, 'utf8');
        console.log('File content:', data);
        return data;
    } catch (err) {
        console.error('Error:', err);
        throw err;
    }
}

// Usage
readFileAsync('file.txt');
```

---

## 🚫 Common Event Loop Pitfalls

### 1. Blocking the Event Loop
```javascript
// ❌ Don't do this
function badLoop() {
    while (true) {
        // This blocks the event loop
    }
}

// ✅ Do this instead
function goodLoop() {
    setImmediate(() => {
        // Do some work
        goodLoop(); // Continue asynchronously
    });
}
```

### 2. Too Many process.nextTick()
```javascript
// ❌ This can starve the event loop
function recursiveNextTick() {
    process.nextTick(recursiveNextTick);
}

// ✅ Use setImmediate for recursive calls
function recursiveImmediate() {
    setImmediate(recursiveImmediate);
}
```

### 3. Synchronous Operations in Callbacks
```javascript
// ❌ Don't mix sync and async
fs.readFile('file.txt', (err, data) => {
    if (err) throw err;
    
    // This blocks the event loop
    const result = fs.readFileSync('another-file.txt');
});

// ✅ Keep everything async
fs.readFile('file.txt', (err, data) => {
    if (err) throw err;
    
    fs.readFile('another-file.txt', (err2, data2) => {
        // Handle both files
    });
});
```

---

## 📊 Performance Monitoring

### Monitoring Event Loop Lag
```javascript
const { performance, PerformanceObserver } = require('perf_hooks');

const obs = new PerformanceObserver((list) => {
    list.getEntries().forEach((entry) => {
        console.log(`${entry.name}: ${entry.duration}ms`);
    });
});

obs.observe({ entryTypes: ['measure'] });

// Measure event loop lag
setInterval(() => {
    const start = performance.now();
    setImmediate(() => {
        const lag = performance.now() - start;
        performance.mark('event-loop-lag', { detail: lag });
        console.log(`Event loop lag: ${lag}ms`);
    });
}, 1000);
```

### CPU Usage Monitoring
```javascript
const os = require('os');

function getCPUUsage() {
    const cpus = os.cpus();
    let user = 0, nice = 0, sys = 0, idle = 0, irq = 0;
    
    for (let cpu of cpus) {
        user += cpu.times.user;
        nice += cpu.times.nice;
        sys += cpu.times.sys;
        idle += cpu.times.idle;
        irq += cpu.times.irq;
    }
    
    const total = user + nice + sys + idle + irq;
    const usage = ((total - idle) / total) * 100;
    
    return usage.toFixed(2);
}

setInterval(() => {
    console.log(`CPU Usage: ${getCPUUsage()}%`);
}, 5000);
```

---

## 🔗 Related Topics
- [[01-Node-JS-Fundamentals]] - Basic Node.js concepts
- [[27-Multithreading-in-Node]] - Advanced threading patterns
- [[21-Error-Handling]] - Handling async errors

---

*Back to [[00-Main-Index]]*
