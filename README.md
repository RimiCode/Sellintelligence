# Sellintelligence — Digital Marketing Campaign Conversion Prediction

![Python](https://img.shields.io/badge/Python-3.13-blue) ![scikit--learn](https://img.shields.io/badge/scikit--learn-ML-orange) ![Power BI](https://img.shields.io/badge/Power%20BI-Dashboard-yellow) ![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

Machine learning capstone project (Boston Institute of Analytics) that predicts which customers will convert from a digital marketing campaign — so ad budget goes to the right people instead of being sprayed at everyone. The predictions power an AI lead-scoring system, a churn-risk analysis, and a fully interactive 3-page Power BI dashboard for ROAS (return on ad spend) optimization.

---

## The Problem

Marketing teams show ads to everyone, but not everyone converts. Every impression served to a customer who was never going to buy is wasted spend that drags down ROAS. The goal: predict conversion per customer from demographic and engagement data, and turn those predictions into concrete budget decisions.

## Key Results

| Model | Accuracy | ROC-AUC | PR-AUC | F1 (weighted) | **Class-0 Recall** |
|---|---|---|---|---|---|
| **Logistic Regression** ⭐ | 0.754 | 0.781 | 0.945 | 0.792 | **0.732** |
| Decision Tree | 0.833 | 0.625 | 0.904 | 0.835 | 0.348 |
| Random Forest | 0.883 | 0.810 | 0.949 | 0.838 | 0.071 |
| Gradient Boosting | 0.910 | 0.817 | 0.950 | 0.892 | 0.333 |

*Held-out test set (n = 1,600), all models evaluated inside the same leak-free pipeline.*

**Champion: Logistic Regression** — chosen for business fit, not the leaderboard. With an 87/13 class imbalance, accuracy is misleading (predicting "converts" for everyone scores 87.65%). What protects the ad budget is **catching non-converters**: the champion catches **73%** of them vs 33% (Gradient Boosting) and 7% (Random Forest), and its coefficients are fully interpretable for marketing stakeholders.

**Business impact (simulated at $50/ad):** suppressing ads to the correctly-identified non-converters saves **$9,750** per campaign cycle at the conservative default threshold — with only 0.9% of real buyers given up.

## Dataset

8,000 customers × 20 features, no missing values (digital marketing campaign dataset):

- **Demographics** — Age, Gender, Income
- **Campaign** — Channel (Email, PPC, Referral, SEO, Social Media), Type (Awareness, Consideration, Conversion, Retention)
- **Engagement** — AdSpend, ClickThroughRate, WebsiteVisits, PagesPerVisit, TimeOnSite, EmailOpens, EmailClicks, SocialShares
- **History** — PreviousPurchases, LoyaltyPoints
- **Target** — `Conversion` (1 = converted: 87.65%, 0 = did not: 12.35%)

## Workflow

The notebook (`Sellintelligence_EDA.ipynb`) runs in 12 phases:

1. **Load & inspect** — structure check, baseline conversion metrics
2. **EDA** — conversions by channel, income vs conversion, correlation heatmap
3. **Feature engineering** — drop ID/masked columns, one-hot encoding (drop-first), standard scaling
4. **Model training** — 4 classifiers on a stratified 80/20 split
5. **Evaluation** — confusion matrix + per-class report (accuracy is misleading at 87/13)
6. **Probability scoring** — conversion probability per customer + ROAS impact labels
7. **Business charts** — ROAS impact summary, top conversion drivers
8. **Lead scoring & ROI** — Hot / Warm / Cold tiers, ad-spend savings simulation
9. **Leak-free rebuild** — preprocessing moved inside sklearn Pipelines, 5-fold stratified CV, out-of-fold probabilities (an honest fix for a scaler leak found in the first pass)
10. **Churn analysis** — proxy churn definition, risk scores and tiers, segment summaries
11. **Model performance exports** — metrics, confusion matrices, ROC points, model dimension table, gain deciles
12. **Precision-recall curves** — the honest lens for imbalanced data

## Churn Analysis

No subscription dates exist in the data, so churn is a stated proxy: an **existing customer** (PreviousPurchases > 0) who **did not convert** in this campaign counts as lapsed.

- 7,162 existing customers → **798 churned (11.1%)**
- Churn risk score = 1 − P(conversion), from out-of-fold predictions
- Low-engagement customers churn at **17.6% vs 9.3%** for engaged ones — the earliest warning signal
- Email-acquired customers churn most (12.1%); PPC customers are most loyal (10.3%)
- 1,445 High-Risk customers identified for retention campaigns

## Power BI Dashboard

`Sellintelligence_ROAS_Optimization_Dashboard.pbix` — three fully interactive, cross-filtering pages (data embedded, no refresh needed to view):

| Page | What it shows |
|---|---|
| **Marketing Summary** | KPI cards, ROAS impact, lead quality, top conversion drivers, conversion by type / channel / age / gender / income, campaign slicers |
| **Churn Patterns** | Churn KPIs, churn rate by channel / age band / campaign type, risk tiers, engagement effect — every chart cross-filters |
| **Model Performance** | Metrics table, confusion matrix, ROC & PR curves, gain chart, with a Model slicer that flips the whole page between the four classifiers |

## Repository Structure

```
├── Sellintelligence_EDA.ipynb                         # Full analysis notebook (12 phases)
├── Sellintelligence_Final_Report.docx                 # Written project report
├── Sellintelligence_Capstone_Presentation.pptx        # Presentation deck (with speaker notes)
├── Sellintelligence_ROAS_Optimization_Dashboard.pbix  # 3-page Power BI dashboard
├── Sellintelligence_Final_Model.pkl                   # Trained champion model (joblib)
├── Sellintelligence_Final_Submission.zip              # Everything bundled for submission
├── Sellintelligence_Dashboard_Data.csv                # Scored customer base (dashboard source)
├── Sellintelligence_Final_Master_Data.csv             # Master data with lead tiers
├── PBI_Churn_Data.csv                                 # Row-level churn data
├── PBI_Churn_Segment_Summary.csv                      # Churn rates per segment
├── PBI_Model_Metrics.csv                              # Overall + per-class metrics, 4 models
├── PBI_Confusion_Matrix.csv                           # Confusion matrices (long format)
├── PBI_ROC_Curve.csv / PBI_PR_Curve.csv               # Curve points, 4 models
├── PBI_Gain_Deciles.csv                               # Gain/lift deciles (champion)
├── PBI_Models.csv                                     # Model dimension table (slicer)
└── data/raw/                                          # Source dataset (not tracked in git;
                                                       #   included in the submission zip)
```

## How to Run

```bash
git clone https://github.com/RimiCode/Sellintelligence-.git
cd Sellintelligence-
pip install pandas scikit-learn matplotlib seaborn joblib
# place digital_marketing_campaign_dataset.csv in data/raw/
jupyter notebook Sellintelligence_EDA.ipynb   # Run All - fully reproducible (fixed random_state)
```

Open the `.pbix` in Power BI Desktop to explore the dashboard (data is embedded). To load the model:

```python
import joblib
model = joblib.load("Sellintelligence_Final_Model.pkl")
```

## Recommendations

1. **Deploy as an ad-suppression list first** — the saving is immediate and measurable
2. **Tier the contact strategy** — premium ads for Hot leads, retargeting for Warm, suppress Cold
3. **Retention campaigns** for High-Risk churn customers; trigger on engagement drops
4. **Retrain quarterly** and monitor Class-0 recall; tune the threshold to the real wasted-ad vs lost-sale cost ratio

**Future work:** real churn labels from purchase timestamps, per-customer ad costs, hyperparameter search, threshold optimisation, SHAP explanations, uplift modelling.

## Tech Stack

Python 3.13 · pandas · scikit-learn · matplotlib · seaborn · joblib · Power BI Desktop

---

*Capstone project — Data Science & Analytics, Boston Institute of Analytics. Author: Rimi ([RimiCode](https://github.com/RimiCode))*
