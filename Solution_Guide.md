# App Insights Unlocked: Solution Guide

Detailed answers and explanations for every question in the *App Insights Unlocked: A Data Analytics Challenge* case study (Google Play Store dataset, Power BI + Python).

> **How to read this guide.** Every answer follows the same pattern: **Answer**, **How to get it** (pandas code), **Explanation**, and **Business impact / action**. All numbers below were computed from `googleplaystore.csv` using the cleaning rules in Section 1. Your Power BI figures may differ slightly if your Power Query steps de-duplicate differently (see the note in 1.3).

---

## Table of Contents

1. [Data Cleaning and Preprocessing](#1-data-cleaning-and-preprocessing)
2. [Basic-Level Questions (1-10)](#2-basic-level-questions)
3. [Medium-Level Questions (1-10)](#3-medium-level-questions)
4. [Advanced-Level Questions (1-5)](#4-advanced-level-questions)
5. [Metric Development](#5-metric-development)
6. [Key Findings and Recommendations](#6-key-findings-and-recommendations)
7. [Limitations](#7-limitations)
8. [Additional Resources](#8-additional-resources)

---

## 1. Data Cleaning and Preprocessing

### 1.1 What the raw file looks like

| Check | Result |
|---|---|
| Raw rows x columns | 10,841 x 13 |
| Missing `Rating` | 1,474 rows |
| Missing `Type` / `Content Rating` / `Current Ver` / `Android Ver` | 1 / 1 (the corrupted row) / 8 / 3 rows |
| Exact duplicate rows | 483 |
| Rows sharing an `App` name | 1,181 (extra copies) |
| Corrupted row | 1 row (`Life Made WI-Fi Touchscreen Photo Frame`) has its columns shifted left, so `Category = "1.9"` |

### 1.2 Cleaning steps and reasoning

| # | Step | Why | Result |
|---|---|---|---|
| 1 | **Drop the shifted row** (`Category == "1.9"`) | Its values are in the wrong columns (it shows a `Rating` of 19 and a `Content Rating` that is blank); fixing it by hand is not worth it for one record. | 10,840 rows |
| 2 | **Remove exact duplicates** | Same record loaded more than once inflates every count. | 10,357 rows |
| 3 | **De-duplicate by `App`**, keep the row with the most `Reviews` | The same app appears in several categories or scrape dates. The row with the most reviews is the most recent and complete snapshot. | **9,659 unique apps** |
| 4 | **Missing `Type`** → `Free` | The one app with a missing `Type` (*Command & Conquer: Rivals*) has `Price = 0`. | 0 missing |
| 5 | **Keep missing `Rating` as null** (do not fill with the mean) | Filling 1,463 ratings with 4.17 would fake a satisfaction signal and shrink the variance. Rating-based answers use the 8,196 apps that have one. | 8,196 rated apps |
| 6 | **Installs** `"1,000,000+"` → `1000000` | Strip `+` and `,`, cast to number. | numeric |
| 7 | **Price** `"$4.99"` → `4.99` | Strip `$`, cast to float. | numeric |
| 8 | **Size** `"19M"`, `"201k"` → MB | `k` ÷ 1024. `"Varies with device"` becomes null (not 0). | 8,431 apps with a size |
| 9 | **Reviews** text → integer | Stored as text in the CSV. | numeric |
| 10 | **Last Updated** → datetime | Format is `"January 7, 2018"`. | `datetime64` |
| 11 | **Standardize text** | `Category` is UPPER_SNAKE_CASE (`ART_AND_DESIGN`), `Genres` is Title Case and can hold two values separated by `;`. Convert with `.str.replace('_',' ').str.title()` for display. | consistent labels |
| 12 | **Add `Year` column** | Needed for the year slicer and ribbon chart in the dashboard. | derived |

### 1.3 Code

```python
import pandas as pd, numpy as np

raw = pd.read_csv("googleplaystore.csv")

df = raw[raw["Category"] != "1.9"].copy()          # corrupted row
df = df.drop_duplicates()                           # exact duplicates
df["Reviews"] = df["Reviews"].astype(int)
df = (df.sort_values("Reviews", ascending=False)
        .drop_duplicates("App", keep="first"))      # one row per app

df["Installs"] = df["Installs"].str.replace(r"[+,]", "", regex=True).astype(float)
df["Price"]    = df["Price"].str.replace("$", "", regex=False).astype(float)

def to_mb(s):
    if s == "Varies with device": return np.nan
    if s.endswith("M"): return float(s[:-1])
    if s.endswith("k"): return float(s[:-1]) / 1024
    return np.nan
df["Size_MB"] = df["Size"].map(to_mb)

df["Last Updated"] = pd.to_datetime(df["Last Updated"], format="%B %d, %Y")
df["Year"] = df["Last Updated"].dt.year
df["Type"] = df["Type"].fillna("Free")

rated = df.dropna(subset=["Rating"])                # 8,196 apps
```

> **Note on matching Power BI.** If your Power Query only removed *exact* duplicate rows (and the shifted row), you will see **10,357** rows instead of **9,659** unique apps, because the same app can appear with slightly different review counts. Averages move only in the second decimal place, but counts such as "Total Apps" will differ. State which rule you used in your report.

---

## 2. Basic-Level Questions

### Q1. What is the average rating of apps in the dataset?

**Answer: 4.17** (median 4.3, based on the 8,196 apps that have a rating).

```python
rated["Rating"].mean(), rated["Rating"].median()
```

**Explanation.** The mean is lower than the median because a small tail of poorly rated apps (1.0 to 3.0) pulls the average down. Most apps sit between 4.0 and 4.7, so users are generally satisfied, and a rating of 4.2 is "average" on this store, not "good".

**Business impact.** Use **4.2 as the minimum benchmark** for launch. An app below 4.0 is in the bottom quarter of the market.

---

### Q2. How many unique categories of apps are there?

**Answer: 33 categories** (and 118 distinct `Genres` values, because genres can be combined such as `Art & Design;Pretend Play`).

```python
df["Category"].nunique()          # 33
df["Category"].value_counts().head()
```

Largest categories by number of apps: **FAMILY 1,876**, **GAME 946**, **TOOLS 829**, BUSINESS 420, MEDICAL 395.

**Explanation.** The store is broad but concentrated: three categories hold 38% of all apps. Small categories (EVENTS, BEAUTY, PARENTING, COMICS) are far less crowded.

**Business impact.** Crowded categories mean more competition for visibility. Small categories with high ratings (see Medium Q2) are low-competition entry points.

---

### Q3. What is the distribution of app sizes?

**Answer.** Mean **20.4 MB**, median **12 MB**, 25th percentile 4.6 MB, 75th percentile 28 MB, maximum 100 MB. A further **1,228 apps (12.7%)** say "Varies with device" and have no fixed size.

| Size band | Apps | Share of apps with a size |
|---|---|---|
| ≤ 5 MB | 2,317 | 27.5% |
| 5-10 MB | 1,613 | 19.1% |
| 10-20 MB | 1,533 | 18.2% |
| 20-50 MB | 2,092 | 24.8% |
| 50-100 MB | 876 | 10.4% |

```python
df["Size_MB"].describe()
pd.cut(df["Size_MB"], [0,5,10,20,50,100]).value_counts().sort_index()
```

**Explanation.** The distribution is right-skewed (mean above median). Nearly half of apps are under 10 MB; only about 1 in 10 is over 50 MB. The data tops out at 100 MB.

**Business impact.** A typical app should target **under 20 MB** unless it is a game. Use a histogram (as in your dashboard) rather than an average for planning.

---

### Q4. How many free vs paid apps are there?

**Answer: 8,905 free (92.2%) vs 754 paid (7.8%).**

```python
df["Type"].value_counts()
```

**Explanation.** The store is overwhelmingly free. Paid apps are a niche: free apps have a median of 100,000 installs, paid apps a median of just 1,000.

**Business impact.** Monetizing through ads or in-app purchases is the norm. A paid-upfront model should be reserved for a clearly differentiated product.

---

### Q5. What is the most common content rating?

**Answer: Everyone, with 7,903 apps (81.8%).**

| Content rating | Apps |
|---|---|
| Everyone | 7,903 |
| Teen | 1,036 |
| Mature 17+ | 393 |
| Everyone 10+ | 322 |
| Adults only 18+ | 3 |
| Unrated | 2 |

```python
df["Content Rating"].value_counts()
```

**Explanation.** Developers aim for the widest possible audience, which also avoids store restrictions.

**Business impact.** Design for "Everyone" by default. Content rating also drives which advertisers will buy your inventory.

---

### Q6. What are the top 5 most installed apps?

**Answer.** The `Installs` field is a **bucket** (`1,000,000,000+`), not an exact count, so **20 apps tie at 1 billion+**. Breaking the tie by number of reviews gives:

| Rank | App | Category | Rating |
|---|---|---|---|
| 1 | Facebook | SOCIAL | 4.1 |
| 2 | WhatsApp Messenger | COMMUNICATION | 4.4 |
| 3 | Instagram | SOCIAL | 4.5 |
| 4 | Messenger | COMMUNICATION | 4.0 |
| 5 | Subway Surfers | GAME | 4.5 |

```python
top = df[df["Installs"] == df["Installs"].max()]
top.sort_values("Reviews", ascending=False).head(5)[["App","Category","Rating"]]
```

**Explanation.** The full 1B+ group is mostly Google's own apps (YouTube, Chrome, Maps, Photos, Gmail, Drive) plus Meta apps and one game. Subway Surfers is the only non-platform app to reach this tier.

**Business impact.** Top installs come from **platform owners and social or communication networks**. A new company should not benchmark against them; use the 10M-100M tier instead (Advanced Q3).

---

### Q7. How many apps have a rating of 4.0 and above?

**Answer: 6,287 apps**, which is **76.7% of rated apps** (65.1% of all apps, counting unrated apps in the denominator).

```python
(rated["Rating"] >= 4.0).sum()
(rated["Rating"] >= 4.0).mean()
```

**Explanation.** Always state the denominator. Rated apps are 8,196; 1,463 apps have no rating yet, usually because they are new or have few users.

**Business impact.** 4.0 is the market's "pass mark". Roughly three in four rated apps clear it, so **falling below 4.0 puts you in the bottom quarter**.

---

### Q8. What is the average number of reviews for free vs paid apps?

**Answer.**

| Type | Apps | Mean reviews | Median reviews |
|---|---|---|---|
| Free | 8,905 | **234,421** | 1,379 |
| Paid | 754 | **8,748** | 88.5 |

```python
df.groupby("Type")["Reviews"].agg(["mean","median","count"])
```

**Explanation.** Free apps get about **27x more reviews on average**. The means are dominated by a few giants (Facebook has 78M reviews), so the **median is the more honest comparison**: 1,379 vs 89, still about 15x.

**Business impact.** Free apps are easier to get feedback from. If you launch a paid app, plan a **free tier or trial** to collect the reviews that drive visibility.

---

### Q9. What is the average app size for each category?

**Answer (top and bottom):**

| Largest average size | MB | Smallest average size | MB |
|---|---|---|---|
| GAME | **41.7** | TOOLS | **8.8** |
| FAMILY | 27.3 | LIBRARIES AND DEMO | 10.6 |
| TRAVEL AND LOCAL | 24.2 | PERSONALIZATION | 11.2 |
| SPORTS | 24.1 | COMMUNICATION | 11.3 |
| ENTERTAINMENT | 23.7 | ART AND DESIGN | 12.2 |

```python
df.groupby("Category")["Size_MB"].mean().sort_values(ascending=False).round(1)
```

**Explanation.** Games ship graphics and audio assets, so they are about **5x larger** than utilities. Apps whose size is "Varies with device" are excluded.

**Business impact.** Set size budgets **per category**. A 30 MB tools app is an outlier; a 30 MB game is below average.

---

### Q10. How many apps were last updated in 2018?

**Answer: 6,287 apps (65.1%).**

```python
(df["Last Updated"].dt.year == 2018).sum()
```

| Year | Apps | | Year | Apps |
|---|---|---|---|---|
| 2018 | 6,287 | | 2015 | 448 |
| 2017 | 1,792 | | 2014 | 203 |
| 2016 | 779 | | ≤ 2013 | 150 |

**Explanation.** The dataset was scraped in **August 2018** (latest date: 8 Aug 2018), so "updated in 2018" really means "updated in the past seven months". The count is a coincidence with Q7 (also 6,287); both were checked separately.

**Business impact.** Two-thirds of apps were updated within about 7 months, so **active maintenance is the norm**. An app untouched for a year looks abandoned.

---

## 3. Medium-Level Questions

### Q1. What is the correlation between installs and rating?

**Answer.** Essentially none.

| Method | Correlation |
|---|---|
| Pearson (installs vs rating) | **0.04** |
| Spearman (rank) | 0.03 |
| Pearson with log10(installs) | 0.08 |
| Spearman (reviews vs rating) | 0.12 |

```python
rated[["Rating","Installs"]].corr()                 # Pearson
rated[["Rating","Installs"]].corr("spearman")
np.corrcoef(rated["Rating"], np.log10(rated["Installs"].clip(lower=1)))
```

**Explanation.** Installs are highly skewed, so use Spearman or log scale. Even then the relationship is very weak. Popular apps are not systematically better rated, and well-rated apps are not systematically more popular. The binned view in Advanced Q3 shows *why*: the pattern is U-shaped, which a straight-line correlation misses.

**Business impact.** **Do not assume that a high rating alone will drive installs**. Marketing and distribution matter independently of quality.

---

### Q2. Which categories have the highest average rating?

**Answer.**

| Rank | Category | Avg rating | Rated apps |
|---|---|---|---|
| 1 | EVENTS | **4.44** | 45 |
| 2 | ART AND DESIGN | 4.36 | 59 |
| 3 | EDUCATION | 4.35 | 105 |
| 4 | BOOKS AND REFERENCE | 4.35 | 169 |
| 5 | PERSONALIZATION | 4.33 | 298 |

Lowest: **DATING 3.98**, MAPS AND NAVIGATION 4.04, TOOLS 4.04, VIDEO PLAYERS 4.05, TRAVEL AND LOCAL 4.07.

```python
(rated.groupby("Category")["Rating"].agg(["mean","count"])
      .sort_values("mean", ascending=False).round(2))
```

**Explanation.** The whole range is narrow (3.98 to 4.44), so differences are modest. Always show the **count** next to the mean: EVENTS has only 45 apps.

**Business impact.** Users rate **content and creative categories** highly and **utility and social-matching categories** more harshly (utilities have to *work* every time; dating apps have mixed experiences by nature).

---

### Q3. How does price affect average rating? (paid apps only)

**Answer.** Slightly negative: **correlation -0.11**, and the **average rating falls as price rises**. Paid apps average **4.26** vs **4.17** for free apps.

| Price band | Avg rating | Rated apps |
|---|---|---|
| $0-1 | 4.30 | 106 |
| $1-2 | 4.29 | 101 |
| $2-5 | 4.26 | 282 |
| $5-10 | 4.26 | 58 |
| $10-20 | 4.24 | 30 |
| $20+ | **3.99** | 25 |

```python
paid = rated[rated["Type"] == "Paid"]
paid["Rating"].corr(paid["Price"])
paid.groupby(pd.cut(paid["Price"], [0,1,2,5,10,20,1000]))["Rating"].agg(["mean","count"])
```

**Explanation.**
- Paid apps are rated a bit *higher* than free apps (self-selection: people who pay are committed, and there are no ad annoyances).
- Within paid apps, rating is flat up to about $20 and then drops.
- The **$20+ group is dominated by "I am Rich"-style novelty apps** ($399.99, no real function). Check these as outliers before drawing a conclusion.

**Business impact.** **Price between $1 and $5** is the sweet spot: highest ratings and by far the most apps (the median paid app costs $2.99). Premium pricing needs clear justification.

---

### Q4. What is the distribution of ratings across content ratings?

**Answer.**

| Content rating | Mean | Median | Rated apps |
|---|---|---|---|
| Adults only 18+ | 4.30 | 4.5 | 3 |
| Everyone 10+ | 4.23 | 4.3 | 305 |
| Teen | 4.23 | 4.3 | 912 |
| Everyone | 4.17 | 4.3 | 6,618 |
| Mature 17+ | 4.12 | 4.2 | 357 |
| Unrated | 4.10 | 4.1 | 1 |

```python
rated.groupby("Content Rating")["Rating"].agg(["mean","median","count"]).round(2)
# For the distribution itself: sns.boxplot(data=rated, x="Content Rating", y="Rating")
```

**Explanation.** Differences are tiny (0.1 points). Ignore "Adults only" and "Unrated": 3 and 1 apps are not enough to conclude anything.

**Business impact.** Content rating is **not a meaningful lever for ratings**. Choose it for audience reach and advertising rules, not for score.

---

### Q5. Which genres have the most apps with over 1 million installs?

**Answer** (apps with more than 1M installs, i.e. the 5M+ buckets and above; **1,981 apps**):

| Rank | Genre | Apps |
|---|---|---|
| 1 | Tools | **173** |
| 2 | Action | 128 |
| 3 | Photography | 124 |
| 4 | Communication | 99 |
| 5 | Productivity | 91 |
| 6 | Entertainment | 83 |
| 7 | Arcade | 81 |
| 8 | Sports | 79 |

```python
big = df[df["Installs"] > 1_000_000]
big["Genres"].value_counts().head(10)
# Splitting combined genres ("Action;Action & Adventure") with .str.split(";").explode() changes counts slightly.
```

**Explanation.** Counting apps rewards *large* genres. Check the success *rate* (share of a genre's apps above 1M) before concluding a genre is "better".

**Business impact.** Tools, action games and photography are where mass-market hits come from, but they are also the most competitive.

---

### Q6. How frequently do apps get updated? Average time between updates?

**Answer.** The dataset **cannot answer this exactly**: it stores only *one* `Last Updated` date per app, so there is no "previous update" to subtract from. Only 42 apps have more than one distinct date among their raw duplicate rows, which is too few to generalize.

**Best available proxy: how stale is each app** (days between its last update and the scrape date, 8 Aug 2018):

| Measure | Value |
|---|---|
| Median days since last update | **96 days** |
| Mean days since last update | 281 days |
| Updated within 30 days | 30.4% |
| Updated within 90 days | 49.3% |
| Updated within 180 days | 61.3% |
| **Not updated for over a year** | **25.0%** |

```python
snapshot = df["Last Updated"].max()
df["Days_Since_Update"] = (snapshot - df["Last Updated"]).dt.days
df["Days_Since_Update"].describe()
```

**Explanation.** Half of apps were updated in the last three months, but a quarter are effectively abandoned. Frequently updated apps are also **better**: apps updated within 30 days have a **4.27** average rating vs **4.06** for apps not updated in over a year, and a median of **1M installs vs 10K**.

**Business impact.** Plan an update at least **every 1-3 months**. Treat this as correlation only (successful apps have more resources to update), but it is a cheap lever.

---

### Q7. What is the impact of app size on installs?

**Answer.** **Larger apps have more installs**, which is the opposite of the common assumption that "small is better".

| Size band | Apps | Median installs | Mean installs |
|---|---|---|---|
| ≤ 5 MB | 2,317 | 5,000 | 0.76M |
| 5-10 MB | 1,613 | 10,000 | 1.6M |
| 10-20 MB | 1,533 | 50,000 | 4.1M |
| 20-50 MB | 2,092 | 100,000 | 4.5M |
| 50-100 MB | 876 | **1,000,000** | 13.0M |

Correlation: Pearson 0.13, **Spearman 0.31** (moderate, monotonic). Size vs rating: Pearson 0.06 (flat; 4.11 for ≤5 MB up to 4.25 for 50-100 MB).

```python
s = df.dropna(subset=["Size_MB"])
s.groupby(pd.cut(s["Size_MB"], [0,5,10,20,50,100]))["Installs"].agg(["median","mean","count"])
s[["Size_MB","Installs"]].corr("spearman")
```

**Explanation.** This is a **confounded** result. Large apps tend to be well-funded games and flagship products with rich features and marketing budgets. Size does not *cause* installs; it is a proxy for investment and feature depth. Tiny apps are often hobby projects with few users.

**Business impact.** Do **not** shrink features just to reduce size. Optimize size where it is cheap (compressed assets, on-demand modules) but do not treat size as a growth lever.

---

### Q8. Which apps have the most reviews and what are their ratings?

| Rank | App | Category | Reviews | Rating | Installs |
|---|---|---|---|---|---|
| 1 | Facebook | SOCIAL | 78.2M | 4.1 | 1B+ |
| 2 | WhatsApp Messenger | COMMUNICATION | 69.1M | 4.4 | 1B+ |
| 3 | Instagram | SOCIAL | 66.6M | 4.5 | 1B+ |
| 4 | Messenger | COMMUNICATION | 56.6M | 4.0 | 1B+ |
| 5 | Clash of Clans | GAME | 44.9M | 4.6 | 100M+ |
| 6 | Clean Master | TOOLS | 42.9M | 4.7 | 500M+ |
| 7 | Subway Surfers | GAME | 27.7M | 4.5 | 1B+ |
| 8 | YouTube | VIDEO PLAYERS | 25.7M | 4.3 | 1B+ |
| 9 | Security Master | TOOLS | 24.9M | 4.7 | 500M+ |
| 10 | Clash Royale | GAME | 23.1M | 4.6 | 100M+ |

```python
df.sort_values("Reviews", ascending=False).head(10)[["App","Category","Reviews","Rating","Installs"]]
```

**Explanation.** Every top-10 app is rated **4.0 or higher** (average about 4.3). The **games and tools** in this list (Clash of Clans, Clean Master) get far more reviews *per install* than social apps, because they prompt users for reviews in-app.

**Business impact.** High review volume and good rating go together at the top. **Prompting for a review after a positive moment** (level completed, task done) is a proven tactic.

---

### Q9. How does content rating distribution differ between free and paid apps?

| Content rating | Free (% of free apps) | Paid (% of paid apps) |
|---|---|---|
| Everyone | 81.4% | **86.6%** |
| Teen | **11.1%** | 6.9% |
| Mature 17+ | **4.2%** | 2.4% |
| Everyone 10+ | 3.3% | 4.1% |
| Adults only / Unrated | 0.05% | 0% |

```python
pd.crosstab(df["Type"], df["Content Rating"], normalize="index").round(4) * 100
```

**Explanation.** Paid apps skew toward **family-friendly** content; free apps include more Teen and Mature content, typically ad-supported games and social apps.

**Business impact.** Match monetization to audience: ads for Teen/Mature free apps, one-time purchase for family and educational apps.

---

### Q10. What are the top 5 categories with the most installs?

| Rank | Category | Total installs | Share of all installs |
|---|---|---|---|
| 1 | GAME | **13.5B** | 17.9% |
| 2 | COMMUNICATION | 11.0B | 14.7% |
| 3 | TOOLS | 8.1B | 10.8% |
| 4 | FAMILY | 6.2B | 8.3% |
| 5 | PRODUCTIVITY | 5.8B | 7.7% |

Total across all apps: about **75.3B** installs.

```python
df.groupby("Category")["Installs"].sum().sort_values(ascending=False).head(5)
```

**Explanation.** Installs are the **lower bound of each bucket**, so totals are conservative. These five categories hold **about 59%** of all installs. GAME is first despite having only 946 apps (versus FAMILY's 1,876), so installs *per app* are much higher.

**Business impact.** Games and communication tools have the **largest audiences**. If the goal is reach, these are the categories; if the goal is less competition and higher ratings, see Medium Q2.

---

## 4. Advanced-Level Questions

### Q1. Top 10 highest-rated apps: how do their reviews and installs compare?

**Answer.** A naive sort returns **271 apps with a perfect 5.0**, and they are **not meaningful**: the median has **4 reviews**, 79% have fewer than 10 reviews, and the median install count is just 100. A rating based on four people is noise.

**Fix: apply a minimum review threshold** (here ≥ 1,000 reviews):

| # | App | Category | Rating | Reviews | Installs |
|---|---|---|---|---|---|
| 1 | JW Library | BOOKS AND REFERENCE | 4.9 | 922,752 | 10M+ |
| 2 | Six Pack in 30 Days - Abs Workout | HEALTH AND FITNESS | 4.9 | 272,337 | 10M+ |
| 3 | Tickets + PDA 2018 Exam | AUTO AND VEHICLES | 4.9 | 197,136 | 1M+ |
| 4 | Learn Japanese, Korean, Chinese Offline | EDUCATION | 4.9 | 133,136 | 1M+ |
| 5 | StrongLifts 5x5 Workout Gym Log | HEALTH AND FITNESS | 4.9 | 66,791 | 1M+ |
| 6 | PixPanda - Color by Number | FAMILY | 4.9 | 55,723 | 1M+ |
| 7 | ipsy: Makeup, Beauty, and Tips | BEAUTY | 4.9 | 49,790 | 1M+ |
| 8 | Hungry Hearts Diner | FAMILY | 4.9 | 46,253 | 500K+ |
| 9 | Lose Belly Fat in 30 Days | HEALTH AND FITNESS | 4.9 | 38,098 | 5M+ |
| 10 | Solitaire: Decked Out Ad Free | GAME | 4.9 | 37,302 | 500K+ |

```python
top = (rated[rated["Reviews"] >= 1000]
       .sort_values(["Rating","Reviews"], ascending=False)
       .head(10)[["App","Category","Rating","Reviews","Installs"]])
```

**Explanation.** Once volume is controlled, every top app has **hundreds of thousands of installs and tens of thousands of reviews**. They are concentrated in **health and fitness, education and hobby apps**: single-purpose apps that solve one clear problem.

**Business impact.** Benchmark against **well-reviewed focused apps**, not "5.0 star" apps. Always apply a minimum-review filter (or a Bayesian average) in dashboards.

---

### Q2. Trend of app updates over time. Patterns or seasonality?

**Answer.**
- **Updates are heavily recent.** Monthly counts rise steadily: 98 (Jan 2017) → 247 (Dec 2017) → 408 (Mar 2018) → 911 (Jun 2018) → **2,318 (Jul 2018)**.
- **July 2018 holds 24% of all apps** (2,318 of 9,659). This is a **snapshot artifact**, not a real seasonal peak: the store only keeps *each app's latest* update date, so old updates are overwritten and the most recent month always looks huge.
- **Weekday pattern:** Monday to Friday each carry about 1,570 to 1,750 apps; **Saturday (710) and Sunday (610) are less than half**. Developers ship on workdays.
- **Seasonality cannot be confirmed**: counts rise steadily year over year (Q4 2017 is the busiest quarter of 2017), and that growth trend is mixed in with any seasonal effect. With only one snapshot there is no way to separate the two.

```python
monthly = df.groupby(df["Last Updated"].dt.to_period("M")).size()
monthly.plot()
df.groupby(df["Last Updated"].dt.dayofweek).size()   # 0 = Monday
```

**Explanation.** "Last updated" data is **right-censored** by the scrape date. To study real update seasonality you need the **full version history** of each app, not just the latest date.

**Business impact.** Ship updates **Tuesday to Thursday**; avoid weekends. Do not read the July 2018 spike as "July is the best month to update".

---

### Q3. How does average rating change with installs? (binned)

| Installs bin | Avg rating | Median rating | Apps |
|---|---|---|---|
| ≤ 100 | **4.42** | 4.8 | 439 |
| 100 - 1K | 4.09 | 4.3 | 895 |
| 1K - 10K | **4.03** | 4.2 | 1,411 |
| 10K - 100K | 4.10 | 4.2 | 1,552 |
| 100K - 1M | 4.20 | 4.3 | 1,918 |
| 1M - 10M | 4.27 | 4.3 | 1,544 |
| 10M - 100M | **4.36** | 4.4 | 393 |
| 100M+ | 4.30 | 4.3 | 44 |

```python
bins   = [-1,100,1e3,1e4,1e5,1e6,1e7,1e8,1e10]
labels = ["<=100","100-1K","1K-10K","10K-100K","100K-1M","1M-10M","10M-100M","100M+"]
rated.groupby(pd.cut(rated["Installs"], bins, labels=labels), observed=True)["Rating"] \
     .agg(["mean","median","count"]).round(3)
```

**Explanation.** The curve is **U-shaped**, which explains the near-zero correlation in Medium Q1:
1. **Very small apps (≤100 installs)** score high because ratings come from friends, family and early adopters. They are unreliable.
2. **The "valley" at 1K-10K installs** is where apps meet real, critical strangers for the first time and ratings fall to the lowest level.
3. **From 10K upward ratings climb steadily**, reaching 4.36 at 10M-100M. Apps that survive to scale are typically better built, and low-rated apps lose installs (survivorship).

**Business impact.** Expect your rating to **dip after the soft launch**. Invest in QA, bug fixing and review responses during the 1K-10K phase, which is when store visibility is won or lost.

---

### Q4. Sentiment analysis on app reviews

**Answer.** **Not possible with the provided file.** `googleplaystore.csv` has only review *counts*, not review *text*. The Kaggle dataset has a companion file, **`googleplaystore_user_reviews.csv`**, which includes `Translated_Review`, `Sentiment`, `Sentiment_Polarity` and `Sentiment_Subjectivity` per app. Use it as follows:

```python
import pandas as pd
from textblob import TextBlob

rev = pd.read_csv("googleplaystore_user_reviews.csv").dropna(subset=["Translated_Review"])
rev["polarity"] = rev["Translated_Review"].map(lambda t: TextBlob(t).sentiment.polarity)

# 1) Link sentiment to rating
summary = rev.groupby("App")["polarity"].mean().rename("avg_polarity")
merged = rated.merge(summary, on="App")
merged[["Rating","avg_polarity"]].corr()

# 2) Themes in high vs low rated apps
high = merged[merged["Rating"] >= 4.5]["App"]
low  = merged[merged["Rating"] <  3.5]["App"]
# Tokenize rev[rev.App.isin(high)] vs rev[rev.App.isin(low)], remove stop-words,
# and compare the most frequent words / bigrams (or run LDA topic modelling).
```

**What to expect / how to interpret.** Positive reviews of highly rated apps typically mention "love", "easy", "helpful", "great"; negative reviews of low-rated apps mention "crash", "ads", "update", "can't", "login", "slow". Verify this with your own run rather than assuming it.

**Business impact.** Recurring complaints (crashes, intrusive ads, login failures) are **fixable product issues**; recurring praise tells you which features to protect and market.

---

### Q5. Relationship between genre and rating. Are some genres consistently higher or lower?

Genres with at least **30 rated apps** (46 of 114 genres qualify; most others have too few apps to compare):

| Highest | Mean | Median | Apps | | Lowest | Mean | Median | Apps |
|---|---|---|---|---|---|---|---|---|
| Events | **4.44** | 4.5 | 45 | | Dating | **3.98** | 4.1 | 133 |
| Puzzle | 4.37 | 4.4 | 100 | | Maps & Navigation | 4.04 | 4.2 | 118 |
| Art & Design | 4.36 | 4.4 | 55 | | Tools | 4.04 | 4.2 | 719 |
| Books & Reference | 4.35 | 4.5 | 169 | | Video Players & Editors | 4.05 | 4.2 | 147 |
| Parenting | 4.34 | 4.5 | 40 | | Travel & Local | 4.07 | 4.2 | 186 |
| Personalization | 4.33 | 4.4 | 298 | | Entertainment | 4.08 | 4.2 | 471 |

```python
g = rated.groupby("Genres")["Rating"].agg(["mean","median","count","std"])
g[g["count"] >= 30].sort_values("mean", ascending=False)
```

**Explanation.**
- **Yes, there are consistent differences**, and the same genres appear at both ends under both mean and median, so the pattern is stable.
- **Without a minimum count, the ranking is misleading:** single-app genres such as "Board;Pretend Play" show 4.8 from just one app.
- **Spread matters too**: Education has a standard deviation of 0.61 (inconsistent quality) while Personalization has 0.36 (consistently good).
- The gap between best and worst genre is only **about 0.5 stars**.

**Business impact.** Puzzle, Education, Books & Reference and Personalization pair **high ratings** with healthy volume. Entering Dating, Tools or Navigation means competing against **lower user satisfaction**, which is both a risk (hard to please) and an opportunity (a genuinely better app stands out).

---

## 5. Metric Development

Metrics used in the dashboard (DAX) and their meaning:

| Metric | Definition | Dashboard measure |
|---|---|---|
| Total Apps | Count of unique apps | `Total Apps` |
| Average Rating | Mean of `Rating` (ignores blanks) | `Average Rating` |
| Total Installs | Sum of numeric `Installs` (lower-bound estimate) | `Total Installs` |
| Total Reviews | Sum of `Reviews` | `Total Reviews` |
| Free Apps / Paid Apps | Count by `Type` | `Free Apps`, `Paid Apps` |

Suggested additional metrics:

| Metric | Formula | Use |
|---|---|---|
| Review rate | `Reviews / Installs` | Engagement per user |
| Success share | Share of apps with rating ≥ 4.0 and installs ≥ 1M | Category health |
| Weighted rating | `(v/(v+m))·R + (m/(v+m))·C` (Bayesian average; `v` = reviews, `m` = review threshold, `C` = mean rating) | Fair ranking of apps with few reviews |
| Days since update | `snapshot date - Last Updated` | Maintenance indicator |

---

## 6. Key Findings and Recommendations

### What drives success on the Play Store?

| Factor | Evidence | Strength |
|---|---|---|
| **Category choice** | GAME + COMMUNICATION = 33% of all installs; Events/Art/Education rate highest | Strong |
| **Free pricing** | Median installs 100K (free) vs 1K (paid); free has 15x more median reviews | Strong |
| **Regular updates** | Updated ≤ 30 days: 4.27 avg rating, 1M median installs; > 1 year: 4.06 and 10K | Strong (correlation) |
| **Review volume** | Apps with 100K+ reviews average 4.37 vs 4.04 for apps with 100-1,000 | Moderate |
| **App size / content richness** | Median installs rise from 5K (≤5 MB) to 1M (50-100 MB) | Moderate (confounded) |
| **Content rating** | Ratings differ by about 0.1 | Weak |
| **Raw rating alone** | Correlation with installs ≈ 0.04 | Not a driver by itself |

### Recommendations for the company

1. **Launch free** (ads or in-app purchase), and if paying upfront, price at **$1-5**.
2. **Target high-satisfaction, moderate-competition categories** (Education, Books & Reference, Art & Design, Personalization) for quality-led wins; target **Games and Communication** for scale.
3. **Set a 4.2 rating target** and treat anything below 4.0 as a red alert.
4. **Plan for the "valley of death" at 1K-10K installs**: heavier QA, support and review replies.
5. **Ship updates every 1-3 months**, on weekdays.
6. **Ask for reviews at positive moments** to build review volume, which feeds visibility.
7. **Budget app size by category** (games ≈ 40 MB, tools ≈ 9 MB) and do not cut features just to save megabytes.
8. **Add review-text sentiment analysis** (Advanced Q4) as the next phase.

---

## 7. Limitations

- **Single snapshot (Aug 2018)**: no time series of ratings or installs; update frequency can only be approximated.
- **Installs are bucketed** (`1,000+`, `5,000+`...) and totals use the bucket floor.
- **Rating is missing for 15%** of apps; results describe apps that have ratings.
- **Association is not causation**: updates, size and reviews correlate with success but may be effects of success.
- **Outliers**: novelty "I am Rich" apps distort the price analysis; a few giants distort means (use medians).
- **No revenue or download-source data**, so "revenue" conclusions are inferred from pricing and installs only.
- **Ethical note**: this dataset has no personal data. If you extend the analysis to review text, handle user-generated text responsibly and anonymize usernames. The case-study PDF's "Additional Considerations" mention patient data and clinical experts; that text is left over from a different (healthcare) case and does not apply here.

---

## 8. Additional Resources

- Dataset: [Google Play Store Apps (Kaggle, lava18)](https://www.kaggle.com/datasets/lava18/google-play-store-apps)
- pandas documentation: https://pandas.pydata.org/docs/
- Power BI documentation (DAX, decomposition tree, ribbon chart): https://learn.microsoft.com/power-bi/
- TextBlob (sentiment): https://textblob.readthedocs.io/
- VADER sentiment: https://github.com/cjhutto/vaderSentiment
- Bayesian average for ratings: search "IMDb weighted rating formula"
