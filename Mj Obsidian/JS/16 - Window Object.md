# Window Object

## Overview
The Window object represents the browser window and serves as the global object in browser environments. It provides access to the browser's functionality, including navigation, timing, storage, and communication between frames. Understanding the Window object is crucial for web development.

## Global Window Object

### Understanding the Global Context
```javascript
// In browser environment, 'window' is the global object
console.log(window === this); // true (in global scope)
console.log(typeof window); // "object"

// Global variables are properties of window
var globalVar = 'I am global';
console.log(window.globalVar); // "I am global"

// Functions declared globally become window methods
function globalFunction() {
    return 'Global function called';
}
console.log(window.globalFunction()); // "Global function called"

// let and const don't create window properties
let blockScoped = 'Block scoped';
const constant = 'Constant value';
console.log(window.blockScoped); // undefined
console.log(window.constant); // undefined

// Check if running in browser
function isBrowser() {
    return typeof window !== 'undefined' && typeof window.document !== 'undefined';
}

// Check if running in web worker
function isWebWorker() {
    return typeof self !== 'undefined' && typeof window === 'undefined';
}

// Check if running in Node.js
function isNode() {
    return typeof global !== 'undefined' && typeof window === 'undefined';
}

console.log('Is Browser:', isBrowser());
console.log('Is Web Worker:', isWebWorker());
console.log('Is Node.js:', isNode());

// Access window properties safely
function safeWindowAccess(property) {
    return typeof window !== 'undefined' ? window[property] : undefined;
}

console.log(safeWindowAccess('navigator'));
console.log(safeWindowAccess('location'));
```

### Window Properties and Methods
```javascript
// Basic window information
console.log('Window dimensions:');
console.log('Inner width:', window.innerWidth);
console.log('Inner height:', window.innerHeight);
console.log('Outer width:', window.outerWidth);
console.log('Outer height:', window.outerHeight);

// Screen information
console.log('Screen information:');
console.log('Screen width:', window.screen.width);
console.log('Screen height:', window.screen.height);
console.log('Available width:', window.screen.availWidth);
console.log('Available height:', window.screen.availHeight);
console.log('Color depth:', window.screen.colorDepth);
console.log('Pixel depth:', window.screen.pixelDepth);

// Window position
console.log('Window position:');
console.log('Screen X:', window.screenX);
console.log('Screen Y:', window.screenY);
console.log('Screen Left:', window.screenLeft);
console.log('Screen Top:', window.screenTop);

// Device pixel ratio (for high-DPI displays)
console.log('Device pixel ratio:', window.devicePixelRatio);

// Window state
console.log('Window state:');
console.log('Is closed:', window.closed);
console.log('Name:', window.name);
console.log('Status:', window.status);

// Browser information
console.log('Browser information:');
console.log('User agent:', window.navigator.userAgent);
console.log('Platform:', window.navigator.platform);
console.log('Language:', window.navigator.language);
console.log('Languages:', window.navigator.languages);
console.log('Online:', window.navigator.onLine);
console.log('Cookie enabled:', window.navigator.cookieEnabled);

// Window methods
function demonstrateWindowMethods() {
    // Alert, confirm, prompt
    // window.alert('This is an alert');
    // const confirmed = window.confirm('Are you sure?');
    // const userInput = window.prompt('Enter your name:', 'Default name');
    
    console.log('Window methods available:');
    console.log('- alert()');
    console.log('- confirm()');
    console.log('- prompt()');
    console.log('- open()');
    console.log('- close()');
    console.log('- focus()');
    console.log('- blur()');
    console.log('- moveBy()');
    console.log('- moveTo()');
    console.log('- resizeBy()');
    console.log('- resizeTo()');
}

demonstrateWindowMethods();

// Feature detection
function detectFeatures() {
    const features = {
        localStorage: 'localStorage' in window,
        sessionStorage: 'sessionStorage' in window,
        indexedDB: 'indexedDB' in window,
        webGL: !!window.WebGLRenderingContext,
        webWorkers: 'Worker' in window,
        serviceWorker: 'serviceWorker' in navigator,
        geolocation: 'geolocation' in navigator,
        notifications: 'Notification' in window,
        fetch: 'fetch' in window,
        websocket: 'WebSocket' in window,
        fileAPI: 'File' in window && 'FileReader' in window,
        dragAndDrop: 'draggable' in document.createElement('div'),
        canvas: !!document.createElement('canvas').getContext,
        touch: 'ontouchstart' in window,
        deviceMotion: 'DeviceMotionEvent' in window,
        deviceOrientation: 'DeviceOrientationEvent' in window
    };
    
    console.log('Feature detection:', features);
    return features;
}

detectFeatures();
```

