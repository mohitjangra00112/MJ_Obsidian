# Events

## Overview
Events are actions or occurrences that happen in the browser that can be detected and responded to with JavaScript. Understanding event handling is crucial for creating interactive web applications.

## Event Types

### Mouse Events
```javascript
// Click events
element.addEventListener('click', function(event) {
    console.log('Element clicked!');
    console.log('Mouse position:', event.clientX, event.clientY);
});

// Double click
element.addEventListener('dblclick', function(event) {
    console.log('Double clicked!');
});

// Mouse movement
element.addEventListener('mouseover', function(event) {
    console.log('Mouse entered element');
});

element.addEventListener('mouseout', function(event) {
    console.log('Mouse left element');
});

element.addEventListener('mousemove', function(event) {
    console.log('Mouse position:', event.clientX, event.clientY);
});

// Mouse button events
element.addEventListener('mousedown', function(event) {
    console.log('Mouse button pressed:', event.button);
    // 0 = left, 1 = middle, 2 = right
});

element.addEventListener('mouseup', function(event) {
    console.log('Mouse button released');
});

// Context menu (right-click)
element.addEventListener('contextmenu', function(event) {
    event.preventDefault(); // Prevent default context menu
    console.log('Right-clicked!');
});
```

### Keyboard Events
```javascript
// Key press events
document.addEventListener('keydown', function(event) {
    console.log('Key pressed:', event.key);
    console.log('Key code:', event.keyCode);
    console.log('Ctrl key:', event.ctrlKey);
    console.log('Shift key:', event.shiftKey);
    console.log('Alt key:', event.altKey);
});

document.addEventListener('keyup', function(event) {
    console.log('Key released:', event.key);
});

// Deprecated but still used
document.addEventListener('keypress', function(event) {
    console.log('Key pressed (deprecated):', event.key);
});

// Practical keyboard shortcuts
document.addEventListener('keydown', function(event) {
    // Save shortcut (Ctrl+S)
    if (event.ctrlKey && event.key === 's') {
        event.preventDefault();
        saveDocument();
    }
    
    // Copy shortcut (Ctrl+C)
    if (event.ctrlKey && event.key === 'c') {
        console.log('Copy triggered');
    }
    
    // Escape key
    if (event.key === 'Escape') {
        closeModal();
    }
    
    // Arrow keys navigation
    switch(event.key) {
        case 'ArrowUp':
            navigateUp();
            break;
        case 'ArrowDown':
            navigateDown();
            break;
        case 'ArrowLeft':
            navigateLeft();
            break;
        case 'ArrowRight':
            navigateRight();
            break;
    }
});

function saveDocument() {
    console.log('Document saved!');
}

function closeModal() {
    console.log('Modal closed!');
}

function navigateUp() { console.log('Navigate up'); }
function navigateDown() { console.log('Navigate down'); }
function navigateLeft() { console.log('Navigate left'); }
function navigateRight() { console.log('Navigate right'); }
```

### Form Events
```javascript
const form = document.getElementById('myForm');
const input = document.getElementById('myInput');

// Form submission
form.addEventListener('submit', function(event) {
    event.preventDefault(); // Prevent default form submission
    
    // Validate form
    if (validateForm()) {
        submitForm();
    } else {
        console.log('Form validation failed');
    }
});

// Input events
input.addEventListener('input', function(event) {
    console.log('Input value changed:', event.target.value);
    validateInput(event.target);
});

input.addEventListener('change', function(event) {
    console.log('Input change event:', event.target.value);
});

input.addEventListener('focus', function(event) {
    console.log('Input focused');
    event.target.style.backgroundColor = '#f0f8ff';
});

input.addEventListener('blur', function(event) {
    console.log('Input lost focus');
    event.target.style.backgroundColor = '';
    validateInput(event.target);
});

// Select events
const select = document.getElementById('mySelect');
select.addEventListener('change', function(event) {
    console.log('Selected value:', event.target.value);
});

// Checkbox and radio events
const checkbox = document.getElementById('myCheckbox');
checkbox.addEventListener('change', function(event) {
    console.log('Checkbox checked:', event.target.checked);
});

function validateForm() {
    // Add validation logic
    return true;
}

function submitForm() {
    console.log('Form submitted successfully!');
}

function validateInput(input) {
    if (input.value.length < 3) {
        input.style.borderColor = 'red';
    } else {
        input.style.borderColor = '';
    }
}
```

