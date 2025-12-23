# PROJECT: NFL PREDICTIVE ENGINE (THE "DUAL-KEY" PROTOCOL)
# VERSION: System Archive 01
# STATUS: Operational

## 1. EXECUTIVE OVERVIEW
This project is a professional-grade sports analytics platform designed to identify market inefficiencies ("edges") in NFL betting lines. Unlike standard prediction models that simply pick winners based on team record, this system employs a **"Dual-Key Protocol."**

It operates two independent modeling tracks:
1.  **The Vegas-Aware (VA) Baseline:** Determines the "Market Consensus" by incorporating betting spreads into the logic.
2.  **The No-Vegas (NV) Ensemble:** A "Pure Football" model that predicts outcomes solely based on efficiency metrics (EPA, Success Rate), completely blind to public sentiment or odds.

An **"Edge"** is identified *only* when the Pure Football model significantly diverges from the Market Consensus.

---

## 2. SYSTEM ARCHITECTURE & FILE STRUCTURE

The system is built as a sequential pipeline. Each notebook feeds into the next via intermediate storage (Pickle/Parquet).

### [Layer 1: Data Ingestion]
* **File:** `01_data_ingestion.ipynb`
* **Core Tech:** `nflreadpy`, `nflverse`
* **Function:**
    * Ingests raw Play-by-Play (PBP) data from 2002–2024.
    * Ingests **Game Officials (Referee)** data to track crew-specific biases (e.g., holding call frequency).
    * Normalizes team abbreviations (e.g., handling the STL -> LA Rams transition).

### [Layer 2: Feature Engineering]
* **File:** `03_master_feature_engineering.ipynb`
* **Core Tech:** `pandas`, `StandardScaler`
* **Key Methodologies:**
    * **Context Over Volume:** Uses EPA (Expected Points Added) per play rather than raw yardage.
    * **Rolling Windows:** Calculates 3-game, 5-game, and 8-game rolling averages to capture recent form vs. long-term talent.
    * **Roster Health:** Generates specific features for `home_key_players_out` and `injury_advantage` using participation reports.
    * **Situational Variables:** Calculates `rest_days` (Short Week vs. Bye Week) and travel distance.

### [Layer 3: The Modeling Core]
* **File:** `04VA_model_baseline.ipynb` (The Benchmark)
    * *Algorithm:* Logistic Regression.
    * *Input:* Advanced Stats + Vegas Spread.
    * *Accuracy:* ~67.3%.
    * *Role:* Acts as the "Control Group."

* **Files:** `05NV_xgboost.ipynb`, `05NVb_svm.ipynb`, `05NVc_neural_network.ipynb` (The Pure Football Models)
    * **XGBoost:** Gradient Boosted Decision Trees. Good at finding non-linear thresholds (e.g., "Wind > 15mph kills Passing EPA").
    * **SVM:** Support Vector Machine with Radial Basis Function (RBF) kernel. Good at finding hyperplanes in high-dimensional defensive metrics.
    * **Neural Network:** Multi-Layer Perceptron (64 -> 32 neurons). Good at capturing abstract patterns that simpler models miss.

### [Layer 4: Ensemble Optimization]
* **File:** `07NV_dual-key_protocol.ipynb`
* **Methodology:**
    * Implemented a **Brute-Force Grid Search** to determine the optimal voting power for each model.
    * **Final Weights:** SVM (14) > Neural Network (8) > XGBoost (2).
    * *Why?* The SVM proved most stable against noise, while XGBoost was prone to overfitting on recent data. The ensemble approach achieved ~59.93% accuracy on unseen test data.

### [Layer 5: The War Room (Execution)]
* **File:** `09VC_war_room.ipynb` & `11_war_room_dashboard.ipynb`
* **Core Tech:** `Streamlit`, `Matplotlib`
* **Function:**
    * Converts live Vegas spreads into Implied Win Probabilities.
    * Compares Implied Probability vs. Model Probability to calculate `Edge`.
    * **Money Management:** Implements the "Quarter Kelly Criterion" to recommend wager sizing based on edge confidence.

---

## 3. TECHNICAL SPECIFICATIONS & REQUIREMENTS

**Primary Stack:**
* Python 3.10+
* Pandas / NumPy (Data Manipulation)
* Scikit-Learn (SVM, Logistic Regression, Scaling, Grid Search)
* XGBoost (Gradient Boosting)
* Streamlit (Frontend Dashboard)

**Key Metrics Defined:**
* **EPA (Expected Points Added):** The difference in Expected Points between the start and end of a play.
* **CPOE:** Completion Percentage Over Expected (measures QB accuracy relative to difficulty).
* **Success Rate:** Percentage of plays that gain positive EPA.

---

## 4. INSTALLATION & USAGE

1.  **Environment Setup:**
    ```bash
    pip install pandas numpy scikit-learn xgboost nflreadpy streamlit matplotlib
    ```

2.  **Data Refresh:**
    Run `01_data_ingestion.ipynb` to pull the latest season data.

3.  **Model Retraining (Optional):**
    If the season has progressed significantly (6+ weeks), run `07NV_dual-key_protocol.ipynb` to re-optimize ensemble weights.

4.  **Launch Dashboard:**
    ```bash
    streamlit run 11_war_room_dashboard.ipynb
    ```

---

## 5. FUTURE ROADMAP (PLANNED UPGRADES)

* **Player-Level Props:** Expansion of the XGBoost module to predict individual rushing/receiving yardage totals.
* **Live Weather Integration:** Real-time API hook for hourly wind/temp updates on game day.
* **Sentiment Analysis:** NLP scanning of beat writer tweets to detect "locker room morale" issues (e.g., Coach on the hot seat).
