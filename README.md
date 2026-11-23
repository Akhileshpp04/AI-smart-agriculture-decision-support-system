# AI-Based Smart Agriculture Decision Support System

An end-to-end full-stack Decision Support System for farmers and agricultural extension workers, featuring **Price-Aware Crop Selection**, **Hybrid U-Net + ResNet AI Disease Identification**, **Real-Time Mandi Market Prices**, and **Multi-Tier Solution Recommendations**.

---

## 1. Project Overview
AgriSmart is designed to empower agricultural stakeholders by combining agronomic suitability models with real-time economic market dynamics and deep learning computer vision:
- **Price-Aware Crop Recommendation**: Ranks crops by combining agronomic soil suitability (60%), real-time market price benchmarks (25%), and weather alignment (15%).
- **Hybrid U-Net + ResNet Disease Detection**: Uses U-Net semantic segmentation to extract leaf/disease regions and ResNet-18 as a classification backbone.
- **Real-Time Market Prices**: Connects to official Agmarknet / Data.gov.in APIs with database caching and fallback `DEMO DATA` labeling.

---

## 2. Features
- 🌾 **Price-Aware Crop Selection**: Soil, season, state, and acreage input to generate weighted rankings and non-guaranteed revenue estimates.
- 🔬 **U-Net + ResNet AI Diagnosis**: Upload plant leaf photos to see side-by-side original input, U-Net binary mask, and segmented region alongside ResNet diagnosis.
- 📊 **Interactive Market Price Dashboard**: Real-time mandi prices, price ranges (min/modal/max), and SVG trend charts for 7d, 30d, 3m, 6m timeframes.
- 💊 **Comprehensive Treatment Plans**: Organic remedies, chemical controls with pesticide safety notes, long-term prevention, irrigation, and fertilizer guidance.
- 🧪 **Fertilizer Calculator**: N-P-K (Urea, DAP, Potash) bag calculations based on farm land area.
- 🌤️ **Weather Advisor**: 5-day weather forecast and rain/irrigation recommendations.

---

## 3. System Architecture

```
                       SMART AGRICULTURE SYSTEM
                                  |
    --------------------------------------------------------------
    |                             |                              |
Crop Selection            Disease Detection               Market Prices
    |                             |                              |
Soil + Weather Inputs       Leaf Image Input           Agmarknet API / DB
    |                             |                              |
Agronomic Suitability        U-Net Segmentation           Live Prices & Trends
    |                             |                              |
    └───────────> Market Price ──> ResNet Classification <───────┘
                       |          |
              Price-Aware Crop   Diagnosis & Solutions
                  Ranking
```

---

## 4. Deep Learning Architecture (U-Net + ResNet)

### U-Net (Leaf Region Segmentation)
- **Input**: Resized `(256x256x3)` leaf image tensor.
- **Architecture**: 4-stage encoder (double conv + maxpool) -> bottleneck -> 4-stage decoder (transpose conv + skip connections).
- **Output**: 1-channel binary segmentation mask highlighting leaf and disease lesions.

### ResNet-18 (Disease Classification)
- **Backbone**: ResNet-18 architecture with transfer learning.
- **Input**: Segmented leaf region cropped from U-Net mask output.
- **Output**: Multi-class softmax classification (e.g., Tomato Early Blight, Tomato Late Blight, Rice Blast, Potato Early Blight, Healthy Leaf) with confidence scores and top alternative predictions.

---

## 5. Market Price Module Architecture
- **API Service Layer**: `backend/services/market_price_service.py` handles communication with Data.gov.in / Agmarknet API.
- **Database Caching**: Stores historical records in SQLite (`market_prices` table) indexed by `commodity`, `state`, `district`, `market`, `price_date`.
- **Demo Fallback**: If `MARKET_PRICE_API_KEY` is not provided or API request fails, returns benchmark data explicitly tagged as `DEMO DATA`.

---

## 6. Project Directory Structure

```
crop-major-project/
├── backend/
│   ├── app.py                      # Flask Application Entry Point
│   ├── config.py                   # Environment Configuration
│   ├── requirements.txt            # Python Dependencies
│   ├── database/
│   │   ├── db.py                   # SQLAlchemy Init
│   │   └── models.py               # MarketPrice, History DB Models
│   ├── models/
│   │   ├── unet_model.py           # U-Net Architecture & Preprocessing
│   │   ├── resnet_model.py         # ResNet Classifier & Disease Database
│   │   └── weights/                # Model Weights Checkpoints (.pth)
│   ├── routes/
│   │   ├── crop_routes.py          # /api/crop-recommendation
│   │   ├── disease_routes.py       # /api/disease-detection
│   │   ├── market_routes.py        # /api/market-prices
│   │   └── health_routes.py        # /api/health
│   ├── services/
│   │   ├── crop_service.py         # Price-Aware Scoring Algorithm
│   │   ├── disease_service.py      # U-Net + ResNet Pipeline
│   │   └── market_price_service.py # Agmarknet API & Demo Fallback
│   ├── train/
│   │   ├── train_unet.py           # Standalone U-Net Trainer
│   │   └── train_resnet.py         # Standalone ResNet Trainer
│   └── tests/                      # Pytest Automated Test Suite
├── src/
│   ├── components/                 # Reusable React UI Components
│   │   ├── MarketPriceCard.jsx
│   │   ├── MarketPriceTable.jsx
│   │   ├── MarketPriceChart.jsx
│   │   ├── MarketFilters.jsx
│   │   ├── CropRecommendationCard.jsx
│   │   └── DiseaseResultCard.jsx
│   ├── pages/                      # Page Views
│   │   ├── Dashboard.jsx
│   │   ├── CropSelection.jsx
│   │   ├── DiseaseId.jsx
│   │   ├── MarketPrices.jsx
│   │   └── Solutions.jsx
│   ├── App.jsx                     # Protected Router & Navbar
│   └── main.jsx
├── .env.example                    # Environment Template
├── vite.config.js                  # Vite Proxy for Flask Backend
└── package.json
```

