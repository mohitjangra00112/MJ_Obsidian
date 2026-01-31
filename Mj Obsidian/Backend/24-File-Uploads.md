# File Uploads

**Navigation**: [[23-Cookies-Sessions]] | [[00-Main-Index]] | Next: [[25-Security-Best-Practices]]

---

## 📁 File Upload Handling in Node.js

**File uploads** are essential for modern web applications. This guide covers secure file handling, image processing, cloud storage integration, and best practices for handling file uploads in Node.js.

---

## 🔧 Multer Configuration

### Basic Multer Setup
```javascript
// middleware/upload.js
const multer = require('multer');
const path = require('path');
const fs = require('fs');
const crypto = require('crypto');

// Ensure upload directories exist
const ensureDir = (dir) => {
    if (!fs.existsSync(dir)) {
        fs.mkdirSync(dir, { recursive: true });
    }
};

// Storage configuration
const storage = multer.diskStorage({
    destination: (req, file, cb) => {
        let uploadPath = 'uploads/';
        
        // Organize by file type and date
        if (file.mimetype.startsWith('image/')) {
            uploadPath += 'images/';
        } else if (file.mimetype.startsWith('video/')) {
            uploadPath += 'videos/';
        } else if (file.mimetype === 'application/pdf') {
            uploadPath += 'documents/';
        } else {
            uploadPath += 'others/';
        }
        
        // Add date-based directory structure
        const date = new Date();
        const year = date.getFullYear();
        const month = String(date.getMonth() + 1).padStart(2, '0');
        uploadPath += `${year}/${month}/`;
        
        ensureDir(uploadPath);
        cb(null, uploadPath);
    },
    filename: (req, file, cb) => {
        // Generate secure filename
        const uniqueSuffix = Date.now() + '-' + crypto.randomUUID();
        const extension = path.extname(file.originalname);
        const baseName = path.basename(file.originalname, extension)
            .replace(/[^a-zA-Z0-9]/g, '_')
            .substring(0, 50); // Limit length
        
        const filename = `${baseName}_${uniqueSuffix}${extension}`;
        cb(null, filename);
    }
});

// File filter for security
const fileFilter = (allowedTypes, maxSize = 10 * 1024 * 1024) => {
    return (req, file, cb) => {
        // Check file type
        const isAllowedType = allowedTypes.some(type => {
            if (type.startsWith('.')) {
                return path.extname(file.originalname).toLowerCase() === type.toLowerCase();
            } else {
                return file.mimetype.startsWith(type);
            }
        });
        
        if (!isAllowedType) {
            return cb(new Error(`File type not allowed. Allowed: ${allowedTypes.join(', ')}`), false);
        }
        
        // Additional security checks
        if (file.originalname.includes('..')) {
            return cb(new Error('Invalid filename'), false);
        }
        
        cb(null, true);
    };
};

// Upload configurations
const uploadConfigs = {
    // Single image upload
    singleImage: multer({
        storage,
        fileFilter: fileFilter(['image/jpeg', 'image/png', 'image/webp']),
        limits: {
            fileSize: 5 * 1024 * 1024, // 5MB
            files: 1
        }
    }).single('image'),
    
    // Multiple images
    multipleImages: multer({
        storage,
        fileFilter: fileFilter(['image/']),
        limits: {
            fileSize: 5 * 1024 * 1024,
            files: 10
        }
    }).array('images', 10),
    
    // Profile picture
    avatar: multer({
        storage,
        fileFilter: fileFilter(['image/jpeg', 'image/png', 'image/webp']),
        limits: {
            fileSize: 2 * 1024 * 1024 // 2MB
        }
    }).single('avatar'),
    
    // Documents
    documents: multer({
        storage,
        fileFilter: fileFilter(['.pdf', '.doc', '.docx', '.txt']),
        limits: {
            fileSize: 10 * 1024 * 1024, // 10MB
            files: 5
        }
    }).array('documents', 5),
    
    // Any file type
    any: multer({
        storage,
        limits: {
            fileSize: 50 * 1024 * 1024 // 50MB
        }
    })
};

module.exports = {
    uploadConfigs,
    ensureDir
};
```

---

## 🖼️ Image Processing

