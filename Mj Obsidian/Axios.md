Great — let's look at **Axios**, a popular JavaScript library for making HTTP requests (often easier and cleaner than `fetch`).

---

## 🔧 Installation

If you're using **Node.js** or a bundler like Webpack/Vite:

```bash
npm install axios
```

In the browser (CDN):

```html
<script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
```

---

## ✅ GET Request with Axios

```javascript
const axios = require('axios'); // Only needed in Node.js

async function getData() {
  try {
    const response = await axios.get('https://jsonplaceholder.typicode.com/posts');
    console.log(response.data); // Axios stores response data in `data`
  } catch (error) {
    console.error('GET error:', error);
  }
}

getData();
```

---

## ✅ POST Request with Axios

```javascript
async function postData() {
  try {
    const response = await axios.post('https://jsonplaceholder.typicode.com/posts', {
      title: 'foo',
      body: 'bar',
      userId: 1
    });

    console.log(response.data); // Response data
  } catch (error) {
    console.error('POST error:', error);
  }
}

postData();
```

---

## 🔒 Adding Headers (e.g., Authorization)

```javascript
axios.get('https://api.example.com/user', {
  headers: {
    Authorization: 'Bearer YOUR_TOKEN_HERE'
  }
});
```

---

## ⚡ Key Differences from `fetch`

|Feature|`axios`|`fetch`|
|---|---|---|
|Auto JSON parse|✅ Yes|❌ No (you call `.json()` manually)|
|Error handling|✅ Errors on bad status codes|❌ Only network errors (you check status manually)|
|Simpler syntax|✅ Yes|🟡 A bit more verbose|

---
