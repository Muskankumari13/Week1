# Capstone Report — Content Refresh Prioritization

- **Author:** Muskan Kumari
- **Lane:** Content Refresh Prioritization
- **Repo:** Muskankumari13/Week1
- **Date:** August 8, 2026

## 1. Problem framing

### Decision

This project supports the decision of **which webpages should be prioritized for content refresh**.

The unit of analysis is **one content page/webpage per row**.

The output is a **refresh-priority score/ranking** that orders pages from higher priority to lower priority.

A human SEO manager or content team can use this ranking to decide which pages should be reviewed and refreshed first.

The cost of a wrong call is mainly inefficient use of editorial resources. If a low-priority page is refreshed first, time and effort may be spent while a more important page remains unaddressed. The opposite error can also occur if a page is ranked highly but does not actually benefit from a refresh.

Data and ML are useful because refresh priority depends on multiple signals at the same time, including content age, time since the last update, search demand, clicks, impressions, CTR, position, engagement, and recent traffic trends. A simple fixed rule may not combine these signals as effectively as a learned model.

This project is therefore designed as **decision support**, not as an automated replacement for editorial judgment.

---

## 2. Data safety

The project uses the provided **anonymized content-refresh dataset**.

The dataset contains **30,000 rows and 44 columns**.

Each row represents one anonymized content page.

The dataset includes signals related to:

- Search demand
- Competition
- Content characteristics
- Search impressions
- Clicks
- Page views and sessions
- Content age
- Days since last update
- CTR
- Average search position
- Engagement
- Scroll behavior
- Recent and previous-period performance
- Trend information

### Columns deliberately excluded

The following identifiers are not used as model features:

- `content_id`
- `client_id`

`content_id` is used only as an identifier for the content page and is not treated as a predictive feature.

`client_id` is pseudonymous and is not used as a feature because it could allow the model to memorize client-specific patterns rather than learn generalizable content-level signals. It may be used for grouping/validation where appropriate, but never as a predictive input.

### Leakage risks

Special attention was given to fields that may contain information derived from the outcome.

In particular:

- `trend_direction`
- `trend_pct`

are treated as **label-derived / outcome-related fields** and are excluded from the predictive feature set when defining the refresh-priority target.

This prevents the model from simply learning the same information used to construct the target.

Other fields that directly encode derived tiers or target information are also excluded when they would duplicate information from the target or create leakage.

The model should therefore use information that would reasonably be available when making a refresh-priority decision.

### Public safety

The dataset is anonymized.

No client names, private URLs, private search queries, or other client-identifying information are included in the report.

Only anonymized content-level information and aggregate/measured results are discussed.

---

## 3. Baseline

The first approach is a **transparent rule/score-based baseline**.

The purpose of the baseline is to provide a simple comparison before using a more complex machine-learning model.

The baseline combines observable refresh-risk signals such as:

- Content age
- Days since last update
- Recent performance
- Search demand
- Engagement/performance signals

The baseline is intentionally simple and interpretable. It represents the type of prioritization that could be created using a fixed scoring rule instead of machine learning.

This is a fair comparison because the model and baseline are evaluated on the **same data and the same validation split**.

### Baseline result

The baseline metric should be copied from the final run of:

`work/notebooks/w04_baseline_score.ipynb`

**Baseline NDCG:** `[INSERT MEASURED BASELINE NDCG FROM W04]`

The baseline provides the reference point against which the ML model is evaluated.

A model should only be considered useful if it improves ranking quality over this simple baseline on the same validation data.

---

## 4. Model / analysis

### Method

The project uses a supervised machine-learning approach for **ranking/priority scoring**.

The goal is not to predict a search-engine ranking algorithm. The goal is to estimate which pages appear more appropriate for refresh prioritization based on the available content and performance signals.

The model was developed after establishing the research question, task framing, data contract, leakage checks, baseline, and signal audit.

### Unit of analysis

One row = one anonymized content page.

### Target / proxy definition

The target is a **refresh-priority proxy**, because the dataset does not contain a direct historical label saying whether a page should have been refreshed.

The proxy represents higher priority for pages showing a combination of signals associated with refresh need, such as older content, longer time since update, and weaker/recently declining performance.

### Feature list

The intended predictive features are content and performance signals available before the prioritization decision, including relevant variables such as:

- `search_volume`
- `competition`
- `cpc`
- `word_count`
- `char_count`
- `impressions_90d`
- `clicks_90d`
- `pageviews_90d`
- `sessions_90d`
- `users_90d`
- `engaged_sessions_90d`
- `ai_sessions_90d`
- `scroll_events_90d`
- `days_with_impressions`
- `days_with_sessions`
- `impressions_last_30d`
- `clicks_last_30d`
- `sessions_last_30d`
- `impressions_prev_30d`
- `clicks_prev_30d`
- `sessions_prev_30d`
- `content_age_days`
- `days_since_last_update`
- `ctr`
- `avg_position`
- `engagement_rate`
- `scroll_rate`
- `ai_traffic_pct`

Categorical content descriptors such as `content_type` and `main_intent` may also be included after appropriate encoding if they are part of the final model pipeline.

### Deliberately excluded

The following are not used as predictive features:

- `content_id`
- `client_id`
- `trend_direction`
- `trend_pct`

Derived tier fields are also excluded when they duplicate information already represented by the underlying numerical variables or create leakage.

### Why this method fits the lane

The task is a prioritization problem rather than a simple yes/no classification problem.

The final output needs to answer:

> "Which pages should the content team look at first?"

A ranking/score is therefore more useful operationally than a binary prediction.

---

## 5. Evaluation

### Validation design

The model and baseline are evaluated on the **same held-out validation split**.