## Window Navigation and Location

### Location Object
```javascript
// Current page location information
console.log('Location information:');
console.log('Full URL:', window.location.href);
console.log('Protocol:', window.location.protocol);
console.log('Host:', window.location.host);
console.log('Hostname:', window.location.hostname);
console.log('Port:', window.location.port);
console.log('Pathname:', window.location.pathname);
console.log('Search:', window.location.search);
console.log('Hash:', window.location.hash);
console.log('Origin:', window.location.origin);

// Parse URL parameters
function parseUrlParams(url = window.location.search) {
    const params = new URLSearchParams(url);
    const result = {};
    
    for (const [key, value] of params.entries()) {
        result[key] = value;
    }
    
    return result;
}

// Example URL: https://example.com/page?name=John&age=30&active=true
const urlParams = parseUrlParams('?name=John&age=30&active=true');
console.log('URL Parameters:', urlParams);

// Advanced URL manipulation
class URLManager {
    constructor(url = window.location.href) {
        this.url = new URL(url);
    }
    
    // Get parameter value
    getParam(key) {
        return this.url.searchParams.get(key);
    }
    
    // Set parameter value
    setParam(key, value) {
        this.url.searchParams.set(key, value);
        return this;
    }
    
    // Remove parameter
    removeParam(key) {
        this.url.searchParams.delete(key);
        return this;
    }
    
    // Get all parameters as object
    getAllParams() {
        const params = {};
        for (const [key, value] of this.url.searchParams.entries()) {
            params[key] = value;
        }
        return params;
    }
    
    // Update hash
    setHash(hash) {
        this.url.hash = hash.startsWith('#') ? hash : `#${hash}`;
        return this;
    }
    
    // Build final URL
    toString() {
        return this.url.toString();
    }
    
    // Navigate to the URL
    navigate(replace = false) {
        if (replace) {
            window.location.replace(this.toString());
        } else {
            window.location.href = this.toString();
        }
    }
    
    // Update current URL without navigation
    updateCurrentUrl(replace = false) {
        if (typeof window.history !== 'undefined') {
            if (replace) {
                window.history.replaceState(null, '', this.toString());
            } else {
                window.history.pushState(null, '', this.toString());
            }
        }
    }
}

// Usage
const urlManager = new URLManager('https://example.com/page?existing=param');
urlManager
    .setParam('newParam', 'value')
    .setParam('another', 'test')
    .setHash('section1');

console.log('Modified URL:', urlManager.toString());
console.log('All params:', urlManager.getAllParams());

// Navigation methods
function demonstrateNavigation() {
    console.log('Navigation methods:');
    
    // Different ways to navigate
    const navigationMethods = {
        // Immediate navigation (can be blocked by beforeunload)
        navigate: (url) => window.location.href = url,
        
        // Replace current page in history
        replace: (url) => window.location.replace(url),
        
        // Assign new location
        assign: (url) => window.location.assign(url),
        
        // Reload current page
        reload: (forceReload = false) => window.location.reload(forceReload),
        
        // Navigation using history API
        pushState: (url, title = '', state = null) => {
            if (window.history.pushState) {
                window.history.pushState(state, title, url);
            }
        },
        
        // Replace state without navigation
        replaceState: (url, title = '', state = null) => {
            if (window.history.replaceState) {
                window.history.replaceState(state, title, url);
            }
        },
        
        // Go back in history
        back: () => window.history.back(),
        
        // Go forward in history
        forward: () => window.history.forward(),
        
        // Go to specific position in history
        go: (steps) => window.history.go(steps)
    };
    
    return navigationMethods;
}

const nav = demonstrateNavigation();