### Sharp Image Processing
```javascript
// utils/imageProcessor.js
const sharp = require('sharp');
const path = require('path');
const fs = require('fs').promises;

class ImageProcessor {
    constructor() {
        this.supportedFormats = ['jpeg', 'png', 'webp', 'avif'];
        this.defaultQuality = 80;
    }
    
    async processImage(inputPath, options = {}) {
        const {
            resize = null,
            quality = this.defaultQuality,
            format = 'jpeg',
            generateThumbnail = true,
            thumbnailSize = { width: 300, height: 300 },
            generateWebP = true,
            watermark = null
        } = options;
        
        try {
            const inputDir = path.dirname(inputPath);
            const inputBasename = path.basename(inputPath, path.extname(inputPath));
            
            const results = {
                original: inputPath,
                processed: [],
                thumbnails: []
            };
            
            // Load image
            let image = sharp(inputPath);
            const metadata = await image.metadata();
            
            // Resize if needed
            if (resize) {
                image = image.resize(resize.width, resize.height, {
                    fit: resize.fit || 'inside',
                    withoutEnlargement: true
                });
            }
            
            // Add watermark if provided
            if (watermark) {
                const watermarkBuffer = await this.createWatermark(watermark);
                image = image.composite([{
                    input: watermarkBuffer,
                    gravity: 'southeast'
                }]);
            }
            
            // Generate main processed image
            const processedPath = path.join(inputDir, `${inputBasename}_processed.${format}`);
            await image
                .toFormat(format, { quality })
                .toFile(processedPath);
            
            results.processed.push({
                path: processedPath,
                format,
                quality
            });
            
            // Generate WebP version
            if (generateWebP && format !== 'webp') {
                const webpPath = path.join(inputDir, `${inputBasename}_processed.webp`);
                await image
                    .toFormat('webp', { quality })
                    .toFile(webpPath);
                
                results.processed.push({
                    path: webpPath,
                    format: 'webp',
                    quality
                });
            }
            
            // Generate thumbnail
            if (generateThumbnail) {
                const thumbnailPath = path.join(inputDir, `${inputBasename}_thumb.${format}`);
                await sharp(inputPath)
                    .resize(thumbnailSize.width, thumbnailSize.height, {
                        fit: 'cover',
                        position: 'centre'
                    })
                    .toFormat(format, { quality: Math.max(quality - 10, 60) })
                    .toFile(thumbnailPath);
                
                results.thumbnails.push({
                    path: thumbnailPath,
                    width: thumbnailSize.width,
                    height: thumbnailSize.height
                });
                
                // WebP thumbnail
                if (generateWebP) {
                    const webpThumbPath = path.join(inputDir, `${inputBasename}_thumb.webp`);
                    await sharp(inputPath)
                        .resize(thumbnailSize.width, thumbnailSize.height, {
                            fit: 'cover',
                            position: 'centre'
                        })
                        .toFormat('webp', { quality: Math.max(quality - 10, 60) })
                        .toFile(webpThumbPath);
                    
                    results.thumbnails.push({
                        path: webpThumbPath,
                        width: thumbnailSize.width,
                        height: thumbnailSize.height
                    });
                }
            }
            
            // Generate responsive images
            const responsiveSizes = [480, 768, 1024, 1440];
            results.responsive = [];
            
            for (const size of responsiveSizes) {
                if (size < metadata.width) {
                    const responsivePath = path.join(inputDir, `${inputBasename}_${size}w.${format}`);
                    await sharp(inputPath)
                        .resize(size, null, {
                            withoutEnlargement: true
                        })
                        .toFormat(format, { quality })
                        .toFile(responsivePath);
                    
                    results.responsive.push({
                        path: responsivePath,
                        width: size
                    });
                }
            }
            
            // Remove original if processed successfully
            if (results.processed.length > 0) {
                await fs.unlink(inputPath);
            }
            
            return results;
            
        } catch (error) {
            throw new Error(`Image processing failed: ${error.message}`);
        }
    }
    
    async createWatermark(options) {
        const { text, fontSize = 24, opacity = 0.5 } = options;
        
        const svg = `
            <svg width="200" height="50">
                <text x="10" y="30" font-family="Arial" font-size="${fontSize}" 
                      fill="white" fill-opacity="${opacity}">${text}</text>
            </svg>
        `;
        
        return Buffer.from(svg);
    }
    
    async generateImageMetadata(imagePath) {
        try {
            const metadata = await sharp(imagePath).metadata();
            const stats = await fs.stat(imagePath);
            
            return {
                width: metadata.width,
                height: metadata.height,
                format: metadata.format,
                size: stats.size,
                density: metadata.density,
                hasAlpha: metadata.hasAlpha,
                orientation: metadata.orientation
            };
        } catch (error) {
            throw new Error(`Failed to get image metadata: ${error.message}`);
        }
    }
    
    async createImageVariants(inputPath, variants) {
        const results = [];
        
        for (const variant of variants) {
            const outputPath = variant.outputPath || 
                inputPath.replace(/\.[^.]+$/, `_${variant.name}$&`);
            
            let image = sharp(inputPath);
            
            if (variant.resize) {
                image = image.resize(variant.resize.width, variant.resize.height, {
                    fit: variant.resize.fit || 'cover'
                });
            }
            
            if (variant.format) {
                image = image.toFormat(variant.format, {
                    quality: variant.quality || this.defaultQuality
                });
            }
            
            await image.toFile(outputPath);
            
            results.push({
                name: variant.name,
                path: outputPath,
                ...variant
            });
        }
        
        return results;
    }
}

module.exports = new ImageProcessor();
```

