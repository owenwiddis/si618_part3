# Preprocessing Summary

## Project Goal

Do cities with similar urban connectivity profiles (Walk Score, Transit Score,
Bike Score) share similar business ecosystems? We aggregate business-level Yelp
data to the city level to create city profiles combining connectivity scores with
business composition, then use clustering to discover city "archetypes."

**Why city-level?** Walk Score, Transit Score, and Bike Score are city-level
metrics — every business in the same city shares identical values. With only 14
cities after merging, clustering at the business level (42K rows) would largely
just rediscover the 14 cities. Aggregating to the city level makes the unit of
analysis match the granularity of the connectivity data.

---

## What Was Done and Why

### 1. Data Cleaning and Handling of Issues (Rubric: 20 pts)

**Missing value analysis** was performed on both raw datasets before any
transformations or merging.

- **Urban dataset (102 rows, 79 columns):** Transit Score was missing for 5 cities
  (4.9%). These were filled with the median value, which is robust to outliers and
  avoids discarding entire cities. The remaining missing columns (park-related
  statistics, field/court counts) are not relevant to the analysis and are excluded
  during column selection.
- **Yelp dataset (150,346 rows, 14 columns):** 103 businesses (0.07%) had no
  categories listed. Since category mapping is essential for encoding business
  type, these rows were dropped. The `attributes` and `hours` columns (9.14% and
  15.45% missing) are not needed and were not carried forward.
- **Duplicates:** Neither dataset contained duplicate rows. Yelp had zero
  duplicate `business_id` values.
- **Filtering:** Only open businesses (`is_open == 1`) were retained, as closed
  businesses do not reflect current business conditions.
- **Post-merge verification:** A final check confirmed zero remaining NaN values
  across all columns.

---

### 2. Feature Scaling and Categorical Variable Handling (Rubric: 20 pts)

**One-hot encoding:** Yelp's granular sub-categories were mapped to 15 major
business categories using keyword matching. Each business receives a 1 in every
major category it belongs to (multi-hot encoding), since a single business can
span multiple types (e.g., a bar that serves food). These per-business binary
columns were then aggregated to **city-level proportions** by taking the mean per
city, making cities with different numbers of businesses comparable.

**City-level aggregation** produced a dataset of 14 rows (one per city) with 21
features:
- 3 connectivity scores (Walk, Transit, Bike)
- 15 category proportion columns
- Average star rating, average review count, and total business count per city

**Feature scaling:** `StandardScaler` was applied to all 21 city-level features.
This is critical because the features span different scales (scores: 0-100,
proportions: 0-1, review counts: 0-hundreds, business counts: 0-thousands).
Without scaling, features with larger magnitudes would dominate Euclidean distance
calculations in clustering.

---

### 3. Dimensionality Reduction (Rubric: 20 pts)

**PCA (Principal Component Analysis)** was applied to the 21 scaled city-level
features. With 21 features and only 14 observations, we have more features than
data points — the "curse of dimensionality" makes distances less meaningful in
such high-dimensional space. PCA compresses correlated features into fewer
uncorrelated components.

- A cumulative explained variance plot was used to select the number of components
  at the 90% variance threshold.
- A 2-component projection was also created for visualization, with city labels
  annotated in the scatter plot.

---

### 4. Preprocessing Justification and Impact (Rubric: 20 pts)

Each preprocessing step includes an inline markdown cell in the notebook
justifying the decision:

| Step | Justification |
|---|---|
| Median imputation for Transit Score | Preserves all cities; median is robust to outliers |
| Dropping rows with no categories | Only 0.07% of data; cannot be meaningfully categorized |
| Filtering to open businesses | Closed businesses do not reflect current conditions |
| City-level aggregation | Matches analysis unit to connectivity score granularity |
| Category proportions (not counts) | Makes cities with different business counts comparable |
| StandardScaler | Equalizes feature influence in distance-based clustering |
| Multi-hot encoding | Preserves multi-category business information |
| PCA | Reduces features below n=14; addresses curse of dimensionality |

---

## Key Variables Available for Clustering

| Variable | Description |
|---|---|
| `city_pca_df` | PCA-reduced city feature matrix (14 rows, 90%+ variance) |
| `city_2d` | 2-component PCA projection for scatter plot visualization |
| `city_scaled_df` | Scaled city-level feature matrix (14 rows x 21 features) |
| `city_df` | Unscaled city-level dataset with city names |
| `combined_deduped` | Per-business dataset (42K rows) for drill-down analysis |
