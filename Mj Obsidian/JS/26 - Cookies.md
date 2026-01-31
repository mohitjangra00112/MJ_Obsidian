# Cookies

## Overview
Cookies are small pieces of data that websites store in a user's browser. They are automatically sent to the server with HTTP requests and are commonly used for session management, personalization, and tracking user behavior.

## Cookie Characteristics

### Properties and Limitations
```javascript
// Cookie characteristics
console.log('Cookie Properties:');
console.log('- Size limit: ~4KB per cookie');
console.log('- Quantity limit: ~300 cookies total, ~20 per domain');
console.log('- Sent to server: Yes (with every HTTP request)');
console.log('- Persistence: Configurable (session or expires)');
console.log('- Cross-subdomain: Configurable');
console.log('- Secure transmission: Configurable (HTTPS only)');
console.log('- HttpOnly: Server-only access (not accessible via JS)');

// Cookie vs other storage
console.log('\nStorage Comparison:');
console.log('Cookies: 4KB, sent to server, has expiration');
console.log('LocalStorage: ~5-10MB, client-only, persistent');
console.log('SessionStorage: ~5-10MB, client-only, session-based');
```

## Basic Cookie Operations

### Setting Cookies
```javascript
// Basic cookie setting
document.cookie = "username=john_doe";
document.cookie = "theme=dark";
document.cookie = "language=en";

// Setting cookies with expiration (7 days from now)
const expires = new Date();
expires.setTime(expires.getTime() + (7 * 24 * 60 * 60 * 1000));
document.cookie = `sessionToken=abc123; expires=${expires.toUTCString()}`;

// Setting cookies with max-age (in seconds)
document.cookie = "tempData=xyz789; max-age=3600"; // 1 hour

// Setting cookies with path
document.cookie = "userPrefs=value; path=/dashboard";

// Setting cookies with domain
document.cookie = "siteWide=value; domain=.example.com";

// Setting secure cookies (HTTPS only)
document.cookie = "secureData=value; secure";

// Setting HttpOnly cookies (server-side only, not accessible via JS)
// Note: HttpOnly can only be set server-side
// document.cookie = "serverOnly=value; HttpOnly"; // Won't work from JS

// Setting SameSite attribute
document.cookie = "csrfToken=value; SameSite=Strict";
document.cookie = "tracking=value; SameSite=Lax";
document.cookie = "crossSite=value; SameSite=None; Secure";
```

### Getting Cookies
```javascript
// Get all cookies (returns a string)
console.log(document.cookie);
// Example output: "username=john_doe; theme=dark; language=en"

// Parse cookies into an object
function getCookies() {
    const cookies = {};
    
    if (document.cookie) {
        document.cookie.split(';').forEach(cookie => {
            const [name, value] = cookie.trim().split('=');
            cookies[name] = decodeURIComponent(value || '');
        });
    }
    
    return cookies;
}

// Get specific cookie value
function getCookie(name) {
    const cookies = getCookies();
    return cookies[name] || null;
}

// Usage examples
const allCookies = getCookies();
console.log(allCookies);
// { username: "john_doe", theme: "dark", language: "en" }

const username = getCookie('username');
console.log(username); // "john_doe"

const nonExistent = getCookie('doesNotExist');
console.log(nonExistent); // null
```

### Deleting Cookies
```javascript
// Delete cookie by setting expiration to past date
function deleteCookie(name, path = '/', domain = '') {
    let cookie = `${name}=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=${path};`;
    
    if (domain) {
        cookie += ` domain=${domain};`;
    }
    
    document.cookie = cookie;
}

// Usage
deleteCookie('username');
deleteCookie('sessionToken');

// Delete cookie with specific path
deleteCookie('userPrefs', '/dashboard');

// Delete cookie with specific domain
deleteCookie('siteWide', '/', '.example.com');

// Alternative method using max-age
function deleteCookieMaxAge(name, path = '/') {
    document.cookie = `${name}=; max-age=0; path=${path}`;
}

// Delete all cookies (for current path)
function deleteAllCookies() {
    const cookies = getCookies();
    
    Object.keys(cookies).forEach(name => {
        deleteCookie(name);
    });
}
```

## Advanced Cookie Management

