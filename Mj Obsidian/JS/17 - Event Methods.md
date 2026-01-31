# Event Methods

## Overview
Event methods in JavaScript provide ways to handle user interactions, system events, and custom events. This includes traditional event handlers, modern event listeners, event delegation, and advanced event management techniques.

## Traditional Event Handlers

### Inline Event Handlers
```html
<!-- Inline event handlers (not recommended) -->
<button onclick="handleClick()">Click me</button>
<input onchange="handleChange(this.value)" />
<form onsubmit="return handleSubmit(event)">
    <input type="text" name="username" />
    <button type="submit">Submit</button>
</form>

<script>
function handleClick() {
    console.log('Button clicked via inline handler');
}

function handleChange(value) {
    console.log('Input changed:', value);
}

function handleSubmit(event) {
    event.preventDefault();
    console.log('Form submitted');
    return false; // Prevent default submission
}
</script>
```

### Property Event Handlers
```javascript
// Property-based event handlers
const button = document.getElementById('myButton');
const input = document.getElementById('myInput');

// Single event handler per element
button.onclick = function(event) {
    console.log('Button clicked via property handler');
    console.log('Event type:', event.type);
    console.log('Target:', event.target);
};

// Override previous handler
button.onclick = function(event) {
    console.log('New click handler - previous one is replaced');
};

// Input events
input.onchange = function(event) {
    console.log('Input changed:', event.target.value);
};

input.oninput = function(event) {
    console.log('Input value:', event.target.value);
};

input.onfocus = function(event) {
    console.log('Input focused');
    event.target.style.backgroundColor = '#f0f0f0';
};

input.onblur = function(event) {
    console.log('Input blurred');
    event.target.style.backgroundColor = '';
};

// Form events
const form = document.getElementById('myForm');
form.onsubmit = function(event) {
    event.preventDefault();
    console.log('Form submitted via property handler');
    
    // Form validation
    const formData = new FormData(event.target);
    for (const [key, value] of formData.entries()) {
        console.log(`${key}: ${value}`);
    }
};

// Window events
window.onload = function() {
    console.log('Page loaded via property handler');
};

window.onresize = function(event) {
    console.log('Window resized:', window.innerWidth, 'x', window.innerHeight);
};

// Mouse events
const div = document.getElementById('myDiv');
div.onmouseenter = function(event) {
    console.log('Mouse entered div');
    event.target.style.backgroundColor = 'lightblue';
};

div.onmouseleave = function(event) {
    console.log('Mouse left div');
    event.target.style.backgroundColor = '';
};

div.onmousedown = function(event) {
    console.log('Mouse down on div');
};

div.onmouseup = function(event) {
    console.log('Mouse up on div');
};

// Keyboard events
document.onkeydown = function(event) {
    console.log('Key pressed:', event.key, event.code);
    
    // Handle specific keys
    switch(event.key) {
        case 'Enter':
            console.log('Enter key pressed');
            break;
        case 'Escape':
            console.log('Escape key pressed');
            break;
        case 'ArrowUp':
        case 'ArrowDown':
        case 'ArrowLeft':
        case 'ArrowRight':
            console.log('Arrow key pressed:', event.key);
            break;
    }
};
```

## Modern Event Listeners (addEventListener)

### Basic addEventListener Usage
```javascript
// Modern event listener approach
const button = document.getElementById('modernButton');

// Add multiple listeners for the same event
button.addEventListener('click', function(event) {
    console.log('First click listener');
});

button.addEventListener('click', function(event) {
    console.log('Second click listener');
});

// Named function listeners (can be removed)
function handleButtonClick(event) {
    console.log('Named function click listener');
    console.log('Button text:', event.target.textContent);
}

button.addEventListener('click', handleButtonClick);

// Remove specific listener
// button.removeEventListener('click', handleButtonClick);

// Event listener options
button.addEventListener('click', function(event) {
    console.log('One-time click listener');
}, { 
    once: true // Remove after first execution
});

button.addEventListener('click', function(event) {
    console.log('Passive click listener');
}, { 
    passive: true // Never calls preventDefault()
});

// Capture phase listener
document.addEventListener('click', function(event) {
    console.log('Document click in capture phase');
}, { 
    capture: true 
});

// Signal-based event listener (AbortController)
const controller = new AbortController();

button.addEventListener('click', function(event) {
    console.log('Abortable click listener');
}, { 
    signal: controller.signal 
});

// Later: controller.abort(); // Removes the listener

// Complex event listener with multiple options
button.addEventListener('mousedown', function(event) {
    console.log('Complex mouse down listener');
    
    if (event.button === 0) {
        console.log('Left mouse button');
    } else if (event.button === 1) {
        console.log('Middle mouse button');
    } else if (event.button === 2) {
        console.log('Right mouse button');
    }
}, {
    capture: false,
    passive: false,
    once: false
});
```

