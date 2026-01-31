# Setup & Installation Guide

## 🎯 Development Environment Setup

Let's get your machine ready for PostgreSQL + Sequelize development!

## 📋 System Requirements

| Component | Minimum Version | Recommended |
|-----------|----------------|-------------|
| Node.js | 14.x | 18.x or later |
| NPM | 6.x | 8.x or later |
| PostgreSQL | 12.x | 14.x or later |
| RAM | 4GB | 8GB+ |
| Storage | 5GB free | 10GB+ free |

## 🐘 PostgreSQL Installation

### Windows Installation

#### Method 1: Official Installer (Recommended)
1. **Download PostgreSQL**
   ```
   Visit: https://www.postgresql.org/download/windows/
   Download: Latest version installer
   ```

2. **Run Installer**
   - Choose installation directory: `C:\Program Files\PostgreSQL\14`
   - Select components: PostgreSQL Server, pgAdmin 4, Command Line Tools
   - Set data directory: `C:\Program Files\PostgreSQL\14\data`
   - Set password for `postgres` user (remember this!)
   - Port: `5432` (default)
   - Locale: Default

3. **Verify Installation**
   ```powershell
   # Open Command Prompt/PowerShell
   psql --version
   # Should output: psql (PostgreSQL) 14.x
   ```

#### Method 2: Using Chocolatey
```powershell
# Install Chocolatey first (if not installed)
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Install PostgreSQL
choco install postgresql
```

### macOS Installation

#### Method 1: Homebrew (Recommended)
```bash
# Install Homebrew (if not installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install PostgreSQL
brew install postgresql
brew services start postgresql
```

#### Method 2: Postgres.app
1. Download from: https://postgresapp.com/
2. Drag to Applications folder
3. Open Postgres.app
4. Click "Initialize" to create a new server

### Linux (Ubuntu/Debian) Installation

```bash
# Update package list
sudo apt update

# Install PostgreSQL
sudo apt install postgresql postgresql-contrib

# Start PostgreSQL service
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Set password for postgres user
sudo -u postgres psql
\password postgres
\q
```

## 🔧 PostgreSQL Configuration

### 1. Create Development Database

```sql
-- Connect to PostgreSQL
psql -U postgres -h localhost

-- Create database for our project
CREATE DATABASE blog_app_dev;
CREATE DATABASE blog_app_test;

-- Create a development user
CREATE USER blog_user WITH ENCRYPTED PASSWORD 'your_password';

-- Grant privileges
GRANT ALL PRIVILEGES ON DATABASE blog_app_dev TO blog_user;
GRANT ALL PRIVILEGES ON DATABASE blog_app_test TO blog_user;

-- Exit
\q
```

### 2. Test Connection

```powershell
# Test connection with new user
psql -U blog_user -d blog_app_dev -h localhost

# If successful, you'll see:
# blog_app_dev=>
```

## 📦 Node.js Project Setup

### 1. Create Project Directory

```powershell
# Create and navigate to project folder
mkdir blog-sequelize-app
cd blog-sequelize-app

# Initialize package.json
npm init -y
```

### 2. Install Dependencies

```powershell
# Core dependencies
npm install express sequelize pg pg-hstore

# Development dependencies
npm install --save-dev nodemon sequelize-cli

# Additional useful packages
npm install cors helmet dotenv bcryptjs jsonwebtoken
```

### 3. Package.json Overview

```json
{
  "name": "blog-sequelize-app",
  "version": "1.0.0",
  "description": "Blog app with PostgreSQL and Sequelize",
  "main": "server.js",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js",
    "db:create": "sequelize-cli db:create",
    "db:migrate": "sequelize-cli db:migrate",
    "db:seed": "sequelize-cli db:seed:all",
    "db:reset": "sequelize-cli db:drop && sequelize-cli db:create && sequelize-cli db:migrate && sequelize-cli db:seed:all"
  },
  "dependencies": {
    "express": "^4.18.2",
    "sequelize": "^6.28.0",
    "pg": "^8.8.0",
    "pg-hstore": "^2.3.4",
    "cors": "^2.8.5",
    "helmet": "^6.0.1",
    "dotenv": "^16.0.3",
    "bcryptjs": "^2.4.3",
    "jsonwebtoken": "^9.0.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.20",
    "sequelize-cli": "^6.5.2"
  }
}
```