### Window Events
```javascript
// Page load events
window.addEventListener('load', function(event) {
    console.log('Page fully loaded (including images, stylesheets)');
});

document.addEventListener('DOMContentLoaded', function(event) {
    console.log('DOM content loaded (HTML parsed)');
    initializeApp();
});

// Page unload events
window.addEventListener('beforeunload', function(event) {
    // Warn user before leaving page
    event.preventDefault();
    event.returnValue = 'Are you sure you want to leave?';
});

window.addEventListener('unload', function(event) {
    console.log('Page is unloading');
    // Clean up resources
    cleanup();
});

// Window resize
window.addEventListener('resize', function(event) {
    console.log('Window resized:', window.innerWidth, 'x', window.innerHeight);
    handleResize();
});

// Scroll events
window.addEventListener('scroll', function(event) {
    console.log('Page scrolled:', window.scrollY);
    handleScroll();
});

// Visibility change (tab switch)
document.addEventListener('visibilitychange', function(event) {
    if (document.hidden) {
        console.log('Tab is hidden');
        pauseOperations();
    } else {
        console.log('Tab is visible');
        resumeOperations();
    }
});

function initializeApp() {
    console.log('App initialized');
}

function cleanup() {
    console.log('Cleaning up resources');
}

function handleResize() {
    // Responsive layout adjustments
    if (window.innerWidth < 768) {
        document.body.classList.add('mobile');
    } else {
        document.body.classList.remove('mobile');
    }
}

function handleScroll() {
    // Infinite scroll, show/hide header, etc.
    const scrollTop = window.scrollY;
    const header = document.getElementById('header');
    
    if (scrollTop > 100) {
        header.classList.add('scrolled');
    } else {
        header.classList.remove('scrolled');
    }
}

function pauseOperations() {
    // Pause timers, animations, etc.
}

function resumeOperations() {
    // Resume timers, animations, etc.
}
```

## Event Object Properties

### Common Event Properties
```javascript
element.addEventListener('click', function(event) {
    // Event type
    console.log('Event type:', event.type); // 'click'
    
    // Target element (where event originated)
    console.log('Target:', event.target);
    
    // Current target (element with event listener)
    console.log('Current target:', event.currentTarget);
    
    // Timestamp
    console.log('Timestamp:', event.timeStamp);
    
    // Prevent default behavior
    event.preventDefault();
    
    // Stop event propagation
    event.stopPropagation();
    
    // Stop immediate propagation
    event.stopImmediatePropagation();
    
    // Check if default was prevented
    console.log('Default prevented:', event.defaultPrevented);
});

// Mouse event specific properties
element.addEventListener('click', function(event) {
    console.log('Client coordinates:', event.clientX, event.clientY);
    console.log('Page coordinates:', event.pageX, event.pageY);
    console.log('Screen coordinates:', event.screenX, event.screenY);
    console.log('Mouse button:', event.button);
    console.log('Buttons pressed:', event.buttons);
    console.log('Modifier keys:', {
        ctrl: event.ctrlKey,
        shift: event.shiftKey,
        alt: event.altKey,
        meta: event.metaKey
    });
});

// Keyboard event specific properties
document.addEventListener('keydown', function(event) {
    console.log('Key:', event.key);
    console.log('Code:', event.code);
    console.log('Key Code:', event.keyCode); // Deprecated
    console.log('Repeat:', event.repeat);
    console.log('Location:', event.location);
});
```

## Event Propagation

### Bubbling and Capturing
```javascript
// Event phases demonstration
const outer = document.getElementById('outer');
const middle = document.getElementById('middle');
const inner = document.getElementById('inner');

// Capturing phase (true parameter)
outer.addEventListener('click', function(event) {
    console.log('Outer - Capturing');
}, true);

middle.addEventListener('click', function(event) {
    console.log('Middle - Capturing');
}, true);

inner.addEventListener('click', function(event) {
    console.log('Inner - Capturing');
}, true);

// Bubbling phase (default)
inner.addEventListener('click', function(event) {
    console.log('Inner - Bubbling');
});

middle.addEventListener('click', function(event) {
    console.log('Middle - Bubbling');
});

outer.addEventListener('click', function(event) {
    console.log('Outer - Bubbling');
});

// Stopping propagation
middle.addEventListener('click', function(event) {
    console.log('Middle clicked - stopping propagation');
    event.stopPropagation(); // Prevents further bubbling
});

// Stopping immediate propagation
inner.addEventListener('click', function(event) {
    console.log('Inner - First listener');
    event.stopImmediatePropagation();
});

inner.addEventListener('click', function(event) {
    console.log('Inner - Second listener (won\'t execute)');
});
```