### Advanced Event Listener Patterns
```javascript
// Event listener manager
class EventManager {
    constructor() {
        this.listeners = new Map();
        this.abortController = new AbortController();
    }
    
    // Add event listener with automatic cleanup
    addEventListener(element, eventType, handler, options = {}) {
        const key = `${element}_${eventType}_${handler.name || 'anonymous'}`;
        
        // Merge with abort signal
        const finalOptions = {
            ...options,
            signal: this.abortController.signal
        };
        
        element.addEventListener(eventType, handler, finalOptions);
        
        // Store reference for manual removal
        this.listeners.set(key, {
            element,
            eventType,
            handler,
            options: finalOptions
        });
        
        return key; // Return key for manual removal
    }
    
    // Remove specific listener
    removeEventListener(key) {
        const listener = this.listeners.get(key);
        if (listener) {
            listener.element.removeEventListener(
                listener.eventType, 
                listener.handler, 
                listener.options
            );
            this.listeners.delete(key);
        }
    }
    
    // Remove all listeners
    removeAllListeners() {
        this.abortController.abort();
        this.listeners.clear();
        this.abortController = new AbortController();
    }
    
    // Get listener count
    getListenerCount() {
        return this.listeners.size;
    }
    
    // Add listener with automatic removal after condition
    addTemporaryListener(element, eventType, handler, condition, options = {}) {
        const wrappedHandler = (event) => {
            handler(event);
            
            if (condition(event)) {
                element.removeEventListener(eventType, wrappedHandler, options);
            }
        };
        
        return this.addEventListener(element, eventType, wrappedHandler, options);
    }
    
    // Add debounced listener
    addDebouncedListener(element, eventType, handler, delay = 300, options = {}) {
        let timeoutId;
        
        const debouncedHandler = (event) => {
            clearTimeout(timeoutId);
            timeoutId = setTimeout(() => handler(event), delay);
        };
        
        return this.addEventListener(element, eventType, debouncedHandler, options);
    }
    
    // Add throttled listener
    addThrottledListener(element, eventType, handler, delay = 100, options = {}) {
        let lastCall = 0;
        
        const throttledHandler = (event) => {
            const now = Date.now();
            if (now - lastCall >= delay) {
                lastCall = now;
                handler(event);
            }
        };
        
        return this.addEventListener(element, eventType, throttledHandler, options);
    }
}

// Usage
const eventManager = new EventManager();

const button = document.getElementById('managedButton');

// Add regular listener
eventManager.addEventListener(button, 'click', function(event) {
    console.log('Managed click listener');
});

// Add debounced listener
eventManager.addDebouncedListener(button, 'click', function(event) {
    console.log('Debounced click listener');
}, 500);

// Add temporary listener (removes after 5 clicks)
let clickCount = 0;
eventManager.addTemporaryListener(
    button, 
    'click', 
    function(event) {
        clickCount++;
        console.log(`Temporary click ${clickCount}`);
    },
    () => clickCount >= 5
);

// Cleanup all listeners when done
// eventManager.removeAllListeners();
```

## Event Delegation

