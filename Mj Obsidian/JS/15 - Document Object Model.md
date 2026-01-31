# Document Object Model (DOM)

## Overview
The Document Object Model (DOM) is a programming interface for HTML and XML documents. It represents the page structure as a tree of objects that JavaScript can manipulate to dynamically change content, structure, and styles.

## DOM Structure

### Document Tree
```html
<!DOCTYPE html>
<html>
<head>
    <title>My Page</title>
</head>
<body>
    <div class="container">
        <h1 id="title">Welcome</h1>
        <p class="description">This is a paragraph.</p>
        <ul>
            <li>Item 1</li>
            <li>Item 2</li>
        </ul>
    </div>
</body>
</html>
```

```javascript
// DOM tree representation:
// document
//   └── html
//       ├── head
//       │   └── title
//       └── body
//           └── div.container
//               ├── h1#title
//               ├── p.description
//               └── ul
//                   ├── li
//                   └── li
```

## Selecting Elements

### getElementById()
```javascript
// Select element by ID
const titleElement = document.getElementById('title');
console.log(titleElement); // <h1 id="title">Welcome</h1>

// Returns null if not found
const nonExistent = document.getElementById('notfound');
console.log(nonExistent); // null
```

### getElementsByClassName()
```javascript
// Returns HTMLCollection (live collection)
const descriptions = document.getElementsByClassName('description');
console.log(descriptions.length); // 1
console.log(descriptions[0]); // <p class="description">This is a paragraph.</p>

// Multiple classes
const elements = document.getElementsByClassName('nav-item active');

// Convert to array for array methods
const descriptionsArray = Array.from(descriptions);
```

### getElementsByTagName()
```javascript
// Select by tag name
const paragraphs = document.getElementsByTagName('p');
const listItems = document.getElementsByTagName('li');

// Select all elements
const allElements = document.getElementsByTagName('*');

// From specific element
const container = document.getElementById('container');
const containerParagraphs = container.getElementsByTagName('p');
```

### querySelector() and querySelectorAll()
```javascript
// querySelector - returns first match
const title = document.querySelector('#title');
const firstParagraph = document.querySelector('p');
const firstDescription = document.querySelector('.description');

// Complex selectors
const firstListItem = document.querySelector('ul li:first-child');
const activeNavItem = document.querySelector('.nav-item.active');

// querySelectorAll - returns NodeList (static collection)
const allParagraphs = document.querySelectorAll('p');
const allListItems = document.querySelectorAll('li');

// Advanced selectors
const evenListItems = document.querySelectorAll('li:nth-child(even)');
const directChildren = document.querySelectorAll('.container > p');

// NodeList has forEach method
allListItems.forEach(item => {
    console.log(item.textContent);
});
```

## Element Properties and Methods

### Content Manipulation
```javascript
const element = document.querySelector('#title');

// Text content (no HTML)
console.log(element.textContent); // "Welcome"
element.textContent = "New Title";

// Inner text (respects styling, slower)
console.log(element.innerText); // "Welcome"
element.innerText = "Another Title";

// HTML content
const container = document.querySelector('.container');
console.log(container.innerHTML); // Full HTML inside
container.innerHTML = '<p>New <strong>content</strong></p>';

// Outer HTML (includes the element itself)
console.log(element.outerHTML); // '<h1 id="title">Welcome</h1>'
```

### Attributes
```javascript
const link = document.querySelector('a');

// Get attribute
const href = link.getAttribute('href');
const className = link.getAttribute('class');

// Set attribute
link.setAttribute('href', 'https://example.com');
link.setAttribute('target', '_blank');

// Remove attribute
link.removeAttribute('target');

// Check if attribute exists
const hasClass = link.hasAttribute('class');

// Direct property access (for standard attributes)
console.log(link.href); // Full URL
console.log(link.className); // Class names as string
console.log(link.id);

// Data attributes
link.setAttribute('data-user-id', '123');
console.log(link.dataset.userId); // "123" (camelCase conversion)
link.dataset.userRole = 'admin';
```

