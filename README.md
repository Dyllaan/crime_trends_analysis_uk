# README: Crime Analysis for Drugs and Guns

## Overview
This project analyzes claims from the ITV documentary *Ross Kemp and the Armed Police* (Sep 6, 2018) using UK crime data. Implemented in a Jupyter Notebook with Apache Spark, Python, and statistical models, it evaluates trends and relationships in violent, firearm, and drug-related crimes across UK cities.

## Claims Analyzed
1. Violent crime is increasing.
2. Crimes involving firearms are closely associated with drug offences.
3. Liverpool has more firearm incidents per capita than any other UK area.

## Datasets
- **Street Level Crime Data** (`all_crimes21_hdr(5).txt.gz`): ~19M rows of crime records (type, LSOA code, month, outcome).
- **LSOA Population Data** (`lsoa_pop_v2.csv`): 2011 population and density by LSOA.
- **Test Data** (`Sample_Data_Only_For_Test(2).csv`): Sample for development.

## Implementation
### Data Pipeline
- **Apache Spark**: Loads and processes data with predefined schemas, repartitions for efficiency.
- **Preprocessing**: 
  - Filters crimes to violent, firearm (defined as `Possession of weapons` with `prison` outcome), and drug-related.
  - Cleans city names (removes trailing codes, e.g., `Liverpool 001A` → `Liverpool`).
  - Joins crime data with population data via LSOA codes.

### Analysis
- **Claim 1: Violent Crime Trends**
  - **Method**: Aggregates violent crime counts by month, applies SARIMA(1,1,1)(1,1,1,12) on log-transformed, differenced data (stationarity via ADF test). Kendall Tau confirms trend.
  - **Findings**: Violent crime rose 158.27% (2011–2019), dipped slightly in 2020 (-1.05%), and fell sharply in 2021 (-58.77%).
  - **Visualization**: Plot of historical and forecasted log-transformed violent crime counts with confidence intervals.

- **Claim 2: Firearm-Drug Association**
  - **Method**: Computes Pearson correlation (0.8796) and OLS regression on log-transformed firearm and drug counts, including violent crime and density as predictors.
  - **Findings**: Strong association confirmed (Adjusted R²: 0.816 for firearms, 0.867 for drugs). Violent crime has a stronger influence (coefficient 0.7048) than drugs (0.4838) on firearms.
  - **Note**: High VIF indicates multicollinearity between violent and drug crimes.

- **Claim 3: Liverpool Firearm Incidents**
  - **Method**: Calculates per capita firearm rates, identifies 95th percentile outliers, applies PCA and Bisecting K-Means (k=4) for city clustering.
  - **Findings**: Liverpool has the highest total firearm incidents but not the highest per capita rate (London boroughs rank higher). It clusters with high-crime cities (silhouette score: 0.5334).
  - **Visualizations**:
    - Bar/line plot of outlier cities’ firearm rates and counts.
    - PCA scatter plot with K-Means clusters, highlighting Liverpool.

### Tools and Libraries
- **Apache Spark**: `pyspark.sql`, `pyspark.ml` (VectorAssembler, KMeans, PCA, Correlation).
- **Python**: `pandas`, `numpy`, `statsmodels` (SARIMA, OLS, ADF, Kendall), `matplotlib`, `seaborn`, `scipy`.

## Key Results
- **Claim 1**: **True**. Violent crime increased significantly from 2011–2019, despite a COVID-related dip.
- **Claim 2**: **True**. Firearm and drug crimes are strongly correlated, with violent crime as a key factor.
- **Claim 3**: **Mostly False**. Liverpool leads in total firearm incidents but not per capita; London boroughs are prominent outliers.

## Limitations
- 2011 population data skews per capita calculations.
- SARIMA lacks exogenous variables (e.g., socioeconomic factors), limiting robustness during shocks like COVID-19.
- No spatial analysis (e.g., urban/rural or longitude/latitude).
- Multicollinearity in OLS models suggests feature refinement.
