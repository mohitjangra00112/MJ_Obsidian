Perfect ✅ Mohit — here’s a **complete documentation** for your **Stock Price Prediction AI using TensorFlow + PERN Stack + PostgreSQL**, fully aligned with your **three-phase plan** (AI → PERN → Integration).

You can copy this directly into your `README.md` or project report — it’s professional, structured, and ready for GitHub or portfolio use.

---

# 📊 Stock Price Prediction AI (TensorFlow + PERN + PostgreSQL)

### 🔹 Author: Mohit Jangra

### 🔹 Tech Stack: TensorFlow, Python, React, Express.js, Node.js, PostgreSQL

### 🔹 Project Type: AI + Full Stack Web Application

### 🔹 Goal: Predict future stock prices using historical and daily financial data.

---

## 🧭 Overview

The **Stock Price Prediction System** is a full-stack AI-powered web application that predicts future stock prices using **machine learning (LSTM neural networks)**.

It fetches **real-time stock data from Yahoo Finance** and merges it with **historical data from Kaggle** to train the model.

The app is built in **three phases** for better development and testing flow:

1. **Phase 1** → AI model development and monitoring dashboard
    
2. **Phase 2** → PERN (PostgreSQL, Express, React, Node.js) base app
    
3. **Phase 3** → Integration of AI model with PERN stack + daily retraining automation
    

---

## 🧩 Phase 1 — AI Model Development and Training Dashboard

### 🎯 Objective

To build, train, and evaluate the AI model using **TensorFlow** and visualize its training progress through a **simple UI dashboard**.

---

### 🧠 AI Workflow

1. **Data Sources**
    
    - Kaggle dataset (historical data)
        
    - Yahoo Finance API (daily data)
        
2. **Data Processing**
    
    - Merge both datasets
        
    - Clean, normalize, and prepare for model training
        
3. **Model**
    
    - LSTM-based neural network
        
    - Predicts the next day's closing stock price
        
4. **Evaluation**
    
    - Loss graph visualization
        
    - Live training progress indicator
        
5. **Output**
    
    - Trained model saved as `stock_model.h5`
        
    - Scaler saved as `scaler.pkl`
        

---

### 📁 Folder Structure

```
ai/
│
├── data/
│   ├── kaggle_data.csv
│   ├── combined_data.csv
│   └── latest_yahoo_data.csv
│
├── models/
│   ├── stock_model.h5
│   └── scaler.pkl
│
├── train_model.py
├── predict.py
├── daily_update.py
└── server.py          # Flask server for monitoring
```

---

### 🧮 Model Architecture (TensorFlow)

```python
model = tf.keras.Sequential([
    tf.keras.layers.LSTM(50, return_sequences=True, input_shape=(X.shape[1], 1)),
    tf.keras.layers.LSTM(50, return_sequences=False),
    tf.keras.layers.Dense(25),
    tf.keras.layers.Dense(1)
])
```

**Optimizer:** Adam  
**Loss Function:** Mean Squared Error (MSE)  
**Training Window:** 60 days (past data → next day prediction)

---

### 🧰 Flask Monitoring Server

- Endpoint `/train`: Start model training
    
- Endpoint `/status`: Returns training progress (% complete)
    
- Endpoint `/metrics`: Returns final loss/accuracy
    

**Example:**

```bash
GET /train
→ "Training started..."

GET /status
→ { "epoch": 5, "loss": 0.00123 }
```

---

### 🖥️ React Dashboard (Phase 1 UI)

Features:

- “Train Model” button
    
- Progress bar showing epoch progress
    
- Graph of loss vs epochs
    
- Status indicator: 🟢 Success / 🔴 Failed / 🟡 In Progress
    

---

### 🧾 Outputs (End of Phase 1)

✅ Working AI model (`stock_model.h5`)  
✅ Dataset combined & cleaned  
✅ Real-time monitoring dashboard  
✅ Model ready for integration

---

## 🧩 Phase 2 — PERN Stack App Development

### 🎯 Objective

To build the **core web application** with **React (frontend)**, **Node.js + Express (backend)**, and **PostgreSQL (database)** — without connecting AI yet.

---

### 📁 Folder Structure

```
backend/
│
├── server.js
├── routes/
│   ├── users.js
│   ├── stocks.js
│   └── predictions.js
├── controllers/
├── db/
│   └── connection.js
└── models/
    └── queries.sql

frontend/
│
├── src/
│   ├── components/
│   │   ├── Navbar.jsx
│   │   ├── StockList.jsx
│   │   ├── Dashboard.jsx
│   │   └── LoginForm.jsx
│   └── pages/
│       ├── Home.jsx
│       ├── Dashboard.jsx
│       └── Profile.jsx
└── App.jsx
```

---

### 🗄️ PostgreSQL Schema

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  username VARCHAR(100),
  email VARCHAR(255),
  password VARCHAR(255)
);

