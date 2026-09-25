# 🛵 Rapido Ride Analytics — Data Cleaning, EDA & Power BI Dashboard

An end-to-end data analytics project on 50,000 Rapido ride records (bike, auto, cab economy, bike lite, and parcel services), covering data cleaning, feature engineering, exploratory data analysis in Python, and an interactive business dashboard built in Power BI.

---

## 📌 1. Project Overview

Rapido is a ride-hailing platform offering multiple service types — bikes, autos, cabs, and parcel delivery. This project analyzes two months of ride-level transaction data to understand **revenue drivers, demand patterns, and cancellation behavior**, then presents those findings as an interactive Power BI dashboard.

The work moves through four stages, each producing an artifact that feeds the next:

```
rides_data.csv  →  rides_Clean_data.csv  →  Rapido_Ride_Analytics.ipynb  →  rapido_final_data.csv  →  Rapido_Ride_Analytics.pbix
   (raw data)       (cleaned + features)         (Python EDA)               (analysis-ready data)      (Power BI dashboard)
```

---

## 🎯 2. Objective

The analysis was driven by a set of core business questions:

- Which services generate the most rides and the most revenue?
- How often do rides get cancelled, and does cancellation vary by service?
- How does ride demand vary by hour of day, day of week, and month?
- Are weekdays or weekends busier?
- Which pickup/drop locations and routes see the most traffic?
- Does ride distance affect fare and volume?
- Which payment methods are used, and how evenly?

---

## 🛠️ 3. Tools & Tech Stack

| Purpose | Tool |
|---|---|
| Data cleaning & feature engineering | Excel / manual preparation (produced `rides_Clean_data.csv`) |
| Exploratory data analysis & validation | Python — `pandas`, `numpy`, `matplotlib`, `seaborn` in Jupyter Notebook |
| Dashboard & reporting | Power BI Desktop |
| Data storage | CSV files |

---

## 📁 4. Repository Structure

```
├── rides_data.csv                  # Raw, untouched ride data (50,000 rows x 13 cols)
├── rides_Clean_data.csv            # Cleaned data with engineered features (50,000 rows x 20 cols)
├── Rapido_Ride_Analytics.ipynb     # Jupyter Notebook — full EDA, validation & analysis
├── rapido_final_data.csv           # Final analysis-ready export from the notebook
├── Rapido_Ride_Analytics.pbix      # Power BI dashboard (4 pages)
└── README.md                       # This file
```

---

## 🧭 5. Step-by-Step Workflow

### Step 1 — Raw Data (`rides_data.csv`)

The starting point is a raw export of **50,000 rides** with **13 columns**:

`services, date, time, ride_status, source, destination, duration, ride_id, distance, ride_charge, misc_charge, total_fare, payment_method`

Key facts about the raw file:
- **Date range:** June 17, 2024 → August 16, 2024 (~2 months)
- **Services:** `bike` (15,128), `auto` (12,327), `cab economy` (10,202), `parcel` (7,459), `bike lite` (4,884)
- **Ride status:** `completed` (44,964) vs `cancelled` (5,036)
- **Missing values:** `ride_charge`, `misc_charge`, `total_fare`, and `payment_method` each had exactly **5,036 nulls** — matching the cancelled-ride count exactly. This made sense on inspection: a cancelled ride never gets charged or paid for, so these fields were legitimately empty rather than corrupted data.

### Step 2 — Data Cleaning & Feature Engineering (`rides_Clean_data.csv`)

Before deep analysis, the raw data was cleaned and enriched into a 20-column dataset:

**Cleaning:**
- `ride_charge`, `misc_charge`, and `total_fare` were filled with **0** for cancelled rides, since a cancelled ride genuinely has no fare — this avoids dropping 10% of the data while keeping the numbers mathematically honest (0 fare, not a missing/unknown fare).
- `payment_method` was deliberately left null at this stage rather than guessed at, so it could be handled explicitly and visibly during the notebook analysis (see Step 3.5).

