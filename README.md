# Customer Segmentation Project

A robust customer segmentation project that transforms e-commerce transaction data into actionable customer groups. The final segmentation uses an RFM-based K-Means model and enriches the resulting segments with behavioural, timing, and product-preference profiling.

View the executive summary presentation [(PDF)](https://github.com/Thoericht/customer-segmentation-rfm-nlp/blob/master/03_results/20260902_OnlineRetail_ExecutiveSummary.pdf) and the modelcard of the final model [(MD)](https://github.com/Thoericht/customer-segmentation-rfm-nlp/blob/master/03_results/Modelcard_KMeans_20260902.md)

## Business Objective

The project answers two related questions:

1. Which customer groups differ in purchase recency, purchase frequency, and historical monetary value?
2. How can these groups be prioritised and addressed through differentiated retention, value-growth, and reactivation actions?

The goal is not only to build clusters, but to translate customer behaviour into interpretable and actionable segment profiles.

## Data Source

This project is based on the **Online Retail II** transaction dataset, accessed via Kaggle:

[Online Retail II – UCI dataset on Kaggle](https://www.kaggle.com/datasets/mashlyn/online-retail-ii-uci/)

The dataset contains historical e-commerce transaction records. It is used to build customer-level purchase profiles, derive Recency, Frequency, and Monetary (RFM) metrics, and develop an actionable customer segmentation.

The data is used for educational and portfolio purposes only. Raw source data is not redistributed in this repository.

## Key Results

- Customer-level profiles were created for **5,879 identifiable customers**.
- The final segmentation uses **Recency, Frequency, and Monetary value (RFM)**.
- The non-identifiable aggregated record (`CustomerID = "Unknown"`) was excluded from customer-level analysis and segmentation.
- To limit the influence of extreme high-frequency and high-monetary observations in distance-based clustering, Frequency and Monetary were capped at the **99th percentile** for model fitting.
- Modelling features were transformed using `log1p` and standardised before K-Means clustering.
- The two-cluster solution achieved the highest silhouette score, but the four-cluster solution was selected because it produced distinct, balanced, and more actionable business profiles.
- The selected four-cluster solution achieved a silhouette score of approximately **0.369** and very high resampling stability: mean Adjusted Rand Index (ARI) = **0.980**, SD = **0.006**.
- Text-derived SVD features and behavioural features did not improve clustering quality relative to the parsimonious Robust RFM model; they are retained for post-clustering profiling and activation.

## Final Customer Segments

| Segment | Customers | Customer share | Typical RFM profile | Suggested focus |
|---|---:|---:|---|---|
| Recent low-value buyers | 1,238 | 21.1% | Recent purchase; low purchase frequency and monetary value | Encourage a second and subsequent purchase |
| Champions | 1,219 | 20.8% | Recent, frequent, and high-value customers | Retain, recognise, cross-sell, and upsell |
| At-risk mid-value customers | 1,457 | 24.8% | Moderate purchase history, but a long time since the last purchase | Prioritised reactivation and retention |
| Dormant low-value customers | 1,964 | 33.4% | Very long inactivity, typically one purchase, and low monetary value | Low-cost reactivation or reduced contact intensity |

> Segment labels are business-oriented descriptions of K-Means clusters. They are based on original, uncapped RFM values used for segment profiling; capping was applied only to the model-training features.

## Methodology

### Data Pipeline

```text
Raw transaction data
    → cleaning and preparation
    → customer-level feature engineering
    → exploratory data analysis
    → robust RFM preprocessing
    → K-Means clustering
    → segment profiling and action recommendations
```

### Feature Engineering

The customer-level dataset includes:

- **RFM metrics:** `recency`, `frequency`, `monetary`
- **Purchase-timing features:** preferred purchase hour, average purchase hour, weekend share, and preferred time-of-day category
- **Purchase-rhythm features:** active purchase days, number of order dates, and average days between orders
- **Product-text features:** TF-IDF representations of product descriptions reduced using Truncated SVD

### Robust RFM Preprocessing

The final RFM model uses the following preparation steps:

1. Exclude records without a valid customer identifier.
2. Cap `frequency` and `monetary` at their respective 99th percentiles.
3. Apply `log1p` transformation to Recency, capped Frequency, and capped Monetary value.
4. Standardise the modelling features with `StandardScaler`.
5. Fit K-Means models across a range of candidate cluster counts.

### Model Evaluation

Candidate K-Means solutions were compared using:

- Silhouette score
- Inertia and elbow inspection
- Cluster size distribution
- Interpretability of RFM profiles
- Resampling stability using the Adjusted Rand Index (ARI)

The two-cluster model had the strongest silhouette score and serves as a useful statistical baseline. The four-cluster model was selected because it retained acceptable internal separation while providing substantially more actionable customer groups.

## Model Comparison

| Model | Purpose | Outcome |
|---|---|---|
| RFM baseline | Initial benchmark using log-transformed RFM features | Establishes a simple comparison model |
| Robust RFM | p99 capping, log transformation, and scaling | Selected for the stable, interpretable, and actionable four-segment solution |
| Robust RFM + NLP | Tests whether product-text features improve segmentation | Did not outperform Robust RFM |
| Robust RFM + Behaviour | Tests whether timing and purchase-rhythm features improve segmentation | Did not outperform Robust RFM; retained for segment profiling |

## Repository Structure

```text
├── 01_data_ingest.ipynb
├── 02_data_preparation.ipynb
├── 03_explorative_data_analysis.ipynb
├── 04_clustering.ipynb
├── data/
│   ├── raw/          # not versioned
│   └── processed/    # not versioned
└── README.md
```

- **Data ingest:** Loads and inspects the source data.
- **Data preparation:** Cleans transactions and creates customer-level analytical features.
- **Exploratory data analysis:** Examines distributions, data quality, relationships, and outliers.
- **Clustering:** Builds, compares, validates, and interprets customer segmentation models.

## Reproducing the Project

1. Download the dataset from the Kaggle source linked above.
2. Store the source file locally in `data/raw/`.
3. Create and activate a Python environment with the required packages.
4. Run the notebooks in numerical order, from `01_data_ingest.ipynb` through `04_clustering.ipynb`.

The repository intentionally excludes raw data and customer-level output files. Update the local file path in the ingest notebook if your source filename differs.

## Technologies

- Python
- pandas
- NumPy
- Matplotlib
- Seaborn
- scikit-learn
- SciPy
- Jupyter Notebook

## Limitations

- RFM segmentation describes historical purchasing behaviour; it does not directly predict future churn, customer lifetime value, or campaign response.
- The clusters are analytical groupings rather than fixed natural customer types.
- The model was designed for interpretability and actionability in a customer-insights context.
- Timing and offer recommendations are hypotheses for testing, not evidence of campaign causality.
- Segment-specific interventions should be validated using controlled campaign experiments, ideally with control groups or A/B tests.

## Next Steps

1. Profile the selected RFM segments by purchase time, weekday, month, seasonality, and purchase rhythm.
2. Connect transaction-level order dates to the final customer-segment labels.
3. Analyse product-category and product-description preferences by segment.
4. Quantify segment value further through revenue contribution and customer-value measures.
5. Translate segment profiles into testable CRM and marketing journeys.
6. Validate recommended communication timing and offers through controlled campaign experiments.