### Style Manipulation
```javascript
const element = document.querySelector('#title');

// Direct style properties
element.style.color = 'red';
element.style.fontSize = '24px';
element.style.backgroundColor = 'yellow';

// CSS properties with hyphens become camelCase
element.style.borderRadius = '5px';
element.style.marginTop = '10px';

// Get computed styles
const computedStyle = window.getComputedStyle(element);
console.log(computedStyle.color); // Computed color value
console.log(computedStyle.fontSize); // Computed font size

// CSS custom properties (variables)
element.style.setProperty('--main-color', 'blue');
const mainColor = computedStyle.getPropertyValue('--main-color');
```

### Classes
```javascript
const element = document.querySelector('.container');

// Add classes
element.classList.add('active');
element.classList.add('highlight', 'important'); // Multiple classes

// Remove classes
element.classList.remove('highlight');

// Toggle class
element.classList.toggle('visible'); // Adds if not present, removes if present

// Check if class exists
const hasActive = element.classList.contains('active');

// Replace class
element.classList.replace('old-class', 'new-class');

// Class list as array
const classes = Array.from(element.classList);
```

## Creating and Modifying Elements

### Creating Elements
```javascript
// Create element
const newDiv = document.createElement('div');
const newParagraph = document.createElement('p');
const newImage = document.createElement('img');

// Set properties
newDiv.className = 'new-container';
newDiv.id = 'dynamic-content';
newParagraph.textContent = 'This is a new paragraph';
newImage.src = 'image.jpg';
newImage.alt = 'Description';

// Create text node
const textNode = document.createTextNode('Hello World');

// Create document fragment (for performance)
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
    const li = document.createElement('li');
    li.textContent = `Item ${i}`;
    fragment.appendChild(li);
}
```

### Inserting Elements
```javascript
const container = document.querySelector('.container');
const newElement = document.createElement('div');
newElement.textContent = 'New content';

// Append as last child
container.appendChild(newElement);

// Insert before specific element
const referenceElement = document.querySelector('.description');
container.insertBefore(newElement, referenceElement);

// Modern insertion methods
const anotherElement = document.createElement('span');

// Insert at different positions
container.prepend(anotherElement); // First child
container.append(anotherElement); // Last child

referenceElement.before(anotherElement); // Before reference
referenceElement.after(anotherElement); // After reference

// Insert adjacent HTML
referenceElement.insertAdjacentHTML('beforebegin', '<div>Before</div>');
referenceElement.insertAdjacentHTML('afterbegin', '<div>At start</div>');
referenceElement.insertAdjacentHTML('beforeend', '<div>At end</div>');
referenceElement.insertAdjacentHTML('afterend', '<div>After</div>');
```

### Removing Elements
```javascript
const element = document.querySelector('#unwanted');

// Modern way
element.remove();

// Old way (still works)
element.parentNode.removeChild(element);

// Remove all children
const container = document.querySelector('.container');
container.innerHTML = ''; // Quick but not ideal for event listeners

// Better way to remove all children
while (container.firstChild) {
    container.removeChild(container.firstChild);
}

// Or using modern methods
container.replaceChildren(); // Removes all children
```

### Cloning Elements
```javascript
const original = document.querySelector('.template');

// Shallow clone (no children or event listeners)
const shallowClone = original.cloneNode();

// Deep clone (includes children but not event listeners)
const deepClone = original.cloneNode(true);

// Clone with event listeners (manual approach)
function cloneWithEvents(element) {
    const clone = element.cloneNode(true);
    // Re-attach event listeners manually
    return clone;
}
```

## Navigation and Relationships

### Parent/Child Relationships
```javascript
const element = document.querySelector('.description');

// Parent relationships
console.log(element.parentNode); // Direct parent
console.log(element.parentElement); // Parent element (same as parentNode for elements)

// Find closest ancestor matching selector
const container = element.closest('.container');
const article = element.closest('article');

// Child relationships
const container = document.querySelector('.container');
console.log(container.childNodes); // All child nodes (including text nodes)
console.log(container.children); // Only element children

console.log(container.firstChild); // First child node
console.log(container.firstElementChild); // First element child
console.log(container.lastChild); // Last child node
console.log(container.lastElementChild); // Last element child

// Check if element has children
console.log(container.hasChildNodes());
console.log(container.children.length > 0);
```

