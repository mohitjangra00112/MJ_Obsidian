# Local Storage

## Overview
Local Storage is a web storage API that allows websites to store data locally within a user's browser. Unlike cookies, Local Storage data is not sent to the server with HTTP requests, making it ideal for client-side data persistence.

## Storage Types Comparison

### Local Storage vs Session Storage vs Cookies
```javascript
// Storage capacity and persistence comparison
console.log('Local Storage:');
console.log('- Capacity: ~5-10MB per origin');
console.log('- Persistence: Until explicitly removed');
console.log('- Scope: Origin (protocol + domain + port)');
console.log('- Server Access: No (client-side only)');

console.log('\nSession Storage:');
console.log('- Capacity: ~5-10MB per origin');
console.log('- Persistence: Until tab/window closes');
console.log('- Scope: Tab/window specific');
console.log('- Server Access: No (client-side only)');

console.log('\nCookies:');
console.log('- Capacity: ~4KB per cookie');
console.log('- Persistence: Configurable expiration');
console.log('- Scope: Domain/path specific');
console.log('- Server Access: Yes (sent with requests)');
```

## Basic Local Storage Operations

### Storing Data
```javascript
// Store string data
localStorage.setItem('username', 'john_doe');
localStorage.setItem('theme', 'dark');

// Store using bracket notation
localStorage['language'] = 'english';

// Store using property notation
localStorage.userEmail = 'john@example.com';

// Store complex data (must be stringified)
const userPreferences = {
    theme: 'dark',
    notifications: true,
    language: 'en',
    fontSize: 16
};

localStorage.setItem('preferences', JSON.stringify(userPreferences));

// Store arrays
const favoriteColors = ['blue', 'green', 'red'];
localStorage.setItem('colors', JSON.stringify(favoriteColors));

// Store numbers (automatically converted to strings)
localStorage.setItem('userAge', 25);
localStorage.setItem('score', 1500);
```

### Retrieving Data
```javascript
// Get string data
const username = localStorage.getItem('username');
const theme = localStorage.getItem('theme');

console.log(username); // "john_doe"
console.log(theme); // "dark"

// Get using bracket notation
const language = localStorage['language'];

// Get using property notation
const email = localStorage.userEmail;

// Get complex data (must be parsed)
const preferencesString = localStorage.getItem('preferences');
const preferences = preferencesString ? JSON.parse(preferencesString) : {};

console.log(preferences.theme); // "dark"

// Get arrays
const colorsString = localStorage.getItem('colors');
const colors = colorsString ? JSON.parse(colorsString) : [];

console.log(colors[0]); // "blue"

// Get numbers (returned as strings)
const ageString = localStorage.getItem('userAge');
const age = parseInt(ageString, 10);

console.log(typeof ageString); // "string"
console.log(typeof age); // "number"
```

### Removing Data
```javascript
// Remove specific item
localStorage.removeItem('username');
localStorage.removeItem('theme');

// Remove using delete operator
delete localStorage.language;
delete localStorage.userEmail;

// Clear all local storage data
localStorage.clear();

// Check if item exists before removing
if (localStorage.getItem('preferences')) {
    localStorage.removeItem('preferences');
}
```

### Checking Data Existence
```javascript
// Check if key exists
function hasLocalStorageItem(key) {
    return localStorage.getItem(key) !== null;
}

// Usage
if (hasLocalStorageItem('username')) {
    console.log('Username is stored');
}

// Alternative method
function isItemStored(key) {
    return key in localStorage;
}

// Get all keys
function getAllLocalStorageKeys() {
    const keys = [];
    for (let i = 0; i < localStorage.length; i++) {
        keys.push(localStorage.key(i));
    }
    return keys;
}

console.log(getAllLocalStorageKeys());
```

## Advanced Local Storage Patterns

