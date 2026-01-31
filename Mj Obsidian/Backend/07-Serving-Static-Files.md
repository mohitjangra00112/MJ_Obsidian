# Serving Static Files

**Navigation**: [[06-Express-Router]] | [[00-Main-Index]] | Next: [[08-Headers-and-Cookies]]

---

## 📁 Serving Static Files in Express

Static files are assets like **images, CSS, JavaScript files, documents** that don't change dynamically. Express provides built-in middleware to serve these files efficiently.

---

## 🚀 Basic Static File Serving

### Using express.static()
```javascript
const express = require('express');
const path = require('path');
const app = express();

// Serve static files from 'public' directory
app.use(express.static('public'));

// Alternative with absolute path
app.use(express.static(path.join(__dirname, 'public')));

// Basic route
app.get('/', (req, res) => {
    res.send(`
        <h1>Static File Server</h1>
        <p><a href="/style.css">View CSS</a></p>
        <p><a href="/script.js">View JavaScript</a></p>
        <p><a href="/image.jpg">View Image</a></p>
    `);
});

app.listen(3000, () => {
    console.log('Static file server running on port 3000');
});
```

### Directory Structure
```
project/
├── app.js
├── public/
│   ├── css/
│   │   ├── style.css
│   │   └── bootstrap.css
│   ├── js/
│   │   ├── script.js
│   │   └── app.js
│   ├── images/
│   │   ├── logo.png
│   │   └── banner.jpg
│   └── documents/
│       ├── manual.pdf
│       └── terms.txt
└── views/
    └── index.html
```

---

## 🎯 Virtual Path Prefix

### Adding URL Prefix
```javascript
const express = require('express');
const path = require('path');
const app = express();

// Serve static files with virtual prefix
app.use('/static', express.static('public'));
app.use('/assets', express.static('assets'));
app.use('/uploads', express.static('uploads'));

// Multiple static directories
app.use('/css', express.static('public/css'));
app.use('/js', express.static('public/js'));
app.use('/images', express.static('public/images'));

app.get('/', (req, res) => {
    res.send(`
        <h1>Static Files with Prefixes</h1>
        <link rel="stylesheet" href="/css/style.css">
        <script src="/js/app.js"></script>
        <img src="/images/logo.png" alt="Logo">
    `);
});

app.listen(3000);
```

### Accessing Files with Prefix
```
Without prefix: http://localhost:3000/style.css
With prefix:    http://localhost:3000/static/style.css
```

---

## 🔧 Advanced Static File Configuration

### Static Middleware Options
```javascript
const express = require('express');
const path = require('path');
const app = express();

// Advanced static file configuration
app.use('/static', express.static('public', {
    // Set Cache-Control header
    maxAge: '1d', // 1 day cache
    
    // Set custom headers
    setHeaders: (res, path, stat) => {
        res.set('X-Custom-Header', 'MyApp');
        
        // Different cache for different file types
        if (path.endsWith('.js') || path.endsWith('.css')) {
            res.set('Cache-Control', 'public, max-age=31536000'); // 1 year
        } else if (path.endsWith('.html')) {
            res.set('Cache-Control', 'public, max-age=3600'); // 1 hour
        }
    },
    
    // Enable/disable directory listing
    index: false, // Don't serve index.html automatically
    
    // Enable/disable dotfiles
    dotfiles: 'deny', // deny, allow, ignore
    
    // Redirect trailing slash
    redirect: false,
    
    // Enable ETag
    etag: true,
    
    // Enable last-modified header
    lastModified: true
}));

app.listen(3000);
```

---

## 🖼️ Complete Static File Server

