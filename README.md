# MATH 189 Project — Airbnb Price Analysis Across European Cities (SP26)

A statistical data analysis project for MATH 189 (Spring 2026) examining what drives Airbnb nightly prices across 10 major European cities. The project covers data merging, exploratory analysis, preprocessing, inferential hypothesis testing, regression modelling, regularisation, and advanced machine-learning techniques.

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

This project investigates factors that influence short-term rental pricing on Airbnb using data from 10 European cities. The analysis is structured across five phases:

**Phase 1 — Data Foundation & EDA ✅**
- Do nightly prices differ significantly between weekday and weekend stays?
- Are price variances equal across cities?
- Is the distribution of room types independent of city or day type?
- Do superhost listings command a price premium?
- Do mean log-prices differ across cities, and if so, which pairs differ?

**Phase 2 — Probability & Statistical Inference ✅**
- Formal hypothesis testing: K-S tests, Levene's test, chi-square tests, Welch's t-test, one-way ANOVA with Tukey HSD post-hoc.

**Phase 3 — Regression ✅**
- Multiple linear regression (baseline) with VIF checks and residual diagnostics.

**Phase 4 — Variable Selection & Regularisation ✅**
- Stepwise selection (AIC/BIC), Ridge, LASSO, and cross-validated comparison.

**Phase 5 — Advanced Techniques 🔄**
- Correlation matrix and PCA scatter (complete); K-Means clustering, XGBoost, SHAP feature importance, Elastic Net with interaction terms, and conformal prediction intervals (in progress).

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
├── phase_3.ipynb                  # Step 4: Multiple linear regression
├── phase_4.ipynb                  # Step 5: Variable selection & regularisation
├── phase_5.ipynb                  # Step 6: PCA, correlation matrix (+ K-Means, XGBoost, SHAP, Elastic Net, conformal — in progress)
│
├── merged_data.csv                # Output of data_merge.ipynb (raw merged data)
├── cleaned_data_transformed.csv   # Output of phase_1.ipynb (cleaned + features)
│
├── dist_overall.png                    # Overall price distribution (raw and log)
├── dist_logsum_kde_by_city.png         # KDE curves by city
├── dist_by_city_boxplot.png            # Box plots by city
├── ks_weekday_vs_weekend.png           # ECDF plots: weekday vs. weekend by city
├── levene_variances_by_city.png        # Per-city variance bar chart
├── chi2_room_type.png                  # Room type proportions by city and day type
│
├── naive_ols_diagnostics.png           # Residual diagnostics: Naive OLS
├── full_model_city_fe_ols_diagnostics.png  # Residual diagnostics: City FE OLS
│
├── stepwise_aic_progression.png        # AIC progression during forward stepwise selection
├── ridge_coef_path.png                 # Ridge coefficient paths vs. λ
├── ridge_cv_curve.png                  # Ridge cross-validation RMSE curve
├── lasso_coef_path.png                 # LASSO coefficient paths vs. λ
├── lasso_cv_curve.png                  # LASSO cross-validation RMSE curve
├── model_comparison_phase4.png         # CV-RMSE and test-RMSE comparison across models
│
├── correlation_matrix_original.png     # Correlation heatmap of original numeric features
├── pca_full_scatter_logprice.png       # PCA scatter of all listings coloured by log(price)
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

The project is structured as sequential notebooks across five phases:

### 1. `data_merge.ipynb` — Data Merging ✅

Reads all 20 raw CSV files from `Data/`, attaches `city` (integer code) and `is_weekday` (0/1) columns, and concatenates them into a single file:

**Output:** `merged_data.csv` (51,707 rows × 21 columns)

### 2. `phase_1.ipynb` — EDA, Cleaning & Preprocessing ✅

Loads `merged_data.csv` and performs:

- **Descriptive statistics** (mean, median, std, skewness, kurtosis) overall and per city — before and after outlier removal
- **Outlier removal:** rows above the 99th percentile of `realSum` (> €1,160.84) are dropped (518 rows removed; 51,189 remaining)
- **Preprocessing:**
  - Log-transforms `realSum` → `logSum`
  - One-hot encodes `room_type` into three binary columns (`room_type_apt`, `room_type_private`, `room_type_shared`)
  - Encodes `host_is_superhost` as `is_superhost` (0/1)
  - Drops redundant columns (`room_shared`, `room_private`, `lng`, `lat`)
  - Renames `weekday` → `is_weekday`