### Sibling Relationships
```javascript
const element = document.querySelector('.description');

// Next siblings
console.log(element.nextSibling); // Next node (might be text)
console.log(element.nextElementSibling); // Next element

// Previous siblings
console.log(element.previousSibling); // Previous node
console.log(element.previousElementSibling); // Previous element

// Get all siblings
function getSiblings(element) {
    const siblings = [];
    let sibling = element.parentNode.firstElementChild;
    
    while (sibling) {
        if (sibling !== element) {
            siblings.push(sibling);
        }
        sibling = sibling.nextElementSibling;
    }
    
    return siblings;
}
```

## Event Handling

### addEventListener()
```javascript
const button = document.querySelector('#myButton');

// Basic event listener
button.addEventListener('click', function() {
    console.log('Button clicked!');
});

// Arrow function
button.addEventListener('click', () => {
    console.log('Button clicked!');
});

// Named function for removal
function handleClick(event) {
    console.log('Button clicked!', event);
}
button.addEventListener('click', handleClick);

// Event listener options
button.addEventListener('click', handleClick, {
    once: true,      // Run only once
    passive: true,   // Never calls preventDefault
    capture: true    // Capture phase
});

// Remove event listener
button.removeEventListener('click', handleClick);
```

### Event Object
```javascript
function handleEvent(event) {
    // Event properties
    console.log(event.type); // Event type (e.g., 'click')
    console.log(event.target); // Element that triggered event
    console.log(event.currentTarget); // Element with event listener
    console.log(event.timeStamp); // When event occurred
    
    // Mouse events
    if (event.type === 'click') {
        console.log(event.clientX, event.clientY); // Mouse coordinates
        console.log(event.button); // Which button (0=left, 1=middle, 2=right)
        console.log(event.ctrlKey, event.shiftKey); // Modifier keys
    }
    
    // Keyboard events
    if (event.type === 'keydown') {
        console.log(event.key); // Key pressed
        console.log(event.code); // Physical key
        console.log(event.keyCode); // Numeric key code (deprecated)
    }
    
    // Prevent default behavior
    event.preventDefault();
    
    // Stop event propagation
    event.stopPropagation();
    event.stopImmediatePropagation(); // Stops other listeners on same element
}
```

### Event Delegation
```javascript
// Instead of adding listeners to many elements
const listItems = document.querySelectorAll('li');
listItems.forEach(item => {
    item.addEventListener('click', handleItemClick);
});

// Use event delegation on parent
const list = document.querySelector('ul');
list.addEventListener('click', function(event) {
    // Check if clicked element is what we want
    if (event.target.tagName === 'LI') {
        handleItemClick(event);
    }
});

// More sophisticated delegation
function delegate(parent, selector, eventType, handler) {
    parent.addEventListener(eventType, function(event) {
        if (event.target.matches(selector)) {
            handler.call(event.target, event);
        }
    });
}

// Usage
delegate(document.body, '.button', 'click', function(event) {
    console.log('Button clicked:', this);
});
```

## Form Handling

### Form Elements
```javascript
const form = document.querySelector('#myForm');
const nameInput = document.querySelector('#name');
const emailInput = document.querySelector('#email');
const selectElement = document.querySelector('#country');
const checkboxes = document.querySelectorAll('input[type="checkbox"]');

// Get form data
function getFormData() {
    return {
        name: nameInput.value,
        email: emailInput.value,
        country: selectElement.value,
        interests: Array.from(checkboxes)
            .filter(cb => cb.checked)
            .map(cb => cb.value)
    };
}

// Set form data
function setFormData(data) {
    nameInput.value = data.name || '';
    emailInput.value = data.email || '';
    selectElement.value = data.country || '';
    
    checkboxes.forEach(cb => {
        cb.checked = data.interests?.includes(cb.value) || false;
    });
}

// Form validation
function validateForm() {
    const errors = [];
    
    if (!nameInput.value.trim()) {
        errors.push('Name is required');
        nameInput.classList.add('error');
    } else {
        nameInput.classList.remove('error');
    }
    
    if (!emailInput.value.includes('@')) {
        errors.push('Valid email is required');
        emailInput.classList.add('error');
    } else {
        emailInput.classList.remove('error');
    }
    
    return errors;
}

// Form submission
form.addEventListener('submit', function(event) {
    event.preventDefault();
    
    const errors = validateForm();
    if (errors.length > 0) {
        console.log('Validation errors:', errors);
        return;
    }
    
    const formData = getFormData();
    console.log('Form data:', formData);
    
    // Submit data
    submitFormData(formData);
});
```