### Local Storage Wrapper Class
```javascript
class LocalStorageManager {
    constructor(prefix = '') {
        this.prefix = prefix;
    }
    
    // Generate prefixed key
    _getKey(key) {
        return this.prefix ? `${this.prefix}_${key}` : key;
    }
    
    // Set item with automatic JSON handling
    set(key, value) {
        try {
            const serializedValue = typeof value === 'string' 
                ? value 
                : JSON.stringify(value);
            localStorage.setItem(this._getKey(key), serializedValue);
            return true;
        } catch (error) {
            console.error('Failed to save to localStorage:', error);
            return false;
        }
    }
    
    // Get item with automatic JSON parsing
    get(key, defaultValue = null) {
        try {
            const item = localStorage.getItem(this._getKey(key));
            
            if (item === null) {
                return defaultValue;
            }
            
            // Try to parse as JSON, fallback to string
            try {
                return JSON.parse(item);
            } catch {
                return item;
            }
        } catch (error) {
            console.error('Failed to get from localStorage:', error);
            return defaultValue;
        }
    }
    
    // Remove item
    remove(key) {
        try {
            localStorage.removeItem(this._getKey(key));
            return true;
        } catch (error) {
            console.error('Failed to remove from localStorage:', error);
            return false;
        }
    }
    
    // Check if item exists
    has(key) {
        return localStorage.getItem(this._getKey(key)) !== null;
    }
    
    // Get all keys with this prefix
    getKeys() {
        const keys = [];
        const fullPrefix = this.prefix ? `${this.prefix}_` : '';
        
        for (let i = 0; i < localStorage.length; i++) {
            const key = localStorage.key(i);
            if (!fullPrefix || key.startsWith(fullPrefix)) {
                keys.push(fullPrefix ? key.substring(fullPrefix.length) : key);
            }
        }
        
        return keys;
    }
    
    // Get all items with this prefix
    getAll() {
        const items = {};
        this.getKeys().forEach(key => {
            items[key] = this.get(key);
        });
        return items;
    }
    
    // Clear all items with this prefix
    clear() {
        const keys = this.getKeys();
        keys.forEach(key => this.remove(key));
    }
    
    // Get storage size in bytes (approximate)
    getSize() {
        let size = 0;
        this.getKeys().forEach(key => {
            const value = localStorage.getItem(this._getKey(key));
            size += (key.length + (value?.length || 0)) * 2; // UTF-16 encoding
        });
        return size;
    }
}

// Usage
const userStorage = new LocalStorageManager('user');
const appStorage = new LocalStorageManager('app');

// Store user data
userStorage.set('profile', {
    name: 'John Doe',
    email: 'john@example.com',
    age: 30
});

userStorage.set('preferences', {
    theme: 'dark',
    notifications: true
});

// Store app data
appStorage.set('version', '1.2.0');
appStorage.set('lastLogin', new Date().toISOString());

// Retrieve data
const profile = userStorage.get('profile');
const theme = userStorage.get('preferences')?.theme;

console.log(profile.name); // "John Doe"
console.log(theme); // "dark"

// Get all user data
console.log(userStorage.getAll());
```

### Expiring Local Storage
```javascript
class ExpiringStorage {
    constructor(storage = localStorage) {
        this.storage = storage;
    }
    
    set(key, value, expirationMinutes = null) {
        const item = {
            value: value,
            timestamp: Date.now(),
            expiration: expirationMinutes 
                ? Date.now() + (expirationMinutes * 60 * 1000)
                : null
        };
        
        this.storage.setItem(key, JSON.stringify(item));
    }
    
    get(key) {
        const itemString = this.storage.getItem(key);
        
        if (!itemString) {
            return null;
        }
        
        try {
            const item = JSON.parse(itemString);
            
            // Check if item has expired
            if (item.expiration && Date.now() > item.expiration) {
                this.storage.removeItem(key);
                return null;
            }
            
            return item.value;
        } catch (error) {
            console.error('Failed to parse stored item:', error);
            this.storage.removeItem(key);
            return null;
        }
    }
    
    remove(key) {
        this.storage.removeItem(key);
    }
    
    // Clean up expired items
    cleanup() {
        const keysToRemove = [];
        
        for (let i = 0; i < this.storage.length; i++) {
            const key = this.storage.key(i);
            const itemString = this.storage.getItem(key);
            
            try {
                const item = JSON.parse(itemString);
                if (item.expiration && Date.now() > item.expiration) {
                    keysToRemove.push(key);
                }
            } catch {
                // Ignore invalid items
            }
        }
        
        keysToRemove.forEach(key => this.storage.removeItem(key));
        return keysToRemove.length;
    }
    
    // Get time until expiration
    getTimeToExpiration(key) {
        const itemString = this.storage.getItem(key);
        
        if (!itemString) {
            return null;
        }
        
        try {
            const item = JSON.parse(itemString);
            if (!item.expiration) {
                return Infinity; // Never expires
            }
            
            const timeLeft = item.expiration - Date.now();
            return timeLeft > 0 ? timeLeft : 0;
        } catch {
            return null;
        }
    }
}

// Usage
const expiringStorage = new ExpiringStorage();

// Store data that expires in 30 minutes
expiringStorage.set('sessionToken', 'abc123xyz', 30);

// Store data that expires in 1 day
expiringStorage.set('cachedData', { data: 'value' }, 24 * 60);

// Store data that never expires
expiringStorage.set('userSettings', { theme: 'dark' });

// Retrieve data
const token = expiringStorage.get('sessionToken');
const settings = expiringStorage.get('userSettings');

// Check expiration time
const timeLeft = expiringStorage.getTimeToExpiration('sessionToken');
console.log(`Token expires in ${timeLeft}ms`);

// Clean up expired items
const removedCount = expiringStorage.cleanup();
console.log(`Removed ${removedCount} expired items`);
```

