# Session Storage

## Overview
Session Storage is a web storage API that provides temporary storage for data within a browser tab or window session. Unlike Local Storage, data stored in Session Storage is automatically cleared when the tab or window is closed.

## Session Storage vs Local Storage

### Key Differences
```javascript
// Session Storage characteristics
console.log('Session Storage:');
console.log('- Lifetime: Tab/window session only');
console.log('- Scope: Per tab/window');
console.log('- Capacity: ~5-10MB per origin');
console.log('- Persistence: Until tab closes');
console.log('- Cross-tab sharing: No');

// Local Storage characteristics
console.log('\nLocal Storage:');
console.log('- Lifetime: Until explicitly removed');
console.log('- Scope: Per origin (all tabs/windows)');
console.log('- Capacity: ~5-10MB per origin');
console.log('- Persistence: Permanent (until cleared)');
console.log('- Cross-tab sharing: Yes');

// Practical demonstration
// Tab 1
sessionStorage.setItem('tabData', 'This is tab 1');
localStorage.setItem('sharedData', 'Available to all tabs');

// Tab 2 (new tab)
console.log(sessionStorage.getItem('tabData')); // null (not shared)
console.log(localStorage.getItem('sharedData')); // "Available to all tabs"
```

## Basic Session Storage Operations

### Storing Data
```javascript
// Store simple strings
sessionStorage.setItem('currentUser', 'john_doe');
sessionStorage.setItem('sessionToken', 'abc123xyz789');

// Store using bracket notation
sessionStorage['currentPage'] = 'dashboard';

// Store using property notation
sessionStorage.userRole = 'admin';

// Store complex data (requires JSON stringification)
const formData = {
    firstName: 'John',
    lastName: 'Doe',
    email: 'john@example.com',
    preferences: {
        newsletter: true,
        notifications: false
    }
};

sessionStorage.setItem('formData', JSON.stringify(formData));

// Store arrays
const visitedPages = ['home', 'about', 'products', 'contact'];
sessionStorage.setItem('visitedPages', JSON.stringify(visitedPages));

// Store timestamps
sessionStorage.setItem('sessionStart', new Date().toISOString());
sessionStorage.setItem('lastActivity', Date.now().toString());
```

### Retrieving Data
```javascript
// Get simple strings
const currentUser = sessionStorage.getItem('currentUser');
const sessionToken = sessionStorage.getItem('sessionToken');

console.log(currentUser); // "john_doe"
console.log(sessionToken); // "abc123xyz789"

// Get using bracket notation
const currentPage = sessionStorage['currentPage'];

// Get using property notation
const userRole = sessionStorage.userRole;

// Get complex data (requires JSON parsing)
const formDataString = sessionStorage.getItem('formData');
const formData = formDataString ? JSON.parse(formDataString) : {};

console.log(formData.firstName); // "John"
console.log(formData.preferences.newsletter); // true

// Get arrays
const visitedPagesString = sessionStorage.getItem('visitedPages');
const visitedPages = visitedPagesString ? JSON.parse(visitedPagesString) : [];

console.log(visitedPages.length); // 4

// Get and convert timestamps
const sessionStartString = sessionStorage.getItem('sessionStart');
const sessionStart = sessionStartString ? new Date(sessionStartString) : new Date();

const lastActivityString = sessionStorage.getItem('lastActivity');
const lastActivity = lastActivityString ? parseInt(lastActivityString, 10) : Date.now();
```

### Removing Data
```javascript
// Remove specific items
sessionStorage.removeItem('currentUser');
sessionStorage.removeItem('sessionToken');

// Remove using delete operator
delete sessionStorage.currentPage;
delete sessionStorage.userRole;

// Clear all session storage
sessionStorage.clear();

// Conditional removal
if (sessionStorage.getItem('formData')) {
    sessionStorage.removeItem('formData');
}

// Remove expired items
function removeExpiredItems() {
    const now = Date.now();
    const maxAge = 30 * 60 * 1000; // 30 minutes
    
    for (let i = sessionStorage.length - 1; i >= 0; i--) {
        const key = sessionStorage.key(i);
        const item = sessionStorage.getItem(key);
        
        try {
            const parsed = JSON.parse(item);
            if (parsed.timestamp && (now - parsed.timestamp) > maxAge) {
                sessionStorage.removeItem(key);
            }
        } catch {
            // Not a timestamped item, skip
        }
    }
}
```

