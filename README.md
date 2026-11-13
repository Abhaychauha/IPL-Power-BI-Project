# 📱 App Insights Unlocked: Google Play Store Analytics

**What makes an app succeed on the Google Play Store?** An end-to-end data analytics project that cleans ~10,800 Play Store records, answers 25 business questions in Python/pandas, and presents the findings in an interactive two-page **Power BI** dashboard.

![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-F2C811?logo=powerbi&logoColor=black)
![Python](https://img.shields.io/badge/Python-pandas-3776AB?logo=python&logoColor=white)
![Dataset](https://img.shields.io/badge/Data-Kaggle-20BEFF?logo=kaggle&logoColor=white)

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Business Questions](#-business-questions)
- [Dataset](#-dataset)
- [Repository Structure](#-repository-structure)
- [Data Cleaning](#-data-cleaning)
- [Dashboard](#-dashboard)
- [Key Insights](#-key-insights)
- [Recommendations](#-recommendations)
- [How to Run](#-how-to-run)
- [Limitations](#-limitations)
- [Tools and Skills](#-tools-and-skills)
- [Author](#-author)

---

## 🎯 Project Overview

**Scenario.** You are a data analyst at a mobile-app company that has acquired Google Play Store data. Product managers, developers, marketers and senior management want to know what drives app ratings, installs and reviews, so they can improve their own app strategy.

**Objectives**
1. Identify the factors that contribute to app success.
2. Show how the company can use those factors to improve its apps.
3. Understand user preferences and behaviour.

**Stakeholders:** app developers, product managers, marketing, senior management (internal); app users, advertisers, partners (external).

---

## ❓ Business Questions

The analysis answers **25 questions** (10 basic, 10 medium, 5 advanced), including:

- What is the average rating, and how many apps score 4.0 or higher?
- Which categories and genres have the highest ratings and the most installs?
- How do **price** and **app size** affect ratings, reviews and installs?
- How does the rating change as an app gains installs (binned analysis)?
- How often are apps updated, and are there update trends?

👉 Full answers with code, explanations and business impact: **[Solution_Guide.md](Solution_Guide.md)**

---

## 📂 Dataset

**Source:** [Google Play Store Apps, Kaggle (lava18)](https://www.kaggle.com/datasets/lava18/google-play-store-apps) · file `googleplaystore.csv`
**Snapshot date:** August 2018 (latest `Last Updated` = 8 Aug 2018)

| Column | Description |
|---|---|
| `App` | App name |
| `Category` | Store category (33 values) |
| `Rating` | Average user rating (0-5) |
| `Reviews` | Number of user reviews |
| `Size` | App size (MB / kB / "Varies with device") |
| `Installs` | Install bucket, e.g. `1,000+` |
| `Type` | Free or Paid |
| `Price` | Price in USD |
| `Content Rating` | Target age group |
| `Genres` | Genre(s), separated by `;` |
| `Last Updated` | Date of latest update |
| `Current Ver` | App version |
| `Android Ver` | Minimum Android version |

| Raw | After cleaning |
|---|---|
| 10,841 rows | **9,659 unique apps** (8,196 with a rating) |

---

## 🗂 Repository Structure

```
.
├── README.md
├── Solution_Guide.md                 # Detailed answers to all 25 questions
├── data/
│   └── googleplaystore.csv           # Raw dataset
├── dashboard/
│   └── Googleplaystore_Dashboard.pbix
├── docs/
│   └── App_Insights__PowerBI_.pdf    # Case study / problem statement
└── images/                           # Dashboard screenshots (add yours here)
    ├── page1_overview.png
    └── page2_installs.png
```

---

## 🧹 Data Cleaning

| Step | Action |
|---|---|
| 1 | Removed one corrupted row (columns shifted; `Category = "1.9"`) |
| 2 | Removed **483** exact duplicate rows |
| 3 | De-duplicated by `App`, keeping the row with the most reviews → **9,659** apps |
| 4 | Converted `Installs` (`"1,000+"` → `1000`), `Price` (`"$4.99"` → `4.99`), `Reviews` to numbers |
| 5 | Converted `Size` to MB (`k` ÷ 1024); `"Varies with device"` set to null |
| 6 | Parsed `Last Updated` to a date and derived `Year` |
| 7 | Filled the single missing `Type` with `Free`; left missing `Rating` empty (not imputed) |
| 8 | Standardized `Category` / `Genres` text for display |

---

## 📊 Dashboard

An interactive **Power BI** report (`dashboard/Googleplaystore_Dashboard.pbix`) with two pages and shared slicers for **Category, Genres, Year and Content Rating**.

**Page 1: Overview**
- KPI cards: Total Apps, Average Rating, Total Installs, Total Reviews, Free Apps, Paid Apps
- Donut chart: Free vs Paid
- Pie chart: Content Rating split
- Bar chart: number of apps per category
- Table: Average Rating, Total Installs and Total Reviews by category

**Page 2: Installs and Engagement**
- Treemap: installs by category
- Decomposition tree: Total Installs explained by Type, Genres, Category and Content Rating
- Column charts: installs and reviews by app
- Ribbon chart: category rank by installs over the years

> 📸 *Add screenshots to `images/` and uncomment the lines below.*
>
> <!-- ![Overview](images/page1_overview.png) -->
> <!-- ![Installs](images/page2_installs.png) -->

**Core DAX measures:** `Total Apps`, `Average Rating`, `Total Installs`, `Total Reviews`, `Free Apps`, `Paid Apps`.

---

## 💡 Key Insights

| # | Insight | Evidence |
|---|---|---|
| 1 | The typical app is rated **4.17**; 77% of rated apps score ≥ 4.0 | 8,196 rated apps |
| 2 | The store is **92% free** | 8,905 free vs 754 paid |
| 3 | **Games and Communication** drive installs | 17.9% and 14.7% of all installs |
| 4 | **Events, Art & Design, Education, Books** rate highest; **Dating, Maps, Tools** lowest | 4.44-4.35 vs 3.98-4.04 |
| 5 | Rating and installs are **barely correlated** (r ≈ 0.04), but the curve is **U-shaped**: ratings dip at 1K-10K installs (4.03) and climb to 4.36 at 10M-100M | Binned analysis |
| 6 | Free apps have **~15x more median reviews** and **100x higher median installs** than paid | 1,379 vs 89 reviews; 100K vs 1K installs |
| 7 | **Price $1-5** is the paid sweet spot; ratings fall above $20 (driven by novelty "I am Rich" apps) | 4.30 → 3.99 |
| 8 | **Recently updated** apps do better | ≤ 30 days: 4.27 rating, 1M median installs; > 1 year: 4.06, 10K |
| 9 | **Bigger apps get more installs**, mostly because of budget and features, not size itself | Median 5K installs (≤ 5 MB) → 1M (50-100 MB) |
| 10 | Perfect **5.0 ratings are misleading**: the median 5.0-star app has just 4 reviews | 271 apps |

Full tables and explanations are in the [Solution Guide](Solution_Guide.md).

---

## ✅ Recommendations

1. **Launch free** with ads or in-app purchases; if paid, price at **$1-5**.
2. **Choose categories deliberately**: Games/Communication for reach, Education/Books/Art for quality-led wins.
3. **Set a 4.2 rating target**; treat < 4.0 as a red flag.
4. **Prepare for the 1K-10K install "valley"** with QA, support and review replies.
5. **Update every 1-3 months**, on weekdays.
6. **Prompt for reviews** after positive moments.
7. **Budget app size per category** (games ≈ 40 MB, tools ≈ 9 MB).
8. **Next phase:** add review-text sentiment analysis using `googleplaystore_user_reviews.csv`.

---

## 🚀 How to Run

**Python analysis**

```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
pip install pandas numpy matplotlib seaborn jupyter
```

```python
import pandas as pd
df = pd.read_csv("data/googleplaystore.csv")
# Follow the cleaning code in Solution_Guide.md, Section 1.3
```

**Power BI dashboard**

1. Install [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (Windows).
2. Open `dashboard/Googleplaystore_Dashboard.pbix`.
3. If prompted, update the data source path: **Transform data → Data source settings → Change source** and point it to `data/googleplaystore.csv`.

---

## ⚠️ Limitations

- Single **August 2018** snapshot, so no time series of ratings or installs.
- **Installs are buckets** (`1,000+`), so totals are lower-bound estimates.
- **15% of apps have no rating** and are excluded from rating analyses.
- Correlations (updates, size, reviews) show **association, not causation**.
- The dataset has **no review text**, so sentiment analysis needs the companion file.
- Update frequency can only be approximated, since only the latest update date is recorded.

---

## 🛠 Tools and Skills

`Power BI` · `DAX` · `Python` · `pandas` · `NumPy` · `Data cleaning` · `Exploratory data analysis` · `Binned and correlation analysis` · `Data visualization` · `Business storytelling`

---

## 👤 Author

**Your Name**
[LinkedIn](https://linkedin.com/in/your-profile) · [GitHub](https://github.com/your-username) · your.email@example.com

⭐ If you found this project useful, consider giving it a star.
