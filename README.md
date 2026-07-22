# SmartInventory AI

## Universal dataset support

Uploaded CSV/XLSX files now pass through `src/schema_adapter.py` before the
existing workflow. The adapter maps industry-specific column names into a
canonical inventory schema using exact aliases, fuzzy matching and optional
Gemini mapping.

Recommended source attributes:

- Product ID/SKU and product name
- Category, brand and supplier
- Current stock and reorder point
- Purchase/cost price and selling price
- Daily/monthly sales or demand
- Expiry date and batch/lot number
- Warehouse/store/location
- Supplier lead time

The upload page displays compatibility, mapped/derived/defaulted fields and
supported analyses. When core fields are absent, the pipeline-safe defaults are
explicitly reported and financial outputs must not be interpreted as factual.

## Gemini business assistant

The app loads Gemini only from the root `.env` file. Open `.env`, paste your
key after the equals sign, save, and restart Streamlit:

```env
GEMINI_API_KEY=paste_your_real_key_here
GEMINI_MODEL=gemini-3.1-flash-lite
```

The API key is intentionally not displayed or accepted in the sidebar. The AI
Business Assistant receives a compact context containing the
uploaded schema, top risk products, computed KPIs and recommendations, plus the
recent conversation. It can answer follow-up questions without repeating the
same generic summary and never invents company-specific numbers.

## Login and startup

Default local login values are stored in `.env` and should be changed before
deployment:

```env
APP_USERNAME=admin
APP_PASSWORD=ChangeMe123!
```

Windows PowerShell:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
python -m streamlit run streamlit/app.py
```

## Business KPI definitions

- Revenue at Risk: selling value of stock predicted as High or Medium expiry risk.
- Expected Recovery: expiry-risk value remaining after the recommended markdown
  (60% for High risk, 80% for Medium risk), capped at Revenue at Risk.
- Waste Reduction Rate: Expected Recovery divided by Revenue at Risk.
- Inventory Health: 0–100 model-derived score based on stock status, expiry risk
  and inventory turnover.

### AI-Powered Revenue Recovery and Inventory Intelligence System

SmartInventory AI is an enterprise-grade **Decision Support System (DSS)** designed to protect retail, grocery, and pharmaceutical supply chains from revenue decay. The platform automates data engineering pipelines, runs machine learning classifiers and regressors, evaluates capital risks, generates strategic business decisions, and updates dashboard systems.

---

## 🛠️ Project Workflow

```text
       Raw Dataset
            │
            ▼
      Data Cleaning (data_cleaning.py)
            │
            ▼
      Cleaned Dataset (SmartInventory_AI_Cleaned_Dataset.csv)
            │
            ▼
     Feature Engineering (feature_engineering.py)
            │
            ▼
      ML-Ready Dataset (SmartInventory_AI_Feature_Engineered.csv)
            │
            ▼
    ┌───────┴──────────────────────────┐
    ▼                                  ▼
EDA Visualizations (eda.py)     Model Training (train_models.py)
                                       │
                                       ▼
                               Save Models (.pkl)
                                       │
                                       ▼
                              Predict (prediction.py)
                                       │
                                       ▼
                         AI Recommendation Engine (recommendation_engine.py)
                                       │
                                       ▼
                           Store Results (database.py)
                                       │
                                       ▼
                       ┌───────────────┼───────────────┐
                       ▼               ▼               ▼
                 Excel Report    Power BI Visuals  Streamlit UI
