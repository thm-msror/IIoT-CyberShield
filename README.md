# IIoT-CyberShield

## DSAI4202 Group Project

### Team Members

- Tehreem Masroor (60302531)
- Djihane Mahraz (60300310)
- Urooj Shah (60300832)

## Project Overview

This project applies big data analytics and machine learning to IoT intrusion detection using Apache Spark.

The dataset used is CIC-IIoT-2025 (approximately 3GB), containing benign and malicious IoT network traffic. We process this large-scale data in a distributed environment, engineer meaningful features, and train multiple Spark ML models to classify traffic as normal or attack.

## Objectives

- Ingest large-scale IoT traffic data in Spark
- Clean and preprocess features for reliable modeling
- Explore attack patterns and class behavior through EDA
- Build and compare multiple distributed classification models
- Use Spark ML Pipeline for scalable, reproducible end-to-end training
- Evaluate generalization and overfitting behavior on unseen data

## Tech Stack

- Apache Spark (PySpark)
- Azure Databricks
- Azure Data Lake Storage Gen2
- Python (Pandas, Matplotlib, Seaborn for supporting analysis and visualization)

## Dataset

- Source: CIC-IIoT-2025
- Scale: approximately 685,671 records (from notebook outputs)
- Features include:
  - Network traffic metrics (packet counts, packet size, TTL, window size, protocols)
  - Time and interval features
  - Log-derived features
  - Multi-level attack labels

## Project Workflow (Stages)

### 1. Data Ingestion

- Dataset was downloaded via Kaggle API and uploaded to ADLS Gen2.
- Loaded into Spark DataFrame from cloud storage in Databricks.

### 2. Data Cleaning and Preprocessing

#### 2.1 Data quality and shape checks

- Verified schema after ingestion to confirm data types across numerical, categorical, and timestamp fields.
- Confirmed large-scale structure was suitable for distributed Spark processing.

#### 2.2 Missing value analysis

- Checked null and empty values across all columns.
- Result: no major missingness that required imputation.
- Decision: keep full dataset without dropping rows for null handling.

#### 2.3 Duplicate analysis

- Compared total row count vs distinct row count.
- Result: no significant duplicate-record issue.
- Decision: preserve records to avoid losing potentially meaningful traffic behavior.

#### 2.4 Semantic renaming of label columns

- Renamed label1 to traffic_class.
- Renamed label2 to attack_category.
- Renamed label3 to attack_type.
- Renamed label4 to attack_subtype.
- Renamed label_full to detailed_label.
- Reason: improve readability and make later model/report sections self-explanatory.

#### 2.5 Column name normalization

- Standardized column names by replacing unsupported/special characters with underscores.
- Reason: improves compatibility with Spark ML transformations and avoids syntax issues.

#### 2.6 Outlier handling strategy

- Distribution checks showed extreme values in packet-related features.
- Decision: outliers were intentionally retained.
- Reason: in intrusion detection, extreme values are often attack signals, not noise.

### 3. Exploratory Data Analysis (EDA)

- Label distribution analysis
- Numerical feature distribution analysis
- Feature-vs-label behavior comparison
- Correlation heatmap for selected features

### 4. Feature Engineering

#### 4.1 Feature exclusion to reduce leakage and improve generalization

- Dropped device_name.
- Dropped device_mac.
- Dropped network_ips_all.
- Dropped network_ips_dst.
- Dropped network_ips_src.
- Dropped network_macs_all.
- Dropped network_macs_dst.
- Dropped network_macs_src.
- Dropped timestamp.
- Reason: these fields are environment-specific identifiers and can hurt transferability to unseen traffic.

#### 4.2 Target encoding

- Encoded traffic_class into numeric label using StringIndexer.
- Reason: Spark ML classification models require numeric target labels.

#### 4.3 Feature vector construction

- Excluded label and label-derived columns from predictors.
- Assembled selected numerical predictors into a single features vector using VectorAssembler.
- Reason: Spark ML estimators consume vectorized features as model input.

#### 4.4 Feature scaling

- Applied StandardScaler to produce scaled_features.
- Reason: scaling keeps features on comparable ranges and improves training stability.