---

## 7. Environment Variables (`.env`)

Copy `.env.example` to `.env` in the root directory:

```env
SECRET_KEY=agrismart-secret-key-2026
PORT=5000
DATABASE_URL=sqlite:///agrismart.db

# Official Agricultural Market Price API (e.g. Data.gov.in / Agmarknet)
MARKET_PRICE_API_KEY=your_api_key_here
MARKET_PRICE_API_URL=https://api.data.gov.in/resource/9ef84268-d588-465a-a308-a864a43d0070

# Recommendation Weight Configuration
WEIGHT_SUITABILITY=0.60
WEIGHT_MARKET=0.25
WEIGHT_WEATHER=0.15
```

---

## 8. Installation & Setup

### Prerequisites
- Node.js (v18+)
- Python (v3.10+)

### Step 1: Install Frontend Dependencies
```bash
npm install
```

### Step 2: Install Backend Dependencies
```bash
pip install -r backend/requirements.txt
```

---

## 9. Running the Application

### Option A: Run Backend Server
```bash
 python -m backend.app
```
*Backend runs on `http://127.0.0.1:5000`*

### Option B: Run Frontend Development Server
In a separate terminal:
```bash
npm run dev
```
*Frontend runs on `http://localhost:5173` (requests to `/api/*` are proxied to Flask backend)*

---

## 10. Model Training (Offline Pipeline)

### How to Train U-Net (Leaf Region Segmentation)
```bash
python backend/train/train_unet.py --dataset_dir ./data/leaf_segmentation --epochs 25 --batch_size 16
```
*Saves trained weights to `backend/models/weights/unet_leaf.pth`*

### How to Train ResNet (Disease Classification)
```bash
python backend/train/train_resnet.py --dataset_dir ./data/plant_village --epochs 20 --batch_size 32
```
*Saves trained weights to `backend/models/weights/resnet_disease.pth`*

---

## 11. API Reference

| Method | Endpoint | Description |
| :--- | :--- | :--- |
| `GET` | `/api/health` | Health check & configuration status |
| `POST` | `/api/crop-recommendation` | Returns price-aware ranked crop list |
| `POST` | `/api/disease-detection` | Runs U-Net segmentation + ResNet diagnosis |
| `GET` | `/api/market-prices` | Returns current market prices (filtered by state/commodity) |
| `GET` | `/api/market-prices/history` | Returns historical price trend points for charting |
| `GET` | `/api/market-prices/commodities` | Returns list of available commodities |
| `GET` | `/api/market-prices/markets` | Returns list of available mandis |

---

## 12. Testing

Run backend tests using pytest:
```bash
python -m pytest backend/tests
```

---

## 13. Limitations & Disclaimers
1. **Price Indicators**: Revenue range calculations are based on historical market prices and estimated average yields; they do not guarantee net farmer profit.
2. **Pesticide Safety**: Chemical recommendations are general guidelines. Farmers should always consult local agricultural extensions and read official product labels before application.

---

## 14. Project Concepts & Video Reference
- **Reference Video**: [AI-Powered Smart Agriculture & Plant Disease Detection | ML Full-Stack Project](https://youtu.be/0P9lFWBVhBs?si=cc1zCQFrb74Z6shx)
- **Core Disease Detection Concepts**:
  - **39 Multi-Crop Plant Classes**: Supports leaf disease identification across 15 agricultural crops (Apple, Blueberry, Cherry, Corn, Grape, Citrus, Peach, Pepper Bell, Potato, Raspberry, Soybean, Squash, Strawberry, Tomato, Wheat).
  - **U-Net + ResNet Hybrid AI**: Combines U-Net binary leaf segmentation mask visualization with ResNet CNN classifier backbone.
  - **Comprehensive Diagnostic Output**: Outputs explicit confidence score percentage, severity rating (None, Low, Medium, High, Critical), pathogen identification, visual symptom checklist, organic remedies, chemical dosages, long-term prevention, and irrigation/fertilizer guidance.