```

---

## 📂 Folder Structure

```text
Teamproject/
│
├── data/
│   ├── raw/                      # Raw dataset input
│   ├── cleaned/                  # Cleaned, standardized dataset
│   └── processed/                # Feature augmented & predictions files
│
├── models/                       # Serialized machine learning models (.pkl)
│
├── reports/
│   └── figures/                  # Saved analytical visualizations
│
├── dashboards/
│   ├── excel/                    # Programmatically generated Excel reports
│   └── powerbi/                  # Power BI layouts and DAX specifications
│
├── src/                          # Back-end modules
│   ├── __init__.py
│   ├── generate_synthetic_data.py # Prepopulate mock raw datasets
│   ├── data_cleaning.py          # Duplicates & Null remediation pipeline
│   ├── feature_engineering.py     # Pareto ABC and health score compiler
│   ├── eda.py                    # Visual analytics generator
│   ├── train_models.py           # ML training script
│   ├── prediction.py             # Inference pipeline
│   ├── recommendation_engine.py  # Strategic business recommendation builder
│   ├── database.py               # SQL database connection schema
│   └── generate_excel_dashboard.py # Programmatic Excel dashboard builder
│
├── streamlit/
│   └── app.py                    # Streamlit web application
│
├── docs/                         # Project documentation
│
├── requirements.txt              # Environment dependencies
├── .gitignore                    # Version control exclusions
└── README.md                     # Project documentation
```

---

## 🚀 Setup & Execution Instructions

### 1. Prerequisites
Ensure you have Python 3.10+ installed on your system.

### 2. Environment Setup
Install the required packages in your local Python environment:
```bash
pip install -r requirements.txt
```

### 3. Database Configurations
By default, the database connector module (`src/database.py`) runs on a local **SQLite** server instance (`smart_inventory.db` created in the project root) for seamless, zero-config final-year demonstrations.

To connect the application to a production **MySQL** database server, configure the environment variable before execution:
* **Windows (PowerShell)**:
  ```powershell
  $env:MYSQL_DATABASE_URL="mysql+mysqlconnector://<username>:<password>@<host>:<port>/<db_name>"
  ```
* **Linux/macOS**:
  ```bash
  export MYSQL_DATABASE_URL="mysql+mysqlconnector://<username>:<password>@<host>:<port>/<db_name>"
  ```

### 4. Running the Streamlit Application
Launch the Streamlit web application interface:
```bash
streamlit run streamlit/app.py
```

---

## 🤖 Machine Learning Modules

The system trains and utilizes four separate machine learning models to analyze different aspects of the inventory:

1. **Expiry Risk Classifier (Random Forest)**:
   * **Target**: Expiry Risk (`High`, `Medium`, `Low`, `None`).
   * **Purpose**: Identifies decaying stock groups by evaluating costs, selling prices, categories, and inventory turnover levels.
2. **Demand Forecast (XGBoost Regressor)**:
   * **Target**: `Sales_Velocity` (Daily units sales).
   * **Purpose**: Models non-linear patterns of product sales velocities based on category, price thresholds, and current stock sizes.
3. **Revenue Forecast (Linear Regression)**:
   * **Target**: `Revenue` ($).
   * **Purpose**: Projects future revenue potential by mapping cost-to-sell ratios and sales velocities.
4. **Inventory Segmentation (K-Means Clustering)**:
   * **Purpose**: Segments products into performance clusters (Fast Moving High Value, Slow Moving High Value, Fast Moving Low Value, Dead Stock) based on turnover, profit margins, and stock value.

---

## 💡 AI Recommendation Engine

Rather than simply predicting values, the engine applies model outputs to generate business-ready recommendations:

* **Discount Recommendation**: Applies calculated markdowns (e.g., 20% or 40%) for products near expiry to accelerate demand velocity and prevent total cost write-off.
* **Transfer Recommendation**: Recommends transferring stock from overstocked locations to other regions with higher demand margins to save carrying costs.
* **Reorder Recommendation**: Alerts procurement when stock is below the minimum threshold and automatically projects the optimal restocking volume.
* **Overstock Recommendation**: Identifies overstock issues and recommends clearance sales to free up tied-up capital.
* **Supplier Recommendation**: Highlights lead time review plans for suppliers of high-velocity, low-stock products.
* **Purchase Planning Recommendation**: Advises establishing bulk contracts for high-demand products to optimize unit costs.
