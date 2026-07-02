# MMS Data Engineering & Deep Learning Challenge

This repository contains a comprehensive data preprocessing, feature engineering, and predictive modeling pipeline developed for the **MediaMarktSaturn (MMS) Engineering Challenge**. The primary objective of this project is to build an end-to-end machine learning solution to accurately predict daily store sales (`sales_per_day`) using structured retail data.

## 📌 Project Overview
The core task is a regression problem focused on predicting daily performance metrics across multi-national retail outlets (MediaMarkt and Saturn). The solution transitions from an initial exploratory baseline to an optimized Deep Learning architecture, demonstrating structured techniques for real-world messy dataset engineering.

### Dataset Schema & Features
* **`sales_per_day`**: Total sales amount for an outlet on a specific day (**Target Label**).
* **`customers_per_day`**: The metric indicating daily store traffic volume.
* **`outlet_id`**: Unique identifier for specific physical retail markets.
* **`brand`**: Retail brand category flags (`MediaMarkt` or `Saturn`).
* **`country`**: Geographical location of the operating outlet.
* **`currency`**: The transaction currency (e.g., EUR, CHF).
* **`week_id` / `weekday`**: Temporal features mapping calendar weeks and days.

---

## 🛠️ Implementation Approach & Pipeline

The notebook implements a structured, 11-step pipeline designed to prevent data leakage and maximize the model's ability to capture patterns:

### 1. Data Cleaning & Imputation
* **Anomaly Handling**: Detected extreme placeholder values (e.g., `-999999.0`) and securely masked them to `0` at the dataframe level to avoid skewing metrics.
* **Missing Values**: Applied custom data imputation mapping fallback strings (`'Unknown'`) and numerical zeroes to preserve structural data instead of blindly dropping rows.

### 2. Feature Engineering & Vectorization
* **Categorical Encoding**: Handled non-numeric attributes (`brand`, `country`, `weekday`) via `LabelEncoder` to efficiently vectorize text features for statistical ingestion.
* **Correlation Filtering**: Conducted Pearson Correlation Matrix analysis using Seaborn heatmaps. Discarded weak structural features showing near-zero dependencies with the target variable to reduce dimensionality and over-fitting risk.
* **Data Splitting**: Partitioned arrays into Train (80%) and Test (20%) datasets prior to computing normalization metrics, completely isolating the test subset from feature scaling leakage.
* **Feature Standardization**: Scaled high-variance numerical metrics (`customers_per_day`) via `StandardScaler` to smooth optimization paths.

### 3. Model Architecture & Evaluation

#### Baseline: Classical Machine Learning
* Built a standard **Linear Regression** model as a benchmark.
* **Baseline Performance**: Achieved an $R^2$ score of approximately `~0.39` on both train and test partitions.
* **Error Metrics**: Evaluated utilizing Mean Absolute Error (MAE: `~13,423`) and Root Mean Squared Error (RMSE: `~17,326`).

#### Main Solution: Deep Learning Regressor
* Implemented a fully connected, deep feedforward Neural Network built using TensorFlow/Keras.
* **Network Topology**:
  * Input layer mapping structured features.
  * 4 Dense hidden layers with decreasing capacities (`10 -> 8 -> 6 -> 4` neurons) utilizing `ReLU` activations.
  * Interwoven **L2 Kernel Regularization** to strictly bound weight matrices and control validation variance.
  * Single node output layer configured without an activation function to suit the unbound regression boundary.
* **Optimization Strategy**: Compiled using the `RMSprop` optimizer with a tuned learning rate (`0.01`) minimizing Mean Squared Error (`mse`).
* **Results**: Significantly outperformed the baseline model, slashing error dimensions down to a final Testing MAE of **`4336.89`**.

---

## 📈 Key Visualizations & Training Convergence
The repository includes comprehensive validation charts demonstrating stable learning curves over `200` training epochs. The convergence profiles highlight a steep, smooth decline in Mean Absolute Error without diverging gaps, confirming robust regularization parameters.

---

## 🚀 Future Roadmap & Real-Time Production Architecture
To push this model into live production environment streams, the planned architecture addresses the challenge's 3 distinct system inputs:
1. **Real-Time Pipeline**: Utilizing **Google Cloud Pub/Sub** to stream instantaneous `sales_per_day` API connectors (System A).
2. **Batch Processing**: Deploying daily automated **Cloud Composer (Airflow)** workflows to parse daily customer batch flat-files from SFTP storage (System B).
3. **Data Warehouse Integration**: Linking static store locations, brand attributes, and currency lookups directly from **BigQuery** (System C).
4. **Orchestrated ETL & Inference**: Streaming all inputs into **Apache Beam / Google Dataflow** for real-time vector preprocessing, feeding directly into **Vertex AI** endpoints for online predictions.