### Cookie Manager Class
```javascript
class CookieManager {
    constructor(options = {}) {
        this.defaults = {
            path: '/',
            domain: '',
            secure: false,
            sameSite: 'Lax',
            ...options
        };
    }
    
    set(name, value, options = {}) {
        const config = { ...this.defaults, ...options };
        
        // Encode the value to handle special characters
        const encodedValue = encodeURIComponent(value);
        
        // Build cookie string
        let cookie = `${name}=${encodedValue}`;
        
        // Add expiration
        if (config.expires) {
            if (typeof config.expires === 'number') {
                // Convert days to date
                const date = new Date();
                date.setTime(date.getTime() + (config.expires * 24 * 60 * 60 * 1000));
                cookie += `; expires=${date.toUTCString()}`;
            } else if (config.expires instanceof Date) {
                cookie += `; expires=${config.expires.toUTCString()}`;
            }
        }
        
        // Add max-age (takes precedence over expires)
        if (config.maxAge !== undefined) {
            cookie += `; max-age=${config.maxAge}`;
        }
        
        // Add path
        if (config.path) {
            cookie += `; path=${config.path}`;
        }
        
        // Add domain
        if (config.domain) {
            cookie += `; domain=${config.domain}`;
        }
        
        // Add secure flag
        if (config.secure) {
            cookie += '; secure';
        }
        
        // Add SameSite
        if (config.sameSite) {
            cookie += `; SameSite=${config.sameSite}`;
        }
        
        // Set the cookie
        document.cookie = cookie;
        
        return this;
    }
    
    get(name) {
        const cookies = this.getAll();
        return cookies[name] || null;
    }
    
    getAll() {
        const cookies = {};
        
        if (document.cookie) {
            document.cookie.split(';').forEach(cookie => {
                const [name, value] = cookie.trim().split('=');
                if (name && value !== undefined) {
                    cookies[name] = decodeURIComponent(value);
                }
            });
        }
        
        return cookies;
    }
    
    remove(name, options = {}) {
        const config = { ...this.defaults, ...options };
        
        // Set expiration to past date
        this.set(name, '', {
            ...config,
            expires: new Date(0),
            maxAge: 0
        });
        
        return this;
    }
    
    exists(name) {
        return this.get(name) !== null;
    }
    
    clear(options = {}) {
        const cookies = this.getAll();
        
        Object.keys(cookies).forEach(name => {
            this.remove(name, options);
        });
        
        return this;
    }
    
    // Set cookie with JSON value
    setJSON(name, value, options = {}) {
        try {
            const jsonValue = JSON.stringify(value);
            return this.set(name, jsonValue, options);
        } catch (error) {
            console.error('Failed to stringify cookie value:', error);
            return this;
        }
    }
    
    // Get cookie with JSON parsing
    getJSON(name, defaultValue = null) {
        const value = this.get(name);
        
        if (value === null) {
            return defaultValue;
        }
        
        try {
            return JSON.parse(value);
        } catch (error) {
            console.error('Failed to parse cookie JSON:', error);
            return defaultValue;
        }
    }
    
    // Check if cookies are enabled
    static isEnabled() {
        try {
            const testCookie = '__cookie_test__';
            document.cookie = `${testCookie}=test; max-age=1`;
            const enabled = document.cookie.includes(testCookie);
            
            // Clean up test cookie
            document.cookie = `${testCookie}=; max-age=0`;
            
            return enabled;
        } catch {
            return false;
        }
    }
    
    // Get cookie size in bytes
    getSize(name) {
        const value = this.get(name);
        if (value === null) return 0;
        
        // Calculate approximate size (name + value + overhead)
        return (name.length + value.length + 10) * 2; // UTF-16 encoding
    }
    
    // Get total cookies size
    getTotalSize() {
        let total = 0;
        const cookies = this.getAll();
        
        Object.entries(cookies).forEach(([name, value]) => {
            total += (name.length + value.length + 10) * 2;
        });
        
        return total;
    }
}

// Usage
const cookieManager = new CookieManager({
    path: '/',
    secure: window.location.protocol === 'https:',
    sameSite: 'Strict'
});

// Check if cookies are enabled
if (CookieManager.isEnabled()) {
    // Set simple cookie
    cookieManager.set('username', 'john_doe');
    
    // Set cookie with expiration (7 days)
    cookieManager.set('sessionToken', 'abc123', { expires: 7 });
    
    // Set cookie with max-age (1 hour)
    cookieManager.set('tempData', 'xyz789', { maxAge: 3600 });
    
    // Set JSON cookie
    cookieManager.setJSON('userPrefs', {
        theme: 'dark',
        language: 'en',
        notifications: true
    });
    
    // Get cookies
    console.log(cookieManager.get('username')); // "john_doe"
    console.log(cookieManager.getJSON('userPrefs')); // { theme: "dark", ... }
    
    // Check cookie size
    console.log(`Cookie size: ${cookieManager.getSize('userPrefs')} bytes`);
    console.log(`Total size: ${cookieManager.getTotalSize()} bytes`);
} else {
    console.warn('Cookies are not enabled');
}
```

