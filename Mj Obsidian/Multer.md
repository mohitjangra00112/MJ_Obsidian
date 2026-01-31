Here are **basic notes on Multer**, a middleware used in Node.js to handle file uploads:

---

## 📦 What is Multer?

- **Multer** is a middleware for handling `multipart/form-data`, primarily used for **uploading files**.
    
- It works with **Express.js** and **Node.js**.
    
- It adds a `body` object and a `file` or `files` object to the `req` object.
    

---

## ⚙️ Installation

```bash
npm install multer
```

---

## 🧠 Basic Concepts

|Concept|Description|
|---|---|
|`req.file`|Used when uploading a **single file**|
|`req.files`|Used when uploading **multiple files**|
|`storage`|Controls where and how files are stored|
|`limits`|Sets limits on file size, number, etc.|
|`fileFilter`|Controls which files are accepted|

---

## 📁 Basic Usage (Single File Upload)

```js
const express = require('express');
const multer = require('multer');
const app = express();

// Set up storage (optional: you can use default memory storage)
const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    cb(null, 'uploads/'); // Folder to store files
  },
  filename: function (req, file, cb) {
    cb(null, Date.now() + '-' + file.originalname); // Rename file
  }
});

const upload = multer({ storage: storage });

// Route for uploading a single file
app.post('/upload', upload.single('myFile'), (req, res) => {
  res.send('File uploaded successfully');
});
```

> `upload.single('myFile')` — the `'myFile'` must match the name in the form/input field.

---

## 📂 Multiple Files Upload

```js
app.post('/uploadMultiple', upload.array('myFiles', 5), (req, res) => {
  res.send('Multiple files uploaded successfully');
});
```

---

## 🔒 File Filter Example

```js
const upload = multer({ 
  storage: storage,
  fileFilter: function (req, file, cb) {
    if (file.mimetype === 'image/jpeg' || file.mimetype === 'image/png') {
      cb(null, true);
    } else {
      cb(new Error('Only JPEG and PNG files are allowed!'), false);
    }
  }
});
```

---

## 📏 File Size Limit

```js
const upload = multer({ 
  storage: storage,
  limits: { fileSize: 1 * 1024 * 1024 } // 1MB
});
```

---

## 🧼 Handling Errors

```js
app.post('/upload', (req, res) => {
  upload.single('myFile')(req, res, function (err) {
    if (err instanceof multer.MulterError) {
      return res.status(400).send('Multer error: ' + err.message);
    } else if (err) {
      return res.status(500).send('Unknown error: ' + err.message);
    }
    res.send('Upload successful');
  });
});
```

---

Let me know if you'd like a working project template or want to store files in memory or cloud (like AWS S3).