### FormData API
```javascript
const form = document.querySelector('#myForm');

form.addEventListener('submit', function(event) {
    event.preventDefault();
    
    // Create FormData from form
    const formData = new FormData(form);
    
    // Add additional data
    formData.append('timestamp', Date.now());
    
    // Get values
    console.log(formData.get('name'));
    console.log(formData.getAll('interests')); // For multiple values
    
    // Iterate over entries
    for (const [key, value] of formData.entries()) {
        console.log(key, value);
    }
    
    // Convert to object
    const data = Object.fromEntries(formData.entries());
    
    // Send with fetch
    fetch('/submit', {
        method: 'POST',
        body: formData // Browser sets correct Content-Type
    });
});
```

## Performance Optimization

### Batch DOM Updates
```javascript
// Inefficient - multiple reflows
const container = document.querySelector('.container');
for (let i = 0; i < 1000; i++) {
    const div = document.createElement('div');
    div.textContent = `Item ${i}`;
    container.appendChild(div); // Reflow on each append
}

// Efficient - single reflow
const fragment = document.createDocumentFragment();
for (let i = 0; i < 1000; i++) {
    const div = document.createElement('div');
    div.textContent = `Item ${i}`;
    fragment.appendChild(div);
}
container.appendChild(fragment); // Single reflow

// Or build HTML string (fastest for simple content)
const html = Array.from({ length: 1000 }, (_, i) => 
    `<div>Item ${i}</div>`
).join('');
container.innerHTML = html;
```

### Minimize DOM Queries
```javascript
// Inefficient - multiple queries
for (let i = 0; i < 10; i++) {
    document.querySelector('.status').textContent = `Processing ${i}`;
}

// Efficient - query once
const statusElement = document.querySelector('.status');
for (let i = 0; i < 10; i++) {
    statusElement.textContent = `Processing ${i}`;
}

// Cache common elements
const DOM = {
    container: document.querySelector('.container'),
    buttons: document.querySelectorAll('.button'),
    form: document.querySelector('#mainForm'),
    status: document.querySelector('.status')
};
```

### Virtual DOM Concept
```javascript
// Simple virtual DOM implementation concept
class VirtualDOM {
    constructor() {
        this.vdom = null;
        this.dom = null;
    }
    
    createElement(tag, props = {}, children = []) {
        return {
            tag,
            props,
            children: children.flat()
        };
    }
    
    render(vdom, container) {
        if (this.dom) {
            this.update(this.vdom, vdom, this.dom);
        } else {
            this.dom = this.create(vdom);
            container.appendChild(this.dom);
        }
        this.vdom = vdom;
    }
    
    create(vdom) {
        if (typeof vdom === 'string') {
            return document.createTextNode(vdom);
        }
        
        const element = document.createElement(vdom.tag);
        
        // Set properties
        Object.keys(vdom.props).forEach(key => {
            element[key] = vdom.props[key];
        });
        
        // Add children
        vdom.children.forEach(child => {
            element.appendChild(this.create(child));
        });
        
        return element;
    }
    
    update(oldVdom, newVdom, dom) {
        // Simplified diffing algorithm
        // In real implementation, this would be much more complex
        if (JSON.stringify(oldVdom) !== JSON.stringify(newVdom)) {
            const newDom = this.create(newVdom);
            dom.parentNode.replaceChild(newDom, dom);
            this.dom = newDom;
        }
    }
}
```

---

## Related Topics
- [[16 - Events]]
- [[17 - Event Methods]]
- [[14 - Window Object]]
- [[32 - jQuery]]

---

*Next: [[16 - Events]]*
