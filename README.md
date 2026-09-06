# Reliable and Efficient IoT Intrusion Detection

A leakage-aware machine-learning study of efficient multiclass IoT intrusion detection using group-aware evaluation, XGBoost feature reduction, and probability calibration.

## Key Result

The final model is an **XGBoost classifier using only 6 original flow features**:

1. `Flow_Duration`
2. `Dst_Port`
3. `Src_Port`
4. `Protocol`
5. `Flow_Pkts/s`
6. `Init_Bwd_Win_Byts`

The reduced XGBoost model retained competitive locked-test performance while using only six original flow features.

On the **locked unique-group test set**, it achieved:

- **Macro F1: 0.9488**
- **Accuracy: 97.33%**
- **Balanced Accuracy: 97.16%**

### Locked Unique-Group Test Performance

| Metric | Result |
|---|---:|
| Accuracy | 0.9733 |
| Balanced Accuracy | 0.9716 |
| Macro Precision | 0.9310 |
| Macro Recall | 0.9716 |
| Macro F1 | **0.9488** |
| Weighted F1 | 0.9744 |

The Top-6 model retained **99.30% of the full 69-feature XGBoost model's validation Macro-F1**, while substantially reducing the original feature representation.

---

## Research Question

Can IoT intrusion detection remain highly accurate and probabilistically reliable while using a substantially reduced network-flow feature representation?

---

## Dataset and Classes

The experiments use the **IoTID20 network intrusion dataset** in a five-class classification setting:

- DoS
- MITM ARP Spoofing
- Mirai
- Normal
- Scan

The original dataset contains **625,783 network-flow records**.

The raw dataset is intentionally excluded from this repository because of file size and redistribution considerations.

---

## Leakage-Aware Experimental Design

A central objective of this project was to prevent duplicated network-flow patterns from producing overly optimistic evaluation results.

The original dataset contained:

- **625,783 rows**
- **79 candidate model predictors**
- **261,419 unique feature vectors**
- **164,087 exact duplicate rows**

A feature-vector audit also identified:

- **114 feature-identical groups with conflicting attack-category labels**
- **2,805 rows** belonging to those ambiguous groups

These conflicting groups were excluded before the final partitions were constructed.

Identical feature vectors were then assigned entirely to a single partition so that the same feature representation could not appear across training, validation, and test sets.

### Frozen Partitions

| Partition | Rows | Unique Feature Groups |
|---|---:|---:|
| Train | 374,506 | 156,782 |
| Validation | 125,170 | 52,262 |
| Test | 123,302 | 52,261 |

Feature-group overlap between all three partitions was:

**0**

This group-aware strategy was used to reduce duplicate-driven information leakage.

---

## Feature Configuration

Direct identifiers and target columns were excluded from the model predictors.

The initial candidate feature space contained **79 features**.

Ten constant features detected using the training partition were removed, leaving:

**69 original modeling features**

`Protocol` was treated as a categorical feature and one-hot encoded, resulting in **71 transformed XGBoost inputs** for the full model.

Preprocessing parameters were fitted using the training partition only.

---

## Duplicate- and Class-Aware Training

The training strategy combines two weighting mechanisms.

### 1. Feature-Group Frequency Weighting

Rows belonging to frequently repeated feature vectors receive lower individual weight.

This prevents highly duplicated traffic patterns from dominating optimization.

### 2. Class-Aware Weighting

Class weights were calculated from **unique training feature groups**, rather than raw row frequencies.

The combined weighting procedure produced approximately equal effective training contribution across the five classes:

| Class | Effective Contribution |
|---|---:|
| DoS | 20% |
| MITM ARP Spoofing | 20% |
| Mirai | 20% |
| Normal | 20% |
| Scan | 20% |

This design addresses both class imbalance and duplicated-pattern dominance.

---

## Baseline Model Comparison

Three model families were evaluated using the same leakage-safe validation partitions:

- Logistic Regression
- Random Forest
- XGBoost

Primary model comparison was performed using **validation unique feature groups**.

| Model | Accuracy | Balanced Accuracy | Macro F1 | Log Loss |
|---|---:|---:|---:|---:|
| Logistic Regression | 0.7452 | 0.7576 | 0.6647 | 0.5697 |
| Random Forest | 0.9553 | 0.9320 | 0.9153 | 0.1307 |
| XGBoost | **0.9774** | **0.9732** | **0.9551** | **0.0572** |

XGBoost achieved the strongest validation performance and was therefore used for the subsequent feature-reduction experiments.

---

## Feature Importance

Feature importance was derived from **XGBoost total gain** using the full 69-feature model fitted on the training partition.

