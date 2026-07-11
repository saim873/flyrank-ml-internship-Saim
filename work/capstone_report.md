

# Capstone Report — Refresh / Content Opportunity Scoring

- **Author:** Muhammad Saim Zafar
- **Lane:** Refresh / Content Opportunity Scoring
- **Repo:** https://github.com/saim873/flyrank-ml-internship-Saim
- **Date:** July 2026
  ## Abstract

This study investigates how historical search-performance signals can identify content that may benefit from editorial review. A Random Forest classifier was trained using anonymized FlyRank internship data and evaluated using grouped client-level validation to reduce client leakage. On the same grouped split, the model achieved 0.796 accuracy and 0.840 F1, outperforming the transparent Week-4 rule baseline. The output is a ranked recommendation queue designed to support human editorial decisions rather than automate publishing actions. The findings are observational, public-safe, and do not claim causation or knowledge of Google’s ranking algorithm.

## 1. Problem framing

This project supports the decision of **which content items should be reviewed first for a possible refresh**.

The unit of analysis is a **single pseudonymized content item**. The output is a **ranked content opportunity score** with supporting reason codes and recommended actions.

A content reviewer or editor can use this ranked queue to decide whether a page should be refreshed, monitored, or left unchanged.

The cost of a false positive is that editorial time may be spent reviewing content that does not actually require updates. The cost of a false negative is that genuinely declining content may remain unnoticed, resulting in missed optimization opportunities.

Machine learning helps because content performance depends on many interacting search signals such as impressions, click-through rate (CTR), average position, content age, and engagement. A Random Forest model can capture complex relationships between these signals better than a simple rule-based approach while still supporting human decision-making.



## 2. Data safety

The analysis uses the anonymized FlyRank ML Internship dataset (`content_refresh_anonymized.csv`). All records are pseudonymized and no client names, domains, URLs, search queries, or credentials are included in this project.

The following columns were deliberately excluded from model training:

- `content_id` – identifier only, not a predictive feature.
- `client_id` – used only for grouped validation, never as a model feature.
- `trend_direction` – used to create the target label and then removed to prevent leakage.
- `trend_pct` – excluded because it directly represents the target outcome and would introduce label leakage.

The remaining features include search visibility, impressions, CTR, ranking position, content characteristics, engagement signals, and content age.

A grouped client-level validation strategy was used so that content from the same client could not appear in both the training and testing sets. This reduces information leakage and provides a more realistic estimate of model performance.

No client-identifying information appears anywhere in the notebooks, outputs, or this report.



## 3. Baseline

A transparent rule-based baseline was developed before training the machine learning model. The baseline assigns a simple score using observable search-performance signals.

The scoring rule included:

- +2 points if the content had not been updated for at least 90 days.
- +2 points if historical impressions (`impressions_90d`) were high.
- +1 point if the click-through rate (CTR) was below 1%.
- +1 point if the average search position was greater than 10.

Content with a baseline score of **4 or higher** was classified as likely to require review.

The baseline was evaluated using the **same grouped client-level train/test split** as the Random Forest model to ensure a fair comparison.

### Baseline Performance

| Method | Accuracy | Precision | Recall | F1 |
|---------|---------:|----------:|--------:|---:|
| Base Rate | 0.617 | 0.617 | 1.000 | 0.763 |
| Week-4 Rule Baseline | 0.393 | 0.516 | 0.268 | 0.353 |

The baseline provides a simple and interpretable reference point but does not capture complex interactions between multiple search-performance signals.



## 4. Model / analysis

A **Random Forest Classifier** was selected because it can model non-linear relationships, handle mixed numerical and categorical features after preprocessing, and provide feature importance scores for interpretation.

### Feature Set

The model was trained using search-performance and content-related features, including:

- Search volume
- Competition and CPC
- Content type and search intent
- Word count and character count
- Days since last update
- Impressions (last 30, previous 30, and 90 days)
- Sessions and engagement metrics
- CTR
- Average search position
- Content age
- AI traffic percentage
- Other engineered categorical tiers

### Excluded Features

The following variables were intentionally excluded:

- `content_id`
- `client_id`
- `trend_direction`
- `trend_pct`

These columns either identify records or directly represent the target outcome and would introduce data leakage.

### Target Definition

The prediction target was a binary label called **is_declining_label**.

Content whose `trend_direction` was **down** was labeled as **1 (declining)**, while all other valid observations were labeled as **0 (not declining)**.

### Model Configuration

The Random Forest model was trained with:

- 200 decision trees
- Balanced class weights
- Grouped client-level validation
- Median imputation for numeric variables
- Most-frequent imputation and one-hot encoding for categorical variables

This configuration provides a strong balance between predictive performance and interpretability while reducing the risk of overfitting.


## 5. Evaluation

A **grouped client-level validation split** was used throughout the project. This ensures that content from the same client does not appear in both the training and testing sets, reducing information leakage and providing a more realistic estimate of model performance.

### Dataset Split

- Training rows: **21,514**
- Test rows: **5,098**
- Training clients: **24**
- Test clients: **7**
- Test base rate: **61.7%**

### Model Comparison

| Method | Accuracy | Precision | Recall | F1 Score |
|---------|---------:|----------:|--------:|---------:|
| Base Rate | 0.617 | 0.617 | 1.000 | 0.763 |
| Week-4 Rule Baseline | 0.393 | 0.516 | 0.268 | 0.353 |
| Random Forest | **0.796** | **0.812** | **0.871** | **0.840** |

The Random Forest model outperformed the transparent baseline on every evaluation metric while using the same grouped validation strategy.

### Error Analysis

The model produced **1,041 incorrect predictions** on the grouped test set.

Inspection of these cases suggests that some errors are associated with:

- Missing feature values.
- Low-volume content with limited historical evidence.
- Seasonal variation not represented in the available dataset.
- Complex interactions between ranking signals that are difficult to capture using the available features.

The evaluation results should be interpreted as **observed model performance on the available dataset** and are intended for **decision-support rather than causal conclusions**.



## 6. Interpretation

The Random Forest model identified several search-performance signals that consistently contributed to the prediction of declining content.

### Most Important Features

The feature importance analysis showed that the following variables contributed the most:

1. Impressions during the last 30 days
2. Impressions from the previous 30 days
3. Impressions over the last 90 days
4. Average search position
5. Content age
6. Sessions during the last 30 days
7. Days with impressions
8. Character count
9. Word count
10. Click-through rate (CTR)

These findings suggest that recent search visibility, historical visibility, ranking position, and content freshness are the strongest observed indicators for prioritizing content review.

### Interpretation

The model appears to rely primarily on measurable search-performance signals rather than any single feature.

The results indicate that content with declining visibility, lower rankings, older update history, and reduced engagement is more likely to require editorial review.

### Limitations

Feature importance should not be interpreted as evidence of causation. The analysis only describes patterns observed in the available dataset.

The recommendations generated by the model are intended to support human decision-making and should not automatically trigger content updates or publishing actions.



## 7. Recommendation

The ranked action queue generated by this project is intended to help content editors prioritize which content items should be reviewed first.

### Recommended Actions

- Review content with the highest action scores first.
- Prioritize content showing declining impressions and weaker ranking positions.
- Review older content that has not been updated recently.
- Investigate pages with low CTR despite maintaining visibility.
- Monitor lower-priority content before deciding on a refresh.

### Human Review

Every recommendation should be reviewed by a human editor before taking action.

The reviewer should confirm:

- Content accuracy
- Search intent
- Technical SEO issues
- Competitor changes
- Business relevance

### Confidence

The recommendations are based on observed historical search-performance patterns.

They should be interpreted as **directional decision-support** rather than guaranteed outcomes.

The model does not prove that refreshing content will improve rankings or traffic.



## 8. Reproducibility

This project can be reproduced using the notebooks stored in the repository under `work/notebooks/`.

### Notebook Order

1. `w04_baseline_score.ipynb` — Transparent baseline scoring.
2. `w05_model.ipynb` — Random Forest model development and comparison.
3. `w06_validation_audit.ipynb` — Grouped validation, leakage audit, and claim review.
4. `w07_action_playbook.ipynb` — Ranked content recommendations and exported action queue.

### Environment

- Python 3.x
- pandas
- NumPy
- scikit-learn
- matplotlib

### Random Seed

All experiments used:

```python
random_state = 42
```

to improve reproducibility.

### Repository Outputs

The repository contains:

- Executed notebooks
- Exported content action queue (`work/outputs/content_action_playbook.csv`)
- Capstone report
- Public research paper

All reported metrics were generated from a fresh notebook execution using the grouped client-level validation strategy.
## 9. Acknowledgements & Data Credit

Built on the FlyRank Machine Learning Internship dataset.

Data source credit: https://flyrank.ai

This project was completed as part of the FlyRank Machine Learning Internship. All findings are based on anonymized, public-safe data and are intended for educational and research purposes.