### Checking Data Existence and Properties
```javascript
// Check if item exists
function hasSessionItem(key) {
    return sessionStorage.getItem(key) !== null;
}

// Check using 'in' operator
function isKeyInSession(key) {
    return key in sessionStorage;
}

// Get session storage size
function getSessionStorageSize() {
    let total = 0;
    for (let key in sessionStorage) {
        if (sessionStorage.hasOwnProperty(key)) {
            total += sessionStorage[key].length + key.length;
        }
    }
    return total * 2; // UTF-16 encoding
}

// Get all keys
function getAllSessionKeys() {
    const keys = [];
    for (let i = 0; i < sessionStorage.length; i++) {
        keys.push(sessionStorage.key(i));
    }
    return keys;
}

// Get all items
function getAllSessionItems() {
    const items = {};
    for (let i = 0; i < sessionStorage.length; i++) {
        const key = sessionStorage.key(i);
        items[key] = sessionStorage.getItem(key);
    }
    return items;
}

// Usage examples
console.log(`Session has ${sessionStorage.length} items`);
console.log(`Total size: ${getSessionStorageSize()} bytes`);
console.log('All keys:', getAllSessionKeys());
```

## Advanced Session Storage Patterns

### Session Storage Manager Class
```javascript
class SessionStorageManager {
    constructor(prefix = '') {
        this.prefix = prefix;
    }
    
    _getKey(key) {
        return this.prefix ? `${this.prefix}_${key}` : key;
    }
    
    set(key, value, options = {}) {
        try {
            const item = {
                value: value,
                timestamp: Date.now(),
                ...options
            };
            
            const serialized = JSON.stringify(item);
            sessionStorage.setItem(this._getKey(key), serialized);
            return true;
        } catch (error) {
            console.error('Failed to save to sessionStorage:', error);
            return false;
        }
    }
    
    get(key, defaultValue = null) {
        try {
            const item = sessionStorage.getItem(this._getKey(key));
            
            if (item === null) {
                return defaultValue;
            }
            
            const parsed = JSON.parse(item);
            
            // Check if item has custom expiration
            if (parsed.expiresAt && Date.now() > parsed.expiresAt) {
                this.remove(key);
                return defaultValue;
            }
            
            return parsed.value;
        } catch (error) {
            console.error('Failed to get from sessionStorage:', error);
            return defaultValue;
        }
    }
    
    setWithExpiration(key, value, expirationMinutes) {
        const expiresAt = Date.now() + (expirationMinutes * 60 * 1000);
        return this.set(key, value, { expiresAt });
    }
    
    remove(key) {
        sessionStorage.removeItem(this._getKey(key));
    }
    
    has(key) {
        return this.get(key) !== null;
    }
    
    getMetadata(key) {
        try {
            const item = sessionStorage.getItem(this._getKey(key));
            if (item) {
                const parsed = JSON.parse(item);
                return {
                    timestamp: parsed.timestamp,
                    expiresAt: parsed.expiresAt,
                    age: Date.now() - parsed.timestamp
                };
            }
        } catch (error) {
            console.error('Failed to get metadata:', error);
        }
        return null;
    }
    
    getKeys() {
        const keys = [];
        const fullPrefix = this.prefix ? `${this.prefix}_` : '';
        
        for (let i = 0; i < sessionStorage.length; i++) {
            const key = sessionStorage.key(i);
            if (!fullPrefix || key.startsWith(fullPrefix)) {
                keys.push(fullPrefix ? key.substring(fullPrefix.length) : key);
            }
        }
        
        return keys;
    }
    
    getAll() {
        const items = {};
        this.getKeys().forEach(key => {
            items[key] = this.get(key);
        });
        return items;
    }
    
    clear() {
        const keys = this.getKeys();
        keys.forEach(key => this.remove(key));
    }
    
    cleanup() {
        const expired = [];
        const keys = this.getKeys();
        
        keys.forEach(key => {
            if (this.get(key) === null) { // Will auto-remove expired items
                expired.push(key);
            }
        });
        
        return expired;
    }
}

// Usage
const userSession = new SessionStorageManager('user');
const appSession = new SessionStorageManager('app');

// Store user session data
userSession.set('currentUser', {
    id: 123,
    name: 'John Doe',
    role: 'admin'
});

// Store with expiration (30 minutes)
userSession.setWithExpiration('tempToken', 'xyz789', 30);

// Store app session data
appSession.set('currentView', 'dashboard');
appSession.set('sidebarCollapsed', false);

// Retrieve data
const currentUser = userSession.get('currentUser');
const tempToken = userSession.get('tempToken');

console.log(currentUser.name); // "John Doe"
console.log(tempToken); // "xyz789" or null if expired

// Get metadata
const tokenMeta = userSession.getMetadata('tempToken');
if (tokenMeta) {
    console.log(`Token age: ${tokenMeta.age}ms`);
}
```