The validation process is designed to reduce overly optimistic results caused by information sharing between related observations.

Where client grouping is used, pages belonging to the same pseudonymous client are kept within the appropriate split rather than being used as both training and validation examples.

The validation design is documented in:

`work/notebooks/w06_validation_audit.ipynb`

### Primary metric

The primary ranking metric is:

**NDCG — Normalized Discounted Cumulative Gain**

NDCG is appropriate because the practical goal is to place the most important pages near the top of the refresh queue.

A higher NDCG means the ranking places higher-priority pages closer to the top.

### Model vs baseline

Both approaches must be evaluated on the same validation split.

| Method | Validation NDCG |
|---|---:|
| Baseline | `[INSERT W04 VALUE]` |
| ML Model | `[INSERT W06 VALUE]` |

**NDCG improvement over baseline:** `[INSERT MEASURED DIFFERENCE]`

The final numbers should be taken directly from the latest successful notebook run.

### Base rate

Because this is a ranking task, the main evaluation metric is NDCG rather than majority-class accuracy.

The project should not describe a high ranking score as proof that the model is correct.

The ranking result is interpreted as **measured decision-support performance relative to the baseline**.

### Error analysis

The main errors occur when pages with similar content age, traffic, or engagement signals receive different priority levels from the underlying proxy.

Some pages can also appear highly important because they have strong search demand or impressions even when their recent performance does not indicate an urgent refresh need.

Conversely, pages with low traffic can receive lower priority even though they may have strategic value that is not represented in the available dataset.

These cases show why the ranking should be reviewed by an editor rather than used as an automatic decision.

---

## 6. Interpretation

The analysis indicates that refresh prioritization is influenced by a combination of **content freshness and performance signals**, rather than by a single variable.

Important signals include:

- How old the content is
- How long it has been since the last update
- Recent impressions and clicks
- Search demand
- CTR
- Average search position
- Engagement
- Recent performance relative to the previous period

### Plain-language interpretation

A page that is old and has not been updated recently may deserve more attention than a recently updated page.

A page with meaningful search demand and impressions may also be more valuable to review because changes to that page could affect a larger amount of existing search visibility.

Performance signals help distinguish between pages that simply look old and pages that also show signs of weaker performance.

### Surprises and negative results

The analysis does not establish that any single feature causes search performance to decline.

Some pages may have high search demand but weak engagement, while others may have lower demand but stronger engagement.

This means that no single signal should be treated as a complete refresh decision.

A useful negative result is that the model should not be interpreted as predicting Google's ranking algorithm. It only learns patterns present in this anonymized dataset and its defined proxy target.

---

## 7. Recommendation

The model output can be converted into a practical refresh queue.

### Ranked action playbook

#### Priority 1 — Review high-priority pages first

Start with pages that receive the highest refresh-priority scores.

Editors should check:

- Whether the information is outdated
- Whether the search intent has changed
- Whether important sections are missing
- Whether the page still satisfies the user's likely intent
- Whether the page has meaningful search demand

#### Priority 2 — Review pages with strong opportunity signals

Pages with meaningful impressions/search demand but weaker recent performance should receive editorial review.

These pages may have more potential value than pages with little measurable search visibility.

#### Priority 3 — Review aging content

Pages with high content age or long periods since the last update should be included in the refresh backlog.

Age alone should not automatically trigger a refresh; it should be combined with performance and editorial relevance.

#### Priority 4 — Defer low-signal pages

Pages with low demand, low visibility, and no strong freshness or performance concern can be placed lower in the queue.

This allows limited editorial resources to focus on higher-priority pages first.

### How an editor could use this tomorrow

A FlyRank editor could:

1. Generate the ranked refresh queue.
2. Review the highest-ranked pages first.
3. Check the page manually for outdated or missing information.
4. Confirm that the page has meaningful search or business value.
5. Refresh the page when the editorial review supports the recommendation.
6. Track the page after the refresh.
7. Compare future performance against the pre-refresh period.

The model is therefore a **prioritization aid**, not an automatic publishing or SEO decision system.

### Confidence and limits

Confidence is **directional rather than causal**.

The model can identify patterns associated with the defined refresh-priority proxy, but it cannot prove that refreshing a page will improve search performance.

The recommendations are limited by:

- The quality of the available dataset
- The definition of the proxy target
- The available time windows
- Possible missing variables
- The validation design
- The fact that historical association does not establish causation

A human editor should make the final decision.

---

## 8. Reproducibility

The project is maintained in the `Week1` repository.

Relevant notebooks are stored under:

`work/notebooks/`

The workflow includes:

1. Research question and lane definition
2. ML task framing
3. Data contract
4. Leakage checks
5. Baseline score
6. Signal audit
7. Model development
8. Validation audit
9. Action playbook
10. Capstone report

### Main notebooks

- `work/notebooks/w01_research_question.ipynb`
- `work/notebooks/w02_ml_task_framing.ipynb`
- `work/notebooks/w03_data_contract.ipynb`
- `work/notebooks/w03_feature_leakage_check.ipynb`
- `work/notebooks/w04_baseline_score.ipynb`
- `work/notebooks/w04_signal_audit.ipynb`
- `work/notebooks/w05_model.ipynb`
- `work/notebooks/w06_validation_audit.ipynb`
- `work/notebooks/w07_action_playbook.ipynb`
- `work/notebooks/capstone.ipynb`

### Data

The raw dataset is:

`data/raw/content_refresh_anonymized.csv`

The dataset contains:

- **30,000 rows**
- **44 columns**

### Re-run

From a fresh clone:

```bash
git clone https://github.com/Muskankumari13/Week1.git
cd Week1
pip install -r requirements.txt