### Basic Event Delegation
```javascript
// Event delegation for dynamic content
const container = document.getElementById('buttonContainer');

// Single listener handles all buttons in container
container.addEventListener('click', function(event) {
    // Check if clicked element is a button
    if (event.target.matches('button')) {
        console.log('Button clicked:', event.target.textContent);
        
        // Handle different button types
        const buttonType = event.target.getAttribute('data-type');
        
        switch(buttonType) {
            case 'primary':
                console.log('Primary button action');
                break;
            case 'secondary':
                console.log('Secondary button action');
                break;
            case 'danger':
                if (confirm('Are you sure?')) {
                    console.log('Danger button confirmed');
                }
                break;
            default:
                console.log('Default button action');
        }
    }
});

// Add buttons dynamically
function addButton(text, type = 'default') {
    const button = document.createElement('button');
    button.textContent = text;
    button.setAttribute('data-type', type);
    container.appendChild(button);
}

// These buttons will automatically work with delegation
addButton('Primary Action', 'primary');
addButton('Secondary Action', 'secondary');
addButton('Delete Item', 'danger');

// Advanced delegation with closest()
const listContainer = document.getElementById('listContainer');

listContainer.addEventListener('click', function(event) {
    // Find the closest list item
    const listItem = event.target.closest('li');
    
    if (listItem) {
        // Handle different click targets within list item
        if (event.target.matches('.edit-btn')) {
            console.log('Edit item:', listItem.dataset.id);
        } else if (event.target.matches('.delete-btn')) {
            console.log('Delete item:', listItem.dataset.id);
            listItem.remove();
        } else if (event.target.matches('.toggle-btn')) {
            listItem.classList.toggle('completed');
        }
    }
});

// Delegation with form handling
const formContainer = document.getElementById('formContainer');

formContainer.addEventListener('input', function(event) {
    const input = event.target;
    
    if (input.matches('input[type="email"]')) {
        validateEmail(input);
    } else if (input.matches('input[type="password"]')) {
        validatePassword(input);
    } else if (input.matches('input[required]')) {
        validateRequired(input);
    }
});

formContainer.addEventListener('submit', function(event) {
    if (event.target.matches('form')) {
        event.preventDefault();
        handleFormSubmit(event.target);
    }
});

function validateEmail(input) {
    const isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(input.value);
    input.setCustomValidity(isValid ? '' : 'Please enter a valid email');
}

function validatePassword(input) {
    const isValid = input.value.length >= 8;
    input.setCustomValidity(isValid ? '' : 'Password must be at least 8 characters');
}

function validateRequired(input) {
    const isValid = input.value.trim() !== '';
    input.setCustomValidity(isValid ? '' : 'This field is required');
}

function handleFormSubmit(form) {
    const formData = new FormData(form);
    console.log('Form submitted:', Object.fromEntries(formData));
}
```

### Advanced Delegation Patterns
```javascript
// Delegation framework
class DelegationManager {
    constructor(container) {
        this.container = container;
        this.delegates = new Map();
        this.setupDelegation();
    }
    
    // Add delegated event handler
    delegate(eventType, selector, handler) {
        const key = `${eventType}_${selector}`;
        
        if (!this.delegates.has(eventType)) {
            this.delegates.set(eventType, new Map());
        }
        
        this.delegates.get(eventType).set(selector, handler);
        
        return () => {
            this.delegates.get(eventType).delete(selector);
        };
    }
    
    // Setup main delegation listeners
    setupDelegation() {
        // Common events to delegate
        const events = ['click', 'change', 'input', 'submit', 'keydown', 'focus', 'blur'];
        
        events.forEach(eventType => {
            this.container.addEventListener(eventType, (event) => {
                this.handleDelegatedEvent(eventType, event);
            });
        });
    }
    
    // Handle delegated events
    handleDelegatedEvent(eventType, event) {
        const delegates = this.delegates.get(eventType);
        if (!delegates) return;
        
        // Check each selector
        for (const [selector, handler] of delegates) {
            let target = event.target;
            
            // Walk up the DOM tree
            while (target && target !== this.container) {
                if (target.matches && target.matches(selector)) {
                    handler.call(target, event);
                    break;
                }
                target = target.parentNode;
            }
        }
    }
    
    // Convenience methods for common patterns
    onClick(selector, handler) {
        return this.delegate('click', selector, handler);
    }
    
    onChange(selector, handler) {
        return this.delegate('change', selector, handler);
    }
    
    onInput(selector, handler) {
        return this.delegate('input', selector, handler);
    }
    
    onSubmit(selector, handler) {
        return this.delegate('submit', selector, handler);
    }
    
    // Remove all delegates
    clear() {
        this.delegates.clear();
    }
}

// Usage
const mainContainer = document.getElementById('mainContainer');
const delegator = new DelegationManager(mainContainer);

// Delegate button clicks
const removeButtonDelegate = delegator.onClick('button.btn-primary', function(event) {
    console.log('Primary button clicked:', this.textContent);
});

delegator.onClick('button.btn-secondary', function(event) {
    console.log('Secondary button clicked:', this.textContent);
});

// Delegate form inputs
delegator.onInput('input[data-validate]', function(event) {
    const validationType = this.getAttribute('data-validate');
    validateInput(this, validationType);
});

// Delegate form submissions
delegator.onSubmit('form.ajax-form', function(event) {
    event.preventDefault();
    submitFormAjax(this);
});

function validateInput(input, type) {
    let isValid = true;
    
    switch(type) {
        case 'email':
            isValid = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(input.value);
            break;
        case 'phone':
            isValid = /^\+?[\d\s-()]+$/.test(input.value);
            break;
        case 'required':
            isValid = input.value.trim() !== '';
            break;
    }
    
    input.classList.toggle('invalid', !isValid);
    input.classList.toggle('valid', isValid);
}

async function submitFormAjax(form) {
    const formData = new FormData(form);
    
    try {
        const response = await fetch(form.action, {
            method: form.method || 'POST',
            body: formData
        });
        
        if (response.ok) {
            console.log('Form submitted successfully');
            form.reset();
        } else {
            console.error('Form submission failed');
        }
    } catch (error) {
        console.error('Form submission error:', error);
    }
}
```