---

## ☁️ Cloud Storage Integration

### AWS S3 Integration
```javascript
// services/CloudStorageService.js
const AWS = require('aws-sdk');
const fs = require('fs');
const path = require('path');
const { v4: uuidv4 } = require('uuid');

// Configure AWS
AWS.config.update({
    accessKeyId: process.env.AWS_ACCESS_KEY_ID,
    secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY,
    region: process.env.AWS_REGION || 'us-east-1'
});

const s3 = new AWS.S3();

class CloudStorageService {
    constructor() {
        this.bucket = process.env.AWS_S3_BUCKET;
        this.cloudFrontUrl = process.env.CLOUDFRONT_URL;
    }
    
    async uploadFile(file, options = {}) {
        const {
            folder = 'uploads',
            makePublic = true,
            generateThumbnail = false,
            contentType = file.mimetype,
            metadata = {}
        } = options;
        
        try {
            // Generate unique filename
            const fileExtension = path.extname(file.originalname);
            const fileName = `${folder}/${Date.now()}-${uuidv4()}${fileExtension}`;
            
            // Upload parameters
            const uploadParams = {
                Bucket: this.bucket,
                Key: fileName,
                Body: fs.createReadStream(file.path),
                ContentType: contentType,
                Metadata: {
                    originalName: file.originalname,
                    uploadedAt: new Date().toISOString(),
                    ...metadata
                }
            };
            
            if (makePublic) {
                uploadParams.ACL = 'public-read';
            }
            
            // Upload to S3
            const result = await s3.upload(uploadParams).promise();
            
            // Generate signed URL if not public
            let url = result.Location;
            if (!makePublic) {
                url = s3.getSignedUrl('getObject', {
                    Bucket: this.bucket,
                    Key: fileName,
                    Expires: 3600 // 1 hour
                });
            } else if (this.cloudFrontUrl) {
                url = `${this.cloudFrontUrl}/${fileName}`;
            }
            
            // Clean up local file
            fs.unlinkSync(file.path);
            
            return {
                key: fileName,
                url,
                bucket: this.bucket,
                size: file.size,
                mimetype: contentType,
                originalName: file.originalname
            };
            
        } catch (error) {
            // Clean up local file on error
            if (fs.existsSync(file.path)) {
                fs.unlinkSync(file.path);
            }
            throw new Error(`Upload failed: ${error.message}`);
        }
    }
    
    async uploadMultipleFiles(files, options = {}) {
        const results = [];
        const errors = [];
        
        for (const file of files) {
            try {
                const result = await this.uploadFile(file, options);
                results.push(result);
            } catch (error) {
                errors.push({
                    file: file.originalname,
                    error: error.message
                });
            }
        }
        
        return { results, errors };
    }
    
    async deleteFile(key) {
        try {
            const deleteParams = {
                Bucket: this.bucket,
                Key: key
            };
            
            await s3.deleteObject(deleteParams).promise();
            return { success: true, message: 'File deleted successfully' };
            
        } catch (error) {
            throw new Error(`Delete failed: ${error.message}`);
        }
    }
    
    async getFileInfo(key) {
        try {
            const params = {
                Bucket: this.bucket,
                Key: key
            };
            
            const result = await s3.headObject(params).promise();
            
            return {
                key,
                size: result.ContentLength,
                lastModified: result.LastModified,
                contentType: result.ContentType,
                metadata: result.Metadata
            };
            
        } catch (error) {
            throw new Error(`Failed to get file info: ${error.message}`);
        }
    }
    
    async generatePresignedUrl(key, expiresIn = 3600) {
        try {
            const url = s3.getSignedUrl('getObject', {
                Bucket: this.bucket,
                Key: key,
                Expires: expiresIn
            });
            
            return url;
            
        } catch (error) {
            throw new Error(`Failed to generate presigned URL: ${error.message}`);
        }
    }
    
    async copyFile(sourceKey, destinationKey) {
        try {
            const copyParams = {
                Bucket: this.bucket,
                CopySource: `${this.bucket}/${sourceKey}`,
                Key: destinationKey
            };
            
            await s3.copyObject(copyParams).promise();
            
            return {
                sourceKey,
                destinationKey,
                message: 'File copied successfully'
            };
            
        } catch (error) {
            throw new Error(`Copy failed: ${error.message}`);
        }
    }
}

module.exports = new CloudStorageService();
```

---

## 📱 File Upload Service

