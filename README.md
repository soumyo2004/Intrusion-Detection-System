# Network Intrusion Detection System (IDS) Using Machine Learning & Ensemble Methods

Due to exponential growth in network traffic and increasingly sophisticated cyber threats, modern networks face serious security vulnerabilities. 
Traditional Intrusion Detection Systems (IDS) rely on signature-based detection (which misses zero-day attacks and novel patterns) or standalone anomaly-based approaches (which suffer from high false positive rates).

This project implements an automated, machine learning-driven IDS for network log analysis. To overcome severe data redundancy and overfitting seen in legacy benchmarks like KDD Cup '99 (which artificially inflated accuracy through ~70% duplicate records), this pipeline transitions to the modern **[UNSW-NB15 dataset](https://research.unsw.edu.au/projects/unsw-nb15-dataset)**. The architecture utilizes a **Weighted Soft-Voting Ensemble Classifier** combining **Decision Tree**, **Random Forest**, and **XGBoost** to achieve multi-class classification across 10 network traffic categories (Normal + 9 distinct attack types).

---

### Project Workflow

1. **Data Preprocessing & Cleaning:** Handle missing values, encode multi-class attack categories, and log-transform skewed bit-rate features.
2. **Feature Scaling & Engineering:** Standardize continuous features via `StandardScaler` and extract top predictive attributes.
3. **Model Training:** Train three distinct supervised classifiers:
   * **Decision Tree** (with depth constraints and balanced class weights)
   * **Random Forest** (ensemble of 100 estimators using parallel CPU cores)
   * **XGBoost** (histogram-based boosting via `tree_method='hist'` for low-latency fitting)
4. **Weighted Soft Voting:** Aggregate class probability vectors using an empirical weighting ratio (DT: 1, RF: 2, XGB: 2).
5. **Performance Diagnostics:** Evaluate model stability and generalization using multi-class confusion matrices, precision/recall breakdowns, and cross-validation learning curves.

---

### Key Methodology & Pipeline

#### 1. Preprocessing & Encoding
* **Categorical Encoding:** `LabelEncoder` is applied across discrete protocol, service, and connection state features (`proto`, `service`, `state`).
* **Normalization & Scaling:** Continuous network vectors are scaled using `StandardScaler` to ensure uniform weighting across gradient and distance computations.
* **Handling Class Imbalance:** Extreme sample distribution differences (e.g., high support for Normal and Generic vs. low support for Backdoor, Shellcode, and Worms) are managed using `class_weight='balanced'`.

#### 2. Base Classifiers
* **Decision Tree (DT):** Provides transparent baseline decision splits (`max_depth=12`, `min_samples_leaf=25`).
* **Random Forest (RF):** Reduces model variance through bagging across 100 decision trees (`max_depth=10`, `n_jobs=-1`).
* **XGBoost (XGB):** Minimizes bias using sequential gradient boosted trees optimized via fast histogram binning (`tree_method='hist'`, `learning_rate=0.1`).

#### 3. Weighted Soft Voting
The final prediction assigns weights based on each estimator's individual empirical performance:

$$\hat{P}(c) = \frac{1 \cdot P_{\text{DT}}(c) + 2 \cdot P_{\text{RF}}(c) + 2 \cdot P_{\text{XGB}}(c)}{5}$$

---

### Experimental Results

* **Evaluation Dataset:** UNSW-NB15 (82,332 training records, 175,341 testing records across 45 features).
* **Multiclass Accuracy:** **76.39%** across 10 classes on the test set.
* **Weighted Metrics:** **0.79 Precision**, **0.76 Recall**, and **0.75 F1-Score**.
* **Training Latency:** Fast ensemble fitting completed in **6.09 seconds**.
* **Generalization:** Successfully eliminated the synthetic memorization of the legacy KDD Cup '99 pipeline, establishing a robust bias-variance trade-off on realistic modern network traffic distributions.

---

### How to Run

1. **Clone the Repository**
git clone [https://github.com/](https://github.com/)<your-username>/Network-IDS-LogAnalysis.git
cd Network-IDS-LogAnalysis

2. **Set Up a Virtual Environment**
python -m venv ids-env

# On Linux / macOS:
source ids-env/bin/activate

# On Windows:
ids-env\Scripts\activate

3. **Install Dependencies**
pip install -r requirements.txt

4. **Dataset Setup**
Ensure UNSW_NB15_training-set.csv and UNSW_NB15_testing-set.csv are placed in the project root directory.

5. **Run the Notebook**
   jupyter notebook
   