### Advanced Static Server
```javascript
const express = require('express');
const path = require('path');
const fs = require('fs');
const app = express();

// Custom middleware for static files with logging
const staticWithLogging = (req, res, next) => {
    const filePath = path.join(__dirname, 'public', req.path);
    
    // Check if file exists
    fs.access(filePath, fs.constants.F_OK, (err) => {
        if (err) {
            console.log(`File not found: ${req.path}`);
        } else {
            console.log(`Serving: ${req.path}`);
        }
    });
    
    next();
};

// Apply logging middleware before static middleware
app.use('/public', staticWithLogging);
app.use('/public', express.static('public', {
    maxAge: '1h',
    setHeaders: (res, path) => {
        res.set('X-Served-By', 'Express Static');
    }
}));

// Serve different types of static content
app.use('/css', express.static('public/css', { maxAge: '1d' }));
app.use('/js', express.static('public/js', { maxAge: '1d' }));
app.use('/images', express.static('public/images', { maxAge: '7d' }));
app.use('/downloads', express.static('public/downloads', { maxAge: '1h' }));

// Custom route for file download
app.get('/download/:filename', (req, res) => {
    const filename = req.params.filename;
    const filePath = path.join(__dirname, 'public/downloads', filename);
    
    // Check if file exists
    fs.access(filePath, fs.constants.F_OK, (err) => {
        if (err) {
            return res.status(404).json({ error: 'File not found' });
        }
        
        // Set download headers
        res.setHeader('Content-Disposition', `attachment; filename="${filename}"`);
        res.setHeader('Content-Type', 'application/octet-stream');
        
        // Send file
        res.download(filePath, filename, (err) => {
            if (err) {
                console.error('Download error:', err);
                res.status(500).json({ error: 'Download failed' });
            }
        });
    });
});

// Serve SPA (Single Page Application)
app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

app.listen(3000, () => {
    console.log('Advanced static server running on port 3000');
});
```

---

## 📷 Image Serving and Processing

### Image Server with Basic Processing
```javascript
const express = require('express');
const path = require('path');
const fs = require('fs');
const app = express();

// Serve original images
app.use('/images', express.static('public/images', {
    maxAge: '30d',
    setHeaders: (res, path) => {
        // Set appropriate content type for images
        if (path.endsWith('.jpg') || path.endsWith('.jpeg')) {
            res.set('Content-Type', 'image/jpeg');
        } else if (path.endsWith('.png')) {
            res.set('Content-Type', 'image/png');
        } else if (path.endsWith('.gif')) {
            res.set('Content-Type', 'image/gif');
        } else if (path.endsWith('.webp')) {
            res.set('Content-Type', 'image/webp');
        }
    }
}));

// Dynamic image resizing (simplified example)
app.get('/images/resize/:width/:height/:filename', (req, res) => {
    const { width, height, filename } = req.params;
    const imagePath = path.join(__dirname, 'public/images', filename);
    
    // Check if original image exists
    fs.access(imagePath, fs.constants.F_OK, (err) => {
        if (err) {
            return res.status(404).json({ error: 'Image not found' });
        }
        
        // In a real application, you would use a library like Sharp
        // For now, just serve the original image
        res.sendFile(imagePath);
    });
});

// Image upload endpoint
app.post('/upload/image', (req, res) => {
    // Implementation would use multer middleware
    res.json({ message: 'Image upload endpoint' });
});

app.listen(3000);
```

---

## 📊 File Management API