### Complete File Upload Service
```javascript
// services/FileUploadService.js
const ImageProcessor = require('../utils/imageProcessor');
const CloudStorageService = require('./CloudStorageService');
const File = require('../models/File');
const fs = require('fs').promises;
const path = require('path');

class FileUploadService {
    async handleFileUpload(file, options = {}) {
        const {
            userId,
            category = 'general',
            isPublic = true,
            processImages = true,
            uploadToCloud = process.env.NODE_ENV === 'production',
            generateVariants = false
        } = options;
        
        try {
            let fileRecord = {
                originalName: file.originalname,
                filename: file.filename,
                path: file.path,
                size: file.size,
                mimetype: file.mimetype,
                category,
                isPublic,
                uploadedBy: userId,
                uploadedAt: new Date()
            };
            
            // Process images
            if (processImages && file.mimetype.startsWith('image/')) {
                const processed = await ImageProcessor.processImage(file.path, {
                    resize: { width: 1920, height: 1080, fit: 'inside' },
                    generateThumbnail: true,
                    generateWebP: true,
                    watermark: process.env.WATERMARK_TEXT ? {
                        text: process.env.WATERMARK_TEXT
                    } : null
                });
                
                fileRecord.processed = processed;
                fileRecord.path = processed.processed[0]?.path || file.path;
            }
            
            // Upload to cloud storage
            if (uploadToCloud) {
                const cloudResult = await CloudStorageService.uploadFile({
                    ...file,
                    path: fileRecord.path
                }, {
                    folder: `${category}/${new Date().getFullYear()}`,
                    makePublic: isPublic,
                    metadata: {
                        category,
                        userId: userId?.toString()
                    }
                });
                
                fileRecord.cloudUrl = cloudResult.url;
                fileRecord.cloudKey = cloudResult.key;
                fileRecord.cloudBucket = cloudResult.bucket;
                
                // Upload processed variants to cloud
                if (fileRecord.processed) {
                    const cloudVariants = [];
                    
                    for (const variant of fileRecord.processed.processed) {
                        const variantResult = await CloudStorageService.uploadFile({
                            ...file,
                            path: variant.path,
                            mimetype: `image/${variant.format}`
                        }, {
                            folder: `${category}/${new Date().getFullYear()}/variants`,
                            makePublic: isPublic
                        });
                        
                        cloudVariants.push({
                            ...variant,
                            cloudUrl: variantResult.url,
                            cloudKey: variantResult.key
                        });
                    }
                    
                    fileRecord.processed.cloudVariants = cloudVariants;
                }
            }
            
            // Save to database
            const savedFile = await File.create(fileRecord);
            
            // Clean up local files if uploaded to cloud
            if (uploadToCloud) {
                await this.cleanupLocalFiles(fileRecord);
            }
            
            return savedFile;
            
        } catch (error) {
            // Clean up on error
            await this.cleanupLocalFiles({ path: file.path });
            throw error;
        }
    }
    
    async handleMultipleFileUpload(files, options = {}) {
        const results = [];
        const errors = [];
        
        for (const file of files) {
            try {
                const result = await this.handleFileUpload(file, options);
                results.push(result);
            } catch (error) {
                errors.push({
                    filename: file.originalname,
                    error: error.message
                });
            }
        }
        
        return { results, errors };
    }
    
    async deleteFile(fileId, userId) {
        try {
            const file = await File.findById(fileId);
            
            if (!file) {
                throw new Error('File not found');
            }
            
            // Check permissions
            if (file.uploadedBy.toString() !== userId.toString() && 
                !this.isAdmin(userId)) {
                throw new Error('Permission denied');
            }
            
            // Delete from cloud storage
            if (file.cloudKey) {
                await CloudStorageService.deleteFile(file.cloudKey);
                
                // Delete variants
                if (file.processed?.cloudVariants) {
                    for (const variant of file.processed.cloudVariants) {
                        await CloudStorageService.deleteFile(variant.cloudKey);
                    }
                }
            }
            
            // Delete local files
            await this.cleanupLocalFiles(file);
            
            // Delete from database
            await File.findByIdAndDelete(fileId);
            
            return { message: 'File deleted successfully' };
            
        } catch (error) {
            throw error;
        }
    }
    
    async getFileInfo(fileId, userId) {
        try {
            const file = await File.findById(fileId);
            
            if (!file) {
                throw new Error('File not found');
            }
            
            // Check permissions for private files
            if (!file.isPublic && 
                file.uploadedBy.toString() !== userId.toString() && 
                !this.isAdmin(userId)) {
                throw new Error('Permission denied');
            }
            
            return file;
            
        } catch (error) {
            throw error;
        }
    }
    
    async getUserFiles(userId, options = {}) {
        const {
            page = 1,
            limit = 20,
            category,
            mimetype,
            sortBy = 'uploadedAt',
            sortOrder = 'desc'
        } = options;
        
        try {
            const query = { uploadedBy: userId };
            
            if (category) query.category = category;
            if (mimetype) query.mimetype = new RegExp(mimetype, 'i');
            
            const files = await File.find(query)
                .sort({ [sortBy]: sortOrder === 'desc' ? -1 : 1 })
                .limit(limit * 1)
                .skip((page - 1) * limit)
                .exec();
            
            const total = await File.countDocuments(query);
            
            return {
                files,
                pagination: {
                    page,
                    limit,
                    total,
                    pages: Math.ceil(total / limit)
                }
            };
            
        } catch (error) {
            throw error;
        }
    }
    
    async cleanupLocalFiles(fileRecord) {
        try {
            // Delete main file
            if (fileRecord.path && await this.fileExists(fileRecord.path)) {
                await fs.unlink(fileRecord.path);
            }
            
            // Delete processed variants
            if (fileRecord.processed) {
                for (const variant of fileRecord.processed.processed) {
                    if (await this.fileExists(variant.path)) {
                        await fs.unlink(variant.path);
                    }
                }
                
                for (const thumbnail of fileRecord.processed.thumbnails) {
                    if (await this.fileExists(thumbnail.path)) {
                        await fs.unlink(thumbnail.path);
                    }
                }
                
                if (fileRecord.processed.responsive) {
                    for (const responsive of fileRecord.processed.responsive) {
                        if (await this.fileExists(responsive.path)) {
                            await fs.unlink(responsive.path);
                        }
                    }
                }
            }
        } catch (error) {
            console.error('Error cleaning up local files:', error);
        }
    }
    
    async fileExists(filePath) {
        try {
            await fs.access(filePath);
            return true;
        } catch {
            return false;
        }
    }
    
    isAdmin(userId) {
        // Implement admin check logic
        return false;
    }
}

module.exports = new FileUploadService();
```