// Hash change detection
function setupHashNavigation() {
    const routes = {
        home: () => console.log('Home page loaded'),
        about: () => console.log('About page loaded'),
        contact: () => console.log('Contact page loaded'),
        default: () => console.log('Default page loaded')
    };
    
    function handleHashChange() {
        const hash = window.location.hash.slice(1) || 'home';
        const route = routes[hash] || routes.default;
        route();
    }
    
    // Listen for hash changes
    window.addEventListener('hashchange', handleHashChange);
    
    // Handle initial load
    handleHashChange();
    
    return {
        navigate: (route) => {
            window.location.hash = route;
        },
        destroy: () => {
            window.removeEventListener('hashchange', handleHashChange);
        }
    };
}

// const hashRouter = setupHashNavigation();
```

### History API
```javascript
// Modern browser history management
class HistoryManager {
    constructor() {
        this.listeners = [];
        this.setupPopstateListener();
    }
    
    // Add state to history
    pushState(state, title, url) {
        if (window.history.pushState) {
            window.history.pushState(state, title, url);
            this.notifyListeners('push', state, url);
        }
    }
    
    // Replace current state
    replaceState(state, title, url) {
        if (window.history.replaceState) {
            window.history.replaceState(state, title, url);
            this.notifyListeners('replace', state, url);
        }
    }
    
    // Navigate back
    back() {
        window.history.back();
    }
    
    // Navigate forward
    forward() {
        window.history.forward();
    }
    
    // Go to specific history entry
    go(steps) {
        window.history.go(steps);
    }
    
    // Get current state
    getCurrentState() {
        return window.history.state;
    }
    
    // Get history length
    getLength() {
        return window.history.length;
    }
    
    // Add navigation listener
    onNavigate(callback) {
        this.listeners.push(callback);
        
        // Return unsubscribe function
        return () => {
            const index = this.listeners.indexOf(callback);
            if (index > -1) {
                this.listeners.splice(index, 1);
            }
        };
    }
    
    // Setup popstate listener
    setupPopstateListener() {
        window.addEventListener('popstate', (event) => {
            this.notifyListeners('pop', event.state, window.location.href);
        });
    }
    
    // Notify all listeners
    notifyListeners(type, state, url) {
        this.listeners.forEach(callback => {
            callback({ type, state, url, timestamp: Date.now() });
        });
    }
    
    // Create navigation entry with metadata
    createEntry(path, data = {}) {
        const entry = {
            path,
            timestamp: Date.now(),
            data,
            id: Math.random().toString(36).substr(2, 9)
        };
        
        this.pushState(entry, '', path);
        return entry;
    }
}

// Usage
const historyManager = new HistoryManager();

// Listen for navigation changes
const unsubscribe = historyManager.onNavigate((event) => {
    console.log('Navigation event:', event);
});

// Simulate single-page app navigation
function simulateSPANavigation() {
    const pages = [
        { path: '/home', data: { title: 'Home', content: 'Welcome to home page' } },
        { path: '/about', data: { title: 'About', content: 'About us page' } },
        { path: '/contact', data: { title: 'Contact', content: 'Contact form' } }
    ];
    
    pages.forEach((page, index) => {
        setTimeout(() => {
            historyManager.createEntry(page.path, page.data);
            console.log(`Navigated to ${page.path}`);
        }, index * 1000);
    });
}

// Browser back/forward button handling
function handleBrowserNavigation() {
    window.addEventListener('popstate', (event) => {
        const state = event.state;
        
        if (state) {
            console.log('Browser navigation to:', state.path);
            console.log('Page data:', state.data);
            
            // Update UI based on state
            updatePageContent(state);
        } else {
            console.log('Browser navigation to initial page');
            updatePageContent({ path: '/', data: { title: 'Initial Page' } });
        }
    });
}

function updatePageContent(state) {
    // Simulate updating page content
    console.log(`Updating page: ${state.data?.title || 'Unknown'}`);
    
    // In real app, you would update DOM here
    if (typeof document !== 'undefined') {
        document.title = state.data?.title || 'Default Title';
    }
}