### Event Delegation
```javascript
// Instead of adding listeners to each item
const listItems = document.querySelectorAll('.list-item');
listItems.forEach(item => {
    item.addEventListener('click', handleItemClick);
});

// Use event delegation on parent
const list = document.getElementById('item-list');
list.addEventListener('click', function(event) {
    // Check if clicked element is a list item
    if (event.target.classList.contains('list-item')) {
        handleItemClick(event);
    }
    
    // Or check by tag name
    if (event.target.tagName === 'LI') {
        handleItemClick(event);
    }
    
    // Or use closest() method
    const listItem = event.target.closest('.list-item');
    if (listItem) {
        handleItemClick(event);
    }
});

function handleItemClick(event) {
    console.log('List item clicked:', event.target.textContent);
}

// Advanced delegation with data attributes
document.addEventListener('click', function(event) {
    // Handle buttons with action data attribute
    const action = event.target.dataset.action;
    
    switch(action) {
        case 'save':
            saveData();
            break;
        case 'delete':
            deleteData(event.target.dataset.id);
            break;
        case 'edit':
            editData(event.target.dataset.id);
            break;
    }
});

// HTML: <button data-action="save">Save</button>
// HTML: <button data-action="delete" data-id="123">Delete</button>

function saveData() { console.log('Saving data...'); }
function deleteData(id) { console.log('Deleting item:', id); }
function editData(id) { console.log('Editing item:', id); }
```

## Custom Events

### Creating and Dispatching Custom Events
```javascript
// Create custom event
const customEvent = new Event('myCustomEvent');

// Create custom event with data
const customEventWithData = new CustomEvent('dataEvent', {
    detail: {
        message: 'Hello World!',
        timestamp: new Date(),
        user: 'John Doe'
    },
    bubbles: true,
    cancelable: true
});

// Listen for custom events
document.addEventListener('myCustomEvent', function(event) {
    console.log('Custom event fired!');
});

document.addEventListener('dataEvent', function(event) {
    console.log('Data event:', event.detail);
});

// Dispatch custom events
document.dispatchEvent(customEvent);
document.dispatchEvent(customEventWithData);

// Practical example: Component communication
class EventEmitter {
    constructor() {
        this.element = document.createElement('div');
    }
    
    emit(eventName, data) {
        const event = new CustomEvent(eventName, {
            detail: data
        });
        this.element.dispatchEvent(event);
    }
    
    on(eventName, callback) {
        this.element.addEventListener(eventName, callback);
    }
    
    off(eventName, callback) {
        this.element.removeEventListener(eventName, callback);
    }
}

// Usage
const emitter = new EventEmitter();

emitter.on('userLogin', function(event) {
    console.log('User logged in:', event.detail.username);
});

emitter.emit('userLogin', { username: 'john_doe', timestamp: Date.now() });

// Application-wide event system
class AppEvents {
    static emit(eventName, data) {
        const event = new CustomEvent(`app:${eventName}`, {
            detail: data
        });
        document.dispatchEvent(event);
    }
    
    static on(eventName, callback) {
        document.addEventListener(`app:${eventName}`, callback);
    }
}

// Usage
AppEvents.on('userLogout', function(event) {
    console.log('User logged out, redirecting...');
    window.location.href = '/login';
});

AppEvents.emit('userLogout', { reason: 'session_expired' });
```

## Advanced Event Handling

### Throttling and Debouncing
```javascript
// Throttle function - limits function calls to once per specified time
function throttle(func, limit) {
    let inThrottle;
    return function() {
        const args = arguments;
        const context = this;
        if (!inThrottle) {
            func.apply(context, args);
            inThrottle = true;
            setTimeout(() => inThrottle = false, limit);
        }
    }
}

// Debounce function - delays function execution until after specified time
function debounce(func, delay) {
    let timeoutId;
    return function() {
        const args = arguments;
        const context = this;
        clearTimeout(timeoutId);
        timeoutId = setTimeout(() => func.apply(context, args), delay);
    }
}

// Usage examples
const throttledScroll = throttle(function(event) {
    console.log('Scroll event (throttled)');
}, 100);

const debouncedSearch = debounce(function(event) {
    console.log('Search query:', event.target.value);
    performSearch(event.target.value);
}, 300);

window.addEventListener('scroll', throttledScroll);
searchInput.addEventListener('input', debouncedSearch);

function performSearch(query) {
    // Perform actual search
    console.log('Searching for:', query);
}

// Practical scroll optimization
let isScrolling = false;

function optimizedScrollHandler() {
    if (!isScrolling) {
        requestAnimationFrame(function() {
            // Perform scroll-related operations
            updateScrollProgress();
            isScrolling = false;
        });
        isScrolling = true;
    }
}

function updateScrollProgress() {
    const scrollPercent = (window.scrollY / (document.body.scrollHeight - window.innerHeight)) * 100;
    document.getElementById('scroll-progress').style.width = scrollPercent + '%';
}

window.addEventListener('scroll', optimizedScrollHandler);
```