- **Distribution visualisations** (see [Generated Figures](#generated-figures))

**Output:** `cleaned_data_transformed.csv` (51,189 rows × 20 columns)

### 3. `phase_2.ipynb` — Hypothesis Testing ✅

Loads `cleaned_data_transformed.csv` and runs formal tests on `logSum`:

| Test | Question |
|------|----------|
| **Kolmogorov–Smirnov (2-sample)** | Does the log(price) distribution differ between weekdays and weekends within each city? |
| **Levene's test** | Are the log(price) variances equal across all 10 cities? |
| **Chi-square (independence)** | Is room type independent of city? |
| **Chi-square (independence)** | Is room type independent of weekday vs. weekend? |
| **Welch's t-test + Cohen's d** | Do superhost listings charge significantly more than non-superhost listings? |
| **One-way ANOVA + η² + Tukey HSD** | Do mean log(price) values differ across cities, and which pairs are significantly different? |

### 4. `phase_3.ipynb` — Regression ✅

Loads `cleaned_data_transformed.csv` and fits two multiple linear regression models:

- **Naive OLS:** numeric features + binary predictors; no city fixed effects. Includes VIF diagnostics and residual plots (QQ-plot, scale-location).
- **City Fixed Effects OLS:** same predictors plus city dummies (Amsterdam as reference). VIF re-checked; residual diagnostics repeated.
- **Model comparison:** R², Adjusted R², AIC, BIC, and RMSE reported side-by-side.

**Output figures:** `naive_ols_diagnostics.png`, `full_model_city_fe_ols_diagnostics.png`

### 5. `phase_4.ipynb` — Variable Selection & Regularisation ✅

Loads `cleaned_data_transformed.csv`, splits into 80/20 train-test, and applies:

- **City FE OLS baseline:** re-fit on training set for fair comparison.
- **Backward stepwise selection (AIC):** starts from full model; drops lowest-AIC predictor at each step.
- **Forward stepwise selection (AIC):** builds model incrementally; AIC progression plotted.
- **Ridge regression:** λ tuned via 10-fold CV on the training set; coefficient path plotted.
- **LASSO regression:** λ tuned via 10-fold CV; coefficient path plotted; zeroed-out features identified.
- **Final comparison:** test RMSE and CV-RMSE across all four models visualised.

**Output figures:** `stepwise_aic_progression.png`, `ridge_coef_path.png`, `ridge_cv_curve.png`, `lasso_coef_path.png`, `lasso_cv_curve.png`, `model_comparison_phase4.png`

### 6. `phase_5.ipynb` — Advanced Techniques 🔄

Loads `cleaned_data_transformed.csv`. Completed so far:

- **Correlation matrix:** heatmap of all numeric predictors including `logSum`.
- **Full-feature PCA:** standardised numeric features projected onto top 2 PCs; scatter coloured by `logSum`.

**Output figures:** `correlation_matrix_original.png`, `pca_full_scatter_logprice.png`

Still in progress:

- **K-Means Clustering:** run in PCA space; elbow method + silhouette scores; crosstab clusters vs. city.
- **XGBoost:** train price regressor; tune hyperparameters with cross-validation.
- **SHAP Feature Importance:** global bar plot + beeswarm; partial dependence plots for top features.
- **Elastic Net with interaction terms:** construct interaction terms (e.g. `superhost_apt`, `apt_capacity`, `dist_weekday`); tune mixing parameter α and λ via CV; compare CV-RMSE to XGBoost.
- **Conformal Prediction Intervals:** apply split-conformal inference on XGBoost residuals; compare interval widths vs. OLS prediction intervals.

---

## Analysis Summary

### Completed (Phases 1–4)

| Test / Technique | Result |
|------------------|--------|
| K-S test (weekday vs. weekend, per city) | Significant in most cities — pricing distributions shift between weekdays and weekends |
| Levene's test (variance across cities) | Rejected H₀ — price variability differs significantly across cities |
| Chi-square (room type × city) | Rejected H₀ — room type distribution is not uniform across cities |
| Chi-square (room type × day type) | Conclusion depends on data; tested at α = 0.05 |
| Welch's t-test + Cohen's d (superhost premium) | Rejected H₀ — superhost listings command a statistically significant price premium |
| One-way ANOVA + η² + Tukey HSD (mean log-price across cities) | Rejected H₀ — at least one city mean differs; Tukey HSD identifies all 45 specific pairs |
| Naive OLS (Phase 3) | Baseline model fit; VIF and residual diagnostics reported |
| City Fixed Effects OLS (Phase 3) | Improved R² over Naive OLS; city dummies jointly significant |
| Backward/Forward Stepwise AIC (Phase 4) | Feature set reduced; AIC-optimal subset identified |
| Ridge regression (Phase 4) | λ tuned via 10-fold CV; coefficients shrunk toward zero |
| LASSO regression (Phase 4) | λ tuned via 10-fold CV; several coefficients zeroed out |
| Model comparison (Phase 4) | CV-RMSE and test-RMSE compared across all four models |

### Partially Complete (Phase 5)

| Technique | Status |
|-----------|--------|
| Correlation matrix | ✅ Complete |
| Full-feature PCA scatter | ✅ Complete |
| K-Means Clustering | 🔄 In progress |
| XGBoost | 🔄 In progress |
| SHAP Feature Importance | 🔄 In progress |
| Elastic Net + interactions | 🔄 In progress |
| Conformal Prediction Intervals | 🔄 In progress |

### Planned Model Summary

| Phase | Technique | Goal |
|-------|-----------|------|
| 5 | K-Means | Unsupervised structure; cluster vs. city alignment |
| 5 | XGBoost | Non-linear price prediction |
| 5 | SHAP | Global and local feature importance |
| 5 | Elastic Net + interactions | Regularised model with engineered features |
| 5 | Conformal Prediction | Calibrated prediction intervals |

---

## Generated Figures

| File | Phase | Description |
|------|-------|-------------|
| `dist_overall.png` | 1 | Histograms of raw price (`realSum`) and log-price (`logSum`) across all cities |
| `dist_logsum_kde_by_city.png` | 1 | Overlapping KDE curves of log(price) for each city |
| `dist_by_city_boxplot.png` | 1 | Box plots of log(price) grouped by city |
| `ks_weekday_vs_weekend.png` | 2 | ECDF comparison plots (weekday vs. weekend) for each of the 10 cities |
| `levene_variances_by_city.png` | 2 | Bar chart of sample variance of log(price) per city, with Levene test annotation |
| `chi2_room_type.png` | 2 | Stacked proportion bar charts of room type by city and by day type |
| `naive_ols_diagnostics.png` | 3 | Residual diagnostic plots (QQ-plot, scale-location) for Naive OLS model |
| `full_model_city_fe_ols_diagnostics.png` | 3 | Residual diagnostic plots for City Fixed Effects OLS model |
| `stepwise_aic_progression.png` | 4 | AIC value at each step of forward stepwise selection |
| `ridge_coef_path.png` | 4 | Ridge coefficient paths as regularisation strength λ varies |
| `ridge_cv_curve.png` | 4 | Cross-validation RMSE curve for Ridge; optimal λ highlighted |
| `lasso_coef_path.png` | 4 | LASSO coefficient paths as λ varies; zeroing-out of features visible |
| `lasso_cv_curve.png` | 4 | Cross-validation RMSE curve for LASSO; optimal λ highlighted |
| `model_comparison_phase4.png` | 4 | CV-RMSE and test-RMSE side-by-side for all Phase 4 models |
| `correlation_matrix_original.png` | 5 | Correlation heatmap of original numeric features including `logSum` |
| `pca_full_scatter_logprice.png` | 5 | All listings projected onto top 2 PCs, coloured by log(price) |

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
xgboost
shap
```

Install all dependencies with:

```bash
pip install pandas numpy scipy scikit-learn statsmodels matplotlib seaborn xgboost shap
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

# Step 4 – regression
jupyter notebook phase_3.ipynb

# Step 5 – variable selection & regularisation
jupyter notebook phase_4.ipynb

# Step 6 – advanced techniques (in progress)
jupyter notebook phase_5.ipynb
```

> **Note:** `data_merge.ipynb` uses a hard-coded local `data_dir` path. Update the `data_dir` and `out_path` variables in that notebook to point to the `Data/` directory and your desired output location before running.