### Cookie Consent Manager
```javascript
class CookieConsentManager {
    constructor(options = {}) {
        this.options = {
            consentCookieName: 'cookie_consent',
            consentDuration: 365, // days
            categories: ['necessary', 'analytics', 'marketing'],
            ...options
        };
        
        this.cookieManager = new CookieManager();
        this.init();
    }
    
    init() {
        this.consent = this.loadConsent();
        
        if (!this.consent) {
            this.showConsentBanner();
        } else {
            this.applyConsent();
        }
    }
    
    loadConsent() {
        const consentData = this.cookieManager.getJSON(this.options.consentCookieName);
        
        if (!consentData) {
            return null;
        }
        
        // Check if consent is still valid
        const expiryDate = new Date(consentData.timestamp + (this.options.consentDuration * 24 * 60 * 60 * 1000));
        
        if (new Date() > expiryDate) {
            this.cookieManager.remove(this.options.consentCookieName);
            return null;
        }
        
        return consentData;
    }
    
    saveConsent(categories) {
        const consentData = {
            categories: categories,
            timestamp: Date.now(),
            version: '1.0'
        };
        
        this.cookieManager.setJSON(
            this.options.consentCookieName,
            consentData,
            { expires: this.options.consentDuration }
        );
        
        this.consent = consentData;
        this.applyConsent();
    }
    
    hasConsent(category = null) {
        if (!this.consent) {
            return false;
        }
        
        if (category) {
            return this.consent.categories.includes(category);
        }
        
        return true; // Has some form of consent
    }
    
    grantConsent(categories) {
        // Always include necessary cookies
        const allowedCategories = ['necessary', ...categories];
        this.saveConsent([...new Set(allowedCategories)]);
        this.hideConsentBanner();
    }
    
    revokeConsent() {
        this.cookieManager.remove(this.options.consentCookieName);
        this.consent = null;
        this.clearNonNecessaryCookies();
        this.showConsentBanner();
    }
    
    applyConsent() {
        // Enable/disable cookie categories based on consent
        this.options.categories.forEach(category => {
            if (this.hasConsent(category)) {
                this.enableCookieCategory(category);
            } else {
                this.disableCookieCategory(category);
            }
        });
        
        // Dispatch consent event
        window.dispatchEvent(new CustomEvent('cookieConsentUpdate', {
            detail: {
                consent: this.consent,
                hasConsent: this.hasConsent.bind(this)
            }
        }));
    }
    
    enableCookieCategory(category) {
        switch (category) {
            case 'analytics':
                // Enable analytics tracking
                this.enableAnalytics();
                break;
            case 'marketing':
                // Enable marketing cookies
                this.enableMarketing();
                break;
            case 'necessary':
                // Necessary cookies are always enabled
                break;
        }
    }
    
    disableCookieCategory(category) {
        switch (category) {
            case 'analytics':
                this.disableAnalytics();
                break;
            case 'marketing':
                this.disableMarketing();
                break;
        }
    }
    
    enableAnalytics() {
        // Example: Enable Google Analytics
        console.log('Analytics cookies enabled');
        // gtag('consent', 'update', { analytics_storage: 'granted' });
    }
    
    disableAnalytics() {
        console.log('Analytics cookies disabled');
        // gtag('consent', 'update', { analytics_storage: 'denied' });
    }
    
    enableMarketing() {
        console.log('Marketing cookies enabled');
        // Enable advertising platforms
    }
    
    disableMarketing() {
        console.log('Marketing cookies disabled');
        // Disable advertising platforms
    }
    
    clearNonNecessaryCookies() {
        // Define patterns for non-necessary cookies
        const nonNecessaryPatterns = [
            /^_ga/, // Google Analytics
            /^_gid/,
            /^_utm/,
            /^_fb/, // Facebook
            /^_pinterest/
        ];
        
        const allCookies = this.cookieManager.getAll();
        
        Object.keys(allCookies).forEach(name => {
            const isNonNecessary = nonNecessaryPatterns.some(pattern => pattern.test(name));
            
            if (isNonNecessary) {
                this.cookieManager.remove(name);
            }
        });
    }
    
    showConsentBanner() {
        // Create and show consent banner
        const banner = this.createConsentBanner();
        document.body.appendChild(banner);
    }
    
    hideConsentBanner() {
        const banner = document.getElementById('cookie-consent-banner');
        if (banner) {
            banner.remove();
        }
    }
    
    createConsentBanner() {
        const banner = document.createElement('div');
        banner.id = 'cookie-consent-banner';
        banner.style.cssText = `
            position: fixed;
            bottom: 0;
            left: 0;
            right: 0;
            background: #333;
            color: white;
            padding: 20px;
            z-index: 10000;
            display: flex;
            align-items: center;
            justify-content: space-between;
            flex-wrap: wrap;
        `;
        
        banner.innerHTML = `
            <div style="flex: 1; margin-right: 20px;">
                <p style="margin: 0;">
                    We use cookies to improve your experience. 
                    <a href="/privacy-policy" style="color: #4CAF50;">Learn more</a>
                </p>
            </div>
            <div style="display: flex; gap: 10px; flex-wrap: wrap;">
                <button id="accept-necessary" style="padding: 8px 16px; background: #666; color: white; border: none; cursor: pointer;">
                    Necessary Only
                </button>
                <button id="accept-all" style="padding: 8px 16px; background: #4CAF50; color: white; border: none; cursor: pointer;">
                    Accept All
                </button>
                <button id="customize-cookies" style="padding: 8px 16px; background: #2196F3; color: white; border: none; cursor: pointer;">
                    Customize
                </button>
            </div>
        `;
        
        // Add event listeners
        banner.querySelector('#accept-necessary').addEventListener('click', () => {
            this.grantConsent([]);
        });
        
        banner.querySelector('#accept-all').addEventListener('click', () => {
            this.grantConsent(this.options.categories.filter(c => c !== 'necessary'));
        });
        
        banner.querySelector('#customize-cookies').addEventListener('click', () => {
            this.showCustomizeModal();
        });
        
        return banner;
    }
    
    showCustomizeModal() {
        // Create customization modal
        const modal = document.createElement('div');
        modal.style.cssText = `
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0,0,0,0.5);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 10001;
        `;
        
        const content = document.createElement('div');
        content.style.cssText = `
            background: white;
            padding: 30px;
            border-radius: 8px;
            max-width: 500px;
            width: 90%;
        `;
        
        content.innerHTML = `
            <h3>Cookie Preferences</h3>
            <div style="margin: 20px 0;">
                <label style="display: block; margin: 10px 0;">
                    <input type="checkbox" checked disabled> 
                    Necessary Cookies (Required)
                </label>
                <label style="display: block; margin: 10px 0;">
                    <input type="checkbox" id="analytics-consent"> 
                    Analytics Cookies
                </label>
                <label style="display: block; margin: 10px 0;">
                    <input type="checkbox" id="marketing-consent"> 
                    Marketing Cookies
                </label>
            </div>
            <div style="text-align: right;">
                <button id="save-preferences" style="padding: 10px 20px; background: #4CAF50; color: white; border: none; cursor: pointer; margin-left: 10px;">
                    Save Preferences
                </button>
            </div>
        `;
        
        modal.appendChild(content);
        document.body.appendChild(modal);
        
        // Handle save
        content.querySelector('#save-preferences').addEventListener('click', () => {
            const categories = [];
            
            if (content.querySelector('#analytics-consent').checked) {
                categories.push('analytics');
            }
            
            if (content.querySelector('#marketing-consent').checked) {
                categories.push('marketing');
            }
            
            this.grantConsent(categories);
            modal.remove();
        });
        
        // Close on background click
        modal.addEventListener('click', (e) => {
            if (e.target === modal) {
                modal.remove();
            }
        });
    }
    
    getConsentStatus() {
        return {
            hasConsent: this.hasConsent(),
            categories: this.consent?.categories || [],
            timestamp: this.consent?.timestamp || null,
            version: this.consent?.version || null
        };
    }
}

// Usage
const consentManager = new CookieConsentManager({
    consentDuration: 365,
    categories: ['necessary', 'analytics', 'marketing']
});

// Listen for consent updates
window.addEventListener('cookieConsentUpdate', (event) => {
    const { consent, hasConsent } = event.detail;
    
    console.log('Consent status:', consentManager.getConsentStatus());
    
    // Only set analytics cookies if consent is given
    if (hasConsent('analytics')) {
        cookieManager.set('_analytics', 'enabled');
    }
    
    // Only set marketing cookies if consent is given
    if (hasConsent('marketing')) {
        cookieManager.set('_marketing', 'enabled');
    }
});
```

