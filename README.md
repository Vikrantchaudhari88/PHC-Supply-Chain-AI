# \# 🏥 AI-Driven PHC Supply Chain Optimization with NHSRC Compliance

# 

# \## 🎯 Project Overview

# This project develops an AI-driven forecasting and inventory optimization engine for Primary Healthcare Centre (PHC) medicine supply chains, fully aligned with \*\*NHSRC (National Health Systems Resource Centre)\*\* guidelines. The system enables predictive inventory management, reduces stockouts of essential medicines, and minimizes wastage through expiry-aware optimization.

# 

# \## 📊 Project Journey: 6 Days of Intensive Development

# 

# \### \*\*Day 1-2: Data Foundation \& NHSRC Compliance\*\*

# \*\*Objective:\*\* Establish clean, standardized data pipeline with NHSRC compliance.

# 

# \#### \*\*Key Components:\*\*

# 1\. \*\*Data Cleaning Pipeline\*\* (`02\_DATA\_CLEANING\_PIPELINE.ipynb`)

# &nbsp;  - Standardized column names and date formats (DD-MM-YYYY → datetime)

# &nbsp;  - Handled missing values with strategic imputation:

# &nbsp;    - `units\_used`: Replaced with 0 (no consumption recorded)

# &nbsp;    - `on\_hand`: Forward-filled by SKU (carry last known stock)

# &nbsp;    - `lead\_time\_days`: Filled with median by SKU

# &nbsp;  - Outlier detection using 3σ rule (flagged, not removed to preserve outbreak signals)

# &nbsp;  - Generated derived fields:

# &nbsp;    - `days\_cover` = `on\_hand` / (7-day rolling average of `units\_used`)

# &nbsp;    - `expiry\_days\_remaining` = `batch\_expiry\_date` - `date`

# &nbsp;    - `expiry\_risk\_bucket`: NHSRC categories (CRITICAL<30d, HIGH30-90d, MEDIUM90-180d, LOW>180d)

# 

# \#### \*\*NHSRC Formulas Implemented:\*\*

# \- \*\*ADC (Average Daily Consumption)\*\*: `Σ(Daily Usage) / Number of Days`

# \- \*\*Safety Stock\*\*: `ADC × Buffer Days` (Vital:10, Essential:7, Desirable:3)

# \- \*\*ROL (Reorder Level)\*\*: `Max Daily Consumption × Max Lead Time + Safety Stock`

# \- \*\*MSL (Minimum Stock Level)\*\*: `ROL - (ADC × Average Lead Time)`

# 

# \*\*Output:\*\* `data/cleaned\_inventory.csv` with 6,480 standardized records

# 

# ---

# 

# \### \*\*Day 3: NHSRC Analytics \& Stock Health Matrix\*\*

# \*\*Objective:\*\* Create comprehensive analytics with NHSRC framework integration.

# 

# \#### \*\*Analytical Layers:\*\*

# 1\. \*\*VED Analysis\*\* (Vital, Essential, Desirable)

# &nbsp;  - Consumption patterns by criticality

# &nbsp;  - Stock coverage by priority level

# &nbsp;  - Risk heatmaps: VED × Expiry Risk

# 

# 2\. \*\*FSN Analysis\*\* (Fast, Slow, Non-moving)

# &nbsp;  - Turnover ratios by movement category

# &nbsp;  - Velocity distribution analysis

# &nbsp;  - FSN-VED matrix for strategic positioning

# 

# 3\. \*\*FEFO Expiry Risk Analysis\*\*

# &nbsp;  - Financial risk quantification by expiry bucket

# &nbsp;  - Monthly expiry projections

# &nbsp;  - Critical SKU identification (high risk + high usage)

# 

# 4\. \*\*Stock Health Matrix\*\* (Core NHSRC Metric)

# &nbsp;  - Per-SKU metrics: ADC, current stock, lead time, ROL, MSL

