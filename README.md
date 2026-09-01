# Customer Segmentation Project

A robust customer segmentation project that transforms e-commerce transaction data into actionable customer groups. The final segmentation uses an RFM-based K-Means model and enriches the resulting segments with behavioural, time, and product-preference profiling.

## Business Objective

The project answers two related questions:

1. Which customer groups differ in purchase recency, purchase frequency, and historical monetary value?
2. How can these groups be prioritised and addressed through differentiated retention, value-growth, and reactivation actions?

The goal is not only to build clusters, but to translate customer behaviour into interpretable and actionable segment profiles.

## Data Source

This project is based on the **Online Retail II** transaction dataset, accessed via Kaggle:

[Online Retail II – UCI dataset on Kaggle](https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci/)

The dataset contains historical e-commerce transaction records. It is used to build customer-level purchase profiles, derive Recency, Frequency, and Monetary value metrics, and develop an actionable customer segmentation.

The data is used for educational and portfolio purposes only. Raw source data is not redistributed in this repository.

## Current Status

**Current stage: Final RFM segmentation selected; behavioural and time-based segment profiling in progress.**

The project has established a robust four-segment RFM solution. Additional NLP and behavioural features have been tested as alternative clustering inputs, but the core RFM model currently provides the clearest and most interpretable segmentation.

## Key Results

- Customer-level profiles were created from transaction data for **5,879 customers**.
- The segmentation is based on **Recency, Frequency, and Monetary value (RFM)**.
- Extreme values were investigated during exploratory analysis.
- A non-identifiable aggregated record (`CustomerID = "Unknown"`) was excluded from customer-level segmentation.
- To reduce the influence of extreme high-frequency and high-monetary observations on distance-based clustering, Frequency and Monetary were capped at the **99th percentile** for model fitting.
- RFM variables were subsequently transformed with `log1p` and standardised before K-Means clustering.
- The two-cluster solution achieved the highest silhouette score, but the four-cluster solution was selected as the final business-oriented segmentation because it produced distinct, balanced, and actionable customer profiles.
- The selected four-cluster solution achieved a silhouette score of approximately **0.369** and showed very high resampling stability: mean Adjusted Rand Index (ARI) = **0.980**, SD = **0.006**.
- Adding text-derived SVD features did not improve clustering quality relative to the parsimonious RFM model.

## Final Customer Segments

| Segment | Customers | Customer share | Typical RFM profile | Suggested focus |
|---|---:|---:|---|---|
| Recent low-value buyers | 1,238 | 21.1% | Recent purchase; low purchase frequency and monetary value | Encourage a second and subsequent purchase |
| Champions | 1,219 | 20.8% | Recent, frequent, and high-value customers | Retain, recognise, cross-sell, and upsell |
| At-risk mid-value customers | 1,457 | 24.8% | Moderate purchase history, but long time since the last purchase | Prioritised reactivation and retention |
| Dormant low-value customers | 1,964 | 33.4% | Very long inactivity, typically one purchase, and low monetary value | Low-cost reactivation or reduced contact intensity |

> Segment labels are business-oriented descriptions of K-Means clusters. They are based on the original, uncapped RFM values used for segment profiling; capping was applied only to the model-training features.

## Methodology

### Data pipeline

```text
Raw transaction data
    → cleaning and preparation
    → customer-level feature engineering
    → exploratory data analysis
    → robust RFM preprocessing
    → K-Means clustering
    → segment profiling and action recommendations
```

### Feature engineering

The customer-level dataset includes:

- **RFM metrics:** `recency`, `frequency`, `monetary`
- **Purchase timing features:** preferred purchase hour, average purchase hour, weekend share, and preferred time-of-day category
- **Purchase rhythm features:** active purchase days, number of order dates, and average days between orders
- **Product-text features:** TF-IDF representations of product descriptions reduced using Truncated SVD

### Robust RFM preprocessing

The final RFM model uses the following preparation steps:

1. Exclude records without a valid customer identifier.
2. Cap `frequency` and `monetary` at their respective 99th percentiles.
3. Apply `log1p` transformation to Recency, capped Frequency, and capped Monetary value.
4. Standardise the modelling features with `StandardScaler`.
5. Fit K-Means models for a range of candidate cluster counts.

### Model evaluation

Candidate K-Means solutions were compared using:

- Silhouette score
- Inertia / elbow inspection
- Cluster size distribution
- Interpretability of RFM profiles
- Resampling stability using the Adjusted Rand Index (ARI)

The two-cluster model had the strongest silhouette score and represents a useful statistical baseline. The four-cluster model was selected because it retained acceptable internal separation while providing substantially more actionable customer groups.

## Model Comparison

| Model | Purpose | Outcome |
|---|---|---|
| RFM baseline | Initial benchmark using log-transformed RFM features | Establishes a simple comparison model |
| Robust RFM | Final model with p99 capping, log transformation, and scaling | Selected for stable, interpretable, and actionable four-segment solution |
| Robust RFM + NLP | Tests whether product-text features improve segmentation | Did not outperform the robust RFM model |
| Robust RFM + Behaviour | Tests whether timing and purchase-rhythm features improve segmentation | Did not outperform the robust RFM model; retained for segment profiling |

## Repository Structure

```text
├── 01_data_ingest.ipynb
├── 02_data_preparation.ipynb
├── 03_explorative_data_analysis.ipynb
├── 04_clustering.ipynb
├── data/
│   ├── raw/
│   └── processed/
└── README.md
```

- **Data ingest:** Loads and inspects the source data.
- **Data preparation:** Cleans transactions and creates customer-level analytical features.
- **Exploratory data analysis:** Examines distributions, data quality, relationships, and outliers.
- **Clustering:** Builds, compares, validates, and interprets customer segmentation models.

## Next Steps

1. Profile the selected RFM segments by purchase time, weekday, month, seasonality, and purchase rhythm.
2. Connect transaction-level order dates to the final customer-segment labels.
3. Analyse product-category and product-description preferences by segment.
4. Quantify segment value further through revenue contribution and customer-value measures.
5. Translate segment profiles into testable CRM and marketing journeys.
6. Validate recommended communication timing and offers through controlled campaign experiments.

## Technologies

- Python
- pandas
- NumPy
- Matplotlib
- Seaborn
- scikit-learn
- SciPy
- Jupyter Notebook

## Notes and Limitations

- RFM segmentation describes historical purchasing behaviour; it does not directly predict future churn, future customer lifetime value, or campaign response.
- The clusters are analytical groupings rather than fixed natural customer types.
- The model was designed for interpretability and actionability in a customer-insights context.
- Segment-specific interventions should be validated using campaign experiments, ideally with control groups or A/B tests.