### Form Data Session Management
```javascript
class FormSessionManager {
    constructor(formId) {
        this.formId = formId;
        this.storageKey = `form_${formId}`;
        this.autoSaveInterval = null;
        this.setupAutoSave();
    }
    
    saveFormData(formElement) {
        const formData = this.extractFormData(formElement);
        
        const sessionData = {
            data: formData,
            timestamp: Date.now(),
            url: window.location.href
        };
        
        sessionStorage.setItem(this.storageKey, JSON.stringify(sessionData));
    }
    
    extractFormData(formElement) {
        const formData = {};
        const elements = formElement.elements;
        
        for (let element of elements) {
            if (element.name) {
                if (element.type === 'checkbox') {
                    formData[element.name] = element.checked;
                } else if (element.type === 'radio') {
                    if (element.checked) {
                        formData[element.name] = element.value;
                    }
                } else if (element.tagName === 'SELECT' && element.multiple) {
                    formData[element.name] = Array.from(element.selectedOptions)
                        .map(option => option.value);
                } else {
                    formData[element.name] = element.value;
                }
            }
        }
        
        return formData;
    }
    
    restoreFormData(formElement) {
        const sessionDataString = sessionStorage.getItem(this.storageKey);
        
        if (!sessionDataString) {
            return false;
        }
        
        try {
            const sessionData = JSON.parse(sessionDataString);
            const formData = sessionData.data;
            
            // Only restore if from same URL (optional security check)
            if (sessionData.url !== window.location.href) {
                return false;
            }
            
            this.populateForm(formElement, formData);
            return true;
        } catch (error) {
            console.error('Failed to restore form data:', error);
            return false;
        }
    }
    
    populateForm(formElement, formData) {
        for (const [name, value] of Object.entries(formData)) {
            const elements = formElement.elements[name];
            
            if (!elements) continue;
            
            if (elements.length) {
                // Multiple elements with same name (radio buttons)
                for (let element of elements) {
                    if (element.type === 'radio') {
                        element.checked = element.value === value;
                    }
                }
            } else {
                // Single element
                const element = elements;
                
                if (element.type === 'checkbox') {
                    element.checked = Boolean(value);
                } else if (element.tagName === 'SELECT' && element.multiple) {
                    for (let option of element.options) {
                        option.selected = value.includes(option.value);
                    }
                } else {
                    element.value = value;
                }
            }
        }
    }
    
    setupAutoSave(intervalMs = 5000) {
        this.stopAutoSave();
        
        const formElement = document.getElementById(this.formId);
        if (!formElement) {
            console.warn(`Form with ID "${this.formId}" not found`);
            return;
        }
        
        this.autoSaveInterval = setInterval(() => {
            this.saveFormData(formElement);
        }, intervalMs);
        
        // Save on form input changes
        formElement.addEventListener('input', () => {
            this.saveFormData(formElement);
        });
        
        // Save on form submission and remove saved data
        formElement.addEventListener('submit', () => {
            this.clearSavedData();
        });
        
        // Restore data on page load
        this.restoreFormData(formElement);
    }
    
    stopAutoSave() {
        if (this.autoSaveInterval) {
            clearInterval(this.autoSaveInterval);
            this.autoSaveInterval = null;
        }
    }
    
    clearSavedData() {
        sessionStorage.removeItem(this.storageKey);
    }
    
    hasSavedData() {
        return sessionStorage.getItem(this.storageKey) !== null;
    }
    
    getSavedDataAge() {
        const sessionDataString = sessionStorage.getItem(this.storageKey);
        
        if (!sessionDataString) {
            return null;
        }
        
        try {
            const sessionData = JSON.parse(sessionDataString);
            return Date.now() - sessionData.timestamp;
        } catch {
            return null;
        }
    }
}

// Usage
// HTML: <form id="contactForm">...</form>
const formManager = new FormSessionManager('contactForm');

// Check if there's saved data
if (formManager.hasSavedData()) {
    const age = formManager.getSavedDataAge();
    const minutes = Math.floor(age / (1000 * 60));
    
    if (confirm(`Restore form data from ${minutes} minutes ago?`)) {
        // Data will be restored automatically
    } else {
        formManager.clearSavedData();
    }
}
```