# &nbsp;  - Coverage gap: `days\_cover - lead\_time\_days`

# &nbsp;  - Action severity score (0-100):

# &nbsp;    - Days cover component (0-40 points): <7d=40, <14d=30, <30d=20, <60d=10

# &nbsp;    - VED component (0-30 points): Vital=30, Essential=20, Desirable=10

# &nbsp;    - Coverage gap component (0-30 points): negative=30, <7=20, <14=10

# &nbsp;  - Risk categorization: Critical(≥70), Warning(50-69), Caution(30-49), Safe(<30)

# 

# \*\*Outputs:\*\* 

# \- `reports/stock\_health\_matrix.csv` (NHSRC-compliant risk assessment)

# \- `reports/ved\_summary.csv`, `reports/fsn\_summary.csv`, `reports/expiry\_summary.csv`

# \- Visualizations for all analytical layers

# 

# ---

# 

# \### \*\*Day 4: Feature Engineering for AI Forecasting\*\*

# \*\*Objective:\*\* Transform raw data into ML-ready features for time series forecasting.

# 

# \#### \*\*Feature Engineering Pipeline:\*\*

# 1\. \*\*Continuous Time Series Creation\*\*

# &nbsp;  - Aggregated duplicate SKU-date entries

# &nbsp;  - Created continuous daily series (180 days × 12 SKUs = 2,160 records)

# &nbsp;  - Forward-filled stock levels, zero-filled consumption gaps

# 

# 2\. \*\*Temporal Features\*\*

# &nbsp;  - `day\_of\_week`, `is\_weekend`, `day\_of\_month`, `month`, `week\_of\_year`

# 

# 3\. \*\*Lag Features\*\* (Forecasting inputs)

# &nbsp;  - `lag\_1`, `lag\_3`, `lag\_7`, `lag\_14`, `lag\_30`

# 

# 4\. \*\*Rolling Statistical Features\*\*

# &nbsp;  - `rolling\_mean\_7/14/30`, `rolling\_std\_7/14/30`

# &nbsp;  - `rolling\_cv\_7/14` (Coefficient of Variation = std/mean)

# 

# 5\. \*\*Derived Features\*\*

# &nbsp;  - `consumption\_trend\_7d` (slope of last 7 days)

# &nbsp;  - `day\_over\_day\_pct\_change`

# 

# \#### \*\*Business Logic:\*\*

# \- \*\*Lag features\*\* capture recent consumption patterns

# \- \*\*Rolling statistics\*\* smooth volatility and identify trends

# \- \*\*CV metrics\*\* quantify demand stability for forecast confidence

# \- \*\*Weekend/holiday flags\*\* account for PHC operation patterns

# 

# \*\*Outputs:\*\* 

# \- `data/forecast\_ready\_timeseries.csv` (37 features, 2,160 records)

# \- `data/sku\_level\_features.csv` (SKU-level summary for model selection)

# 

# ---

# 

# \### \*\*Day 5: Baseline Forecasting \& Model Benchmarking\*\*

# \*\*Objective:\*\* Establish forecasting baselines and select best model per SKU.

# 

# \#### \*\*Model Portfolio:\*\*

# 1\. \*\*Naive Forecast\*\*: `forecast = last\_observed\_value`

# &nbsp;  - Baseline control model

# &nbsp;  - Shows minimum acceptable accuracy

# 

# 2\. \*\*Moving Average\*\*: `forecast = mean(last\_window\_values)`

# &nbsp;  - Window sizes: 7, 14, 30 days

# &nbsp;  - Smooths volatility, captures trends

# &nbsp;  - 7-day: Weekly patterns, 30-day: Monthly trends

# 

# 3\. \*\*Exponential Smoothing (ETS)\*\*: Adaptive weighting

# &nbsp;  - Formula: `S\_t = α × Y\_t + (1-α) × S\_{t-1}`

# &nbsp;  - α=0.3 gives more weight to recent observations