---

## 🎮 File Upload Controllers

### Upload Controller
```javascript
// controllers/FileUploadController.js
const FileUploadService = require('../services/FileUploadService');
const { asyncErrorHandler } = require('../middleware/errorHandler');
const { uploadConfigs } = require('../middleware/upload');

class FileUploadController {
    // Single file upload
    static uploadSingle = [
        uploadConfigs.singleImage,
        asyncErrorHandler(async (req, res) => {
            if (!req.file) {
                return res.status(400).json({
                    success: false,
                    message: 'No file uploaded'
                });
            }
            
            const options = {
                userId: req.user?.id,
                category: req.body.category || 'general',
                isPublic: req.body.isPublic !== 'false',
                processImages: req.body.processImages !== 'false'
            };
            
            const result = await FileUploadService.handleFileUpload(req.file, options);
            
            res.status(201).json({
                success: true,
                message: 'File uploaded successfully',
                file: result
            });
        })
    ];
    
    // Multiple file upload
    static uploadMultiple = [
        uploadConfigs.multipleImages,
        asyncErrorHandler(async (req, res) => {
            if (!req.files || req.files.length === 0) {
                return res.status(400).json({
                    success: false,
                    message: 'No files uploaded'
                });
            }
            
            const options = {
                userId: req.user?.id,
                category: req.body.category || 'general',
                isPublic: req.body.isPublic !== 'false',
                processImages: req.body.processImages !== 'false'
            };
            
            const result = await FileUploadService.handleMultipleFileUpload(req.files, options);
            
            res.status(201).json({
                success: true,
                message: `${result.results.length} files uploaded successfully`,
                files: result.results,
                errors: result.errors
            });
        })
    ];
    
    // Avatar upload
    static uploadAvatar = [
        uploadConfigs.avatar,
        asyncErrorHandler(async (req, res) => {
            if (!req.file) {
                return res.status(400).json({
                    success: false,
                    message: 'No avatar file uploaded'
                });
            }
            
            const options = {
                userId: req.user.id,
                category: 'avatars',
                isPublic: true,
                processImages: true
            };
            
            const result = await FileUploadService.handleFileUpload(req.file, options);
            
            // Update user's avatar URL
            const User = require('../models/User');
            await User.findByIdAndUpdate(req.user.id, {
                avatarUrl: result.cloudUrl || result.path
            });
            
            res.json({
                success: true,
                message: 'Avatar uploaded successfully',
                avatar: result
            });
        })
    ];
    
    // Get file info
    static getFileInfo = asyncErrorHandler(async (req, res) => {
        const { fileId } = req.params;
        const file = await FileUploadService.getFileInfo(fileId, req.user?.id);
        
        res.json({
            success: true,
            file
        });
    });
    
    // Delete file
    static deleteFile = asyncErrorHandler(async (req, res) => {
        const { fileId } = req.params;
        const result = await FileUploadService.deleteFile(fileId, req.user.id);
        
        res.json({
            success: true,
            message: result.message
        });
    });
    
    // Get user files
    static getUserFiles = asyncErrorHandler(async (req, res) => {
        const options = {
            page: parseInt(req.query.page) || 1,
            limit: parseInt(req.query.limit) || 20,
            category: req.query.category,
            mimetype: req.query.mimetype,
            sortBy: req.query.sortBy || 'uploadedAt',
            sortOrder: req.query.sortOrder || 'desc'
        };
        
        const result = await FileUploadService.getUserFiles(req.user.id, options);
        
        res.json({
            success: true,
            ...result
        });
    });
    
    // Render upload page
    static renderUploadPage = (req, res) => {
        res.render('upload/index', {
            title: 'File Upload',
            user: req.user
        });
    };
}

module.exports = FileUploadController;
```

