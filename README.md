# MATH 189 Project — Airbnb Price Analysis Across European Cities (SP26)

A statistical data analysis project for MATH 189 (Spring 2026) examining what drives Airbnb nightly prices across 10 major European cities. The project covers data merging, exploratory analysis, preprocessing, and a suite of inferential hypothesis tests.

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Repository Structure](#repository-structure)
3. [Dataset](#dataset)
4. [Workflow](#workflow)
5. [Analysis Summary](#analysis-summary)
6. [Generated Figures](#generated-figures)
7. [Requirements](#requirements)
8. [Usage](#usage)

---

## Project Overview

This project investigates factors that influence short-term rental pricing on Airbnb using data from 10 European cities. The analysis tests several research questions through formal hypothesis tests:

- Do nightly prices differ significantly between weekday and weekend stays?
- Are price variances equal across cities?
- Is the distribution of room types independent of city or day type?
- Do superhost listings command a price premium?
- Do mean log-prices differ across cities, and if so, which pairs differ?

---

## Repository Structure

```
.
├── Data/                          # Raw Airbnb CSV files (one per city × day-type)
│   ├── amsterdam_weekdays.csv
│   ├── amsterdam_weekends.csv
│   ├── athens_weekdays.csv
│   ├── athens_weekends.csv
│   ├── barcelona_weekdays.csv
│   ├── barcelona_weekends.csv
│   ├── berlin_weekdays.csv
│   ├── berlin_weekends.csv
│   ├── budapest_weekdays.csv
│   ├── budapest_weekends.csv
│   ├── lisbon_weekdays.csv
│   ├── lisbon_weekends.csv
│   ├── london_weekdays.csv
│   ├── london_weekends.csv
│   ├── paris_weekdays.csv
│   ├── paris_weekends.csv
│   ├── rome_weekdays.csv
│   ├── rome_weekends.csv
│   ├── vienna_weekdays.csv
│   └── vienna_weekends.csv
│
├── data_merge.ipynb               # Step 1: Merge raw CSVs into merged_data.csv
├── phase_1.ipynb                  # Step 2: EDA, cleaning, preprocessing
├── phase_2.ipynb                  # Step 3: Hypothesis tests
│
├── merged_data.csv                # Output of data_merge.ipynb (raw merged data)
├── cleaned_data_transformed.csv   # Output of phase_1.ipynb (cleaned + features)
│
├── dist_overall.png               # Overall price distribution (raw and log)
├── dist_logsum_kde_by_city.png    # KDE curves by city
├── dist_by_city_boxplot.png       # Box plots by city
├── ks_weekday_vs_weekend.png      # ECDF plots: weekday vs. weekend by city
├── levene_variances_by_city.png   # Per-city variance bar chart
├── chi2_room_type.png             # Room type proportions by city and day type
│
├── 1-s2.0-S0261517721000388-main.pdf  # Reference paper
└── housing.pdf                        # Supplementary reference
```

---

## Dataset

### Source

Raw data is stored in `Data/` as 20 CSV files (10 cities × 2 day types: weekdays / weekends). City names are encoded as integers 1–10 in alphabetical order:

| Code | City      |
|------|-----------|
| 1    | Amsterdam |
| 2    | Athens    |
| 3    | Barcelona |
| 4    | Berlin    |
| 5    | Budapest  |
| 6    | Lisbon    |
| 7    | London    |
| 8    | Paris     |
| 9    | Rome      |
| 10   | Vienna    |

### Features

| Column | Description |
|--------|-------------|
| `realSum` | Nightly price in euros (€) |
| `logSum` | Natural log of `realSum` (added during preprocessing) |
| `room_type` | Listing type: *Entire apt*, *Private room*, or *Shared room* |
| `room_type_apt` | One-hot: Entire apt |
| `room_type_private` | One-hot: Private room |
| `room_type_shared` | One-hot: Shared room |
| `person_capacity` | Maximum number of guests |
| `is_superhost` | 1 if host holds Superhost status, 0 otherwise |
| `multi` | 1 if host has multiple listings |
| `biz` | 1 if host is a business account |
| `cleanliness_rating` | Cleanliness score from guest reviews |
| `guest_satisfaction_overall` | Overall guest satisfaction score |
| `bedrooms` | Number of bedrooms |
| `dist` | Distance to city centre (km) |
| `metro_dist` | Distance to nearest metro station (km) |
| `attr_index` / `attr_index_norm` | Attraction index (raw / normalised) |
| `rest_index` / `rest_index_norm` | Restaurant index (raw / normalised) |
| `city` | Integer city code (1–10) |
| `is_weekday` | 1 = weekday listing, 0 = weekend listing |

---

## Workflow

The project is structured as three sequential notebooks:

### 1. `data_merge.ipynb` — Data Merging

Reads all 20 raw CSV files from `Data/`, attaches `city` (integer code) and `weekday` (0/1) columns, and concatenates them into a single file:

**Output:** `merged_data.csv`

### 2. `phase_1.ipynb` — EDA, Cleaning & Preprocessing

Loads `merged_data.csv` and performs:

- **Descriptive statistics** (mean, median, std, skewness, kurtosis) overall and per city
- **Outlier removal:** rows above the 99th percentile of `realSum` are dropped
- **Preprocessing:**
  - Log-transforms `realSum` → `logSum`
  - One-hot encodes `room_type` into three binary columns
  - Encodes `host_is_superhost` as `is_superhost` (0/1)
  - Drops redundant columns (`room_shared`, `room_private`, `lng`, `lat`)
  - Renames `weekday` → `is_weekday`
- **Distribution visualisations** (see [Generated Figures](#generated-figures))

**Output:** `cleaned_data_transformed.csv`

### 3. `phase_2.ipynb` — Hypothesis Testing

Loads `cleaned_data_transformed.csv` and runs five formal tests on `logSum`:

| Test | Question |
|------|----------|
| **Kolmogorov–Smirnov (2-sample)** | Does the log(price) distribution differ between weekdays and weekends within each city? |
| **Levene's test** | Are the log(price) variances equal across all 10 cities? |
| **Chi-square (independence)** | Is room type independent of city? |
| **Chi-square (independence)** | Is room type independent of weekday vs. weekend? |
| **Welch's t-test** | Do superhost listings charge significantly more than non-superhost listings? |
| **One-way ANOVA + Tukey HSD** | Do mean log(price) values differ across cities, and which pairs are significantly different? |

---

## Analysis Summary

| Test | Result |
|------|--------|
| K-S test (weekday vs. weekend, per city) | Significant in most cities — pricing distributions shift between weekdays and weekends |
| Levene's test (variance across cities) | Rejected H₀ — price variability differs significantly across cities |
| Chi-square (room type × city) | Rejected H₀ — room type distribution is not uniform across cities |
| Chi-square (room type × day type) | Conclusion depends on data; tested at α = 0.05 |
| Welch's t-test (superhost premium) | Rejected H₀ — superhost listings command a statistically significant price premium |
| One-way ANOVA (mean log-price across cities) | Rejected H₀ — at least one city mean differs; Tukey HSD identifies specific pairs |

---

## Generated Figures

| File | Description |
|------|-------------|
| `dist_overall.png` | Histograms of raw price (`realSum`) and log-price (`logSum`) across all cities |
| `dist_logsum_kde_by_city.png` | Overlapping KDE curves of log(price) for each city |
| `dist_by_city_boxplot.png` | Box plots of log(price) grouped by city |
| `ks_weekday_vs_weekend.png` | ECDF comparison plots (weekday vs. weekend) for each of the 10 cities |
| `levene_variances_by_city.png` | Bar chart of sample variance of log(price) per city, with Levene test annotation |
| `chi2_room_type.png` | Stacked proportion bar charts of room type by city and by day type |

---

## Requirements

The notebooks require Python 3 and the following libraries:

```
pandas
numpy
scipy
scikit-learn
statsmodels
matplotlib
seaborn
```

Install all dependencies with:

```bash
pip install pandas numpy scipy scikit-learn statsmodels matplotlib seaborn
```

---

## Usage

Run the notebooks in order:

```bash
# Step 1 – merge raw data
jupyter notebook data_merge.ipynb

# Step 2 – EDA, cleaning, and preprocessing
jupyter notebook phase_1.ipynb

# Step 3 – hypothesis testing
jupyter notebook phase_2.ipynb
```

> **Note:** `data_merge.ipynb` uses a hard-coded local `data_dir` path. Update the `data_dir` and `out_path` variables in that notebook to point to the `Data/` directory and your desired output location before running.