**Feature engineering** — 7 new columns were added to support time, geography, and distance-based analysis that the raw fields couldn't answer on their own:

| New Column | Purpose |
|---|---|
| `Hour` | Extracted from `time`, to analyze ride demand by hour of day |
| `day_name` | Day of week (Monday–Sunday), to compare demand across days |
| `month`, `month_number` | To track monthly ride/revenue trends |
| `day_type` | `Weekday` vs `Weekend` flag, to compare usage patterns |
| `Route` | `source > destination` combined string, to analyze demand at the route level |
| `Distance_Category` | Distance binned into `Short` (1–3 km), `Medium` (3–7 km), `Long` (7–15 km), `Very Long` (15–50 km), to segment rides by trip length |

### Step 3 — Exploratory Data Analysis (`Rapido_Ride_Analytics.ipynb`)

The notebook picks up from `rides_Clean_data.csv` and works through inspection, validation, and analysis in this order:

**3.1 Setup** — imported `pandas`, `numpy`, `matplotlib`, `seaborn`.

**3.2 Load & inspect** — loaded the cleaned CSV and checked `.head()`, `.columns`, `.shape` (50,000 × 20), and `.info()` to confirm structure and dtypes before doing anything else.

**3.3 Missing value audit** — re-ran `.isnull().sum()` and confirmed the only remaining gap was `payment_method` (5,036 nulls, all cancelled rides).

**3.4 Handling `payment_method`** — filled the nulls with `'Not Applicable'` rather than dropping the rows or leaving them blank, so cancelled rides stay in the dataset for cancellation-rate analysis without polluting the payment-method breakdown of *completed* rides.

**3.5 Data type correction** — converted `date` to proper `datetime` and `time` to a time object, since the raw columns were plain strings and couldn't be used for any date/time-based grouping otherwise.

**3.6 Data integrity checks** — before trusting any number, the data was stress-tested:
- **Duplicate `ride_id` check** → 0 duplicates, confirming each ride is recorded exactly once.
- **Negative value check** on `distance`, `ride_charge`, `misc_charge`, `total_fare` → 0 negative values found.
- **Fare consistency check** — verified `total_fare == ride_charge + misc_charge` for every row → 0 mismatches, confirming the fare math is internally consistent.

**3.7 Descriptive statistics** — ran `.describe()` on both numeric and categorical columns to get a baseline feel for the data (e.g., average duration ≈ 64.3 minutes, average distance ≈ 25.5 km).

**3.8 Top-line KPIs** — computed the headline business numbers:

| Metric | Value |
|---|---|
| Total Revenue | ₹24,612,983.05 |
| Average Fare | ₹492.26 |
| Average Distance | 25.53 km |
| Average Duration | 64.32 min |
| Overall Cancellation Rate | 10.07% |

**3.9 Service-level analysis** — grouped by `services` to compare rides, completions, cancellations, revenue, average fare, distance, and duration per service, then visualized **Total Revenue by Service** and **Total Rides by Service** as bar charts. `bike` leads on both volume (15,128 rides) and revenue (₹74.3L); `cab economy` has the highest cancellation rate.

**3.10 Time-based analysis:**
- **Hourly demand** — extracted `Hour` and plotted a line chart of rides per hour. Demand turned out to be **remarkably flat across the day** (roughly 2,000–2,200 rides in every hour), with no sharp rush-hour spike.
- **Day-of-week demand** — bar chart by `day_name`. Monday was busiest (7,432 rides), Sunday quietest (6,540).
- **Weekday vs. weekend** — bar chart comparing `day_type`. Weekdays account for 36,876 rides vs. 13,124 on weekends (~74% / 26% split).
- **Monthly trend** — line charts of rides and revenue by month. July was the strongest month (25,552 rides, ₹1.25 Cr revenue) — expected, since it's the only fully-captured calendar month in the June 17–Aug 16 window.

