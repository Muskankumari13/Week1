
````markdown
# Content Refresh Prioritization

A machine-learning decision-support project that helps content teams prioritize which webpages should be reviewed and refreshed first.

## Overview

This capstone investigates **content refresh prioritization** using FlyRank's anonymized content-refresh dataset.

The project combines content freshness, search demand, traffic, engagement, and recent performance signals to identify pages that may deserve editorial attention.

The final system is designed as a **decision-support tool**, not as an automated replacement for human editorial judgment.

The practical question is:

> **Which webpages should the content team look at first?**

---

## Problem

Content teams often have a large number of webpages but limited time and editorial resources.

A manual process may make it difficult to determine which pages should be reviewed first. This project uses multiple content and performance signals to create a prioritization approach that can help organize an editorial refresh queue.

The main signals include:

- Content age
- Days since last update
- Search volume
- Impressions
- Clicks
- CTR
- Sessions
- Engagement
- Recent performance

The output is converted into actionable priority categories:

- Refresh immediately
- Review soon
- Monitor
- No action

---

## Intended User

The intended user is an:

- SEO manager
- Content strategist
- Editorial team
- Content operations team

The model provides a ranked or categorized queue that a human can review before making a final refresh decision.

---

## Dataset

The project uses FlyRank's anonymized content-refresh dataset.

Dataset characteristics:

- **30,000 rows**
- **44 columns**
- One row represents one anonymized content page

The dataset contains signals related to:

- Search demand
- Competition
- Content characteristics
- Search impressions
- Clicks
- Page views
- Sessions
- Content age
- Days since last update
- CTR
- Average search position
- Engagement
- Recent and previous-period performance

No client-identifying information is used in this project.

---

## Data Safety and Leakage Prevention

The following identifiers were excluded from predictive features:

- `content_id`
- `client_id`

Outcome-related fields such as:

- `trend_direction`
- `trend_pct`

were also excluded from the predictive feature set where they could duplicate target information or introduce leakage.

The validation audit checked that:

- The target column was not used as a feature.
- Missing values were checked.
- Duplicate rows were checked.
- Future information was considered during validation.

Final validation checks:

| Check | Result |
|---|---|
| Target column used as feature | False |
| Missing values | 0 |
| Duplicate rows | 0 |

---

## Methodology

The project follows this workflow:

```text
Problem Framing
      ↓
Data & Leakage Checks
      ↓
Rule-Based Baseline
      ↓
Random Forest Model
      ↓
Time-Aware Validation
      ↓
Action Prioritization
      ↓
Editorial Decision Support
````

The workflow was developed across the Week 1–7 notebooks.

---

## W04 — Rule-Based Baseline

A transparent rule-based baseline was created before the machine-learning model.

The baseline combines observable signals such as:

* Content age
* Days since last update
* Recent performance
* Search demand
* Engagement/performance signals

The purpose of the baseline was to provide a simple and interpretable reference approach.

The baseline produces a prioritization/action score and ranked queue.

A separate numerical baseline NDCG value was not recorded in the W04 notebook, so no baseline NDCG improvement is claimed.

---

## W05 — Random Forest Model

The Week 5 model uses a **Random Forest classifier**.

### Model result

**Random Forest Accuracy: 0.668**

This corresponds to an observed accuracy of approximately **66.8%** on the random train-test split.

### Feature importance

The strongest model features were:

| Feature                  | Importance |
| ------------------------ | ---------: |
| `impressions_90d`        |   0.315003 |
| `content_age_days`       |   0.210479 |
| `sessions_90d`           |   0.154071 |
| `ctr`                    |   0.104537 |
| `search_volume`          |   0.085307 |
| `clicks_90d`             |   0.075994 |
| `days_since_last_update` |   0.054609 |

`impressions_90d` was the most important feature in the trained Random Forest, followed by `content_age_days` and `sessions_90d`.

Feature importance indicates model reliance within this trained model; it does not establish that a feature causes content performance changes.

---

## W06 — Validation

The Week 6 validation audit tested the model using **TimeSeriesSplit**.

This provides a more conservative and time-aware evaluation than relying only on a random train-test split.

### Validation results

| Evaluation                  |    Accuracy |
| --------------------------- | ----------: |
| W05 Random Train-Test Split |       0.668 |
| W06 TimeSeriesSplit         | **0.66064** |

### Fold scores

| Fold | Accuracy |
| ---- | -------: |
| 1    |   0.6524 |
| 2    |   0.6558 |
| 3    |   0.6664 |
| 4    |   0.6666 |
| 5    |   0.6620 |

The average TimeSeriesSplit accuracy was **0.66064**.

The lower time-aware result suggests that the random split may have provided a slightly optimistic estimate.

Therefore, the TimeSeriesSplit result is treated as the more conservative validation result.

### Validation integrity

The audit found:

* Target column used as feature: **False**
* Missing values: **0**
* Duplicate rows: **0**

No obvious target leakage was identified in the final feature set during the validation audit.

---

## W07 — Action Playbook

The model output was converted into practical editorial action categories.

### Final action distribution

| Action              |      Pages | Percentage |
| ------------------- | ---------: | ---------: |
| Refresh immediately |      5,956 |     19.85% |
| Review soon         |      6,018 |     20.06% |
| Monitor             |      6,052 |     20.17% |
| No action           |     11,974 |     39.91% |
| **Total**           | **30,000** |   **100%** |

Overall:

* **5,956 pages** should be considered for immediate refresh.
* **6,018 pages** should be reviewed soon.
* **6,052 pages** should be monitored.
* **11,974 pages** were assigned no action.

In total, **18,026 pages (60.09%)** received some level of review or monitoring recommendation.

The action categories are intended to organize editorial work rather than automatically trigger publishing or content changes.

---

## How an Editor Could Use the Output

A content team could use the output as follows:

1. Generate the refresh-priority queue.
2. Start with pages marked **Refresh immediately**.
3. Manually review the page for outdated or missing information.
4. Check whether search intent has changed.
5. Confirm that the page has meaningful search or business value.
6. Refresh the page when editorial review supports the recommendation.
7. Track performance after the refresh.
8. Compare future performance with the pre-refresh period.

The human editor remains responsible for the final decision.

---

## Evaluation Interpretation

The model achieved:

* **0.668 accuracy** on the random train-test split.
* **0.66064 average accuracy** using TimeSeriesSplit.

These are observed measurements on this dataset and validation setup.

They should not be interpreted as proof that the model will perform identically on future datasets.

The project also does not establish that refreshing a high-priority page will cause its future search performance to improve.

The target is a **refresh-priority proxy**, not a confirmed historical editorial outcome.

Therefore, the model provides directional decision support rather than causal predictions.

---

## Limitations

Several limitations should be considered.

### 1. Proxy target

The dataset does not contain a direct historical label stating whether a webpage actually needed a refresh.

The target is therefore a proxy based on available signals.

### 2. No causal evidence

The model identifies patterns associated with the defined prioritization target.

It does not prove that refreshing a page will improve traffic, rankings, clicks, or conversions.

### 3. Dataset limitations

The model learns from the available anonymized dataset and may not generalize to every website, industry, content type, or future dataset.

### 4. Missing business context

Some editorial decisions depend on information not represented in the dataset, such as:

* Strategic importance
* Brand priorities
* Business goals
* Legal considerations
* Editorial expertise
* Major product or market changes

### 5. Human review is required

A high model priority should be treated as a recommendation for review, not an automatic instruction to modify a page.

---

## Reproducibility

The project is maintained in the `Week1` repository.

Relevant notebooks are located under:

```text
work/notebooks/
```

Main notebooks:

```text
w01_research_question.ipynb
w02_ml_task_framing.ipynb
w03_data_contract.ipynb
w03_feature_leakage_check.ipynb
w04_signal_audit.ipynb
w04_baseline_score.ipynb
w05_model.ipynb
w06_validation_audit.ipynb
w07_action_playbook.ipynb
capstone.ipynb
```

### Repository structure

```text
Week1/
├── data/
│   └── raw/
├── work/
│   ├── notebooks/
│   ├── figures/
│   └── capstone_report.md
├── scripts/
├── docs/
├── outputs/
├── requirements.txt
└── README.md
```

The raw dataset should not be committed as a new dataset file or copied into the repository outside the provided project structure.

---

## Setup

Clone the repository:

```bash
git clone https://github.com/Muskankumari13/Week1.git
cd Week1
```

Install the required Python packages:

```bash
pip install -r requirements.txt
```

The notebooks can also be run using Google Colab.

For the capstone workflow, open the relevant notebooks in:

```text
work/notebooks/
```

and run the workflow in sequence.

---

## Usage

The general workflow is:

```text
Load anonymized dataset
        ↓
Check data quality
        ↓
Define prioritization target
        ↓
Train Random Forest
        ↓
Validate using TimeSeriesSplit
        ↓
Generate priority/action categories
        ↓
Review recommendations manually
```

The resulting action categories can be used to create an editorial review queue.

---

## Project Files

| File                         | Purpose                                 |
| ---------------------------- | --------------------------------------- |
| `w04_baseline_score.ipynb`   | Rule-based baseline                     |
| `w05_model.ipynb`            | Random Forest model                     |
| `w06_validation_audit.ipynb` | Time-aware validation and leakage audit |
| `w07_action_playbook.ipynb`  | Action categories and recommendations   |
| `capstone.ipynb`             | Final capstone analysis                 |
| `capstone_report.md`         | Written capstone report                 |

---

## AI Transparency

AI tools were used as a thinking and development assistant during parts of the project, including helping with code explanations, debugging, structuring documentation, and interpreting workflow requirements.

The model results, validation checks, and reported metrics were reviewed against the executed notebooks and outputs.

The final decisions about the methodology, claims, limitations, and interpretation remain the author's responsibility.

---

## Conclusion

This project demonstrates how content freshness, search demand, traffic, engagement, and recent performance signals can be combined to support content refresh prioritization.

The Random Forest model achieved an observed accuracy of **0.668** on the random split and **0.66064** under TimeSeriesSplit validation.

The final action playbook classified 30,000 pages into practical editorial categories, with **5,956 pages marked for immediate refresh, 6,018 for review soon, 6,052 for monitoring, and 11,974 requiring no action**.

The main conclusion is not that the model can automatically decide which pages should be refreshed. Instead, it provides a structured and measurable way to help editorial teams decide **where to look first**.

Human editorial judgment remains essential.

---

## Author

**Muskan Kumari**

FlyRank ML Internship — Content Refresh Prioritization

August 2026

```

