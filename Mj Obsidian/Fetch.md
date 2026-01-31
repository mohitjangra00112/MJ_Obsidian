Here’s a clear breakdown of how to make **GET** and **POST** requests using the `fetch` API in JavaScript:

---

## ✅ GET Request (Fetching data)

```javascript
async function getData() {
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/posts');

    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }

    const data = await response.json(); // Parse JSON
    console.log(data); // Output fetched data
  } catch (error) {
    console.error('Error fetching data:', error);
  }
}

getData();
```

---

## ✅ POST Request (Sending data)

```javascript
async function postData() {
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'  // Tells server the payload is JSON
      },
      body: JSON.stringify({
        title: 'foo',
        body: 'bar',
        userId: 1
      })
    });

    if (!response.ok) {
      throw new Error(`HTTP error! Status: ${response.status}`);
    }

    const result = await response.json(); // Get response data
    console.log(result); // Output result
  } catch (error) {
    console.error('Error posting data:', error);
  }
}

postData();
```

---

## 🧠 Quick Summary

|Feature|GET|POST|
|---|---|---|
|Purpose|Retrieve data|Send data to server|
|Method|`'GET'` (default)|`'POST'`|
|`body` allowed?|❌ No|✅ Yes|
|Use case|Load info from API|Submit forms, create new records|

---

Let me know if you also want examples with PUT, DELETE, or with auth headers (e.g. tokens).