// handleBrowserNavigation();
```

## Window Events

### Window Event Handling
```javascript
// Window load events
function setupWindowEvents() {
    const events = {
        // Page load events
        beforeunload: (event) => {
            console.log('Page is about to unload');
            // event.returnValue = 'Are you sure you want to leave?';
            // return 'Are you sure you want to leave?';
        },
        
        unload: () => {
            console.log('Page is unloading');
        },
        
        load: () => {
            console.log('Page fully loaded');
        },
        
        DOMContentLoaded: () => {
            console.log('DOM content loaded');
        },
        
        // Window resize
        resize: () => {
            console.log(`Window resized to: ${window.innerWidth}x${window.innerHeight}`);
        },
        
        // Window focus/blur
        focus: () => {
            console.log('Window gained focus');
        },
        
        blur: () => {
            console.log('Window lost focus');
        },
        
        // Scroll events
        scroll: () => {
            console.log(`Page scrolled to: ${window.pageXOffset}, ${window.pageYOffset}`);
        },
        
        // Orientation change (mobile)
        orientationchange: () => {
            console.log('Orientation changed to:', window.orientation);
        },
        
        // Network status
        online: () => {
            console.log('Connection restored');
        },
        
        offline: () => {
            console.log('Connection lost');
        },
        
        // History changes
        popstate: (event) => {
            console.log('History state changed:', event.state);
        },
        
        // Hash changes
        hashchange: (event) => {
            console.log('Hash changed from', event.oldURL, 'to', event.newURL);
        },
        
        // Page visibility API
        visibilitychange: () => {
            if (document.hidden) {
                console.log('Page is now hidden');
            } else {
                console.log('Page is now visible');
            }
        },
        
        // Error handling
        error: (event) => {
            console.error('Global error:', event.error);
        },
        
        unhandledrejection: (event) => {
            console.error('Unhandled promise rejection:', event.reason);
            event.preventDefault(); // Prevent default handling
        }
    };
    
    // Add all event listeners
    Object.keys(events).forEach(eventName => {
        if (eventName === 'DOMContentLoaded') {
            document.addEventListener(eventName, events[eventName]);
        } else {
            window.addEventListener(eventName, events[eventName]);
        }
    });
    
    // Return cleanup function
    return () => {
        Object.keys(events).forEach(eventName => {
            if (eventName === 'DOMContentLoaded') {
                document.removeEventListener(eventName, events[eventName]);
            } else {
                window.removeEventListener(eventName, events[eventName]);
            }
        });
    };
}

// Throttled resize handler
function createThrottledResizeHandler(callback, delay = 250) {
    let timeoutId;
    let lastCall = 0;
    
    function throttledHandler() {
        const now = Date.now();
        
        if (now - lastCall >= delay) {
            lastCall = now;
            callback({
                width: window.innerWidth,
                height: window.innerHeight,
                timestamp: now
            });
        } else {
            clearTimeout(timeoutId);
            timeoutId = setTimeout(() => {
                lastCall = Date.now();
                callback({
                    width: window.innerWidth,
                    height: window.innerHeight,
                    timestamp: Date.now()
                });
            }, delay - (now - lastCall));
        }
    }
    
    window.addEventListener('resize', throttledHandler);
    
    return () => {
        window.removeEventListener('resize', throttledHandler);
        clearTimeout(timeoutId);
    };
}

// Usage
const cleanup = setupWindowEvents();
const cleanupResize = createThrottledResizeHandler((dimensions) => {
    console.log('Throttled resize:', dimensions);
});

