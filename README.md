# Phishing Detection Robustness
 
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/tentorifrancescaDev/phishing-detection-robustness/blob/main/Progetto_AD.ipynb)
 
**Phishing Detection Robustness** is a study of how Machine Learning models for phishing website detection behave when their input data is imperfect or deliberately manipulated. A **Decision Tree** and a **Neural Network** are trained on the UCI PhiUSIIL Phishing URL dataset and then tested under four data corruption techniques, from random missing values to a targeted **mimicry evasion attack**. The project was developed as a university assignment for the Data Architecture course.
 
---
 
## Technologies
 
* **Python & Google Colab:** development and execution of the whole project in a Jupyter notebook.
* **ucimlrepo:** dataset loaded directly from the UCI Machine Learning Repository, with no local files.
* **pandas & NumPy:** data manipulation and implementation of the corruption techniques.
* **Matplotlib & Seaborn:** correlation heatmaps, confusion matrices and performance degradation plots with confidence intervals.
* **scikit-learn:** preprocessing (`StandardScaler`, `SimpleImputer`), stratified splitting (`StratifiedShuffleSplit`), `DecisionTreeClassifier` and evaluation metrics.
* **TensorFlow / Keras:** design and training of the Neural Network.
* **SciPy:** Student's t confidence intervals and statistical tests (paired t-test, Wilcoxon).
---
 
## Project Pipeline
 
