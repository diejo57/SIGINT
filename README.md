# RF Emitter Identification and Geolocation Analysis

This project is a comprehensive solution for processing raw radio frequency (RF) signal data to classify and locate unknown radar emitters. It simulates a real-world signals intelligence (SIGINT) and Electronic Intelligence (ELINT) workflow, turning noisy sensor data into actionable intelligence.

---
## Key Features

* **Advanced Clustering:** Utilizes the DBSCAN algorithm on a 4-dimensional feature space (`Frequency`, `PRI`, `PW`, `Time`) to accurately classify emitters.
* **Interactive Parameter Selection:** Includes an interactive, two-step workflow to generate a diagnostic K-Distance plot, allowing the analyst to make a data-driven choice for the critical `eps` hyperparameter.
* **High-Accuracy Geolocation:** Implements a robust hybrid location estimator that:
    1.  First attempts a high-precision, non-linear optimization (SciPy) based on spherical geometry.
    2.  Automatically falls back to a linear least-squares method on a UTM projection (`pyproj`) if the primary solver fails, ensuring a location estimate is always provided.
* **Configurable & Reusable:** The script is designed for easy use with new datasets, with key parameters and column names defined in a central configuration section.

---
## Methodology

The analysis is performed in two primary stages: Emitter Classification and Emitter Geolocation.

### 1. Emitter Classification

The goal of this stage is to group the thousands of raw, unlabeled signal observations into distinct clusters, where each cluster represents a unique radar emitter.

* **Feature Engineering:** Four key features are used to create a unique "fingerprint" for each signal: `Freq` (Frequency), `PRI` (Pulse Repetition Interval), `PW` (Pulse Width), and `Time`.
* **Scaling:** All features are scaled using `StandardScaler` to have zero mean and unit variance. This is essential for distance-based algorithms like DBSCAN to function correctly.
* **Clustering:** The DBSCAN algorithm is used to identify core samples of high density and expand clusters from them. It's well-suited for this task due to its ability to find arbitrarily shaped clusters and handle noise effectively.

### 2. Emitter Geolocation

Once the observations are clustered, this stage calculates the geographic position of each identified emitter.

* **Angle of Arrival (AOA):** The method uses the aircraft's position (`Lat`, `Lon`) and the measured `Angle` of arrival for each signal in a cluster.
* **Hybrid Solver:** To ensure both accuracy and robustness, a hybrid location solver is used. It first attempts to find the location using a non-linear solver (`scipy.optimize.minimize`) that minimizes the true geometric distance on a sphere (cross-track distance). If this high-accuracy method fails to converge (often due to poor signal geometry), it falls back to a more robust, linear least-squares solver on a Universal Transverse Mercator (UTM) projection.

---
## How to Use

This script uses an interactive, two-step workflow to ensure the best parameters are chosen for the analysis.

### 1. Prerequisites

Make sure you have the required Python libraries installed:
```bash
pip install pandas numpy scikit-learn scipy pyproj matplotlib
```

### 2. Configuration

Open the script and set the following variables in the configuration section:
* `INPUT_FILE`: The name of the observation data file (e.g., `'Observations_011.xlsx'`).
* `CHOSEN_EPS`: **Leave this as `0.0` for the first run.**
* `MIN_SAMPLES`: Set the desired `min_samples` value (e.g., `20`).
* Column Name Definitions: Ensure the column name variables match the headers in your input file.

### 3. Execution Workflow

1.  **First Run (Generate Plot):** Run the script from your terminal. It will load the data and generate a diagnostic plot named `k_distance_plot.png`. It will then stop.
 
2.  **Inspect and Update:** Open the `k_distance_plot.png` file. Look for the "elbow" of the curve and note the corresponding value on the Y-axis. This is your optimal `eps`. Update the `CHOSEN_EPS` variable in the script with this value.

3.  **Second Run (Final Analysis):** Run the exact same script again. This time, because `CHOSEN_EPS` has been updated, the script will skip the plotting and perform the full analysis, printing a final report and saving the results to a CSV file.