// Performance monitoring
function monitorPagePerformance() {
    if (typeof performance !== 'undefined') {
        // Navigation timing
        window.addEventListener('load', () => {
            setTimeout(() => {
                const navigation = performance.getEntriesByType('navigation')[0];
                
                if (navigation) {
                    const metrics = {
                        dns: navigation.domainLookupEnd - navigation.domainLookupStart,
                        tcp: navigation.connectEnd - navigation.connectStart,
                        ssl: navigation.connectEnd - navigation.secureConnectionStart,
                        request: navigation.responseStart - navigation.requestStart,
                        response: navigation.responseEnd - navigation.responseStart,
                        dom: navigation.domContentLoadedEventEnd - navigation.domContentLoadedEventStart,
                        load: navigation.loadEventEnd - navigation.loadEventStart,
                        total: navigation.loadEventEnd - navigation.fetchStart
                    };
                    
                    console.log('Performance metrics (ms):', metrics);
                }
                
                // Core Web Vitals (if available)
                if ('PerformanceObserver' in window) {
                    // Largest Contentful Paint
                    new PerformanceObserver((list) => {
                        const entries = list.getEntries();
                        const lcp = entries[entries.length - 1];
                        console.log('LCP:', lcp.startTime);
                    }).observe({ entryTypes: ['largest-contentful-paint'] });
                    
                    // First Input Delay
                    new PerformanceObserver((list) => {
                        const entries = list.getEntries();
                        entries.forEach(entry => {
                            console.log('FID:', entry.processingStart - entry.startTime);
                        });
                    }).observe({ entryTypes: ['first-input'] });
                    
                    // Cumulative Layout Shift
                    new PerformanceObserver((list) => {
                        let clsValue = 0;
                        const entries = list.getEntries();
                        
                        entries.forEach(entry => {
                            if (!entry.hadRecentInput) {
                                clsValue += entry.value;
                            }
                        });
                        
                        console.log('CLS:', clsValue);
                    }).observe({ entryTypes: ['layout-shift'] });
                }
            }, 0);
        });
    }
}

// monitorPagePerformance();
```

## Window Communication

### Cross-Window Communication
```javascript
// Window communication utilities
class WindowCommunicator {
    constructor() {
        this.channels = new Map();
        this.setupMessageListener();
    }
    
    // Open new window
    openWindow(url, name, features = '') {
        const newWindow = window.open(url, name, features);
        
        if (newWindow) {
            // Store reference for communication
            this.channels.set(name, newWindow);
            
            // Check if window is closed periodically
            const checkClosed = setInterval(() => {
                if (newWindow.closed) {
                    this.channels.delete(name);
                    clearInterval(checkClosed);
                    console.log(`Window '${name}' was closed`);
                }
            }, 1000);
            
            return newWindow;
        } else {
            console.error('Failed to open window - popup blocked?');
            return null;
        }
    }
    
    // Send message to specific window
    sendMessage(windowName, message, targetOrigin = '*') {
        const targetWindow = this.channels.get(windowName);
        
        if (targetWindow && !targetWindow.closed) {
            targetWindow.postMessage({
                type: 'WINDOW_MESSAGE',
                from: window.name || 'parent',
                to: windowName,
                payload: message,
                timestamp: Date.now()
            }, targetOrigin);
        } else {
            console.error(`Window '${windowName}' not found or closed`);
        }
    }
    
    // Broadcast message to all windows
    broadcast(message, targetOrigin = '*') {
        this.channels.forEach((targetWindow, windowName) => {
            if (!targetWindow.closed) {
                this.sendMessage(windowName, message, targetOrigin);
            }
        });
    }
    
    // Setup message listener
    setupMessageListener() {
        window.addEventListener('message', (event) => {
            // Validate origin for security
            // if (event.origin !== 'https://trusted-domain.com') return;
            
            const { type, from, to, payload, timestamp } = event.data;
            
            if (type === 'WINDOW_MESSAGE') {
                console.log(`Message from '${from}':`, payload);
                
                // Emit custom event for application code
                window.dispatchEvent(new CustomEvent('windowMessage', {
                    detail: { from, to, payload, timestamp }
                }));
            }
        });
    }
    
    // Close specific window
    closeWindow(windowName) {
        const targetWindow = this.channels.get(windowName);
        
        if (targetWindow && !targetWindow.closed) {
            targetWindow.close();
            this.channels.delete(windowName);
        }
    }
    
    // Close all windows
    closeAllWindows() {
        this.channels.forEach((targetWindow, windowName) => {
            if (!targetWindow.closed) {
                targetWindow.close();
            }
        });
        this.channels.clear();
    }
    
    // Get list of open windows
    getOpenWindows() {
        const openWindows = [];
        
        this.channels.forEach((targetWindow, windowName) => {
            if (!targetWindow.closed) {
                openWindows.push({
                    name: windowName,
                    closed: targetWindow.closed,
                    location: targetWindow.location?.href || 'unknown'
                });
            }
        });
        
        return openWindows;
    }
}

// Usage
const communicator = new WindowCommunicator();

// Listen for messages
window.addEventListener('windowMessage', (event) => {
    console.log('Received window message:', event.detail);
});