#### 4.5 Pre-pipeline cleanup for unified workflow

- Removed pre-existing features and scaled_features before building the full pipeline block.
- Reason: avoids duplicate transformation columns and ensures a clean end-to-end pipeline execution.

### 5. Model Selection and Training

#### Compared four Spark ML classifiers:

- Logistic Regression
- Decision Tree
- Random Forest
- Gradient Boosted Trees

### 6. Model Evaluation

Evaluated using:

- Accuracy
- F1-score
- Weighted Precision
- Weighted Recall

### 7. Spark ML Pipeline and Tuning

- Rebuilt the workflow as a unified Spark ML Pipeline:
  - `VectorAssembler` -> `StandardScaler` -> `RandomForestClassifier`
- Performed hyperparameter tuning with `ParamGridBuilder` and 3-fold cross-validation
- Validated final model on independent test set

## Model Comparison Results (From Notebook)

| Model                  | Accuracy |     F1 | Precision | Recall |
| ---------------------- | -------: | -----: | --------: | -----: |
| Logistic Regression    |   0.9084 | 0.9066 |    0.9158 | 0.9084 |
| Decision Tree          |   0.9526 | 0.9521 |    0.9559 | 0.9526 |
| Random Forest          |   0.9396 | 0.9387 |    0.9451 | 0.9396 |
| Gradient Boosted Trees |   0.9560 | 0.9556 |    0.9588 | 0.9560 |

### Tuned Final Pipeline (Random Forest + CV)

| Metric    |  Score |
| --------- | -----: |
| Accuracy  | 0.9552 |
| F1        | 0.9547 |
| Precision | 0.9583 |
| Recall    | 0.9552 |

### Overfitting Check

- Train Accuracy: 0.9555
- Test Accuracy: 0.9552

The train-test gap is very small, indicating good generalization.

## Manual vs Spark ML Pipeline Execution Time Comparison

The notebook currently indicates that the pipeline approach is faster, but exact execution timings must be collected from Databricks run history.

Fill the table below once timings are retrieved:

| Workflow               | Step/Cell Group                                                              | Execution Time (seconds) | Notes                                 |
| ---------------------- | ---------------------------------------------------------------------------- | -----------------------: | ------------------------------------- |
| Manual Step-by-Step ML | Feature assembly + scaling + individual model training/prediction/evaluation |                      TBD | Collect from Databricks cell runtimes |
| Spark ML Pipeline      | Pipeline fit + transform (`4.7`)                                             |                      TBD | Collect from Databricks cell runtimes |
| Spark ML Pipeline + CV | Cross-validation + tuned inference (`4.8` onward)                            |                      TBD | Optional comparison row               |

## Why Spark ML Pipeline and Vectorization Matter

### Spark ML Pipeline

Spark ML Pipeline packages preprocessing and modeling into one reproducible workflow. This improves:

- Scalability on distributed clusters
- Maintainability of production ML code
- Consistency of transformations between training and inference

### Vectorization

Spark ML algorithms expect a single feature vector per row. `VectorAssembler` and scaling steps convert many columns into optimized vector form, enabling:

- Faster distributed computation
- Cleaner integration with ML algorithms
- Easier extension for tuning and cross-validation

## Spark vs Traditional Processing

Apache Spark is preferred for this project because the dataset size and iterative ML workflow benefit from distributed in-memory computation.

Compared with single-machine processing, Spark provides:

- Better scalability for large IoT traffic data
- Faster iterative model development
- Easier pipeline orchestration for ML at scale

Hadoop ecosystem concepts are relevant in the architecture context (distributed storage/processing foundations), while this implementation primarily leverages Spark on Databricks with ADLS.

## Conclusion

This project demonstrates a complete big-data ML lifecycle for intrusion detection:

- Data ingestion from cloud storage
- Robust preprocessing and feature engineering
- Comparative model benchmarking
- Final tuned Spark pipeline with strong generalization

The results show that tree-based methods are highly effective for this problem, and Spark ML Pipeline provides the best structure for scalable, reproducible deployment-ready workflows.
