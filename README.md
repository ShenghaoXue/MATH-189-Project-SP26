# MATH 189 Airbnb Price Modeling Project

This repository contains a complete analysis workflow for a MATH 189 project on Airbnb listing prices across ten European cities. The project is motivated by Gyódi and Nawaro's 2021 study, ["Determinants of Airbnb prices in European cities: A spatial econometrics approach"](https://www.sciencedirect.com/science/article/pii/S0261517721000388), which studies Airbnb price determinants with an emphasis on location, accessibility, and spatial dependence.

The project uses the supplementary dataset released with that study on [Zenodo](https://zenodo.org/records/4446043). Starting from those raw weekday/weekend city CSV files, this repository builds a cleaned modeling dataset, runs exploratory analysis and hypothesis tests, develops linear-regression baselines, evaluates feature-selection and regularized linear models, and finishes with advanced XGBoost, SHAP, and targeted Elastic Net modeling.

The main target variable is `logSum`, the log-transformed Airbnb listing price. The raw price variable `realSum` is preserved for descriptive analysis.

## Repository Structure

```text
.
├── Data/                                  # Raw city weekday/weekend CSV files
├── plots/                                 # Generated PNG figures used by notebooks/report
├── merged_data.csv                        # Cleaned merged dataset produced by Section 1
├── section1_cleaning_and_EDA.ipynb        # Data cleaning and exploratory analysis
├── section2_hypothesis_testing.ipynb      # Statistical hypothesis tests
├── section3_linear_regression.ipynb       # OLS and city fixed-effects baselines
├── section4_feature_selectionCV.ipynb     # Stepwise, Ridge, LASSO, and CV comparison
├── section5_advanced.ipynb                # XGBoost, SHAP, and targeted Elastic Net
└── README.md
```

## Data

The raw data live in `Data/` and come from Gyódi and Nawaro's Zenodo supplementary material. The Zenodo record provides weekday and weekend Airbnb listing files for each city, along with variables covering price, room type, host/listing attributes, ratings, distance to city center and metro, attraction accessibility, restaurant accessibility, and coordinates.

This project uses the weekday/weekend Airbnb listings for:

- Amsterdam
- Athens
- Barcelona
- Berlin
- Budapest
- Lisbon
- London
- Paris
- Rome
- Vienna

There are 20 raw CSV files, one weekday and one weekend file per city. Section 1 merges these files, adds city and weekday/weekend indicators, removes missing values and extreme outliers, creates `logSum`, and writes the cleaned dataset to `merged_data.csv`.

The current cleaned dataset has:

- `51,189` rows
- `18` columns
- `10` cities
- weekday/weekend indicator counts close to balanced

### Cleaned Dataset Columns

| Column | Description |
|---|---|
| `realSum` | Raw listing price |
| `logSum` | Log-transformed listing price; main modeling target |
| `room_type_apt` | Indicator for entire apartment/home listing |
| `room_type_private` | Indicator for private-room listing |
| `room_type_shared` | Indicator for shared-room listing |
| `person_capacity` | Number of guests the listing can accommodate |
| `is_superhost` | Superhost indicator |
| `multi` | Multiple-listing host indicator |
| `biz` | Business-host indicator |
| `cleanliness_rating` | Cleanliness rating |
| `guest_satisfaction_overall` | Overall guest satisfaction rating |
| `bedrooms` | Bedroom count |
| `dist` | Distance from city center |
| `metro_dist` | Distance from nearest metro/transit station |
| `attr_index` | Attraction accessibility index |
| `rest_index` | Restaurant accessibility index |
| `city` | City name |
| `is_weekday` | Weekday indicator; weekend listings are encoded as `0` |

## Notebook Workflow

Run the notebooks in order, because later sections depend on files and modeling choices created earlier.

### Section 1: Data Cleaning and Exploratory Analysis

Notebook: `section1_cleaning_and_EDA.ipynb`

This section loads and merges the raw city CSV files, checks the merged structure, summarizes prices before cleaning, removes missing values and outliers, creates `logSum`, and saves `merged_data.csv`.

Main outputs:

- `merged_data.csv`
- overall price distribution plot
- city-level log-price KDE plot
- city-level price boxplot

### Section 2: Hypothesis Testing

Notebook: `section2_hypothesis_testing.ipynb`

This section uses the cleaned dataset to test distributional and group-level differences in price. It includes weekday/weekend distribution tests, city variance testing, room-type independence tests, superhost premium testing, ANOVA for city mean differences, and pairwise city comparisons.

Main outputs:

- weekday/weekend ECDF comparison
- city variance plot
- room-type stacked proportion plot
- hypothesis-test summary tables

### Section 3: Linear Regression Baselines

Notebook: `section3_linear_regression.ipynb`

This section builds interpretable OLS baselines for `logSum`. It compares a naive linear model with a city fixed-effects model, reports coefficients, checks multicollinearity through VIF, and saves residual diagnostic plots.

Main outputs:

- naive OLS residual diagnostics
- city fixed-effects OLS residual diagnostics
- coefficient tables
- VIF tables
- linear baseline comparison table

### Section 4: Feature Selection and Cross-Validation

Notebook: `section4_feature_selectionCV.ipynb`

This section improves the linear-model workflow with train/test splitting, training-only preprocessing, stepwise selection, Ridge tuning, LASSO tuning, coefficient paths, cross-validation curves, and a final linear-model comparison.

Main outputs:

- backward and forward stepwise traces
- stepwise AIC progression plot
- Ridge coefficient path and CV curve
- LASSO coefficient path and CV curve
- final linear-model comparison plot

### Section 5: Advanced Modeling

Notebook: `section5_advanced.ipynb`

This section adds nonlinear and model-explanation methods. It starts with original-feature correlations and PCA, fits baseline and tuned XGBoost models, explains them with native feature importance and SHAP values, then builds a targeted Elastic Net model using only interaction terms motivated by SHAP dependence plots and PCA structure.

Main outputs:

- original-feature correlation heatmap
- PCA price projection
- baseline and tuned XGBoost native feature-importance plots
- baseline and tuned SHAP violin plots
- tuned SHAP global-importance plot
- ordered mutual SHAP dependence plots
- targeted Elastic Net coefficient plot
- advanced model-comparison plot

## Advanced Modeling Details

Section 5 uses three advanced modeling layers:

1. **Baseline XGBoost**  
   Fits a tree-based nonlinear model using standardized numeric features, binary indicators, and city dummy variables.

2. **Tuned XGBoost**  
   Uses Optuna to tune XGBoost hyperparameters by cross-validated RMSE. The tuned model is then evaluated on the held-out test set.

3. **Targeted Elastic Net**  
   Starts from the city fixed-effects linear baseline and adds only selected pairwise interactions. The interaction set is deliberately compact: it is based on SHAP dependence plots and PCA feature groupings rather than a broad expansion of all possible interactions.

The targeted Elastic Net interaction set includes relationships among:

- distance and attraction access
- distance and restaurant access
- attraction and restaurant access
- city-center distance and metro distance
- guest satisfaction and cleanliness
- guest satisfaction and location/access measures
- superhost status and rating variables
- room type and guest satisfaction

This keeps the design interpretable and avoids the earlier high-dimensional interaction expansion.

## Figures

All generated figures are stored in `plots/`. File names follow this convention:

```text
sectionX_PlotFunctionY_optionalDescription.png
```

Examples:

- `section1_PlotDistribution1_overall.png`
- `section4_PlotComparison1_linearModels.png`
- `section5_PlotSHAP3_tunedXGBBarImportance.png`
- `section5_PlotComparison1_advancedModels.png`

The notebooks save figures directly into `plots/`, so rerunning notebooks should refresh the relevant PNG files in place.

## Reproducing the Analysis

### 1. Clone the repository

```bash
git clone <repository-url>
cd "189 Project"
```

### 2. Create an environment

The project uses standard scientific Python packages plus XGBoost, Optuna, and SHAP. One workable setup is:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install pandas numpy matplotlib seaborn scipy statsmodels scikit-learn xgboost optuna shap jupyter
```

If using Conda:

```bash
conda create -n math189-airbnb python=3.11
conda activate math189-airbnb
pip install pandas numpy matplotlib seaborn scipy statsmodels scikit-learn xgboost optuna shap jupyter
```

### 3. Run notebooks in order

```text
section1_cleaning_and_EDA.ipynb
section2_hypothesis_testing.ipynb
section3_linear_regression.ipynb
section4_feature_selectionCV.ipynb
section5_advanced.ipynb
```

Section 1 creates `merged_data.csv`, which is required by Sections 2-5.

### 4. Check outputs

After running the notebooks:

- `merged_data.csv` should exist in the project root.
- PNG figures should be saved in `plots/`.
- model-comparison tables should display inside the relevant notebooks.

## Reproducibility Notes

- Notebooks use `RANDOM_STATE = 42` where stochastic modeling or splitting is involved.
- Train/test splits are stratified by `city` where appropriate.
- Section 4 and Section 5 use cross-validation for regularized and tuned models.
- XGBoost and Optuna may take longer to run than the linear-model notebooks.
- SHAP dependence plots in Section 5 generate many figures and may take noticeable time to render.

## Project Outputs by Section

| Section | Notebook | Primary focus | Key artifacts |
|---|---|---|---|
| 1 | `section1_cleaning_and_EDA.ipynb` | Cleaning and EDA | `merged_data.csv`, distribution plots |
| 2 | `section2_hypothesis_testing.ipynb` | Hypothesis testing | test summaries, ECDF/variance/room-type plots |
| 3 | `section3_linear_regression.ipynb` | OLS baselines | diagnostics, coefficients, VIF tables |
| 4 | `section4_feature_selectionCV.ipynb` | Feature selection and regularization | stepwise traces, Ridge/LASSO plots, linear comparison |
| 5 | `section5_advanced.ipynb` | XGBoost, SHAP, Elastic Net | SHAP plots, XGBoost importance, advanced comparison |

## Notes for Future Work

Potential extensions include:

- adding a pinned `requirements.txt` or Conda environment file,
- exporting final notebook outputs into a reproducible report build,
- adding automated checks that every notebook plot save path points to `plots/`,
- comparing Elastic Net interactions against a small set of nonlinear splines,
- using permutation importance alongside SHAP for additional model interpretation.