## Custom Events

### Creating and Dispatching Custom Events
```javascript
// Basic custom events
const customButton = document.getElementById('customButton');

// Create custom event
const customEvent = new CustomEvent('myCustomEvent', {
    detail: {
        message: 'Hello from custom event!',
        timestamp: Date.now(),
        data: { userId: 123, action: 'button_click' }
    },
    bubbles: true,
    cancelable: true
});

// Listen for custom event
customButton.addEventListener('myCustomEvent', function(event) {
    console.log('Custom event received:', event.detail);
});

// Dispatch custom event
customButton.addEventListener('click', function() {
    this.dispatchEvent(customEvent);
});

// Advanced custom event system
class EventBus {
    constructor() {
        this.events = {};
        this.element = document.createElement('div'); // Hidden element for events
    }
    
    // Subscribe to event
    on(eventName, callback) {
        if (!this.events[eventName]) {
            this.events[eventName] = [];
        }
        
        this.events[eventName].push(callback);
        
        // Add DOM listener
        this.element.addEventListener(eventName, callback);
        
        // Return unsubscribe function
        return () => {
            this.off(eventName, callback);
        };
    }
    
    // Unsubscribe from event
    off(eventName, callback) {
        if (this.events[eventName]) {
            const index = this.events[eventName].indexOf(callback);
            if (index > -1) {
                this.events[eventName].splice(index, 1);
            }
        }
        
        this.element.removeEventListener(eventName, callback);
    }
    
    // Subscribe to event once
    once(eventName, callback) {
        const unsubscribe = this.on(eventName, (event) => {
            callback(event);
            unsubscribe();
        });
        
        return unsubscribe;
    }
    
    // Emit event
    emit(eventName, data = {}) {
        const event = new CustomEvent(eventName, {
            detail: {
                ...data,
                timestamp: Date.now(),
                eventName
            }
        });
        
        this.element.dispatchEvent(event);
    }
    
    // Emit event with delay
    emitAfter(eventName, data, delay) {
        setTimeout(() => {
            this.emit(eventName, data);
        }, delay);
    }
    
    // Get event listener count
    getListenerCount(eventName) {
        return this.events[eventName] ? this.events[eventName].length : 0;
    }
    
    // Remove all listeners
    removeAllListeners() {
        Object.keys(this.events).forEach(eventName => {
            this.events[eventName].forEach(callback => {
                this.element.removeEventListener(eventName, callback);
            });
        });
        this.events = {};
    }
}

// Usage
const eventBus = new EventBus();

// Subscribe to events
const unsubscribe1 = eventBus.on('user:login', (event) => {
    console.log('User logged in:', event.detail);
});

const unsubscribe2 = eventBus.on('user:logout', (event) => {
    console.log('User logged out:', event.detail);
});

// One-time subscription
eventBus.once('app:ready', (event) => {
    console.log('App is ready:', event.detail);
});

// Emit events
eventBus.emit('user:login', {
    userId: 123,
    username: 'john_doe',
    role: 'admin'
});

eventBus.emit('app:ready', {
    version: '1.0.0',
    features: ['auth', 'dashboard', 'reports']
});

// Delayed event
eventBus.emitAfter('user:logout', { userId: 123 }, 5000);

// Application-specific events
class UserManager extends EventBus {
    constructor() {
        super();
        this.users = new Map();
    }
    
    addUser(userData) {
        const user = {
            id: Date.now(),
            ...userData,
            createdAt: new Date()
        };
        
        this.users.set(user.id, user);
        
        // Emit user added event
        this.emit('user:added', { user });
        
        return user;
    }
    
    removeUser(userId) {
        const user = this.users.get(userId);
        
        if (user) {
            this.users.delete(userId);
            this.emit('user:removed', { user });
        }
        
        return user;
    }
    
    updateUser(userId, updates) {
        const user = this.users.get(userId);
        
        if (user) {
            Object.assign(user, updates, { updatedAt: new Date() });
            this.emit('user:updated', { user, updates });
        }
        
        return user;
    }
}

const userManager = new UserManager();

// Listen for user events
userManager.on('user:added', (event) => {
    console.log('New user added:', event.detail.user);
});

userManager.on('user:updated', (event) => {
    console.log('User updated:', event.detail.user);
});

// Add and update users
const newUser = userManager.addUser({
    name: 'John Doe',
    email: 'john@example.com'
});

userManager.updateUser(newUser.id, {
    name: 'John Smith'
});
```