The highest-ranked original features were:

| Rank | Feature | Total-Gain Importance |
|---:|---|---:|
| 1 | `Flow_Duration` | 21.51% |
| 2 | `Dst_Port` | 20.96% |
| 3 | `Src_Port` | 20.88% |
| 4 | `Protocol` | 7.76% |
| 5 | `Flow_Pkts/s` | 6.65% |
| 6 | `Init_Bwd_Win_Byts` | 5.77% |

Together, the six highest-ranked features accounted for approximately **83.52% of the accumulated total gain**.

---

## Feature Reduction

Candidate feature subsets were constructed from the XGBoost total-gain ranking.

The evaluated subset sizes were:

- Top-6
- Top-9
- Top-16
- Top-25
- Top-35
- Full-69

A selection rule was defined **before the locked test set was opened**:

> Select the smallest feature subset retaining at least 99% of the full-model validation unique-group Macro-F1.

### Feature-Subset Results

| Feature Set | Original Features | Validation Macro F1 | Full-Model Retention |
|---|---:|---:|---:|
| Top-6 | 6 | 0.9485 | 99.30% |
| Top-9 | 9 | 0.9496 | 99.42% |
| Top-16 | 16 | 0.9547 | 99.95% |
| Top-25 | 25 | 0.9553 | 100.02% |
| Top-35 | 35 | 0.9555 | 100.04% |
| Full-69 | 69 | 0.9551 | 100.00% |

According to the predefined selection criterion, the **Top-6 feature model** was selected.

The feature representation was therefore reduced from **69 original modeling features to only 6**.

![Feature reduction](results/figures/feature_subset_macro_f1.png)

---

## Probability Calibration

High classification accuracy does not automatically imply reliable probability estimates.

The selected Top-6 XGBoost model was therefore evaluated using **temperature scaling**.

Calibration development was performed using validation unique groups with a five-fold out-of-fold calibration audit.

The fitted fold temperatures were highly consistent, and the final temperature was:

`T = 1.099329`

Temperature scaling does not change the predicted class because applying a positive scalar temperature preserves the probability ordering. Instead, it adjusts prediction confidence.

### Locked-Test Unique-Group Calibration

| Metric | Before | After |
|---|---:|---:|
| Log Loss | 0.066754 | **0.066308** |
| Multiclass Brier Score | 0.037325 | **0.036973** |
| Equal-width ECE | 0.005717 | **0.005011** |
| Adaptive ECE | 0.005777 | **0.004458** |
| Confidence-Accuracy Gap | 0.005298 | **0.002718** |

The calibrated model preserved classification accuracy while improving the main probability-reliability measures on the primary unique-group test evaluation.

![Reliability diagram](results/figures/final_test_reliability_diagram.png)

---

## Final Locked-Test Performance

The final test set was opened only after:

- model family selection;
- feature ranking;
- feature-subset selection;
- sample-weight configuration; and
- calibration temperature

had been frozen.

### Unique-Group Evaluation

The unique-group evaluation is treated as the **primary final evaluation**, because each unique feature vector contributes once regardless of its frequency in the raw dataset.

| Metric | Result |
|---|---:|
| Accuracy | **0.9733** |
| Balanced Accuracy | **0.9716** |
| Macro Precision | 0.9310 |
| Macro Recall | 0.9716 |
| Macro F1 | **0.9488** |
| Weighted F1 | 0.9744 |

### Per-Class Performance

| Class | Precision | Recall | F1 |
|---|---:|---:|---:|
| DoS | 0.9997 | 0.9987 | 0.9992 |
| MITM ARP Spoofing | 0.7494 | 0.9488 | 0.8374 |
| Mirai | 0.9952 | 0.9638 | 0.9792 |
| Normal | 0.9918 | 0.9900 | 0.9909 |
| Scan | 0.9191 | 0.9565 | 0.9375 |

MITM ARP Spoofing remains the most challenging category, primarily because of lower precision despite high recall.

![Per-class F1](results/figures/final_test_class_f1.png)

### Confusion Matrix

![Locked-test confusion matrix](results/figures/final_test_confusion_matrix_normalized.png)

---

## Row-Level Test Evaluation

A secondary row-level evaluation was also retained to show model behavior under the original repetition frequencies.

| Metric | Result |
|---|---:|
| Accuracy | 0.9827 |
| Balanced Accuracy | 0.9865 |
| Macro Precision | 0.9530 |
| Macro Recall | 0.9865 |
| Macro F1 | 0.9681 |
| Weighted F1 | 0.9833 |

The unique-group evaluation remains the primary reported result because it reduces the influence of repeated feature patterns.

---

## Experimental Integrity