## 🏗️ Project Structure

```
blog-sequelize-app/
├── config/
│   ├── database.js          # Database configuration
│   └── config.json          # Sequelize CLI config
├── controllers/             # Route handlers
├── middleware/              # Custom middleware
├── migrations/              # Database migrations
├── models/                  # Sequelize models
│   └── index.js            # Models index file
├── routes/                  # API routes
├── seeders/                # Database seeders
├── utils/                  # Utility functions
├── .env                    # Environment variables
├── .gitignore             # Git ignore file
├── server.js              # Main application file
└── package.json           # Project dependencies
```

### Create Basic Structure

```powershell
# Create directories
mkdir config, controllers, middleware, migrations, models, routes, seeders, utils

# Create essential files
New-Item -ItemType File -Path ".env"
New-Item -ItemType File -Path ".gitignore"
New-Item -ItemType File -Path "server.js"
```

## ⚙️ Environment Configuration

### 1. Environment Variables (.env)

```bash
# Database Configuration
DB_HOST=localhost
DB_PORT=5432
DB_NAME=blog_app_dev
DB_USER=blog_user
DB_PASSWORD=your_password
DB_DIALECT=postgres

# Test Database
TEST_DB_NAME=blog_app_test

# Server Configuration
PORT=3000
NODE_ENV=development

# JWT Secret
JWT_SECRET=your_super_secret_jwt_key_here

# Other configs
BCRYPT_ROUNDS=12
```

### 2. Database Config (config/database.js)

```javascript
require('dotenv').config();

module.exports = {
  development: {
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    dialect: process.env.DB_DIALECT,
    logging: console.log, // Enable SQL logging in development
    pool: {
      max: 5,
      min: 0,
      acquire: 30000,
      idle: 10000
    }
  },
  test: {
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.TEST_DB_NAME,
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    dialect: process.env.DB_DIALECT,
    logging: false // Disable logging in tests
  },
  production: {
    username: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
    database: process.env.DB_NAME,
    host: process.env.DB_HOST,
    port: process.env.DB_PORT,
    dialect: process.env.DB_DIALECT,
    logging: false,
    pool: {
      max: 20,
      min: 5,
      acquire: 30000,
      idle: 10000
    }
  }
};
```

### 3. Sequelize CLI Config (config/config.json)

```json
{
  "development": {
    "username": "blog_user",
    "password": "your_password",
    "database": "blog_app_dev",
    "host": "localhost",
    "port": 5432,
    "dialect": "postgres"
  },
  "test": {
    "username": "blog_user",
    "password": "your_password",
    "database": "blog_app_test",
    "host": "localhost",
    "port": 5432,
    "dialect": "postgres"
  },
  "production": {
    "use_env_variable": "DATABASE_URL",
    "dialect": "postgres",
    "dialectOptions": {
      "ssl": {
        "require": true,
        "rejectUnauthorized": false
      }
    }
  }
}
```

## 🚀 Basic Server Setup

### Create server.js