# &nbsp;  - Handles level shifts without trend/seasonality assumptions

# 

# \#### \*\*Evaluation Metric:\*\*

# \- \*\*MAPE (Mean Absolute Percentage Error)\*\*: 

# MAPE = (1/n) × Σ(|Actual - Forecast| / |Actual|) × 100%

# 

# text

# \- Skip first 30 days to avoid initialization bias

# \- Handle zero actual values (replace with 0.1 to avoid division by zero)

# 

# \#### \*\*Results Interpretation:\*\*

# \- \*\*ETS dominated (6 SKUs)\*\*: Data has clear level shifts + variability

# \- \*\*MA models won (5 SKUs)\*\*: Weekly cycles or long smoothing windows

# \- \*\*Naive won (1 SKU)\*\*: Minimal pattern, validates data authenticity

# \- \*\*MAPE range 14-15%\*\*: Excellent for PHC forecasting (<6 month window)

# 

# \*\*Outputs:\*\*

# \- `reports/model\_performance\_baseline.csv` (Model leaderboard by MAPE)

# \- `reports/best\_model\_selection.csv` (Best model per SKU with confidence)

# 

# ---

# 

# \### \*\*Day 6: Replenishment Policy Engine\*\*

# \*\*Objective:\*\* Transform forecasts into actionable procurement decisions.

# 

# \#### \*\*Decision Logic Framework:\*\*

# 

# 1\. \*\*14-Day Demand Forecast\*\*

# forecast\_14d = best\_model(historical\_data) × 14

# 

# text

# 

# 2\. \*\*Stock Coverage Calculation\*\*

# expected\_days\_available = current\_stock / (forecast\_14d / 14)

# 

# text

# 

# 3\. \*\*Reorder Trigger\*\*

# reorder\_flag = "YES" if expected\_days\_available < lead\_time\_days

# 

# text

# 

# 4\. \*\*Order Quantity\*\*

# recommended\_order\_qty = max(ROL - current\_stock, 0)

# 

# text

# 

# 5\. \*\*NHSRC Priority Tiering\*\*

# \- \*\*URGENT REPLENISH\*\*: Vital + <7 days stock

# \- \*\*PRIORITY 1\*\*: Vital + below ROL

# \- \*\*PRIORITY 2\*\*: Essential + below ROL

# \- \*\*PRIORITY 3\*\*: Desirable + below ROL

# \- \*\*REDISTRIBUTE\*\*: High expiry risk (<30d)

# \- \*\*MONITOR EXCESS\*\*: >90 days cover

# \- \*\*HOLD HEALTHY\*\*: Good coverage + safety stock

# 

# 6\. \*\*Forecast Confidence\*\*

# \- \*\*HIGH\*\*: CV < 0.3 (stable demand)

# \- \*\*MEDIUM\*\*: CV 0.3-0.6 (moderate variability)

# \- \*\*LOW\*\*: CV > 0.6 (highly volatile)

# 

# \#### \*\*Business Impact:\*\*

# \- \*\*Stockout Prevention\*\*: Proactive reordering before critical levels

# \- \*\*Waste Reduction\*\*: Expiry-aware redistribution

# \- \*\*Capital Optimization\*\*: Avoid overstocking slow-moving items

# \- \*\*PHC Manager Focus\*\*: Prioritize Vital medicines

# 

# \*\*Output:\*\* `reports/replenishment\_recommendations.csv` (Actionable procurement plan)

# 

# ---

# 

# \## 📁 Repository Structure

# phc-supply-chain-ai/

# ├── data/ # Structured datasets

# │ ├── sample\_inventory.csv # Raw transactional data

# │ ├── cleaned\_inventory.csv # Standardized data (Day 2)

# │ ├── medicine\_master.csv # Medicine classifications

# │ ├── forecast\_ready\_timeseries.csv # ML-ready features (Day 4)

