# RESULTS - Explainable Intrusion Detection (EXID)

## 1) Project Overview (TL;DR)

The EXID pipeline uses LightGBM on the NSL-KDD dataset in a two-stage strategy: a high-recall binary filter (`Normal` vs. `Anomaly`) followed by multiclass attack categorization. Beyond aggregate accuracy, the project prioritizes detection of rare and stealthy attacks (`R2L`, `U2R`) and explains model behavior with SHAP to make decisions interpretable for security analysts.

> **Key takeaway:** EXID is optimized not only to classify common attacks well, but also to reduce blind spots for low-frequency, high-impact intrusions.

## 2) Stage 1: The Binary Filter (Normal vs. Anomaly)

Model 0 acts as the perimeter filter before deeper categorization.

| Class | Precision | Recall | F1-score | Support |
|---|---:|---:|---:|---:|
| **Normal** | 0.82 | 0.97 | 0.89 | 9,711 |
| **Anomaly** | 0.96 | 0.76 | 0.85 | 9,083 |

Additional interpretation from the notebook output:
- **True Positives (caught anomalies):** 6,939  
- **False Positives (false alarms):** 264

![Binary Confusion Matrix](path/to/image.png)

> **Key takeaway:** The binary stage is conservative against missed benign traffic (high `Normal` recall) while still capturing most malicious activity, creating a reliable first-pass IDS screen.

## 3) Stage 2: The Multiclass Challenge & The "Maximization Protocol"

### The "Normal Siphon" Effect in the Baseline 5-Category Model

In the baseline 5-category setup (Model 2), the classifier performed strongly on dominant categories (`DoS`, `Probe`) but under-detected stealth attacks. This created a **Normal Siphon** behavior: subtle `R2L`/`U2R` patterns were frequently absorbed into decision regions associated with non-rare behavior, yielding poor recall where detection matters most.

### Model 2.1 Maximization Protocol (Rare-Attack Rescue)

To counter this, Model 2.1 introduced a targeted intervention stack:

1. **SMOTE-NC balancing** across all five categories to address data sparsity in rare classes.
2. **Custom class weighting** during LightGBM training:  
   `{Normal: 1, DoS: 1, Probe: 2, R2L: 10, U2R: 20}`
3. **Probability-threshold overrides** at inference:  
   assign `R2L` when `P(R2L) > 0.15`, assign `U2R` when `P(U2R) > 0.15` (with deterministic precedence based on higher probability if both fire).

### Baseline vs. Maximized Performance (Model 2 vs. Model 2.1)

| Class | Baseline Recall | Maximized Recall | Delta Recall | Baseline F1 | Maximized F1 | Delta F1 |
|---|---:|---:|---:|---:|---:|---:|
| **R2L** | 0.13 | 0.40 | **+0.27** | 0.23 | 0.57 | **+0.34** |
| **U2R** | 0.22 | 0.43 | **+0.21** | 0.32 | 0.09 | **-0.23** |
| **Probe** | 1.00 | 1.00 | 0.00 | 0.90 | 0.90 | 0.00 |

![Model 2.1 Confusion Matrix](path/to/image.png)

> **Key takeaway:** The maximization protocol substantially increases **rare-attack recall**, especially for `R2L` and `U2R`, but introduces a precision tradeoff for extremely scarce `U2R` samples. This is an intentional security-oriented bias toward surfacing stealth threats.

## 4) Explainable AI (SHAP) Insights - The "Why"

![Multiclass SHAP Summary Plot](path/to/image.png)

SHAP analysis reveals a clear feature-separation pattern in how the model reasons about attack families:

- **Volumetric traffic signals** (notably `src_bytes` and `count`) strongly drive decisions for broad, high-volume behaviors such as `DoS` and `Probe`.
- For stealth-oriented categories, the model shifts toward **payload and access behavior indicators** (`dst_bytes`, `hot`, `num_failed_logins`) that better represent low-and-slow, privilege-oriented intrusion dynamics.
- This transition in feature attention aligns with domain intuition: noisy flood-like attacks are detected via traffic intensity, while subtle compromise attempts are detected via access-pattern anomalies.

> **Key takeaway:** EXID does not rely on a single global heuristic; it adapts its decision logic by attack family, which improves trustworthiness and analyst interpretability in operational SOC settings.