### Navigation State Management
```javascript
class NavigationStateManager {
    constructor() {
        this.storageKey = 'nav_state';
        this.setupNavigationTracking();
    }
    
    saveState(state) {
        const navigationState = {
            ...state,
            timestamp: Date.now(),
            url: window.location.href,
            userAgent: navigator.userAgent
        };
        
        sessionStorage.setItem(this.storageKey, JSON.stringify(navigationState));
    }
    
    getState() {
        const stateString = sessionStorage.getItem(this.storageKey);
        
        if (!stateString) {
            return null;
        }
        
        try {
            return JSON.parse(stateString);
        } catch (error) {
            console.error('Failed to parse navigation state:', error);
            return null;
        }
    }
    
    setupNavigationTracking() {
        // Track page visits
        this.trackPageVisit();
        
        // Track scroll position
        this.trackScrollPosition();
        
        // Track tab visibility
        this.trackTabVisibility();
        
        // Save state before page unload
        window.addEventListener('beforeunload', () => {
            this.saveCurrentState();
        });
    }
    
    trackPageVisit() {
        const currentState = this.getState() || { visitedPages: [] };
        
        const currentPage = {
            url: window.location.href,
            title: document.title,
            timestamp: Date.now()
        };
        
        currentState.visitedPages = currentState.visitedPages || [];
        currentState.visitedPages.push(currentPage);
        
        // Keep only last 10 pages
        if (currentState.visitedPages.length > 10) {
            currentState.visitedPages = currentState.visitedPages.slice(-10);
        }
        
        this.saveState(currentState);
    }
    
    trackScrollPosition() {
        let scrollTimeout;
        
        window.addEventListener('scroll', () => {
            clearTimeout(scrollTimeout);
            scrollTimeout = setTimeout(() => {
                const currentState = this.getState() || {};
                currentState.scrollPosition = {
                    x: window.scrollX,
                    y: window.scrollY,
                    timestamp: Date.now()
                };
                this.saveState(currentState);
            }, 250);
        });
    }
    
    trackTabVisibility() {
        document.addEventListener('visibilitychange', () => {
            const currentState = this.getState() || {};
            currentState.visibility = {
                hidden: document.hidden,
                timestamp: Date.now()
            };
            this.saveState(currentState);
        });
    }
    
    saveCurrentState() {
        const currentState = this.getState() || {};
        
        currentState.currentSession = {
            duration: Date.now() - (currentState.sessionStart || Date.now()),
            endTime: Date.now(),
            finalUrl: window.location.href,
            finalTitle: document.title,
            finalScrollPosition: {
                x: window.scrollX,
                y: window.scrollY
            }
        };
        
        this.saveState(currentState);
    }
    
    getVisitedPages() {
        const state = this.getState();
        return state?.visitedPages || [];
    }
    
    getLastScrollPosition() {
        const state = this.getState();
        return state?.scrollPosition || { x: 0, y: 0 };
    }
    
    restoreScrollPosition() {
        const scrollPos = this.getLastScrollPosition();
        if (scrollPos.x !== undefined && scrollPos.y !== undefined) {
            window.scrollTo(scrollPos.x, scrollPos.y);
        }
    }
    
    getSessionDuration() {
        const state = this.getState();
        if (state?.sessionStart) {
            return Date.now() - state.sessionStart;
        }
        return 0;
    }
    
    clearState() {
        sessionStorage.removeItem(this.storageKey);
    }
}

// Usage
const navManager = new NavigationStateManager();

// Get navigation information
console.log('Visited pages:', navManager.getVisitedPages());
console.log('Session duration:', navManager.getSessionDuration(), 'ms');

// Restore scroll position (useful for back button navigation)
window.addEventListener('load', () => {
    setTimeout(() => {
        navManager.restoreScrollPosition();
    }, 100);
});
```