The locked test set was **not used** for:

- model selection;
- feature ranking;
- feature-subset selection;
- weighting design;
- probability-calibration fitting; or
- hyperparameter selection.

The final test evaluation was performed only after the complete modeling pipeline had been frozen.

No model configuration was modified in response to locked-test performance.

---

## Repository Structure

```text
reliable-iot-intrusion-detection/
├── README.md
├── LICENSE
├── requirements.txt
├── notebooks/
│   └── 01_reliable_iot_intrusion_detection.ipynb
├── data/
│   ├── README.md
│   └── processed/
├── models/
│   ├── README.md
│   └── feature_selection/
├── results/
│   ├── figures/
│   └── tables/
└── .gitignore
```

Large raw datasets, prediction-level dumps, and large trained-model artifacts are intentionally excluded from version control.

---

## How to Reproduce

### 1. Clone the Repository

```bash
git clone https://github.com/maram-almomani/reliable-iot-intrusion-detection.git
cd reliable-iot-intrusion-detection
```

### 2. Install Dependencies

```bash
pip install -r requirements.txt
```

### 3. Prepare the Dataset

Place the IoTID20 dataset at:

```text
data/raw/IoT Network Intrusion Dataset.csv
```

The exact completed experiment also relies on frozen leakage-control artifacts generated during preprocessing, including:

```text
data/processed/split_manifest.csv
data/processed/group_split_map.csv
```

These large artifacts are intentionally excluded from the public repository.

Therefore, reproducing the **exact original partition assignment** requires access to the corresponding frozen split artifacts.

### 4. Run the Experiment Notebook

Open:

```text
notebooks/01_reliable_iot_intrusion_detection.ipynb
```

Run the notebook sequentially from top to bottom.

The research workflow covers:

1. project initialization and dataset validation;
2. restoration of frozen leakage-safe partitions;
3. feature configuration and preprocessing;
4. group-aware and class-aware weighting;
5. baseline model comparison;
6. full 69-feature XGBoost training;
7. total-gain feature-importance analysis;
8. reduced feature-subset evaluation;
9. Top-6 model selection;
10. probability-calibration analysis;
11. final locked-test evaluation; and
12. generation of final performance and reliability figures.

> **Important:** The locked test set should only be evaluated after model selection, feature selection, weighting, and calibration parameters have been frozen.

---

## Requirements

The project was developed using Python and the following primary libraries:

- NumPy
- pandas
- SciPy
- scikit-learn
- XGBoost
- Matplotlib
- joblib

Exact package versions used in the experimental environment are listed in:

```text
requirements.txt
```

---

## Limitations

This study was evaluated on a **single benchmark dataset, IoTID20**. Although the feature-group-aware design reduces duplicate-driven leakage, performance on other IoT environments, network distributions, and previously unseen attack families has not yet been externally validated.

The evaluation also represents a fixed network-traffic dataset rather than a longitudinal deployment. Consequently, **temporal concept drift, evolving attack behavior, and changes in network topology or device populations** remain outside the scope of the current experiment.

Finally, the study evaluates computational efficiency within the experimental environment rather than directly benchmarking the final model on a resource-constrained IoT or edge device.

Future work should therefore investigate **cross-dataset validation, temporal evaluation, robustness under distribution shift, and deployment on resource-constrained IoT or edge-security platforms**.

---

## Research Significance

The project demonstrates that strong multiclass IoT intrusion-detection performance can be maintained with a substantially smaller flow-feature representation.

The main methodological contributions are:

- duplicate-aware leakage control;
- feature-group-based partitioning;
- combined duplicate-frequency and class-aware sample weighting;
- explicit validation-based feature reduction;
- locked-test evaluation; and
- post-hoc probability calibration.

Together, these components emphasize not only predictive performance, but also **evaluation reliability, computational efficiency, and confidence quality**.

---

<h2 dir="ltr" align="left">Research Areas</h2>

<ul dir="ltr">
  <li>IoT Security</li>
  <li>Network Intrusion Detection</li>
  <li>Machine Learning for Cybersecurity</li>
  <li>Leakage-Aware Evaluation</li>
  <li>Feature-Efficient Detection</li>
  <li>Probability Calibration</li>
  <li>Reliable AI for Cybersecurity</li>
</ul>

---

<h2 dir="ltr" align="left">Author</h2>

<p dir="ltr" align="left">
<strong>Maram M. Momani</strong><br>
Cybersecurity Researcher<br>
Machine Learning for Cybersecurity | Network & IoT Security | Cyber-Physical Systems Security
</p>

---

## License

This project is released under the license provided in the repository's `LICENSE` file.