// Example: Open popup and communicate
function demoWindowCommunication() {
    // Open a popup window
    const popup = communicator.openWindow(
        'about:blank',
        'popup1',
        'width=400,height=300,scrollbars=yes'
    );
    
    if (popup) {
        // Send message after popup loads
        setTimeout(() => {
            communicator.sendMessage('popup1', {
                type: 'greeting',
                message: 'Hello from parent window!'
            });
        }, 1000);
        
        // Send periodic updates
        let counter = 0;
        const interval = setInterval(() => {
            counter++;
            communicator.sendMessage('popup1', {
                type: 'update',
                counter: counter,
                timestamp: new Date().toISOString()
            });
            
            if (counter >= 5) {
                clearInterval(interval);
            }
        }, 2000);
    }
}

// SharedArrayBuffer communication (if available)
function createSharedMemoryChannel() {
    if (typeof SharedArrayBuffer !== 'undefined') {
        // Create shared buffer
        const sharedBuffer = new SharedArrayBuffer(1024);
        const sharedArray = new Int32Array(sharedBuffer);
        
        return {
            buffer: sharedBuffer,
            array: sharedArray,
            
            writeInt32(index, value) {
                Atomics.store(sharedArray, index, value);
                Atomics.notify(sharedArray, index);
            },
            
            readInt32(index) {
                return Atomics.load(sharedArray, index);
            },
            
            waitForChange(index, expectedValue, timeout = Infinity) {
                return Atomics.wait(sharedArray, index, expectedValue, timeout);
            }
        };
    } else {
        console.warn('SharedArrayBuffer not available');
        return null;
    }
}
```

### Broadcast Channel API
```javascript
// Modern cross-tab communication
class TabCommunicator {
    constructor(channelName = 'app-channel') {
        this.channelName = channelName;
        this.listeners = new Map();
        
        if ('BroadcastChannel' in window) {
            this.channel = new BroadcastChannel(channelName);
            this.setupBroadcastListener();
        } else {
            console.warn('BroadcastChannel not supported, falling back to localStorage');
            this.setupLocalStorageFallback();
        }
    }
    
    // Send message to all tabs
    broadcast(type, data) {
        const message = {
            type,
            data,
            timestamp: Date.now(),
            tabId: this.getTabId()
        };
        
        if (this.channel) {
            this.channel.postMessage(message);
        } else {
            // Fallback to localStorage
            localStorage.setItem('broadcast-message', JSON.stringify(message));
            localStorage.removeItem('broadcast-message'); // Trigger storage event
        }
    }
    
    // Listen for specific message types
    on(type, callback) {
        if (!this.listeners.has(type)) {
            this.listeners.set(type, []);
        }
        this.listeners.get(type).push(callback);
        
        // Return unsubscribe function
        return () => {
            const callbacks = this.listeners.get(type);
            if (callbacks) {
                const index = callbacks.indexOf(callback);
                if (index > -1) {
                    callbacks.splice(index, 1);
                }
            }
        };
    }
    
    // Setup BroadcastChannel listener
    setupBroadcastListener() {
        this.channel.addEventListener('message', (event) => {
            this.handleMessage(event.data);
        });
    }
    
    // Setup localStorage fallback
    setupLocalStorageFallback() {
        window.addEventListener('storage', (event) => {
            if (event.key === 'broadcast-message' && event.newValue) {
                try {
                    const message = JSON.parse(event.newValue);
                    this.handleMessage(message);
                } catch (error) {
                    console.error('Failed to parse broadcast message:', error);
                }
            }
        });
    }
    
    // Handle incoming messages
    handleMessage(message) {
        const { type, data, timestamp, tabId } = message;
        
        // Don't process messages from same tab
        if (tabId === this.getTabId()) return;
        
        const callbacks = this.listeners.get(type);
        if (callbacks) {
            callbacks.forEach(callback => {
                try {
                    callback(data, { timestamp, tabId });
                } catch (error) {
                    console.error('Error in message callback:', error);
                }
            });
        }
    }
    
    // Get unique tab ID
    getTabId() {
        if (!this.tabId) {
            this.tabId = sessionStorage.getItem('tab-id');
            if (!this.tabId) {
                this.tabId = Math.random().toString(36).substr(2, 9);
                sessionStorage.setItem('tab-id', this.tabId);
            }
        }
        return this.tabId;
    }
    