### Event-Driven Architecture
```javascript
// Component communication via events
class Component {
    constructor(element) {
        this.element = element;
        this.eventBus = new EventBus();
        this.setupEventListeners();
    }
    
    setupEventListeners() {
        // Override in subclasses
    }
    
    emit(eventName, data) {
        this.eventBus.emit(eventName, data);
        
        // Also emit on the DOM element
        const domEvent = new CustomEvent(eventName, {
            detail: data,
            bubbles: true
        });
        this.element.dispatchEvent(domEvent);
    }
    
    on(eventName, callback) {
        return this.eventBus.on(eventName, callback);
    }
    
    destroy() {
        this.eventBus.removeAllListeners();
    }
}

// Todo list component
class TodoList extends Component {
    constructor(element) {
        super(element);
        this.todos = [];
        this.render();
    }
    
    setupEventListeners() {
        // Listen for form submissions
        this.element.addEventListener('submit', (event) => {
            if (event.target.matches('.todo-form')) {
                event.preventDefault();
                this.addTodo(event.target);
            }
        });
        
        // Listen for todo actions
        this.element.addEventListener('click', (event) => {
            const todoItem = event.target.closest('.todo-item');
            if (!todoItem) return;
            
            const todoId = parseInt(todoItem.dataset.id);
            
            if (event.target.matches('.complete-btn')) {
                this.toggleTodo(todoId);
            } else if (event.target.matches('.delete-btn')) {
                this.deleteTodo(todoId);
            }
        });
    }
    
    addTodo(form) {
        const formData = new FormData(form);
        const todo = {
            id: Date.now(),
            text: formData.get('text'),
            completed: false,
            createdAt: new Date()
        };
        
        this.todos.push(todo);
        this.emit('todo:added', { todo });
        this.render();
        form.reset();
    }
    
    toggleTodo(id) {
        const todo = this.todos.find(t => t.id === id);
        if (todo) {
            todo.completed = !todo.completed;
            this.emit('todo:toggled', { todo });
            this.render();
        }
    }
    
    deleteTodo(id) {
        const index = this.todos.findIndex(t => t.id === id);
        if (index > -1) {
            const todo = this.todos[index];
            this.todos.splice(index, 1);
            this.emit('todo:deleted', { todo });
            this.render();
        }
    }
    
    render() {
        const completedCount = this.todos.filter(t => t.completed).length;
        const totalCount = this.todos.length;
        
        this.element.innerHTML = `
            <form class="todo-form">
                <input type="text" name="text" placeholder="Add todo..." required>
                <button type="submit">Add</button>
            </form>
            
            <div class="todo-stats">
                ${completedCount} of ${totalCount} completed
            </div>
            
            <ul class="todo-list">
                ${this.todos.map(todo => `
                    <li class="todo-item ${todo.completed ? 'completed' : ''}" data-id="${todo.id}">
                        <span class="todo-text">${todo.text}</span>
                        <button class="complete-btn">${todo.completed ? 'Undo' : 'Complete'}</button>
                        <button class="delete-btn">Delete</button>
                    </li>
                `).join('')}
            </ul>
        `;
        
        this.emit('todo:rendered', { 
            todos: this.todos, 
            completedCount, 
            totalCount 
        });
    }
}

// Statistics component
class TodoStats extends Component {
    constructor(element) {
        super(element);
        this.stats = {
            total: 0,
            completed: 0,
            pending: 0
        };
    }
    
    updateStats(todos) {
        this.stats.total = todos.length;
        this.stats.completed = todos.filter(t => t.completed).length;
        this.stats.pending = this.stats.total - this.stats.completed;
        
        this.render();
        this.emit('stats:updated', { stats: this.stats });
    }
    
    render() {
        this.element.innerHTML = `
            <div class="stats">
                <div class="stat">
                    <span class="label">Total:</span>
                    <span class="value">${this.stats.total}</span>
                </div>
                <div class="stat">
                    <span class="label">Completed:</span>
                    <span class="value">${this.stats.completed}</span>
                </div>
                <div class="stat">
                    <span class="label">Pending:</span>
                    <span class="value">${this.stats.pending}</span>
                </div>
            </div>
        `;
    }
}

// Connect components
const todoListElement = document.getElementById('todoList');
const statsElement = document.getElementById('todoStats');

const todoList = new TodoList(todoListElement);
const todoStats = new TodoStats(statsElement);

// Connect todo list events to stats
todoList.on('todo:rendered', (event) => {
    todoStats.updateStats(event.detail.todos);
});

// Global event logging
['todo:added', 'todo:toggled', 'todo:deleted'].forEach(eventName => {
    todoList.on(eventName, (event) => {
        console.log(`Event: ${eventName}`, event.detail);
    });
});
```