---

## 🎨 Upload Templates

### File Upload Form
```html
<!-- views/upload/index.ejs -->
<% layout('layouts/main') -%>

<div class="container mt-4">
    <div class="row">
        <div class="col-lg-8 mx-auto">
            <h2>File Upload</h2>
            
            <!-- Upload Form -->
            <div class="card mb-4">
                <div class="card-header">
                    <ul class="nav nav-tabs card-header-tabs" role="tablist">
                        <li class="nav-item" role="presentation">
                            <button class="nav-link active" data-bs-toggle="tab" data-bs-target="#single-upload">
                                Single File
                            </button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" data-bs-toggle="tab" data-bs-target="#multiple-upload">
                                Multiple Files
                            </button>
                        </li>
                        <li class="nav-item" role="presentation">
                            <button class="nav-link" data-bs-toggle="tab" data-bs-target="#drag-drop">
                                Drag & Drop
                            </button>
                        </li>
                    </ul>
                </div>
                <div class="card-body">
                    <div class="tab-content">
                        <!-- Single File Upload -->
                        <div class="tab-pane fade show active" id="single-upload">
                            <form id="singleUploadForm" enctype="multipart/form-data">
                                <div class="mb-3">
                                    <label for="singleFile" class="form-label">Choose File</label>
                                    <input type="file" class="form-control" id="singleFile" name="image" 
                                           accept="image/*" required>
                                    <div class="form-text">Supported formats: JPEG, PNG, WebP (Max: 5MB)</div>
                                </div>
                                
                                <div class="mb-3">
                                    <label for="category" class="form-label">Category</label>
                                    <select class="form-select" name="category">
                                        <option value="general">General</option>
                                        <option value="profile">Profile</option>
                                        <option value="product">Product</option>
                                        <option value="gallery">Gallery</option>
                                    </select>
                                </div>
                                
                                <div class="mb-3 form-check">
                                    <input type="checkbox" class="form-check-input" id="isPublic" 
                                           name="isPublic" checked>
                                    <label class="form-check-label" for="isPublic">
                                        Make file public
                                    </label>
                                </div>
                                
                                <div class="mb-3 form-check">
                                    <input type="checkbox" class="form-check-input" id="processImages" 
                                           name="processImages" checked>
                                    <label class="form-check-label" for="processImages">
                                        Process images (resize, thumbnails, WebP)
                                    </label>
                                </div>
                                
                                <button type="submit" class="btn btn-primary">
                                    <span class="btn-text">Upload File</span>
                                    <span class="spinner-border spinner-border-sm ms-2 d-none"></span>
                                </button>
                            </form>
                        </div>
                        
                        <!-- Multiple File Upload -->
                        <div class="tab-pane fade" id="multiple-upload">
                            <form id="multipleUploadForm" enctype="multipart/form-data">
                                <div class="mb-3">
                                    <label for="multipleFiles" class="form-label">Choose Files</label>
                                    <input type="file" class="form-control" id="multipleFiles" name="images" 
                                           accept="image/*" multiple required>
                                    <div class="form-text">Select up to 10 images (Max: 5MB each)</div>
                                </div>
                                
                                <div class="row">
                                    <div class="col-md-6">
                                        <label for="multipleCategory" class="form-label">Category</label>
                                        <select class="form-select" name="category">
                                            <option value="general">General</option>
                                            <option value="gallery">Gallery</option>
                                            <option value="product">Product</option>
                                        </select>
                                    </div>
                                    <div class="col-md-6">
                                        <div class="form-check mt-4">
                                            <input type="checkbox" class="form-check-input" name="isPublic" checked>
                                            <label class="form-check-label">Make files public</label>
                                        </div>
                                    </div>
                                </div>
                                
                                <button type="submit" class="btn btn-primary mt-3">
                                    <span class="btn-text">Upload Files</span>
                                    <span class="spinner-border spinner-border-sm ms-2 d-none"></span>
                                </button>
                            </form>
                        </div>
                        
                        <!-- Drag & Drop Upload -->
                        <div class="tab-pane fade" id="drag-drop">
                            <div id="dropZone" class="border border-dashed rounded p-5 text-center">
                                <i class="fas fa-cloud-upload-alt fa-3x text-muted mb-3"></i>
                                <h5>Drag & Drop Files Here</h5>
                                <p class="text-muted">Or click to select files</p>
                                <input type="file" id="dragDropFiles" multiple accept="image/*" style="display: none;">
                                <button type="button" class="btn btn-outline-primary" onclick="document.getElementById('dragDropFiles').click()">
                                    Select Files
                                </button>
                            </div>
                            
                            <!-- File List -->
                            <div id="fileList" class="mt-3"></div>
                            
                            <!-- Upload Progress -->
                            <div id="uploadProgress" class="mt-3 d-none">
                                <div class="progress">
                                    <div class="progress-bar" role="progressbar" style="width: 0%"></div>
                                </div>
                                <small class="text-muted">Uploading files...</small>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
            
            <!-- Upload Results -->
            <div id="uploadResults" class="d-none">
                <div class="card">
                    <div class="card-header">
                        <h5>Upload Results</h5>
                    </div>
                    <div class="card-body" id="resultsContent">
                        <!-- Results will be populated here -->
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

<script>
// File upload handling
class FileUploader {
    constructor() {
        this.setupEventListeners();
        this.setupDragDrop();
    }
    
    setupEventListeners() {
        // Single file upload
        document.getElementById('singleUploadForm').addEventListener('submit', (e) => {
            e.preventDefault();
            this.uploadSingleFile(e.target);
        });
        
        // Multiple file upload
        document.getElementById('multipleUploadForm').addEventListener('submit', (e) => {
            e.preventDefault();
            this.uploadMultipleFiles(e.target);
        });
        
        // File input change for drag & drop
        document.getElementById('dragDropFiles').addEventListener('change', (e) => {
            this.handleFileSelection(e.target.files);
        });
    }
    
    setupDragDrop() {
        const dropZone = document.getElementById('dropZone');
        
        dropZone.addEventListener('dragover', (e) => {
            e.preventDefault();
            dropZone.classList.add('border-primary', 'bg-light');
        });
        
        dropZone.addEventListener('dragleave', (e) => {
            e.preventDefault();
            dropZone.classList.remove('border-primary', 'bg-light');
        });
        
        dropZone.addEventListener('drop', (e) => {
            e.preventDefault();
            dropZone.classList.remove('border-primary', 'bg-light');
            this.handleFileSelection(e.dataTransfer.files);
        });
    }
    
    async uploadSingleFile(form) {
        const formData = new FormData(form);
        const button = form.querySelector('button[type="submit"]');
        
        this.setLoadingState(button, true);
        
        try {
            const response = await fetch('/api/upload/single', {
                method: 'POST',
                body: formData
            });
            
            const result = await response.json();
            
            if (result.success) {
                this.showSuccess('File uploaded successfully!');
                this.displayResults([result.file]);
                form.reset();
            } else {
                this.showError(result.message || 'Upload failed');
            }
        } catch (error) {
            this.showError('Upload failed: ' + error.message);
        } finally {
            this.setLoadingState(button, false);
        }
    }
    
    async uploadMultipleFiles(form) {
        const formData = new FormData(form);
        const button = form.querySelector('button[type="submit"]');
        
        this.setLoadingState(button, true);
        
        try {
            const response = await fetch('/api/upload/multiple', {
                method: 'POST',
                body: formData
            });
            
            const result = await response.json();
            
            if (result.success) {
                this.showSuccess(`${result.files.length} files uploaded successfully!`);
                this.displayResults(result.files);
                if (result.errors.length > 0) {
                    this.showWarning(`${result.errors.length} files failed to upload`);
                }
                form.reset();
            } else {
                this.showError(result.message || 'Upload failed');
            }
        } catch (error) {
            this.showError('Upload failed: ' + error.message);
        } finally {
            this.setLoadingState(button, false);
        }
    }
    
    handleFileSelection(files) {
        const fileList = document.getElementById('fileList');
        fileList.innerHTML = '';
        
        Array.from(files).forEach((file, index) => {
            const fileItem = document.createElement('div');
            fileItem.className = 'border rounded p-2 mb-2';
            fileItem.innerHTML = `
                <div class="d-flex justify-content-between align-items-center">
                    <div>
                        <strong>${file.name}</strong>
                        <small class="text-muted d-block">${this.formatFileSize(file.size)}</small>
                    </div>
                    <button class="btn btn-sm btn-outline-danger" onclick="this.parentElement.parentElement.remove()">
                        <i class="fas fa-times"></i>
                    </button>
                </div>
            `;
            fileList.appendChild(fileItem);
        });
        
        if (files.length > 0) {
            const uploadButton = document.createElement('button');
            uploadButton.className = 'btn btn-primary mt-2';
            uploadButton.textContent = 'Upload Selected Files';
            uploadButton.onclick = () => this.uploadDragDropFiles(files);
            fileList.appendChild(uploadButton);
        }
    }
    
    async uploadDragDropFiles(files) {
        const formData = new FormData();
        Array.from(files).forEach(file => {
            formData.append('images', file);
        });
        formData.append('category', 'general');
        formData.append('isPublic', 'true');
        
        const progressContainer = document.getElementById('uploadProgress');
        const progressBar = progressContainer.querySelector('.progress-bar');
        
        progressContainer.classList.remove('d-none');
        
        try {
            const response = await fetch('/api/upload/multiple', {
                method: 'POST',
                body: formData
            });
            
            const result = await response.json();
            
            if (result.success) {
                progressBar.style.width = '100%';
                this.showSuccess(`${result.files.length} files uploaded successfully!`);
                this.displayResults(result.files);
                document.getElementById('fileList').innerHTML = '';
            } else {
                this.showError(result.message || 'Upload failed');
            }
        } catch (error) {
            this.showError('Upload failed: ' + error.message);
        } finally {
            setTimeout(() => {
                progressContainer.classList.add('d-none');
                progressBar.style.width = '0%';
            }, 2000);
        }
    }
    
    displayResults(files) {
        const resultsContainer = document.getElementById('uploadResults');
        const resultsContent = document.getElementById('resultsContent');
        
        let html = '<div class="row">';
        files.forEach(file => {
            html += `
                <div class="col-md-4 mb-3">
                    <div class="card">
                        ${file.mimetype.startsWith('image/') ? 
                            `<img src="${file.cloudUrl || file.path}" class="card-img-top" style="height: 200px; object-fit: cover;">` :
                            `<div class="card-img-top d-flex align-items-center justify-content-center bg-light" style="height: 200px;">
                                <i class="fas fa-file fa-3x text-muted"></i>
                            </div>`
                        }
                        <div class="card-body">
                            <h6 class="card-title">${file.originalName}</h6>
                            <p class="card-text">
                                <small class="text-muted">
                                    Size: ${this.formatFileSize(file.size)}<br>
                                    Type: ${file.mimetype}
                                </small>
                            </p>
                            <div class="btn-group w-100">
                                <button class="btn btn-sm btn-outline-primary" onclick="this.copyToClipboard('${file.cloudUrl || file.path}')">
                                    Copy URL
                                </button>
                                <button class="btn btn-sm btn-outline-danger" onclick="this.deleteFile('${file._id}')">
                                    Delete
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            `;
        });
        html += '</div>';
        
        resultsContent.innerHTML = html;
        resultsContainer.classList.remove('d-none');
    }
    
    setLoadingState(button, loading) {
        const text = button.querySelector('.btn-text');
        const spinner = button.querySelector('.spinner-border');
        
        if (loading) {
            text.textContent = 'Uploading...';
            spinner.classList.remove('d-none');
            button.disabled = true;
        } else {
            text.textContent = 'Upload Files';
            spinner.classList.add('d-none');
            button.disabled = false;
        }
    }
    
    formatFileSize(bytes) {
        if (bytes === 0) return '0 Bytes';
        const k = 1024;
        const sizes = ['Bytes', 'KB', 'MB', 'GB'];
        const i = Math.floor(Math.log(bytes) / Math.log(k));
        return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
    }
    
    copyToClipboard(text) {
        navigator.clipboard.writeText(text).then(() => {
            this.showSuccess('URL copied to clipboard!');
        });
    }
    
    showSuccess(message) {
        this.showAlert(message, 'success');
    }
    
    showError(message) {
        this.showAlert(message, 'danger');
    }
    
    showWarning(message) {
        this.showAlert(message, 'warning');
    }
    
    showAlert(message, type) {
        const alert = document.createElement('div');
        alert.className = `alert alert-${type} alert-dismissible fade show`;
        alert.innerHTML = `
            ${message}
            <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
        `;
        
        document.querySelector('.container').insertBefore(alert, document.querySelector('.container').firstChild);
        
        setTimeout(() => {
            alert.remove();
        }, 5000);
    }
}

// Initialize uploader
new FileUploader();
</script>
```

---

## 🔧 Routes Configuration

### File Upload Routes
```javascript
// routes/uploadRoutes.js
const express = require('express');
const router = express.Router();
const FileUploadController = require('../controllers/FileUploadController');
const { requireAuth } = require('../middleware/auth');

// All upload routes require authentication
router.use(requireAuth);

// Upload routes
router.post('/single', FileUploadController.uploadSingle);
router.post('/multiple', FileUploadController.uploadMultiple);
router.post('/avatar', FileUploadController.uploadAvatar);

// File management
router.get('/files', FileUploadController.getUserFiles);
router.get('/files/:fileId', FileUploadController.getFileInfo);
router.delete('/files/:fileId', FileUploadController.deleteFile);

// Web routes
router.get('/', FileUploadController.renderUploadPage);

module.exports = router;
```

---

## 🔗 Related Topics
- [[23-Cookies-Sessions]] - Session management for uploads
- [[25-Security-Best-Practices]] - File upload security
- [[21-Middlewares]] - Upload middleware
- [[26-Caching-Strategies]] - File caching strategies

---

*Back to [[00-Main-Index]]*