### Complete File Management System
```javascript
const express = require('express');
const path = require('path');
const fs = require('fs').promises;
const multer = require('multer');
const app = express();

app.use(express.json());

// Configure multer for file uploads
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        cb(null, 'uploads/');
    },
    filename: (req, file, cb) => {
        const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
        cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
    }
});

const upload = multer({ 
    storage: storage,
    limits: {
        fileSize: 10 * 1024 * 1024 // 10MB limit
    },
    fileFilter: (req, file, cb) => {
        // Allow only certain file types
        const allowedTypes = /jpeg|jpg|png|gif|pdf|doc|docx/;
        const extname = allowedTypes.test(path.extname(file.originalname).toLowerCase());
        const mimetype = allowedTypes.test(file.mimetype);
        
        if (mimetype && extname) {
            return cb(null, true);
        } else {
            cb(new Error('Invalid file type'));
        }
    }
});

// Serve uploaded files
app.use('/uploads', express.static('uploads', {
    maxAge: '1d',
    setHeaders: (res, path, stat) => {
        res.set('X-File-Size', stat.size);
        res.set('X-Upload-Time', stat.mtime);
    }
}));

// File upload endpoint
app.post('/upload', upload.single('file'), (req, res) => {
    if (!req.file) {
        return res.status(400).json({ error: 'No file uploaded' });
    }
    
    res.json({
        message: 'File uploaded successfully',
        file: {
            filename: req.file.filename,
            originalname: req.file.originalname,
            size: req.file.size,
            path: req.file.path,
            url: `/uploads/${req.file.filename}`
        }
    });
});

// Multiple file upload
app.post('/upload-multiple', upload.array('files', 5), (req, res) => {
    if (!req.files || req.files.length === 0) {
        return res.status(400).json({ error: 'No files uploaded' });
    }
    
    const files = req.files.map(file => ({
        filename: file.filename,
        originalname: file.originalname,
        size: file.size,
        url: `/uploads/${file.filename}`
    }));
    
    res.json({
        message: 'Files uploaded successfully',
        files: files
    });
});

// List files endpoint
app.get('/files', async (req, res) => {
    try {
        const files = await fs.readdir('uploads');
        const fileDetails = await Promise.all(
            files.map(async (filename) => {
                const filePath = path.join('uploads', filename);
                const stats = await fs.stat(filePath);
                return {
                    filename,
                    size: stats.size,
                    created: stats.birthtime,
                    modified: stats.mtime,
                    url: `/uploads/${filename}`
                };
            })
        );
        
        res.json({ files: fileDetails });
    } catch (error) {
        res.status(500).json({ error: 'Unable to list files' });
    }
});

// Delete file endpoint
app.delete('/files/:filename', async (req, res) => {
    try {
        const filename = req.params.filename;
        const filePath = path.join('uploads', filename);
        
        await fs.unlink(filePath);
        res.json({ message: 'File deleted successfully' });
    } catch (error) {
        if (error.code === 'ENOENT') {
            res.status(404).json({ error: 'File not found' });
        } else {
            res.status(500).json({ error: 'Unable to delete file' });
        }
    }
});

// File info endpoint
app.get('/files/:filename/info', async (req, res) => {
    try {
        const filename = req.params.filename;
        const filePath = path.join('uploads', filename);
        const stats = await fs.stat(filePath);
        
        res.json({
            filename,
            size: stats.size,
            created: stats.birthtime,
            modified: stats.mtime,
            isFile: stats.isFile(),
            isDirectory: stats.isDirectory(),
            url: `/uploads/${filename}`
        });
    } catch (error) {
        if (error.code === 'ENOENT') {
            res.status(404).json({ error: 'File not found' });
        } else {
            res.status(500).json({ error: 'Unable to get file info' });
        }
    }
});

app.listen(3000, () => {
    console.log('File management server running on port 3000');
});
```

---

## 🔒 Security Considerations

### Secure Static File Serving
```javascript
const express = require('express');
const path = require('path');
const rateLimit = require('express-rate-limit');
const app = express();

// Rate limiting for static files
const staticLimiter = rateLimit({
    windowMs: 15 * 60 * 1000, // 15 minutes
    max: 1000, // limit each IP to 1000 requests per windowMs
    message: 'Too many requests for static files'
});

// Security middleware for static files
const secureStatic = (req, res, next) => {
    // Prevent directory traversal
    if (req.path.includes('..')) {
        return res.status(403).json({ error: 'Access denied' });
    }
    
    // Block access to sensitive files
    const sensitiveFiles = ['.env', '.git', 'package.json', 'server.js'];
    const filename = path.basename(req.path);
    
    if (sensitiveFiles.includes(filename)) {
        return res.status(403).json({ error: 'Access denied' });
    }
    
    next();
};

// Apply security and rate limiting
app.use('/public', staticLimiter);
app.use('/public', secureStatic);
app.use('/public', express.static('public', {
    dotfiles: 'deny', // Deny access to dotfiles
    maxAge: '1d',
    setHeaders: (res, path) => {
        // Security headers
        res.set('X-Content-Type-Options', 'nosniff');
        res.set('X-Frame-Options', 'DENY');
        res.set('X-XSS-Protection', '1; mode=block');
    }
}));

app.listen(3000);
```

---

## 📱 SPA (Single Page Application) Support

### SPA with Fallback Routing
```javascript
const express = require('express');
const path = require('path');
const app = express();

// Serve static assets
app.use('/static', express.static('build/static', {
    maxAge: '1y' // Long cache for hashed assets
}));

// Serve other static files
app.use(express.static('build', {
    maxAge: '1h' // Shorter cache for HTML
}));

// API routes
app.use('/api', require('./routes/api'));

// SPA fallback - serve index.html for all non-API routes
app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, 'build/index.html'));
});

app.listen(3000);
```

---

## 🔗 Related Topics
- [[05-Express-Framework]] - Basic Express setup
- [[06-Express-Router]] - Organizing routes
- [[08-Headers-and-Cookies]] - Working with HTTP headers
- [[21-Error-Handling]] - Handling file serving errors

---

*Back to [[00-Main-Index]]*
