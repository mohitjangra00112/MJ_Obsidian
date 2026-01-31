# Frontend Frameworks

## Overview
Frontend frameworks provide structured approaches to building interactive web applications. They offer tools, patterns, and libraries that help developers create maintainable, scalable, and efficient user interfaces. This guide covers the major frameworks including React, Vue.js, Angular, and emerging frameworks.

## React

### Introduction to React
React is a declarative, efficient, and flexible JavaScript library for building user interfaces, developed by Facebook.

```javascript
// Basic React component
import React, { useState, useEffect } from 'react';

// Functional component with hooks
function UserProfile({ userId }) {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);
    
    useEffect(() => {
        async function fetchUser() {
            try {
                setLoading(true);
                const response = await fetch(`/api/users/${userId}`);
                if (!response.ok) throw new Error('User not found');
                const userData = await response.json();
                setUser(userData);
            } catch (err) {
                setError(err.message);
            } finally {
                setLoading(false);
            }
        }
        
        fetchUser();
    }, [userId]);
    
    if (loading) return <div className="loading">Loading...</div>;
    if (error) return <div className="error">Error: {error}</div>;
    if (!user) return <div>No user found</div>;
    
    return (
        <div className="user-profile">
            <img src={user.avatar} alt={user.name} />
            <h2>{user.name}</h2>
            <p>{user.email}</p>
            <div className="user-stats">
                <span>Posts: {user.postCount}</span>
                <span>Followers: {user.followerCount}</span>
            </div>
        </div>
    );
}

// Class component (legacy but still used)
class Counter extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            count: 0
        };
    }
    
    increment = () => {
        this.setState(prevState => ({
            count: prevState.count + 1
        }));
    }
    
    render() {
        return (
            <div className="counter">
                <h3>Count: {this.state.count}</h3>
                <button onClick={this.increment}>Increment</button>
            </div>
        );
    }
}

// React Hooks examples
function useLocalStorage(key, initialValue) {
    const [storedValue, setStoredValue] = useState(() => {
        try {
            const item = window.localStorage.getItem(key);
            return item ? JSON.parse(item) : initialValue;
        } catch (error) {
            return initialValue;
        }
    });
    
    const setValue = (value) => {
        try {
            setStoredValue(value);
            window.localStorage.setItem(key, JSON.stringify(value));
        } catch (error) {
            console.error('Error saving to localStorage:', error);
        }
    };
    
    return [storedValue, setValue];
}

// Custom hook usage
function Settings() {
    const [theme, setTheme] = useLocalStorage('theme', 'light');
    const [language, setLanguage] = useLocalStorage('language', 'en');
    
    return (
        <div className={`settings theme-${theme}`}>
            <h2>Settings</h2>
            <div>
                <label>
                    Theme:
                    <select value={theme} onChange={(e) => setTheme(e.target.value)}>
                        <option value="light">Light</option>
                        <option value="dark">Dark</option>
                    </select>
                </label>
            </div>
            <div>
                <label>
                    Language:
                    <select value={language} onChange={(e) => setLanguage(e.target.value)}>
                        <option value="en">English</option>
                        <option value="es">Spanish</option>
                        <option value="fr">French</option>
                    </select>
                </label>
            </div>
        </div>
    );
}

// Context API for state management
const ThemeContext = React.createContext();

function ThemeProvider({ children }) {
    const [theme, setTheme] = useLocalStorage('theme', 'light');
    
    const toggleTheme = () => {
        setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
    };
    
    return (
        <ThemeContext.Provider value={{ theme, toggleTheme }}>
            {children}
        </ThemeContext.Provider>
    );
}

function useTheme() {
    const context = React.useContext(ThemeContext);
    if (!context) {
        throw new Error('useTheme must be used within a ThemeProvider');
    }
    return context;
}

// Component using context
function Header() {
    const { theme, toggleTheme } = useTheme();
    
    return (
        <header className={`header header-${theme}`}>
            <h1>My App</h1>
            <button onClick={toggleTheme}>
                Switch to {theme === 'light' ? 'dark' : 'light'} mode
            </button>
        </header>
    );
}

// React Router example
import { BrowserRouter as Router, Routes, Route, Link, useParams } from 'react-router-dom';

function App() {
    return (
        <ThemeProvider>
            <Router>
                <div className="app">
                    <nav>
                        <Link to="/">Home</Link>
                        <Link to="/users">Users</Link>
                        <Link to="/settings">Settings</Link>
                    </nav>
                    
                    <Routes>
                        <Route path="/" element={<Home />} />
                        <Route path="/users" element={<UserList />} />
                        <Route path="/users/:id" element={<UserDetail />} />
                        <Route path="/settings" element={<Settings />} />
                    </Routes>
                </div>
            </Router>
        </ThemeProvider>
    );
}

function UserDetail() {
    const { id } = useParams();
    return <UserProfile userId={id} />;
}
```