# │ └── sku\_level\_features.csv # SKU-level summaries (Day 4)

# ├── notebooks/ # Analytical pipeline

# │ ├── 02\_DATA\_CLEANING\_PIPELINE.ipynb # Day 2

# │ ├── 03\_EDA\_NHSRC\_ANALYSIS.ipynb # Day 3

# │ ├── 04\_FEATURE\_ENGINEERING\_FORECASTING.ipynb # Day 4

# │ ├── 05\_BASELINE\_FORECASTING.ipynb # Day 5

# │ └── 06\_REPLENISHMENT\_POLICY\_ENGINE.ipynb # Day 6

# ├── reports/ # Analysis outputs

# │ ├── stock\_health\_matrix.csv # NHSRC risk assessment (Day 3)

# │ ├── model\_performance\_baseline.csv # Model benchmarks (Day 5)

# │ ├── best\_model\_selection.csv # Best model per SKU (Day 5)

# │ └── replenishment\_recommendations.csv # Procurement plan (Day 6)

# ├── docs/

# │ └── data\_quality\_report.md # Data validation report

# └── README.md # This documentation

# 

# text

# 

# ---

# 

# \## 🔧 Technical Stack

# \- \*\*Data Processing\*\*: Pandas, NumPy

# \- \*\*Visualization\*\*: Matplotlib, Seaborn

# \- \*\*Forecasting\*\*: Custom implementations (Naive, MA, ETS)

# \- \*\*Evaluation\*\*: scikit-learn (MAPE)

# \- \*\*Version Control\*\*: Git, GitHub

# \- \*\*Environment\*\*: Python 3.9+, Jupyter Notebooks

# 

# ---

# 

# \## 🎯 Business Impact Metrics

# 1\. \*\*Stockout Risk Reduction\*\*: 11/12 SKUs now have proactive reorder triggers

# 2\. \*\*Forecast Accuracy\*\*: 14-15% MAPE (beats industry 25-40% benchmark)

# 3\. \*\*Waste Prevention\*\*: Expiry risk categorization for 100% of batches

# 4\. \*\*Capital Efficiency\*\*: Priority-based ordering optimizes inventory investment

# 5\. \*\*PHC Manager Empowerment\*\*: Clear action priorities for 12 medicines

# 

# ---

# 

# \## 🚀 Next Steps

# 1\. \*\*Day 7\*\*: Model monitoring dashboard + alert simulation

# 2\. \*\*Pipeline Automation\*\*: Schedule daily runs with new data

# 3\. \*\*Dashboard Integration\*\*: Power BI/Tableau for PHC managers

# 4\. \*\*Advanced Models\*\*: Prophet, XGBoost, LSTM for improved accuracy

# 5\. \*\*Multi-Facility Scaling\*\*: Extend to network of PHCs

# 

# ---

# 

# \## 📋 NHSRC Compliance Status

# | Component | Status | Notes |

# |-----------|--------|-------|

# | VED Analysis | ✅ Complete | Vital/Essential/Desirable classification |

# | FSN Analysis | ✅ Complete | Fast/Slow/Non-moving velocity tracking |

# | FEFO Logic | ✅ Complete | Expiry-aware stock rotation |

# | Safety Stock | ✅ Complete | VED-based buffer calculation |

# | ROL/MSL Formulas | ✅ Complete | Official NHSRC calculations |

# | Forecasting | ✅ Complete | Multi-model benchmarked approach |

# | Decision Engine | ✅ Complete | NHSRC priority tiering implemented |

# 

# ---

# 

# \## 👥 Contributors

# \- \*\*Vikrant\*\*: Project Lead, Data Engineering, AI Implementation

# \- \*\*NHSRC Guidelines\*\*: Framework compliance

# \- \*\*PHC Operational Data\*\*: Realistic synthetic dataset

# 

# ---

# 

# \*Last Updated: Day 6 - Replenishment Policy Engine Complete\*