### Observable Local Storage
```javascript
class ObservableStorage {
    constructor() {
        this.listeners = new Map();
        this.setupStorageListener();
    }
    
    setupStorageListener() {
        // Listen for storage changes from other tabs/windows
        window.addEventListener('storage', (event) => {
            if (event.storageArea === localStorage) {
                this.notifyListeners(event.key, event.newValue, event.oldValue);
            }
        });
    }
    
    set(key, value) {
        const oldValue = localStorage.getItem(key);
        const newValue = typeof value === 'string' ? value : JSON.stringify(value);
        
        localStorage.setItem(key, newValue);
        this.notifyListeners(key, newValue, oldValue);
    }
    
    get(key, defaultValue = null) {
        const value = localStorage.getItem(key);
        if (value === null) {
            return defaultValue;
        }
        
        try {
            return JSON.parse(value);
        } catch {
            return value;
        }
    }
    
    remove(key) {
        const oldValue = localStorage.getItem(key);
        localStorage.removeItem(key);
        this.notifyListeners(key, null, oldValue);
    }
    
    // Subscribe to changes
    subscribe(key, callback) {
        if (!this.listeners.has(key)) {
            this.listeners.set(key, new Set());
        }
        
        this.listeners.get(key).add(callback);
        
        // Return unsubscribe function
        return () => {
            const callbacks = this.listeners.get(key);
            if (callbacks) {
                callbacks.delete(callback);
                if (callbacks.size === 0) {
                    this.listeners.delete(key);
                }
            }
        };
    }
    
    // Subscribe to all changes
    subscribeAll(callback) {
        return this.subscribe('*', callback);
    }
    
    notifyListeners(key, newValue, oldValue) {
        // Notify specific key listeners
        const keyListeners = this.listeners.get(key);
        if (keyListeners) {
            keyListeners.forEach(callback => {
                try {
                    callback(newValue, oldValue, key);
                } catch (error) {
                    console.error('Storage listener error:', error);
                }
            });
        }
        
        // Notify global listeners
        const globalListeners = this.listeners.get('*');
        if (globalListeners) {
            globalListeners.forEach(callback => {
                try {
                    callback(newValue, oldValue, key);
                } catch (error) {
                    console.error('Storage listener error:', error);
                }
            });
        }
    }
}

// Usage
const observableStorage = new ObservableStorage();

// Subscribe to specific key changes
const unsubscribeUser = observableStorage.subscribe('user', (newValue, oldValue, key) => {
    console.log(`User data changed:`, { newValue, oldValue, key });
});

// Subscribe to all changes
const unsubscribeAll = observableStorage.subscribeAll((newValue, oldValue, key) => {
    console.log(`Storage changed:`, { key, newValue, oldValue });
});

// Update storage
observableStorage.set('user', { name: 'John', age: 30 });
observableStorage.set('theme', 'dark');

// Unsubscribe
unsubscribeUser();
unsubscribeAll();
```

## Error Handling and Validation