## Event Performance and Best Practices

### Performance Optimization
```javascript
// Efficient event handling patterns
class PerformantEventHandler {
    constructor() {
        this.throttledHandlers = new Map();
        this.debouncedHandlers = new Map();
    }
    
    // Throttle function
    throttle(func, delay) {
        let timeoutId;
        let lastExecTime = 0;
        
        return function(...args) {
            const currentTime = Date.now();
            
            if (currentTime - lastExecTime > delay) {
                func.apply(this, args);
                lastExecTime = currentTime;
            } else {
                clearTimeout(timeoutId);
                timeoutId = setTimeout(() => {
                    func.apply(this, args);
                    lastExecTime = Date.now();
                }, delay - (currentTime - lastExecTime));
            }
        };
    }
    
    // Debounce function
    debounce(func, delay) {
        let timeoutId;
        
        return function(...args) {
            clearTimeout(timeoutId);
            timeoutId = setTimeout(() => func.apply(this, args), delay);
        };
    }
    
    // Add throttled event listener
    addThrottledListener(element, eventType, handler, delay = 100) {
        const throttledHandler = this.throttle(handler, delay);
        element.addEventListener(eventType, throttledHandler);
        
        return () => element.removeEventListener(eventType, throttledHandler);
    }
    
    // Add debounced event listener
    addDebouncedListener(element, eventType, handler, delay = 300) {
        const debouncedHandler = this.debounce(handler, delay);
        element.addEventListener(eventType, debouncedHandler);
        
        return () => element.removeEventListener(eventType, debouncedHandler);
    }
    
    // Passive event listeners for better performance
    addPassiveListener(element, eventType, handler) {
        element.addEventListener(eventType, handler, { passive: true });
        
        return () => element.removeEventListener(eventType, handler, { passive: true });
    }
    
    // Event listener with performance monitoring
    addMonitoredListener(element, eventType, handler, options = {}) {
        const monitoredHandler = (event) => {
            const startTime = performance.now();
            
            try {
                handler(event);
            } catch (error) {
                console.error('Event handler error:', error);
            } finally {
                const endTime = performance.now();
                const duration = endTime - startTime;
                
                if (duration > 16) { // Longer than a frame (60fps)
                    console.warn(`Slow event handler: ${eventType} took ${duration.toFixed(2)}ms`);
                }
            }
        };
        
        element.addEventListener(eventType, monitoredHandler, options);
        
        return () => element.removeEventListener(eventType, monitoredHandler, options);
    }
}

// Usage
const performantHandler = new PerformantEventHandler();

// Throttled scroll handler
const removeScrollHandler = performantHandler.addThrottledListener(
    window,
    'scroll',
    () => {
        console.log('Throttled scroll:', window.pageYOffset);
    },
    100
);

// Debounced resize handler
const removeResizeHandler = performantHandler.addDebouncedListener(
    window,
    'resize',
    () => {
        console.log('Debounced resize:', window.innerWidth, 'x', window.innerHeight);
    },
    250
);

// Passive touch handlers
const removeTouchHandler = performantHandler.addPassiveListener(
    document,
    'touchstart',
    (event) => {
        console.log('Passive touch start');
    }
);

// Memory leak prevention
class EventCleanupManager {
    constructor() {
        this.cleanupFunctions = [];
        this.weakMapListeners = new WeakMap();
    }
    
    // Add listener with automatic cleanup tracking
    addListener(element, eventType, handler, options) {
        element.addEventListener(eventType, handler, options);
        
        const cleanup = () => {
            element.removeEventListener(eventType, handler, options);
        };
        
        this.cleanupFunctions.push(cleanup);
        
        // Store in WeakMap for element-specific cleanup
        if (!this.weakMapListeners.has(element)) {
            this.weakMapListeners.set(element, []);
        }
        this.weakMapListeners.get(element).push(cleanup);
        
        return cleanup;
    }
    
    // Clean up listeners for specific element
    cleanupElement(element) {
        const cleanupFunctions = this.weakMapListeners.get(element);
        if (cleanupFunctions) {
            cleanupFunctions.forEach(cleanup => cleanup());
            this.weakMapListeners.delete(element);
        }
    }
    
    // Clean up all listeners
    cleanupAll() {
        this.cleanupFunctions.forEach(cleanup => cleanup());
        this.cleanupFunctions = [];
    }
    
    // Auto cleanup when element is removed from DOM
    observeElementRemoval(element) {
        if ('MutationObserver' in window) {
            const observer = new MutationObserver((mutations) => {
                mutations.forEach((mutation) => {
                    mutation.removedNodes.forEach((node) => {
                        if (node === element || (node.contains && node.contains(element))) {
                            this.cleanupElement(element);
                            observer.disconnect();
                        }
                    });
                });
            });
            
            observer.observe(document.body, {
                childList: true,
                subtree: true
            });
        }
    }
}

// Best practices for event handling
const bestPractices = {
    // Use event delegation for dynamic content
    setupDelegation() {
        document.addEventListener('click', (event) => {
            // More efficient than adding individual listeners
            if (event.target.matches('.dynamic-button')) {
                console.log('Dynamic button clicked');
            }
        });
    },
    
    // Avoid anonymous functions for removable listeners
    setupRemovableListeners() {
        function namedHandler(event) {
            console.log('Named handler called');
        }
        
        // Can be removed later
        document.addEventListener('click', namedHandler);
        
        // Later: document.removeEventListener('click', namedHandler);
    },
    
    // Use AbortController for modern cleanup
    setupAbortableListeners() {
        const controller = new AbortController();
        
        document.addEventListener('click', (event) => {
            console.log('Abortable listener');
        }, { signal: controller.signal });
        
        document.addEventListener('scroll', (event) => {
            console.log('Another abortable listener');
        }, { signal: controller.signal });
        
        // Remove all listeners at once
        // controller.abort();
    },
    
    // Prefer passive listeners for touch/scroll events
    setupPassiveListeners() {
        document.addEventListener('touchstart', (event) => {
            // Can't call preventDefault() in passive listener
            console.log('Passive touch start');
        }, { passive: true });
        
        document.addEventListener('wheel', (event) => {
            // Better scrolling performance
            console.log('Passive wheel');
        }, { passive: true });
    }
};

// Initialize best practices
// bestPractices.setupDelegation();
// bestPractices.setupAbortableListeners();
// bestPractices.setupPassiveListeners();
```

---

## Related Topics
- [[14 - Events]]
- [[15 - Document Object Model]]
- [[16 - Window Object]]
- [[18 - Callbacks]]
- [[35 - JavaScript OOP]]

---

*Next: [[21 - XMLHttpRequest]]*