    // Close channel
    close() {
        if (this.channel) {
            this.channel.close();
        }
        this.listeners.clear();
    }
}

// Usage example
const tabComm = new TabCommunicator('my-app');

// Listen for user login/logout across tabs
const unsubscribeAuth = tabComm.on('auth', (data, meta) => {
    console.log('Auth event from another tab:', data, meta);
    
    if (data.type === 'login') {
        console.log('User logged in from another tab');
        // Update UI to reflect logged in state
    } else if (data.type === 'logout') {
        console.log('User logged out from another tab');
        // Update UI to reflect logged out state
    }
});

// Listen for data synchronization
const unsubscribeSync = tabComm.on('sync', (data) => {
    console.log('Data sync from another tab:', data);
    // Update local data cache
});

// Broadcast user login
function broadcastLogin(user) {
    tabComm.broadcast('auth', {
        type: 'login',
        user: user
    });
}

// Broadcast data update
function broadcastDataUpdate(entityType, entityData) {
    tabComm.broadcast('sync', {
        type: 'update',
        entity: entityType,
        data: entityData
    });
}

// Tab coordination - prevent multiple tabs from doing the same work
class TabCoordinator {
    constructor(channelName = 'coordinator') {
        this.tabComm = new TabCommunicator(channelName);
        this.isLeader = false;
        this.leaderCheckInterval = null;
        this.setupLeaderElection();
    }
    
    setupLeaderElection() {
        // Check if we should become leader
        this.checkLeadership();
        
        // Listen for leader announcements
        this.tabComm.on('leader', (data) => {
            if (data.action === 'announce') {
                this.isLeader = false;
                console.log('Another tab is now the leader');
            } else if (data.action === 'stepping-down') {
                // Start new election
                setTimeout(() => this.checkLeadership(), 100);
            }
        });
        
        // Periodic leadership check
        this.leaderCheckInterval = setInterval(() => {
            this.checkLeadership();
        }, 5000);
        
        // Step down when page unloads
        window.addEventListener('beforeunload', () => {
            if (this.isLeader) {
                this.stepDown();
            }
        });
    }
    
    checkLeadership() {
        const now = Date.now();
        const lastLeaderAnnounce = localStorage.getItem('leader-announce');
        
        // Become leader if no recent announcement
        if (!lastLeaderAnnounce || (now - parseInt(lastLeaderAnnounce)) > 10000) {
            this.becomeLeader();
        }
    }
    
    becomeLeader() {
        if (!this.isLeader) {
            this.isLeader = true;
            console.log('This tab is now the leader');
            
            // Announce leadership
            this.announceLeadership();
            
            // Start leader duties
            this.startLeaderDuties();
        }
    }
    
    announceLeadership() {
        const now = Date.now();
        localStorage.setItem('leader-announce', now.toString());
        
        this.tabComm.broadcast('leader', {
            action: 'announce',
            timestamp: now
        });
    }
    
    stepDown() {
        if (this.isLeader) {
            this.isLeader = false;
            console.log('Stepping down as leader');
            
            this.tabComm.broadcast('leader', {
                action: 'stepping-down',
                timestamp: Date.now()
            });
            
            this.stopLeaderDuties();
        }
    }
    
    startLeaderDuties() {
        // Only leader performs these tasks
        console.log('Starting leader duties');
        
        // Example: Periodic data sync
        this.leaderInterval = setInterval(() => {
            if (this.isLeader) {
                console.log('Leader performing periodic tasks');
                // Sync data, check for updates, etc.
            }
        }, 30000);
    }
    
    stopLeaderDuties() {
        if (this.leaderInterval) {
            clearInterval(this.leaderInterval);
            this.leaderInterval = null;
        }
    }
    
    destroy() {
        this.stepDown();
        if (this.leaderCheckInterval) {
            clearInterval(this.leaderCheckInterval);
        }
        this.tabComm.close();
    }
}

// const coordinator = new TabCoordinator();
```

---

## Related Topics
- [[15 - Document Object Model]]
- [[17 - Event Methods]]
- [[24 - Local Storage]]
- [[25 - Session Storage]]
- [[26 - Cookies]]

---

*Next: [[17 - Event Methods]]*
