Perfect 👏 Mohit!  
Let’s assume your PC is **freshly set up** (no Python, no database, nothing).  
We’ll go step-by-step to **prepare your local environment** for **Phase 1** — AI model training with TensorFlow, Yahoo, and Kaggle.

---

# 🧰 PHASE 1 — COMPLETE LOCAL SETUP GUIDE

## 🪟 Works for Windows (you can follow same flow on macOS/Linux with minor terminal changes)

---

## ⚙️ Step 1: Install Python

### ✅ Download:

👉 [https://www.python.org/downloads/](https://www.python.org/downloads/)

### During installation:

✅ **Check both boxes:**

- ☑️ “Add Python to PATH”
    
- ☑️ “Install pip (Python package manager)”
    

### Verify installation:

```bash
python --version
pip --version
```

You should see something like:

```
Python 3.12.1
pip 24.0
```

---

## 🗃️ Step 2: Create a Project Folder

```bash
mkdir stock-ai
cd stock-ai
```

Now your folder structure:

```
stock-ai/
```

---

## 🧱 Step 3: Create a Virtual Environment (Best Practice)

This isolates dependencies for your project.

```bash
python -m venv venv
```

Activate it:

```bash
venv\Scripts\activate
```

If you see `(venv)` at the start of your terminal line → ✅ activated.

Deactivate any time:

```bash
deactivate
```

---

## 📦 Step 4: Install Core Dependencies

Now install all required libraries.

```bash
pip install tensorflow yfinance pandas numpy scikit-learn joblib matplotlib streamlit plotly kaggle
```

### What each does:

|Library|Purpose|
|---|---|
|**tensorflow**|Deep learning (LSTM model)|
|**yfinance**|Fetch stock data from Yahoo Finance|
|**pandas**|Handle CSV data|
|**numpy**|Array/matrix operations|
|**scikit-learn**|Preprocessing (scaling data)|
|**joblib**|Save model scaler|
|**matplotlib**|Plot training graphs|
|**streamlit**|Build monitoring UI|
|**plotly**|Interactive visualization|
|**kaggle**|Download datasets from Kaggle|

---

## 📁 Step 5: Create Folder Structure

```bash
mkdir data model scripts ui
```

Now your folder tree:

```
stock-ai/
│
├── data/
├── model/
├── scripts/
└── ui/
```

---

## 🧩 Step 6: Set Up Kaggle for Data Download

### 1️⃣ Create a Kaggle account

👉 [https://www.kaggle.com](https://www.kaggle.com/)

### 2️⃣ Get API credentials

- Go to your Kaggle profile → “Account” tab
    
- Scroll to “API” → click **Create New API Token**
    
- It downloads a file: `kaggle.json`
    

### 3️⃣ Place it properly

Move it to your user folder:

```
C:\Users\<YourUser>\.kaggle\kaggle.json
```

### 4️⃣ Test it:

```bash
kaggle datasets list
```

If it lists datasets, ✅ setup complete.

---

## 💾 Step 7: Create Your Data Fetch Script

In `scripts/fetch_data.py` paste:

```python
import yfinance as yf
import pandas as pd
import os

def fetch_data(symbol='AAPL', start='2020-01-01', end='2025-01-01'):
    print("📥 Fetching stock data...")
    data = yf.download(symbol, start=start, end=end)
    os.makedirs("data", exist_ok=True)
    data.to_csv("data/latest_data.csv")
    print("✅ Data saved to data/latest_data.csv")

if __name__ == "__main__":
    fetch_data()
```

Run to check:

```bash
python scripts/fetch_data.py
```

✅ Should create `data/latest_data.csv`.

---

## 🧠 Step 8: Create the Training Script

Create: `scripts/train_model.py`

(Paste the TensorFlow model training code I gave earlier.)

Run it:

```bash
python scripts/train_model.py
```

If everything is installed properly, you’ll see the model training with lines like:

```
Epoch 1/15
500/500 [==============================] - 10s 50ms/step - loss: 0.0025 - val_loss: 0.0018
...
✅ Model trained and saved.
```

---

## 📊 Step 9: Visualize Model Performance

Run:

```bash
python scripts/evaluate_model.py
```

This opens a graph showing loss decreasing across epochs.

---

## 🖥️ Step 10: Run Monitoring Dashboard

In `ui/dashboard.py` paste the Streamlit dashboard code.

Then run:

```bash
streamlit run ui/dashboard.py
```

✅ A local web UI opens (default: [http://localhost:8501](http://localhost:8501/))  
You’ll see:

- Training graphs
    
- A button to retrain the model
    

---

## 🧹 Step 11: Save All Dependencies

Create `requirements.txt`:

```bash
pip freeze > requirements.txt
```

So in the future, you can reinstall everything with:

```bash
pip install -r requirements.txt
```

---

## ✅ Final Check

At this point, your system will have:

- Python (working)
    
- Virtual environment (`venv`)
    
- TensorFlow + ML libraries
    
- Kaggle + Yahoo data access
    
- Your AI model training properly
    
- Dashboard UI working
    

---

Would you like me to give you a **“Phase 1 Verification Checklist”** next — a simple list of things to test (so you can confirm everything works correctly before moving to Phase 2)?