### Event Listener Management
```javascript
class EventManager {
    constructor() {
        this.listeners = new Map();
    }
    
    addListener(element, eventType, callback, options = {}) {
        const wrappedCallback = (event) => {
            try {
                callback.call(element, event);
            } catch (error) {
                console.error('Event handler error:', error);
            }
        };
        
        element.addEventListener(eventType, wrappedCallback, options);
        
        // Store for later removal
        const key = `${eventType}_${callback.name || 'anonymous'}`;
        this.listeners.set(key, {
            element,
            eventType,
            callback: wrappedCallback,
            options
        });
        
        return key;
    }
    
    removeListener(key) {
        const listener = this.listeners.get(key);
        if (listener) {
            listener.element.removeEventListener(
                listener.eventType,
                listener.callback,
                listener.options
            );
            this.listeners.delete(key);
        }
    }
    
    removeAllListeners() {
        this.listeners.forEach((listener, key) => {
            this.removeListener(key);
        });
    }
    
    // Add temporary listener that removes itself after first execution
    addOnceListener(element, eventType, callback, options = {}) {
        const onceCallback = (event) => {
            callback.call(element, event);
            element.removeEventListener(eventType, onceCallback, options);
        };
        
        element.addEventListener(eventType, onceCallback, options);
    }
}

// Usage
const eventManager = new EventManager();

const clickKey = eventManager.addListener(button, 'click', function(event) {
    console.log('Button clicked!');
});

// Remove specific listener
eventManager.removeListener(clickKey);

// One-time listener
eventManager.addOnceListener(document, 'click', function(event) {
    console.log('This will only run once');
});

// Cleanup all listeners
eventManager.removeAllListeners();
```

### Touch Events (Mobile)
```javascript
let startX, startY, currentX, currentY;

element.addEventListener('touchstart', function(event) {
    const touch = event.touches[0];
    startX = touch.clientX;
    startY = touch.clientY;
    console.log('Touch started at:', startX, startY);
}, { passive: true });

element.addEventListener('touchmove', function(event) {
    const touch = event.touches[0];
    currentX = touch.clientX;
    currentY = touch.clientY;
    
    const deltaX = currentX - startX;
    const deltaY = currentY - startY;
    
    console.log('Touch moved:', deltaX, deltaY);
    
    // Prevent scrolling if needed
    // event.preventDefault();
}, { passive: false });

element.addEventListener('touchend', function(event) {
    const deltaX = currentX - startX;
    const deltaY = currentY - startY;
    
    // Detect swipe direction
    if (Math.abs(deltaX) > Math.abs(deltaY)) {
        if (deltaX > 50) {
            console.log('Swipe right');
        } else if (deltaX < -50) {
            console.log('Swipe left');
        }
    } else {
        if (deltaY > 50) {
            console.log('Swipe down');
        } else if (deltaY < -50) {
            console.log('Swipe up');
        }
    }
}, { passive: true });

// Touch gestures
element.addEventListener('gesturestart', function(event) {
    console.log('Gesture started');
});

element.addEventListener('gesturechange', function(event) {
    console.log('Gesture scale:', event.scale);
    console.log('Gesture rotation:', event.rotation);
});

element.addEventListener('gestureend', function(event) {
    console.log('Gesture ended');
});
```

## Event Best Practices

### Performance Optimization
```javascript
// Use passive listeners for scroll/touch events
window.addEventListener('scroll', handleScroll, { passive: true });
element.addEventListener('touchstart', handleTouch, { passive: true });

// Remove event listeners when no longer needed
function cleanup() {
    window.removeEventListener('scroll', handleScroll);
    element.removeEventListener('click', handleClick);
}

// Use event delegation instead of multiple listeners
// Bad: Adding listener to each item
document.querySelectorAll('.item').forEach(item => {
    item.addEventListener('click', handleItemClick);
});

// Good: Single listener on parent
document.getElementById('container').addEventListener('click', function(event) {
    if (event.target.classList.contains('item')) {
        handleItemClick(event);
    }
});

// Avoid memory leaks with proper cleanup
class Component {
    constructor(element) {
        this.element = element;
        this.boundHandleClick = this.handleClick.bind(this);
        this.element.addEventListener('click', this.boundHandleClick);
    }
    
    handleClick(event) {
        console.log('Component clicked');
    }
    
    destroy() {
        this.element.removeEventListener('click', this.boundHandleClick);
        this.element = null;
        this.boundHandleClick = null;
    }
}

function handleScroll() { /* scroll handler */ }
function handleTouch() { /* touch handler */ }
function handleClick() { /* click handler */ }
function handleItemClick() { /* item click handler */ }
```

---

## Related Topics
- [[15 - Document Object Model]]
- [[17 - Event Methods]]
- [[43 - Browser APIs]]
- [[46 - Debugging Techniques]]

---

*Next: [[17 - Event Methods]]*