### Storage Quota Management
```javascript
class SafeLocalStorage {
    static isSupported() {
        try {
            const test = '__localStorage_test__';
            localStorage.setItem(test, 'test');
            localStorage.removeItem(test);
            return true;
        } catch {
            return false;
        }
    }
    
    static getQuotaInfo() {
        if (!navigator.storage || !navigator.storage.estimate) {
            return null;
        }
        
        return navigator.storage.estimate().then(estimate => ({
            quota: estimate.quota,
            usage: estimate.usage,
            usageBytesByOrigin: estimate.usageDetails
        }));
    }
    
    static getCurrentUsage() {
        let totalSize = 0;
        
        for (let key in localStorage) {
            if (localStorage.hasOwnProperty(key)) {
                totalSize += localStorage[key].length + key.length;
            }
        }
        
        return totalSize * 2; // UTF-16 encoding
    }
    
    static safeSet(key, value, maxRetries = 3) {
        for (let attempt = 1; attempt <= maxRetries; attempt++) {
            try {
                const serializedValue = typeof value === 'string' 
                    ? value 
                    : JSON.stringify(value);
                
                localStorage.setItem(key, serializedValue);
                return { success: true, attempt };
            } catch (error) {
                if (error.name === 'QuotaExceededError') {
                    if (attempt < maxRetries) {
                        // Try to free up space
                        const freedSpace = this.cleanupOldItems();
                        console.warn(`Storage quota exceeded. Freed ${freedSpace} bytes. Retry ${attempt}/${maxRetries}`);
                    } else {
                        return { 
                            success: false, 
                            error: 'Storage quota exceeded',
                            usage: this.getCurrentUsage()
                        };
                    }
                } else {
                    return { 
                        success: false, 
                        error: error.message 
                    };
                }
            }
        }
    }
    
    static cleanupOldItems(maxAge = 7 * 24 * 60 * 60 * 1000) { // 7 days
        let freedSpace = 0;
        const now = Date.now();
        const itemsToRemove = [];
        
        for (let key in localStorage) {
            if (!localStorage.hasOwnProperty(key)) continue;
            
            try {
                const value = localStorage[key];
                const parsed = JSON.parse(value);
                
                if (parsed.timestamp && (now - parsed.timestamp) > maxAge) {
                    itemsToRemove.push(key);
                    freedSpace += value.length + key.length;
                }
            } catch {
                // Not a timestamped item, skip
            }
        }
        
        itemsToRemove.forEach(key => localStorage.removeItem(key));
        return freedSpace * 2; // UTF-16 encoding
    }
    
    static safeGet(key, defaultValue = null) {
        try {
            const value = localStorage.getItem(key);
            return value !== null ? JSON.parse(value) : defaultValue;
        } catch (error) {
            console.error(`Failed to get item "${key}":`, error);
            return defaultValue;
        }
    }
}

// Usage
if (SafeLocalStorage.isSupported()) {
    const result = SafeLocalStorage.safeSet('largeData', { 
        data: new Array(1000).fill('sample data'),
        timestamp: Date.now()
    });
    
    if (result.success) {
        console.log(`Data saved on attempt ${result.attempt}`);
    } else {
        console.error('Failed to save data:', result.error);
    }
    
    // Get quota information (modern browsers)
    SafeLocalStorage.getQuotaInfo().then(info => {
        if (info) {
            console.log(`Storage quota: ${info.quota} bytes`);
            console.log(`Storage usage: ${info.usage} bytes`);
        }
    });
} else {
    console.error('Local Storage is not supported');
}
```

## Practical Applications