### React Performance Optimization
```javascript
import React, { memo, useMemo, useCallback, lazy, Suspense } from 'react';

// React.memo for preventing unnecessary re-renders
const ExpensiveComponent = memo(function ExpensiveComponent({ data, onUpdate }) {
    console.log('ExpensiveComponent rendering');
    
    const processedData = useMemo(() => {
        return data.map(item => ({
            ...item,
            processed: item.value * 2
        }));
    }, [data]);
    
    const handleClick = useCallback((id) => {
        onUpdate(id);
    }, [onUpdate]);
    
    return (
        <div>
            {processedData.map(item => (
                <div key={item.id} onClick={() => handleClick(item.id)}>
                    {item.name}: {item.processed}
                </div>
            ))}
        </div>
    );
});

// Lazy loading components
const LazyChart = lazy(() => import('./Chart'));

function Dashboard() {
    return (
        <div>
            <h1>Dashboard</h1>
            <Suspense fallback={<div>Loading chart...</div>}>
                <LazyChart />
            </Suspense>
        </div>
    );
}

// Virtual scrolling for large lists
function VirtualList({ items, itemHeight, containerHeight }) {
    const [scrollTop, setScrollTop] = useState(0);
    
    const visibleStart = Math.floor(scrollTop / itemHeight);
    const visibleEnd = Math.min(
        visibleStart + Math.ceil(containerHeight / itemHeight),
        items.length - 1
    );
    
    const visibleItems = items.slice(visibleStart, visibleEnd + 1);
    
    return (
        <div 
            style={{ height: containerHeight, overflow: 'auto' }}
            onScroll={(e) => setScrollTop(e.target.scrollTop)}
        >
            <div style={{ height: items.length * itemHeight, position: 'relative' }}>
                {visibleItems.map((item, index) => (
                    <div
                        key={item.id}
                        style={{
                            position: 'absolute',
                            top: (visibleStart + index) * itemHeight,
                            height: itemHeight,
                            width: '100%'
                        }}
                    >
                        {item.name}
                    </div>
                ))}
            </div>
        </div>
    );
}
```

## Vue.js

### Vue.js Fundamentals
Vue.js is a progressive framework for building user interfaces with a gentle learning curve.

