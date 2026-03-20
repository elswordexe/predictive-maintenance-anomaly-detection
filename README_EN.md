# Predictive Maintenance Project (ai4i2020)

This project analyzes the `ai4i2020` dataset to predict machine failures using a hybrid artificial intelligence system.

## About the Dataset

This synthetic dataset is modeled after an existing milling machine and consists of 10,000 data points stored as rows with 14 features in columns.

### Features:
- **UID**: Unique identifier ranging from 1 to 10,000.
- **Product ID**: Consisting of a letter L, M, or H for quality variants (Low 50%, Medium 30%, High 20%) and a variant-specific serial number.
- **Type**: Product type (L, M, or H).
- **Air temperature [K]**: Generated using a normalized random walk process (300 K +/- 2 K).
- **Process temperature [K]**: Generated using a random walk process (Air temperature + 10 K +/- 1 K).
- **Rotational speed [rpm]**: Calculated from a power of 2860 W, overlaid with normally distributed noise.
- **Torque [Nm]**: Torque values are normally distributed around 40 Nm (SD = 10 Nm, no negative values).
- **Tool wear [min]**: Tool wear increases by 5/3/2 minutes depending on the quality variant (H/M/L).
- **Machine failure**: Binary label indicating whether the machine has failed.

### Failure Modes:
The machine fails if at least one of the following modes is true:
1. **Tool Wear Failure (TWF)**: Tool wear between 200 and 240 min.
2. **Heat Dissipation Failure (HDF)**: Temperature difference < 8.6 K and speed < 1380 rpm.
3. **Power Failure (PWF)**: Power (Torque * Speed) < 3500 W or > 9000 W.
4. **Overstrain Failure (OSF)**: Product of tool wear and torque > threshold (L: 11k, M: 12k, H: 13k).
5. **Random Failures (RNF)**: 0.1% chance of random failure.

### Attribution:
S. Matzka, "Explainable Artificial Intelligence for Predictive Maintenance Applications," 2020 Third International Conference on Artificial Intelligence for Industries (AI4I), 2020, pp. 69-74, doi: 10.1109/AI4I49448.2020.00023.

## Data Analysis

### 1. Data Understanding
- **Data Type**: Structural analysis with `df.info()`.
- **Descriptive Statistics**: Statistical overview with `df.describe()`.
- **Preprocessing**: Removal of unnecessary columns (`UDI`, `Product ID`).

### 2. Distribution and Correlation Analysis
- **Distribution**: Most variables follow normal or multimodal distributions.
![Distribution](NoteBooks/figures/distribution.png)
- **Correlation**: The matrix shows strong relationships between torque and speed (inverse), as well as between temperatures.
![Correlation](NoteBooks/figures/correlation.png)

### 3. Boxplot Analysis (Key Observations)
Comparing variables against the `Machine failure` target reveals crucial insights:

![Boxplots](NoteBooks/figures/boxplots.png)

1. **Outliers and Box-plots**: The presence of atypical observations is noted via box-plots. For predictive maintenance, these points are not errors but weak signals of failure.
2. **Trends (Horizontal Reading)**: Charts (e.g., Torque vs Failure) show an upward trend in torque during failures. A machine under high torque stress has a significantly higher probability of failure.
3. **Dispersion (Vertical Reading)**: Increased heterogeneity of physical parameters is observed during failures (notably speed and torque). Measurement instability is often a precursor to a machine stop.

### 4. Outlier Management and Feature Engineering
Extreme values of torque (`Torque`) and speed (`Rotational speed`) have been transformed into alert variables:
- **Torque_Alert**: Based on IQR, identifies extreme mechanical stress.
- **Speed_Alert**: Identifies overspeed.

---

## Model Performance

### 1. Random Forest (ML Component)
The Random Forest model achieves an accuracy of over **99%**.
- **Accuracy**: ~0.99
- **ROC AUC**: ~0.98

#### Confusion Matrix:
![Confusion Matrix](NoteBooks/figures/confusion_matrix.png)
The matrix shows that the model successfully identifies almost all failures (True Positives) with a very low number of false positives, which is crucial for avoiding unnecessary maintenance.

### 2. Isolation Forest (Anomaly Component)
This unsupervised model identifies approximately 5% of the data as statistical anomalies. it captures "invisible" or random failures that the supervised model might miss.

---

## Hybrid AI System

The final decision system combines the three pillars to classify risk:

![Hybrid Results](NoteBooks/figures/hybrid_results.png)

### Defined Risk Levels:
- **High Risk**: Critical. The machine shows statistical anomalies AND ML/Expert failure predictions. Immediate action required.
- **Medium Risk**: Increased monitoring. One of the components signals unusual behavior.
- **Low Risk**: Nominal operation.

### Hybrid Decision Results:
Crossing with actual failures shows that the **"High Risk"** level captures the majority of real failures, while the **"Medium Risk"** level serves as a buffer zone for preventive maintenance.
