# NIFTY Options IV Surface Reconstruction

## Overview
This project reconstructs missing implied volatility (IV) values in a NIFTY options volatility surface.

The solution combines:

1. **Cross-sectional polynomial smile fitting**
   - Uses neighboring strikes within the same timestamp.
   - Fits a weighted polynomial volatility smile.
   - Produces a baseline IV estimate.

2. **LightGBM residual correction**
   - Learns the residual error of the smile model.
   - Uses only historical information (strictly causal).
   - Improves prediction accuracy without introducing look-ahead bias.

---

## Key Features

### Strictly Causal Design
The model never uses future observations when predicting a missing value.

For every timestamp:

- Historical residuals are used.
- Future rows are ignored.
- No leakage from validation data.

### Smile-Based Surface Reconstruction
For a missing strike:

- Nearby observed strikes are collected.
- A weighted polynomial smile is fitted.
- The missing IV is estimated from the fitted curve.

### Residual Learning with LightGBM
A LightGBM regressor is trained on smile-model residuals.

Features include:

- Strike
- Call/Put flag
- Smile prediction
- Previous IV
- Previous residual
- Row index (time)

The final prediction is:

Final IV = Smile Prediction + Residual Correction

---

## Project Structure

### Input Files

#### dataset.csv
Raw dataset containing:

- datetime
- underlying_price
- Multiple option IV columns (CE and PE)

#### filled_dataset.csv
Dataset after filling all missing IV values.

#### submission.csv
Competition submission file containing reconstructed values.

---

## Workflow

### Step 1: Load Data

- Read dataset.
- Identify IV columns.
- Separate Call (CE) and Put (PE) contracts.

### Step 2: Cross-Sectional Prediction

For each strike:

- Collect neighboring observed strikes.
- Fit a weighted polynomial smile.
- Generate baseline IV prediction.

### Step 3: Build Residual Dataset

For observed IV values:

Residual = Actual IV − Smile Prediction

These residuals become the LightGBM training target.

### Step 4: Train LightGBM

Train on residuals using causal historical features.

### Step 5: Fill Missing Values

For every missing IV:

1. Generate smile estimate.
2. Predict residual correction.
3. Combine both estimates.
4. Enforce positive IV floor.

### Step 6: Validation

Evaluate reconstruction quality using:

- MSE
- RMSE
- MAE

---

## LightGBM Configuration

- Objective: Regression
- Metric: RMSE
- Learning Rate: 0.05
- Number of Leaves: 31
- Estimators: 300
- Random State: 42

---

## Outputs

### filled_dataset.csv

Contains:
- Original observations
- Reconstructed missing IV values

### submission.csv

Competition-ready prediction file.

---

## Model Advantages

- No look-ahead bias
- Utilizes volatility smile structure
- Captures temporal residual patterns
- Handles sparse strikes robustly
- Fast training and inference

---

## Warning Fix

The notebook was updated to remove:

UserWarning:
X does not have valid feature names,
but LGBMRegressor was fitted with feature names

Fix:
- Train LightGBM using DataFrames with explicit column names.
- Predict using DataFrames containing the same feature names.

This removes warning spam while preserving model behavior.

---

## Author :- Ashish Kumar Agrawal

NIFTY Options IV Surface Reconstruction
Cross-sectional Polynomial Smile + LightGBM Residual Correction