### Session-based Shopping Cart
```javascript
class SessionShoppingCart {
    constructor() {
        this.storageKey = 'session_cart';
        this.setupEventListeners();
    }
    
    addItem(product, quantity = 1) {
        const cart = this.getCart();
        const existingIndex = cart.items.findIndex(item => item.id === product.id);
        
        if (existingIndex >= 0) {
            cart.items[existingIndex].quantity += quantity;
        } else {
            cart.items.push({
                id: product.id,
                name: product.name,
                price: product.price,
                quantity: quantity,
                addedAt: Date.now()
            });
        }
        
        cart.lastModified = Date.now();
        this.saveCart(cart);
    }
    
    removeItem(productId) {
        const cart = this.getCart();
        cart.items = cart.items.filter(item => item.id !== productId);
        cart.lastModified = Date.now();
        this.saveCart(cart);
    }
    
    updateQuantity(productId, quantity) {
        const cart = this.getCart();
        const item = cart.items.find(item => item.id === productId);
        
        if (item) {
            if (quantity <= 0) {
                this.removeItem(productId);
            } else {
                item.quantity = quantity;
                cart.lastModified = Date.now();
                this.saveCart(cart);
            }
        }
    }
    
    getCart() {
        const cartString = sessionStorage.getItem(this.storageKey);
        
        if (!cartString) {
            return {
                items: [],
                createdAt: Date.now(),
                lastModified: Date.now()
            };
        }
        
        try {
            return JSON.parse(cartString);
        } catch (error) {
            console.error('Failed to parse cart data:', error);
            return {
                items: [],
                createdAt: Date.now(),
                lastModified: Date.now()
            };
        }
    }
    
    saveCart(cart) {
        sessionStorage.setItem(this.storageKey, JSON.stringify(cart));
        this.notifyCartUpdate(cart);
    }
    
    getItems() {
        return this.getCart().items;
    }
    
    getTotal() {
        const items = this.getItems();
        return items.reduce((total, item) => total + (item.price * item.quantity), 0);
    }
    
    getItemCount() {
        const items = this.getItems();
        return items.reduce((count, item) => count + item.quantity, 0);
    }
    
    clear() {
        sessionStorage.removeItem(this.storageKey);
        this.notifyCartUpdate({ items: [], createdAt: Date.now(), lastModified: Date.now() });
    }
    
    isEmpty() {
        return this.getItems().length === 0;
    }
    
    getCartAge() {
        const cart = this.getCart();
        return Date.now() - cart.createdAt;
    }
    
    setupEventListeners() {
        // Clear cart when tab is closed (optional)
        window.addEventListener('beforeunload', () => {
            // You might want to save to server here instead of clearing
            // this.clear();
        });
        
        // Handle storage events (though sessionStorage doesn't trigger these)
        window.addEventListener('storage', (event) => {
            if (event.storageArea === sessionStorage && event.key === this.storageKey) {
                this.notifyCartUpdate(this.getCart());
            }
        });
    }
    
    notifyCartUpdate(cart) {
        window.dispatchEvent(new CustomEvent('sessionCartUpdate', {
            detail: {
                items: cart.items,
                total: this.getTotal(),
                count: this.getItemCount(),
                age: Date.now() - cart.createdAt
            }
        }));
    }
    
    // Export cart for transfer to persistent storage
    exportCart() {
        return JSON.stringify(this.getCart());
    }
    
    // Import cart from another source
    importCart(cartData) {
        try {
            const cart = typeof cartData === 'string' ? JSON.parse(cartData) : cartData;
            cart.lastModified = Date.now();
            this.saveCart(cart);
            return true;
        } catch (error) {
            console.error('Failed to import cart:', error);
            return false;
        }
    }
}

// Usage
const sessionCart = new SessionShoppingCart();

// Listen for cart updates
window.addEventListener('sessionCartUpdate', (event) => {
    const { items, total, count, age } = event.detail;
    console.log(`Session Cart: ${count} items, $${total.toFixed(2)}`);
    console.log(`Cart age: ${Math.floor(age / 1000)} seconds`);
    
    // Update UI elements
    updateCartDisplay(count, total);
});

// Add items to cart
sessionCart.addItem({ id: 1, name: 'T-Shirt', price: 19.99 }, 2);
sessionCart.addItem({ id: 2, name: 'Jeans', price: 49.99 }, 1);

// Get cart information
console.log('Cart items:', sessionCart.getItems());
console.log('Total:', sessionCart.getTotal());
console.log('Item count:', sessionCart.getItemCount());

function updateCartDisplay(count, total) {
    const cartIcon = document.querySelector('.cart-icon');
    if (cartIcon) {
        cartIcon.dataset.count = count;
        cartIcon.title = `${count} items - $${total.toFixed(2)}`;
    }
}
```

---

## Related Topics
- [[24 - Local Storage]]
- [[26 - Cookies]]
- [[27 - JSON]]
- [[43 - Browser APIs]]

---

*Next: [[26 - Cookies]]*