### User Preferences Management
```javascript
class UserPreferences {
    constructor() {
        this.storage = new LocalStorageManager('userPrefs');
        this.defaults = {
            theme: 'light',
            language: 'en',
            fontSize: 16,
            notifications: true,
            autoSave: true,
            privacy: {
                analytics: false,
                cookies: true
            }
        };
        
        this.loadPreferences();
    }
    
    loadPreferences() {
        this.preferences = {
            ...this.defaults,
            ...this.storage.get('preferences', {})
        };
    }
    
    get(key) {
        return this.getNestedValue(this.preferences, key);
    }
    
    set(key, value) {
        this.setNestedValue(this.preferences, key, value);
        this.savePreferences();
        this.notifyChange(key, value);
    }
    
    getNestedValue(obj, path) {
        return path.split('.').reduce((current, key) => current?.[key], obj);
    }
    
    setNestedValue(obj, path, value) {
        const keys = path.split('.');
        const lastKey = keys.pop();
        const target = keys.reduce((current, key) => {
            if (!current[key] || typeof current[key] !== 'object') {
                current[key] = {};
            }
            return current[key];
        }, obj);
        
        target[lastKey] = value;
    }
    
    savePreferences() {
        this.storage.set('preferences', this.preferences);
    }
    
    reset() {
        this.preferences = { ...this.defaults };
        this.savePreferences();
    }
    
    export() {
        return JSON.stringify(this.preferences, null, 2);
    }
    
    import(jsonString) {
        try {
            const imported = JSON.parse(jsonString);
            this.preferences = { ...this.defaults, ...imported };
            this.savePreferences();
            return true;
        } catch (error) {
            console.error('Failed to import preferences:', error);
            return false;
        }
    }
    
    notifyChange(key, value) {
        // Dispatch custom event for preference changes
        window.dispatchEvent(new CustomEvent('preferenceChange', {
            detail: { key, value, preferences: this.preferences }
        }));
    }
}

// Usage
const userPrefs = new UserPreferences();

// Listen for preference changes
window.addEventListener('preferenceChange', (event) => {
    console.log('Preference changed:', event.detail);
    
    // Apply theme change
    if (event.detail.key === 'theme') {
        document.body.className = `theme-${event.detail.value}`;
    }
});

// Get and set preferences
console.log(userPrefs.get('theme')); // 'light'
console.log(userPrefs.get('privacy.analytics')); // false

userPrefs.set('theme', 'dark');
userPrefs.set('privacy.analytics', true);

// Export/import preferences
const exported = userPrefs.export();
userPrefs.import(exported);
```

### Shopping Cart Storage
```javascript
class ShoppingCart {
    constructor() {
        this.storage = new LocalStorageManager('cart');
        this.items = this.storage.get('items', []);
    }
    
    addItem(product, quantity = 1) {
        const existingIndex = this.items.findIndex(item => item.id === product.id);
        
        if (existingIndex >= 0) {
            this.items[existingIndex].quantity += quantity;
        } else {
            this.items.push({
                id: product.id,
                name: product.name,
                price: product.price,
                quantity: quantity,
                addedAt: new Date().toISOString()
            });
        }
        
        this.save();
    }
    
    removeItem(productId) {
        this.items = this.items.filter(item => item.id !== productId);
        this.save();
    }
    
    updateQuantity(productId, quantity) {
        const item = this.items.find(item => item.id === productId);
        if (item) {
            if (quantity <= 0) {
                this.removeItem(productId);
            } else {
                item.quantity = quantity;
                this.save();
            }
        }
    }
    
    getItems() {
        return [...this.items];
    }
    
    getTotal() {
        return this.items.reduce((total, item) => {
            return total + (item.price * item.quantity);
        }, 0);
    }
    
    getItemCount() {
        return this.items.reduce((count, item) => count + item.quantity, 0);
    }
    
    clear() {
        this.items = [];
        this.save();
    }
    
    save() {
        this.storage.set('items', this.items);
        this.storage.set('lastUpdated', new Date().toISOString());
        
        // Dispatch cart update event
        window.dispatchEvent(new CustomEvent('cartUpdate', {
            detail: {
                items: this.getItems(),
                total: this.getTotal(),
                count: this.getItemCount()
            }
        }));
    }
    
    // Merge cart from another session (e.g., after login)
    merge(otherCartItems) {
        otherCartItems.forEach(otherItem => {
            const existingItem = this.items.find(item => item.id === otherItem.id);
            if (existingItem) {
                existingItem.quantity = Math.max(existingItem.quantity, otherItem.quantity);
            } else {
                this.items.push({ ...otherItem });
            }
        });
        this.save();
    }
}

// Usage
const cart = new ShoppingCart();

// Add items
cart.addItem({ id: 1, name: 'T-Shirt', price: 19.99 }, 2);
cart.addItem({ id: 2, name: 'Jeans', price: 49.99 }, 1);

// Listen for cart updates
window.addEventListener('cartUpdate', (event) => {
    const { items, total, count } = event.detail;
    console.log(`Cart: ${count} items, Total: $${total.toFixed(2)}`);
    
    // Update UI
    document.getElementById('cart-count').textContent = count;
    document.getElementById('cart-total').textContent = `$${total.toFixed(2)}`;
});

// Get cart information
console.log(cart.getItems());
console.log(`Total: $${cart.getTotal().toFixed(2)}`);
```

---

## Related Topics
- [[25 - Session Storage]]
- [[26 - Cookies]]
- [[27 - JSON]]
- [[43 - Browser APIs]]

---

*Next: [[25 - Session Storage]]*