```javascript
// Vue 3 Composition API
import { ref, reactive, computed, watch, onMounted } from 'vue';

// Single File Component
export default {
    name: 'UserProfile',
    props: {
        userId: {
            type: String,
            required: true
        }
    },
    setup(props) {
        // Reactive data
        const user = ref(null);
        const loading = ref(true);
        const error = ref(null);
        
        // Reactive object
        const form = reactive({
            name: '',
            email: '',
            bio: ''
        });
        
        // Computed property
        const displayName = computed(() => {
            return user.value ? `${user.value.firstName} ${user.value.lastName}` : '';
        });
        
        // Methods
        const fetchUser = async () => {
            try {
                loading.value = true;
                const response = await fetch(`/api/users/${props.userId}`);
                user.value = await response.json();
            } catch (err) {
                error.value = err.message;
            } finally {
                loading.value = false;
            }
        };
        
        const updateUser = async () => {
            try {
                const response = await fetch(`/api/users/${props.userId}`, {
                    method: 'PUT',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(form)
                });
                
                if (response.ok) {
                    await fetchUser();
                }
            } catch (err) {
                error.value = err.message;
            }
        };
        
        // Watchers
        watch(() => props.userId, (newId) => {
            if (newId) {
                fetchUser();
            }
        });
        
        watch(user, (newUser) => {
            if (newUser) {
                form.name = newUser.name;
                form.email = newUser.email;
                form.bio = newUser.bio;
            }
        });
        
        // Lifecycle hooks
        onMounted(() => {
            fetchUser();
        });
        
        return {
            user,
            loading,
            error,
            form,
            displayName,
            updateUser
        };
    }
};

// Template
/*
<template>
  <div class="user-profile">
    <div v-if="loading" class="loading">Loading...</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else-if="user" class="user-content">
      <h2>{{ displayName }}</h2>
      
      <form @submit.prevent="updateUser">
        <div class="form-group">
          <label for="name">Name:</label>
          <input 
            id="name" 
            v-model="form.name" 
            type="text" 
            required 
          />
        </div>
        
        <div class="form-group">
          <label for="email">Email:</label>
          <input 
            id="email" 
            v-model="form.email" 
            type="email" 
            required 
          />
        </div>
        
        <div class="form-group">
          <label for="bio">Bio:</label>
          <textarea 
            id="bio" 
            v-model="form.bio" 
            rows="4"
          ></textarea>
        </div>
        
        <button type="submit">Update Profile</button>
      </form>
    </div>
  </div>
</template>
*/

// Vuex store (state management)
import { createStore } from 'vuex';

const store = createStore({
    state: {
        users: [],
        currentUser: null,
        loading: false
    },
    
    mutations: {
        SET_USERS(state, users) {
            state.users = users;
        },
        
        SET_CURRENT_USER(state, user) {
            state.currentUser = user;
        },
        
        SET_LOADING(state, loading) {
            state.loading = loading;
        },
        
        UPDATE_USER(state, updatedUser) {
            const index = state.users.findIndex(user => user.id === updatedUser.id);
            if (index !== -1) {
                state.users.splice(index, 1, updatedUser);
            }
        }
    },
    
    actions: {
        async fetchUsers({ commit }) {
            commit('SET_LOADING', true);
            try {
                const response = await fetch('/api/users');
                const users = await response.json();
                commit('SET_USERS', users);
            } catch (error) {
                console.error('Failed to fetch users:', error);
            } finally {
                commit('SET_LOADING', false);
            }
        },
        
        async updateUser({ commit }, user) {
            try {
                const response = await fetch(`/api/users/${user.id}`, {
                    method: 'PUT',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(user)
                });
                
                if (response.ok) {
                    const updatedUser = await response.json();
                    commit('UPDATE_USER', updatedUser);
                    return updatedUser;
                }
            } catch (error) {
                console.error('Failed to update user:', error);
                throw error;
            }
        }
    },
    
    getters: {
        activeUsers: state => state.users.filter(user => user.active),
        getUserById: state => id => state.users.find(user => user.id === id)
    }
});

// Vue Router
import { createRouter, createWebHistory } from 'vue-router';

const routes = [
    {
        path: '/',
        name: 'Home',
        component: () => import('./views/Home.vue')
    },
    {
        path: '/users',
        name: 'Users',
        component: () => import('./views/Users.vue'),
        children: [
            {
                path: ':id',
                name: 'UserDetail',
                component: () => import('./views/UserDetail.vue'),
                props: true
            }
        ]
    },
    {
        path: '/login',
        name: 'Login',
        component: () => import('./views/Login.vue'),
        meta: { requiresGuest: true }
    }
];

const router = createRouter({
    history: createWebHistory(),
    routes
});

// Navigation guards
router.beforeEach((to, from, next) => {
    const isAuthenticated = store.state.currentUser !== null;
    
    if (to.meta.requiresAuth && !isAuthenticated) {
        next('/login');
    } else if (to.meta.requiresGuest && isAuthenticated) {
        next('/');
    } else {
        next();
    }
});

// Custom composables
import { ref, onMounted, onUnmounted } from 'vue';

// useApi composable
export function useApi(url) {
    const data = ref(null);
    const error = ref(null);
    const loading = ref(false);
    
    const execute = async (options = {}) => {
        try {
            loading.value = true;
            error.value = null;
            
            const response = await fetch(url, options);
            if (!response.ok) throw new Error(`HTTP ${response.status}`);
            
            data.value = await response.json();
        } catch (err) {
            error.value = err.message;
        } finally {
            loading.value = false;
        }
    };
    
    return { data, error, loading, execute };
}

// useEventListener composable
export function useEventListener(target, event, handler) {
    onMounted(() => {
        target.addEventListener(event, handler);
    });
    
    onUnmounted(() => {
        target.removeEventListener(event, handler);
    });
}
```

## Angular

### Angular Fundamentals
Angular is a platform and framework for building single-page client applications using HTML and TypeScript.

