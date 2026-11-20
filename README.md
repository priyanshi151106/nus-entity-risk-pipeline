# Entity-Risk-Score Fraud Model (Pipeline 2)

**Author:** Priyanshi Suneja

I built this pipeline as my part of a team capstone for **SP2718F Data Science in Action: Financial Transactions & Payments** at the NUS Global Science Summer Programme 2026 (Track D: Network & Entity Linkage Fraud).

## Problem

Does knowing a card's, address's or email domain's fraud history improve card-not-present fraud detection beyond a standard transaction-level model?

## Approach

- **Data:** IEEE-CIS Fraud Detection, 590,540 transactions with a 3.5% fraud rate. The split is time-based, with the earliest 80% for training and the latest 20% (118,108 rows) for testing. The data isn't included here; it's available on Kaggle.
- **Entity features:** `card1_fraud_rate`, `card1_txn_count`, `addr1_fraud_rate`, `email_domain_fraud_rate` and a combined `entity_risk_score`.
- **Leakage-safe recipe:** entity rates are calculated on training rows only, then mapped onto test rows. Unseen entities fall back to the global training fraud rate, never zero. The rates are never refit on test data.
- **Model:** LightGBM using the baseline transaction features plus the entity features, retrained as a single model.
- **Verification:** a manual spot-check recomputes 5 random test rows' entity rates from training data only. All 5 match.

## Results (vs the transaction-only baseline, same review threshold of 0.574)

| Metric | Baseline | This pipeline |
|---|---|---|
| Recall | 60.0% | **61.4%** |
| Precision | 23.5% | **29.3%** |
| False positives | 7,924 | **6,022 (−24%)** |
| Transactions sent to review | 10,363 | **8,516 (−17.8%)** |
| ROC-AUC | 0.878 | 0.860 |
| PR-AUC | 0.445 | 0.451 |

- Entity features account for 38% of the model's total feature importance. `card1_fraud_rate` is the single most important feature.
- Illustrative net saving: about $100.6K on the test set, assuming $100 per missed fraud and $50 per false flag.
- ROC-AUC fell slightly while PR-AUC and threshold-level performance improved. So the recommendation is to use this as a review-queue enhancement on top of the baseline, not as a replacement.

## Note

The notebook imports helper modules (`src/`) from the shared team codebase, which aren't included here. The notebook is shown with its saved outputs for reference.