```javascript
require('dotenv').config();
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');

// Import database
const db = require('./models');

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(helmet());
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Basic route
app.get('/', (req, res) => {
  res.json({ 
    message: 'Welcome to Blog API with PostgreSQL + Sequelize!',
    timestamp: new Date().toISOString(),
    environment: process.env.NODE_ENV
  });
});

// Health check route
app.get('/health', async (req, res) => {
  try {
    // Test database connection
    await db.sequelize.authenticate();
    res.json({ 
      status: 'OK', 
      database: 'Connected',
      timestamp: new Date().toISOString()
    });
  } catch (error) {
    res.status(500).json({ 
      status: 'Error', 
      database: 'Disconnected',
      error: error.message 
    });
  }
});

// Error handling middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ 
    message: 'Something went wrong!',
    error: process.env.NODE_ENV === 'development' ? err.message : 'Internal Server Error'
  });
});

// 404 handler
app.use((req, res) => {
  res.status(404).json({ message: 'Route not found' });
});

// Start server
const startServer = async () => {
  try {
    // Test database connection
    await db.sequelize.authenticate();
    console.log('✅ Database connection established successfully.');
    
    // Sync database (in development)
    if (process.env.NODE_ENV === 'development') {
      await db.sequelize.sync({ alter: true });
      console.log('✅ Database synchronized successfully.');
    }
    
    app.listen(PORT, () => {
      console.log(`🚀 Server running on port ${PORT}`);
      console.log(`📝 Environment: ${process.env.NODE_ENV}`);
      console.log(`🗄️  Database: ${process.env.DB_NAME}`);
    });
  } catch (error) {
    console.error('❌ Unable to start server:', error);
    process.exit(1);
  }
};

startServer();

module.exports = app;
```

## ✅ Testing Your Setup

### 1. Initialize Sequelize

```powershell
# Initialize Sequelize in your project
npx sequelize-cli init
```

This creates:
- `config/config.json`
- `models/index.js`
- `migrations/` folder
- `seeders/` folder

### 2. Test Database Connection

```powershell
# Start your server
npm run dev

# Visit in browser or use curl
curl http://localhost:3000/health
```

Expected response:
```json
{
  "status": "OK",
  "database": "Connected",
  "timestamp": "2025-01-06T10:30:00.000Z"
}
```

### 3. Verify PostgreSQL Connection

```sql
-- Connect to your database
psql -U blog_user -d blog_app_dev -h localhost

-- List all tables (should be empty for now)
\dt

-- Check connection info
\conninfo
```

## 🔧 Development Tools

### 1. pgAdmin 4 (GUI Tool)
- **URL**: http://localhost:5050 (if installed)
- **Default email**: admin@admin.com
- **Password**: Set during installation

### 2. VS Code Extensions
```
Recommended extensions:
- PostgreSQL by Chris Kolkman
- SQL Tools by Matheus Teixeira
- Sequelize Snippets
- Thunder Client (API testing)
```

### 3. Useful Commands

```powershell
# Development workflow
npm run dev          # Start server with nodemon
npm run db:create    # Create database
npm run db:migrate   # Run migrations
npm run db:seed      # Run seeders
npm run db:reset     # Reset entire database

# PostgreSQL commands
psql -U postgres     # Connect as superuser
\l                   # List databases
\dt                  # List tables
\d table_name        # Describe table
\q                   # Quit
```

## 🎯 Verification Checklist

- [ ] PostgreSQL installed and running
- [ ] Node.js project created with dependencies
- [ ] Database created and accessible
- [ ] Environment variables configured
- [ ] Server starts without errors
- [ ] Health check endpoint returns success
- [ ] Sequelize CLI working

## 🚨 Troubleshooting

### Common Issues

#### 1. PostgreSQL Connection Refused
```bash
# Check if PostgreSQL is running
# Windows:
Get-Service postgresql*

# Start service if stopped
Start-Service postgresql-x64-14
```

#### 2. Authentication Failed
```sql
-- Reset password for postgres user
sudo -u postgres psql
\password postgres
```

#### 3. Port Already in Use
```bash
# Find process using port 3000
netstat -ano | findstr :3000

# Kill process (replace PID)
taskkill /PID <PID> /F
```

#### 4. Sequelize CLI Commands Not Working
```powershell
# Install globally
npm install -g sequelize-cli

# Or use npx
npx sequelize-cli --help
```

## 🎉 Next Steps

Great! Your environment is ready. Now let's dive into [[03-Database-Basics|Database Fundamentals]] to understand how PostgreSQL works before we start building with Sequelize!

---

## 🔗 Related Topics
- [[03-Database-Basics|Database Fundamentals]]
- [[04-Sequelize-Introduction|Sequelize Introduction]]
- [[22-Troubleshooting|Troubleshooting Guide]]