```typescript
// Component
import { Component, OnInit, Input, Output, EventEmitter } from '@angular/core';
import { Observable } from 'rxjs';
import { map, catchError } from 'rxjs/operators';

@Component({
    selector: 'app-user-profile',
    template: `
        <div class="user-profile" *ngIf="!loading; else loadingTemplate">
            <div *ngIf="error" class="error">{{ error }}</div>
            
            <div *ngIf="user" class="user-content">
                <h2>{{ user.name }}</h2>
                <p>{{ user.email }}</p>
                
                <form (ngSubmit)="onSubmit()" #userForm="ngForm">
                    <div class="form-group">
                        <label for="name">Name:</label>
                        <input 
                            id="name"
                            type="text"
                            [(ngModel)]="user.name"
                            name="name"
                            required
                            #name="ngModel"
                        />
                        <div *ngIf="name.invalid && name.touched" class="error">
                            Name is required
                        </div>
                    </div>
                    
                    <div class="form-group">
                        <label for="email">Email:</label>
                        <input 
                            id="email"
                            type="email"
                            [(ngModel)]="user.email"
                            name="email"
                            required
                            email
                            #email="ngModel"
                        />
                        <div *ngIf="email.invalid && email.touched" class="error">
                            <span *ngIf="email.errors?.['required']">Email is required</span>
                            <span *ngIf="email.errors?.['email']">Invalid email format</span>
                        </div>
                    </div>
                    
                    <button 
                        type="submit" 
                        [disabled]="userForm.invalid || saving"
                    >
                        {{ saving ? 'Saving...' : 'Save' }}
                    </button>
                </form>
            </div>
        </div>
        
        <ng-template #loadingTemplate>
            <div class="loading">Loading...</div>
        </ng-template>
    `,
    styles: [`
        .user-profile {
            padding: 20px;
        }
        .error {
            color: red;
            margin: 10px 0;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            margin-bottom: 5px;
        }
        input {
            width: 100%;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        button:disabled {
            opacity: 0.6;
            cursor: not-allowed;
        }
    `]
})
export class UserProfileComponent implements OnInit {
    @Input() userId!: string;
    @Output() userUpdated = new EventEmitter<User>();
    
    user: User | null = null;
    loading = false;
    saving = false;
    error: string | null = null;
    
    constructor(private userService: UserService) {}
    
    ngOnInit() {
        this.loadUser();
    }
    
    loadUser() {
        if (!this.userId) return;
        
        this.loading = true;
        this.error = null;
        
        this.userService.getUser(this.userId).subscribe({
            next: (user) => {
                this.user = user;
                this.loading = false;
            },
            error: (error) => {
                this.error = error.message;
                this.loading = false;
            }
        });
    }
    
    onSubmit() {
        if (!this.user) return;
        
        this.saving = true;
        
        this.userService.updateUser(this.user).subscribe({
            next: (updatedUser) => {
                this.user = updatedUser;
                this.userUpdated.emit(updatedUser);
                this.saving = false;
            },
            error: (error) => {
                this.error = error.message;
                this.saving = false;
            }
        });
    }
}

// Service
import { Injectable } from '@angular/core';
import { HttpClient, HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError, retry } from 'rxjs/operators';

export interface User {
    id: string;
    name: string;
    email: string;
    avatar?: string;
}

@Injectable({
    providedIn: 'root'
})
export class UserService {
    private apiUrl = '/api/users';
    
    constructor(private http: HttpClient) {}
    
    getUsers(): Observable<User[]> {
        return this.http.get<User[]>(this.apiUrl)
            .pipe(
                retry(2),
                catchError(this.handleError)
            );
    }
    
    getUser(id: string): Observable<User> {
        return this.http.get<User>(`${this.apiUrl}/${id}`)
            .pipe(
                catchError(this.handleError)
            );
    }
    
    updateUser(user: User): Observable<User> {
        return this.http.put<User>(`${this.apiUrl}/${user.id}`, user)
            .pipe(
                catchError(this.handleError)
            );
    }
    
    createUser(user: Omit<User, 'id'>): Observable<User> {
        return this.http.post<User>(this.apiUrl, user)
            .pipe(
                catchError(this.handleError)
            );
    }
    
    deleteUser(id: string): Observable<void> {
        return this.http.delete<void>(`${this.apiUrl}/${id}`)
            .pipe(
                catchError(this.handleError)
            );
    }
    
    private handleError(error: HttpErrorResponse) {
        let errorMessage = 'An unknown error occurred';
        
        if (error.error instanceof ErrorEvent) {
            // Client-side error
            errorMessage = error.error.message;
        } else {
            // Server-side error
            errorMessage = `Error Code: ${error.status}\nMessage: ${error.message}`;
        }
        
        console.error(errorMessage);
        return throwError(() => new Error(errorMessage));
    }
}

// Module
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
import { HttpClientModule } from '@angular/common/http';
import { RouterModule, Routes } from '@angular/router';

import { AppComponent } from './app.component';
import { UserProfileComponent } from './user-profile/user-profile.component';
import { UserListComponent } from './user-list/user-list.component';

const routes: Routes = [
    { path: '', redirectTo: '/users', pathMatch: 'full' },
    { path: 'users', component: UserListComponent },
    { path: 'users/:id', component: UserProfileComponent },
    { path: '**', redirectTo: '/users' }
];

@NgModule({
    declarations: [
        AppComponent,
        UserProfileComponent,
        UserListComponent
    ],
    imports: [
        BrowserModule,
        FormsModule,
        ReactiveFormsModule,
        HttpClientModule,
        RouterModule.forRoot(routes)
    ],
    providers: [],
    bootstrap: [AppComponent]
})
export class AppModule { }

// Reactive Forms
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
    selector: 'app-reactive-form',
    template: `
        <form [formGroup]="userForm" (ngSubmit)="onSubmit()">
            <div class="form-group">
                <label for="name">Name:</label>
                <input 
                    id="name"
                    type="text"
                    formControlName="name"
                    [class.is-invalid]="name?.invalid && name?.touched"
                />
                <div *ngIf="name?.invalid && name?.touched" class="invalid-feedback">
                    <div *ngIf="name?.errors?.['required']">Name is required</div>
                    <div *ngIf="name?.errors?.['minlength']">Name must be at least 2 characters</div>
                </div>
            </div>
            
            <div class="form-group">
                <label for="email">Email:</label>
                <input 
                    id="email"
                    type="email"
                    formControlName="email"
                    [class.is-invalid]="email?.invalid && email?.touched"
                />
                <div *ngIf="email?.invalid && email?.touched" class="invalid-feedback">
                    <div *ngIf="email?.errors?.['required']">Email is required</div>
                    <div *ngIf="email?.errors?.['email']">Invalid email format</div>
                </div>
            </div>
            
            <button type="submit" [disabled]="userForm.invalid">Submit</button>
        </form>
    `
})
export class ReactiveFormComponent {
    userForm: FormGroup;
    
    constructor(private fb: FormBuilder) {
        this.userForm = this.fb.group({
            name: ['', [Validators.required, Validators.minLength(2)]],
            email: ['', [Validators.required, Validators.email]]
        });
    }
    
    get name() { return this.userForm.get('name'); }
    get email() { return this.userForm.get('email'); }
    
    onSubmit() {
        if (this.userForm.valid) {
            console.log(this.userForm.value);
        }
    }
}
```

## Framework Comparison

### Choosing the Right Framework
```javascript
// Framework comparison criteria

const frameworkComparison = {
    react: {
        pros: [
            'Large ecosystem and community',
            'Flexible and unopinionated',
            'Great developer tools',
            'Strong performance with Virtual DOM',
            'Backed by Facebook',
            'Excellent for complex UIs'
        ],
        cons: [
            'Steep learning curve',
            'Requires additional libraries for full functionality',
            'JSX syntax learning requirement',
            'Rapid ecosystem changes'
        ],
        useCase: [
            'Complex single-page applications',
            'When you need maximum flexibility',
            'Large development teams',
            'Projects requiring custom solutions'
        ]
    },
    
    vue: {
        pros: [
            'Gentle learning curve',
            'Excellent documentation',
            'Good performance',
            'Template syntax familiar to HTML',
            'Progressive adoption possible',
            'Great tooling (Vue CLI, Vite)'
        ],
        cons: [
            'Smaller ecosystem compared to React',
            'Fewer job opportunities',
            'Less enterprise adoption',
            'Breaking changes between major versions'
        ],
        useCase: [
            'Small to medium projects',
            'Teams new to modern frameworks',
            'Rapid prototyping',
            'Progressive enhancement'
        ]
    },
    
    angular: {
        pros: [
            'Full-featured framework',
            'Strong TypeScript support',
            'Excellent CLI tools',
            'Great for enterprise applications',
            'Comprehensive testing utilities',
            'Strong opinionated structure'
        ],
        cons: [
            'Steep learning curve',
            'Heavy framework',
            'Complex for simple applications',
            'Verbose syntax',
            'Breaking changes between versions'
        ],
        useCase: [
            'Large enterprise applications',
            'Teams preferring opinionated structure',
            'Projects requiring comprehensive features',
            'TypeScript-first development'
        ]
    }
};

// Performance comparison example
function measureFrameworkPerformance() {
    const metrics = {
        bundleSize: {
            react: '42.2kb (React + ReactDOM gzipped)',
            vue: '34kb (Vue 3 gzipped)',
            angular: '130kb (Angular framework gzipped)'
        },
        
        renderingSpeed: {
            react: 'Fast (Virtual DOM diffing)',
            vue: 'Fast (Optimized reactivity system)',
            angular: 'Good (Change detection with zones)'
        },
        
        memoryUsage: {
            react: 'Moderate (depends on state management)',
            vue: 'Low (efficient reactivity)',
            angular: 'Higher (comprehensive framework overhead)'
        },
        
        developmentSpeed: {
            react: 'Moderate (requires setup decisions)',
            vue: 'Fast (conventions and tooling)',
            angular: 'Fast (comprehensive CLI and features)'
        }
    };
    
    return metrics;
}

// Migration strategies
const migrationStrategies = {
    // From jQuery to modern framework
    jqueryToReact: {
        steps: [
            '1. Identify DOM manipulation patterns',
            '2. Convert jQuery selectors to React refs',
            '3. Replace event handlers with React events',
            '4. Convert animations to CSS or React Transition Group',
            '5. Migrate AJAX calls to fetch/axios with hooks'
        ],
        example: `
            // jQuery
            $('#button').click(function() {
                $('#content').slideDown();
            });
            
            // React
            function Component() {
                const [visible, setVisible] = useState(false);
                return (
                    <div>
                        <button onClick={() => setVisible(!visible)}>Toggle</button>
                        <div className={visible ? 'visible' : 'hidden'}>Content</div>
                    </div>
                );
            }
        `
    },
    
    // Between modern frameworks
    reactToVue: {
        steps: [
            '1. Convert JSX to Vue templates',
            '2. Replace React hooks with Vue Composition API',
            '3. Convert React Router to Vue Router',
            '4. Migrate state management (Redux to Vuex/Pinia)',
            '5. Update build tools and configuration'
        ]
    },
    
    // Legacy to modern
    legacyToModern: {
        approaches: [
            'Big Bang: Complete rewrite',
            'Strangler Fig: Gradual replacement',
            'Micro-frontends: Modular migration',
            'Component-by-component: Incremental updates'
        ]
    }
};

// Framework-agnostic patterns
const universalPatterns = {
    // Component composition
    composition: {
        // Higher-order components (React)
        reactHOC: `
            function withLoading(WrappedComponent) {
                return function WithLoadingComponent(props) {
                    if (props.loading) {
                        return <div>Loading...</div>;
                    }
                    return <WrappedComponent {...props} />;
                };
            }
        `,
        
        // Mixins (Vue)
        vueMixin: `
            const loadingMixin = {
                data() {
                    return { loading: false };
                },
                methods: {
                    async fetchData() {
                        this.loading = true;
                        // fetch logic
                        this.loading = false;
                    }
                }
            };
        `,
        
        // Services (Angular)
        angularService: `
            @Injectable()
            export class LoadingService {
                private loading = new BehaviorSubject(false);
                
                setLoading(loading: boolean) {
                    this.loading.next(loading);
                }
                
                getLoading() {
                    return this.loading.asObservable();
                }
            }
        `
    },
    
    // State management patterns
    stateManagement: {
        flux: 'Unidirectional data flow',
        redux: 'Predictable state container',
        mobx: 'Reactive state management',
        vuex: 'Centralized state for Vue',
        ngrx: 'Reactive state for Angular'
    },
    
    // Testing strategies
    testing: {
        unit: 'Component behavior testing',
        integration: 'Component interaction testing',
        e2e: 'Full application workflow testing',
        snapshot: 'Render output comparison',
        accessibility: 'ARIA and keyboard navigation'
    }
};
```

---

## Related Topics
- [[33 - JavaScript OOP]]
- [[37 - ES6+ Features]]
- [[38 - Modules]]
- [[31 - jQuery Basics]]
- [[42 - Error Handling]]

---

*Next: [[33 - JavaScript OOP]]*
