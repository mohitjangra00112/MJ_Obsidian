Got it ✅ — you want a **loading animation that covers the whole `<iframe>`** until it’s fully loaded.

Here’s a **single HTML file example** that does it:

---

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Iframe Loading Animation</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 0;
    }

    /* Container to hold iframe + loader */
    .iframe-container {
      position: relative;
      width: 100%;
      height: 500px;
      border: 2px solid #ccc;
    }

    /* Loader overlay */
    .loader {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(255, 255, 255, 0.8);
      display: flex;
      align-items: center;
      justify-content: center;
      z-index: 10;
    }

    /* Spinner animation */
    .spinner {
      width: 50px;
      height: 50px;
      border: 6px solid #ddd;
      border-top: 6px solid #3498db;
      border-radius: 50%;
      animation: spin 0.8s linear infinite;
    }

    @keyframes spin {
      0% { transform: rotate(0deg); }
      100% { transform: rotate(360deg); }
    }

    /* Iframe styling */
    iframe {
      width: 100%;
      height: 100%;
      border: none;
    }
  </style>
</head>
<body>

  <h2>Iframe with Loading Animation</h2>

  <div class="iframe-container">
    <!-- Loader -->
    <div class="loader" id="loader">
      <div class="spinner"></div>
    </div>

    <!-- Iframe -->
    <iframe id="myIframe" src="https://example.com"></iframe>
  </div>

  <script>
    const iframe = document.getElementById("myIframe");
    const loader = document.getElementById("loader");

    // Hide loader when iframe finishes loading
    iframe.addEventListener("load", () => {
      loader.style.display = "none";
    });
  </script>

</body>
</html>
```

---

### **How It Works**

1. The `.loader` div covers the entire iframe with a semi-transparent background.
    
2. The spinner runs until the iframe’s `load` event fires.
    
3. Once the iframe finishes loading, the loader is hidden (`display: none`).
    

---

If you want, I can also give you a **version where the loader fades out smoothly** instead of disappearing instantly. That will make it look more professional.