CREATE TABLE predictions (
  id SERIAL PRIMARY KEY,
  user_id INT REFERENCES users(id),
  symbol VARCHAR(10),
  predicted_price NUMERIC,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

### 🧰 Backend Features (Node + Express)

- **User Routes**
    
    - `/signup` → Create user
        
    - `/login` → Authenticate user
        
- **Stock Routes**
    
    - `/stocks` → Get stock list
        
    - `/predictions` → Get past predictions
        
- **Auth**
    
    - JWT-based authentication
        
    - bcrypt for password hashing
        

---

### 🖥️ Frontend Features (React)

- Login / Register page
    
- Stock dashboard
    
- Prediction table
    
- Search bar for stock symbols
    
- Button “Predict with AI” (inactive until Phase 3)
    

---

### 🧾 Outputs (End of Phase 2)

✅ Full working PERN app  
✅ User authentication  
✅ Stock dashboard UI  
✅ Database connected successfully

---

## 🧩 Phase 3 — AI Integration + Daily Retraining

### 🎯 Objective

Integrate the trained TensorFlow model into the PERN backend, enable **real-time predictions**, and schedule **daily auto-retraining** using Yahoo data.

---

### 🔗 Integration Flow

1. **Frontend (React):**
    
    - User enters stock symbol → calls backend `/predict/:symbol`
        
2. **Backend (Node):**
    
    - Executes Python script `predict.py` with `child_process.spawn`
        
    - Receives predicted price and stores it in PostgreSQL
        
3. **Python (AI):**
    
    - Loads model and scaler
        
    - Fetches last 60 days of data from Yahoo
        
    - Predicts next day price → sends back to Node
        
4. **Database (PostgreSQL):**
    
    - Saves `symbol`, `predicted_price`, and timestamp
        
5. **Daily Retraining:**
    
    - Node cron job runs `daily_update.py` at a fixed time every day.
        

---

### 🧱 Node.js Backend Example

```javascript
const express = require('express');
const { spawn } = require('child_process');
const app = express();

app.get('/predict/:symbol', (req, res) => {
  const symbol = req.params.symbol;
  const python = spawn('python', ['../ai/predict.py', symbol]);

  python.stdout.on('data', (data) => {
    res.send({ prediction: data.toString() });
  });

  python.stderr.on('data', (data) => {
    console.error(`Error: ${data}`);
  });
});

app.listen(4000, () => console.log("Server running on port 4000"));
```

---

### ⏰ Daily Retraining (Automation)

```javascript
const cron = require('node-cron');
const { spawn } = require('child_process');

cron.schedule('0 20 * * *', () => {
  console.log("Starting daily retraining...");
  spawn('python', ['../ai/daily_update.py']);
});
```

🕗 Runs every day at 8 PM (you can modify the schedule).

---

### 🗄️ Database Integration (Store Predictions)

```javascript
await pool.query(
  "INSERT INTO predictions (symbol, predicted_price) VALUES ($1, $2)",
  [symbol, prediction]
);
```

---

### 🖥️ Final UI Features

- Input box for stock symbol
    
- “Predict” button
    
- Predicted price + graph
    
- Historical predictions table
    
- Model training status (optional from Phase 1)
    
- Auto-updated data daily
    

---

## 🚀 Final Output

✅ AI model trained and auto-updating daily  
✅ PERN app connected to TensorFlow model  
✅ Predictions saved in PostgreSQL  
✅ Professional, real-world deployable web app

---

## 💾 Deployment (Free Hosting Options)

|Component|Service|Free Tier|
|---|---|---|
|**Backend (Node.js)**|Render / Railway|✅|
|**Frontend (React)**|Vercel / Netlify|✅|
|**Database (PostgreSQL)**|ElephantSQL|✅|
|**Python Model API**|Render (Flask Server)|✅|

---

## 🧩 Tech Stack Summary

|Layer|Technology|
|---|---|
|Frontend|React, Tailwind CSS|
|Backend|Node.js, Express|
|Database|PostgreSQL|
|AI|TensorFlow (Python)|
|APIs|Yahoo Finance, Kaggle|
|Scheduler|node-cron|
|Authentication|JWT, bcrypt|

---

## 📈 Future Enhancements

- 📰 **News Sentiment Analysis** (NLP model for trend impact)
    
- 📊 **Stock comparison charts**
    
- 📉 **Accuracy visualization (Predicted vs Actual)**
    
- 👤 **User watchlist**
    
- 📱 **Mobile-friendly UI (React + Tailwind)**
    

---

## 🏁 Summary

|Phase|Description|Status|
|---|---|---|
|**Phase 1**|Build & visualize AI model|🧠 ML + Python|
|**Phase 2**|Develop PERN stack app|🧱 Web + DB|
|**Phase 3**|Integrate AI + Automation|🚀 Smart App|

---

Would you like me to now create a **separate `README.md` file (Markdown formatted)** that you can directly upload to your GitHub repo (with formatting, emojis, and sections ready)?  
It’ll look great on your project page.