* **Exploratory Data Analysis (EDA):** 235,795 URLs described by 54 features, with no missing values and a slight class imbalance (57.2% legitimate, 42.8% phishing). Pearson and Spearman correlation matrices were compared: Spearman, robust to outliers and non-linear monotonic relationships, revealed 28 strongly correlated feature pairs (|r| > 0.8) against the 5 found by Pearson.
* **Feature Dropping:** 13 features removed (from 54 to 41): high-cardinality text fields (`URL`, `Domain`, `TLD`, `Title`), leakage-prone derived scores (`URLSimilarityIndex`, `TLDLegitimateProb`) and redundant absolute counts, keeping their more informative ratio counterparts.
* **Stratified Split and Standardization:** 80% training and 20% test with the same class proportions; `StandardScaler` fitted on the training set only to prevent Data Leakage.
* **Baseline Training:** both models trained and evaluated on clean data.
* **Data Corruption:** four corruption techniques applied to the test set, measuring the performance degradation with respect to the baseline.
* **Statistical Validation:** every experiment is repeated over **10 stratified hold-out splits**, retraining the models from scratch each time. Results are reported as the mean with **95% confidence intervals** (Student's t distribution).
---
 
## Models
 
* **Decision Tree:** `max_depth = 7` to limit overfitting, `min_samples_split = 2` and `class_weight = 'balanced'`. Trained on the original (non-standardized) features.
* **Neural Network (Keras):** two hidden Dense layers with 32 and 16 ReLU neurons and a Sigmoid output neuron, Adam optimizer and binary cross-entropy loss. Trained on standardized features for up to 30 epochs (batch size 512), with Early Stopping on the validation true negatives.
Since a missed phishing website means a risk of credential theft, the models are compared mainly on the **Recall of the phishing class**.
 
---
 
## Data Corruption Techniques
 
* **Missing Data (MCAR):** test set cells are randomly set to `NaN` with a Missing Completely At Random mechanism, at rates of 0%, 5%, 10%, 20% and 40%, then imputed with the training-set median. It simulates pages whose content cannot be downloaded or rendered (timeouts, dynamic JavaScript content, anti-bot mechanisms).
* **Feature Occlusion:** at inference time, one feature at a time is replaced with its training-set mean, measuring how much each model relies on it. This identifies the critical features of each model.
* **Targeted Feature Noise:** Gaussian and uniform noise (0, 0.5, 1, 2 and 5 times the feature's standard deviation) applied only to the critical features found by the occlusion study: `NoOfImage`, `LineOfCode`, `NoOfSelfRef` and `NoOfExternalRef` for the Decision Tree, `LetterRatioInURL` for the Neural Network.
* **Mimicry Attack:** an evasion attack in which phishing samples are shifted towards the median profile of legitimate websites along the four critical structural features, with intensity α ∈ {0%, 25%, 50%, 75%, 100%}:
  `X_mimic = X_phish + α · (M_legit − X_phish)`
---
 
## Results
 
**Baseline** on the clean test set (47,159 URLs):
 
| Model           | Accuracy | Precision (phishing) | Recall (phishing) | F1-Score (phishing) |
| --------------- | :------: | :------------------: | :---------------: | :-----------------: |
| Decision Tree   | 0.9961   | 0.9962               | 0.9947            | 0.9955              |
| Neural Network  | 0.9997   | 0.9998               | 0.9995            | 0.9996              |
 
**Robustness:** Recall on the phishing class, from the clean baseline to the strongest level of each corruption technique (mean over 10 splits):
 
| Technique              | Strongest level             | Decision Tree        | Neural Network        |
| ---------------------- | --------------------------- | :------------------: | :-------------------: |
| Missing Data (MCAR)    | 40% missing values          | 0.995 → 0.716        | 1.000 → 0.920         |
| Feature Occlusion      | most critical feature       | −66 pp (`NoOfImage`) | −3 pp (`LetterRatioInURL`) |
| Targeted Feature Noise | maximum noise level         | 0.995 → 0.817        | 1.000 → 0.970         |
| Mimicry Attack         | α = 100%                    | 0.995 → 0.67         | 1.000 → 0.91          |
 
* The **Neural Network** degrades gradually under all four techniques: the information is distributed across many neurons, so corrupting a few features does not compromise the decision.
* The **Decision Tree** collapses abruptly as soon as a critical value crosses a split threshold, and becomes much less stable: in the occlusion study its confidence intervals span up to 20 percentage points, against 2 for the Neural Network.
* Under the mimicry attack the Decision Tree shows a characteristic **step-wise degradation**, a direct effect of its rigid split thresholds.
* In real-world scenarios where data quality is not guaranteed, the Neural Network is the more reliable choice.
---
 
## Limitations and Future Work
 
* **Dataset specificity:** PhiUSIIL features follow a very specific extraction methodology, so the conclusions should be validated on datasets collected in different contexts.
* **Feature independence:** each corruption technique perturbs features independently, ignoring their correlations (e.g. the mimicry attack changes `LineOfCode` but leaves the correlated `NoOfJS` and `NoOfCSS` untouched). A future *Constrained Mimicry Attack* could propagate the perturbation to co-dependent features using the Spearman correlation matrix and a Cholesky decomposition.
* **Statistical testing:** repeated hold-out splits are not independent, so a standard paired t-test may be too liberal. A corrected variance estimator, such as the Nadeau-Bengio correction, would give a more conservative significance estimate.
---
 
## Development Team
 
University project developed by:
* [Alessandro Messa](https://github.com/DiagonDev)
* [Matteo Ronchi](https://github.com/MatteoRonchiDev)
* [Francesca Tentori](https://github.com/tentorifrancescaDev)
---
 
## Project Structure
 
```
├── Progetto_AD.ipynb       # Colab notebook: EDA, preprocessing, training and data corruption experiments
├── Report_ProgettoAD.pdf   # Full project report (in Italian)
└── README.md
```
 
> **Documentation Note:** the full analysis, including the correlation study, the formal description of each corruption technique, all degradation plots and the discussion of the results, is available in the **`Report_ProgettoAD.pdf`** file.
 
---
 
## How to Run
 
* **On Google Colab:** open the notebook with the *Open in Colab* badge at the top of this page and run all cells with *Runtime > Run all*. The dataset is downloaded automatically from the UCI Machine Learning Repository. Since every experiment retrains both models over 10 splits, a full run takes some time.
 
Neural Network training is not fully deterministic, so re-running the notebook may give slightly different values from those reported above.
 
---
 
## Dataset
 
The project uses the **PhiUSIIL Phishing URL** dataset (235,795 URLs, 54 features), released under the [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) license:
 
> Prasad, A., & Chandra, S. (2024). *PhiUSIIL Phishing URL (Website)* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.1016/j.cose.2023.103545