**3.11 Route & location analysis:**
- **Route-level demand** (`source > destination`) — the busiest routes only had 2 rides each, showing the ~13,000 pickup/drop points are highly dispersed with almost no repeated point-to-point commuting pattern.
- **Top pickup locations** — bar chart of top 10 `source` values (busiest: Kothanur Landing, 23 rides).
- **Top destinations** — bar charts of top 10 `destination` values by both ride count and revenue.

**3.12 Distance category analysis** — grouped by `Distance_Category` and visualized rides and revenue per bucket. `Very Long` rides (15–50 km) dominate: 35,747 rides (71.5% of all rides) and ₹1.76 Cr (71% of all revenue).

**3.13 Payment method analysis** — bar charts of rides and revenue by `payment_method`. The four digital methods are almost perfectly split (Paytm, GPay, Amazon Pay, QR scan each ~11,200 rides / ~22% share), with no dominant option.

**3.14 Cancellation analysis:**
- Overall cancellation rate: **10.07%**.
- Countplot of completed vs. cancelled rides.
- Cancellation rate **by service**, sorted descending:

| Service | Cancellation Rate |
|---|---|
| cab economy | 10.33% |
| bike | 10.32% |
| bike lite | 10.16% |
| auto | 9.84% |
| parcel | 9.55% |

**3.15 Export** — the fully cleaned, validated, and feature-enriched dataframe was exported to `rapido_final_data.csv` (50,000 × 20), which became the single data source for the Power BI dashboard.

### Step 4 — Power BI Dashboard (`Rapido_Ride_Analytics.pbix`)

Python was used for deep, statistics-driven analysis and validation; Power BI was then used to turn the same findings into an **interactive, filterable dashboard** that non-technical stakeholders can explore on their own, built on top of `rapido_final_data.csv`.

The report has 4 pages:

**Page 1 — Executive Overview**
High-level KPI cards plus: Revenue by Service (bar), Monthly Revenue Trend (line), Ride Status Distribution (donut), Revenue Contribution by Service (bar), and 4 slicers so viewers can filter the whole page by service, date, status, etc.

**Page 2 — Operations & Demand**
KPI cards plus: Hourly Ride Demand (line), Cancellation Rate by Service (bar), Completed vs. Cancelled Rides by Service (column), Day-wise Ride Demand (column), Ride Distribution by Distance Category (bar), Average Ride Duration by Service (bar).

**Page 3 — Revenue & Business**
KPI cards plus: Revenue trend (line), Revenue by Service (bar), Revenue Contribution by Service (donut), and additional revenue breakdown charts.

**Page 4 — Map view**
A geographic map visual for exploring ride locations spatially.

### Step 5 — Key Insights

- **₹2.46 Cr** in total revenue from 50,000 rides at an average fare of **₹492**, over a two-month window.
- **`bike`** is the highest-volume and highest-revenue service; **`cab economy`** has the highest cancellation rate.
- The overall cancellation rate is **~10%**, and it's fairly uniform across services (9.5%–10.3%) — cancellations aren't concentrated in any one service type.
- Ride demand is **nearly flat across all 24 hours** — there's no strong rush-hour pattern in this data.
- **Weekdays drive ~74%** of all rides vs. 26% on weekends.
- **"Very Long" trips (15–50 km) dominate**, making up 71.5% of rides and 71% of revenue — this is a long-distance-heavy ride mix.
- Payment method usage is **evenly split** across four digital wallets, with no single preferred method.
- Demand is **geographically dispersed** — no single route or location acts as a major hub; most routes appear only once or twice in two months.

---

## ▶️ 6. How to Reproduce

1. Clone this repository.
2. Install dependencies:
   ```bash
   pip install pandas numpy matplotlib seaborn
   ```
3. Open `Rapido_Ride_Analytics.ipynb` and run all cells top to bottom — it reads `rides_Clean_data.csv` and produces `rapido_final_data.csv`.
4. Open `Rapido_Ride_Analytics.pbix` in **Power BI Desktop** to explore the interactive dashboard. If the data source path has changed, refresh the connection to point at `rapido_final_data.csv`.

---

## 👤 Author

**Ritik** — Data Analyst (Python, SQL, Excel/Power BI)