### Cross-Domain Cookie Sharing
```javascript
class CrossDomainCookieManager {
    constructor(domains = []) {
        this.domains = domains;
        this.cookieManager = new CookieManager();
    }
    
    // Set cookie across multiple domains
    setCrossDomain(name, value, options = {}) {
        // Set for current domain
        this.cookieManager.set(name, value, options);
        
        // Set for each specified domain
        this.domains.forEach(domain => {
            this.cookieManager.set(name, value, {
                ...options,
                domain: domain
            });
        });
    }
    
    // Share data via postMessage to iframe
    shareViaPostMessage(targetOrigin, data) {
        // Create hidden iframe for target domain
        const iframe = document.createElement('iframe');
        iframe.style.display = 'none';
        iframe.src = `${targetOrigin}/cookie-receiver.html`;
        
        iframe.onload = () => {
            iframe.contentWindow.postMessage({
                type: 'SET_COOKIES',
                data: data
            }, targetOrigin);
            
            // Remove iframe after a delay
            setTimeout(() => {
                document.body.removeChild(iframe);
            }, 1000);
        };
        
        document.body.appendChild(iframe);
    }
    
    // Listen for cross-domain cookie data
    setupCrossDomainReceiver() {
        window.addEventListener('message', (event) => {
            // Verify origin is allowed
            if (!this.domains.includes(event.origin)) {
                return;
            }
            
            if (event.data.type === 'SET_COOKIES') {
                Object.entries(event.data.data).forEach(([name, value]) => {
                    this.cookieManager.set(name, value);
                });
                
                // Send confirmation back
                event.source.postMessage({
                    type: 'COOKIES_SET',
                    success: true
                }, event.origin);
            }
        });
    }
    
    // Sync cookies between domains using a central service
    syncWithCentralService(serviceUrl) {
        const currentCookies = this.cookieManager.getAll();
        
        fetch(`${serviceUrl}/sync-cookies`, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                domain: window.location.hostname,
                cookies: currentCookies
            }),
            credentials: 'include'
        })
        .then(response => response.json())
        .then(data => {
            if (data.cookies) {
                Object.entries(data.cookies).forEach(([name, value]) => {
                    this.cookieManager.set(name, value);
                });
            }
        })
        .catch(error => {
            console.error('Cookie sync failed:', error);
        });
    }
}

// Usage
const crossDomainManager = new CrossDomainCookieManager([
    'https://example.com',
    'https://app.example.com',
    'https://api.example.com'
]);

// Setup receiver for cross-domain messages
crossDomainManager.setupCrossDomainReceiver();

// Set cookie across domains
crossDomainManager.setCrossDomain('userToken', 'abc123', {
    expires: 7,
    secure: true,
    sameSite: 'None'
});

// Share specific data with another domain
crossDomainManager.shareViaPostMessage('https://app.example.com', {
    sessionId: '12345',
    userId: 'user123'
});
```

---

## Related Topics
- [[24 - Local Storage]]
- [[25 - Session Storage]]
- [[27 - JSON]]
- [[43 - Browser APIs]]
- [[47 - Security Best Practices]]

---

*Next: [[28 - Object Methods]]*
