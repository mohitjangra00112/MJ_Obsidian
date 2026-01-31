Alright, here’s a **single HTML file** that shows a loading animation for dynamic API elements using JavaScript.

---

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Loading Animation Example</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      padding: 20px;
    }

    /* Spinner Styling */
    .loading-spinner {
      width: 40px;
      height: 40px;
      border: 4px solid #ddd;
      border-top: 4px solid #3498db;
      border-radius: 50%;
      animation: spin 0.8s linear infinite;
      margin: 20px auto;
    }

    @keyframes spin {
      0% { transform: rotate(0deg); }
      100% { transform: rotate(360deg); }
    }

    .item {
      margin: 10px 0;
      padding: 10px;
      background: #f0f0f0;
      border-radius: 5px;
    }
  </style>
</head>
<body>
  <h2>Dynamic API Data with Loading Animation</h2>

  <div id="data-container">
    <!-- Loading animation -->
    <div id="loading" class="loading-spinner" style="display:none;"></div>

    <!-- Data will be injected here -->
    <div id="data"></div>
  </div>

  <button onclick="fetchData()">Reload Data</button>

  <script>
    const loading = document.getElementById("loading");
    const dataDiv = document.getElementById("data");

    async function fetchData() {
      // Show loading spinner
      loading.style.display = "block";
      dataDiv.innerHTML = "";

      try {
        const res = await fetch("https://jsonplaceholder.typicode.com/posts?_limit=5");
        const data = await res.json();

        // Hide loading spinner
        loading.style.display = "none";

        // Show API data
        data.forEach(item => {
          const div = document.createElement("div");
          div.className = "item";
          div.textContent = item.title;
          dataDiv.appendChild(div);
        });
      } catch (error) {
        loading.style.display = "none";
        dataDiv.textContent = "Error loading data.";
      }
    }

    // Load data on page load
    fetchData();
  </script>
</body>
</html>
```

---

This is **self-contained** — no external CSS or JS files needed.

- **Spinner appears** while the API is fetching.
    
- **Data replaces the spinner** once loaded.
    
- A **Reload Data** button lets you re-test the animation.
    

---

If you want, I can now give you a **version where each card/element has its own mini spinner** so multiple sections can load independently. That’s especially useful for tables and grids.