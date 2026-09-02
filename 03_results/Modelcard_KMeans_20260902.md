# Model Card: Robust RFM K-Means Segmentation

## Model Summary

| Field | Description |
|---|---|
| **Model name** | Robust RFM K-Means Segmentation |
| **Model type** | Unsupervised clustering / customer segmentation |
| **Primary use** | Group identifiable customers into actionable profiles for retention, value growth, and reactivation planning |
| **Selected configuration** | K-Means with 4 clusters |
| **Training population** | 5,879 identifiable customers from the Online Retail II transaction dataset |
| **Unit of analysis** | Customer |
| **Output** | One of four business-oriented segment labels per customer |

## Intended Use

### Appropriate uses

- Prioritising customer-retention and reactivation efforts.
- Designing differentiated CRM journeys, including VIP recognition, second-purchase activation, and low-cost win-back campaigns.
- Profiling customer groups by purchase timing, purchase rhythm, and product-preference signals after segmentation.
- Supporting exploratory customer-insights work and hypothesis generation.

### Out-of-scope uses

- Predicting churn, future customer lifetime value, campaign response, or individual purchase propensity.
- Automatically deciding eligibility, pricing, creditworthiness, or access to services.
- Treating segment membership as a fixed or causal customer characteristic.
- Using recommendations as proof that a campaign, offer, or contact time will cause a business outcome.

## Data and Scope

The model is based on the **Online Retail II** transaction dataset accessed through Kaggle. The dataset contains historical e-commerce transactions from 2009 to 2011.

Customer-level analysis includes only records with a valid customer identifier. The non-identifiable aggregated record (`CustomerID = "Unknown"`) is excluded because its transactions cannot be assigned to an individual purchase history.

| Item | Scope |
|---|---|
| **Analysis period** | 2009–2011 historical transactions |
| **Customers modelled** | 5,879 identifiable customers |
| **Core input data** | Transaction dates, invoice identifiers, quantities, unit prices, and customer identifiers |
| **Excluded from model fitting** | Non-identifiable customer records, cancellations, returns, and non-positive-price transactions |
| **Data availability** | Raw source data is not redistributed in this repository |

## Features

The selected model uses only three customer-level RFM variables:

| Feature | Definition | Interpretation |
|---|---|---|
| **Recency** | Days since the customer's most recent purchase | Lower values indicate more recent activity |
| **Frequency** | Number of distinct purchase transactions | Higher values indicate repeat purchasing |
| **Monetary** | Historical monetary value of purchases | Higher values indicate greater historical customer value |

Purchase-timing, purchase-rhythm, and text-derived product-preference features were evaluated as extensions. They were not included in the final clustering input because they did not improve cluster separation; they are used for post-clustering profiling and campaign activation.

## Preprocessing

The final Robust RFM pipeline applies the following steps:

1. Retain customers with valid customer identifiers.
2. Construct customer-level Recency, Frequency, and Monetary features from cleaned transactions.
3. Cap Frequency and Monetary values at their respective 99th percentiles for model fitting.
4. Apply `log1p` transformation to Recency, capped Frequency, and capped Monetary.
5. Standardise the transformed features with `StandardScaler`.
6. Fit K-Means models for candidate values of `k` from 2 to 9.

Capping and log transformation reduce the influence of extreme observations on Euclidean-distance-based clustering. Segment profiles are interpreted using the original, uncapped RFM variables.

## Model Selection

The Robust RFM model was selected over baseline RFM, RFM plus product-text features, and RFM plus behavioural features because it provided the strongest overall combination of separation, stability, interpretability, and business actionability.

| Model | Best silhouette score | Assessment |
|---|---:|---|
| RFM baseline | 0.439 | Strong initial benchmark |
| Robust RFM | 0.440 | Best overall separation; selected core approach |
| Robust RFM + NLP | 0.383 | Product-text features did not improve segmentation |
| Robust RFM + Behaviour | 0.209 | Behavioural features reduced cluster separation |

The two-cluster Robust RFM solution achieved the highest silhouette score. The four-cluster solution was selected for operational use because it retained acceptable separation while producing more distinct, balanced, and actionable customer groups.

## Performance and Validation

| Measure | Result | Interpretation |
|---|---:|---|
| **Selected number of clusters** | 4 | Business-oriented final configuration |
| **Silhouette score, selected model** | 0.369 | Acceptable internal separation for the selected four-cluster solution |
| **Best Robust RFM silhouette score** | 0.440 at `k = 2` | Statistical benchmark; too coarse for business use |
| **Mean ARI, resampling stability** | 0.980 | Very stable cluster assignments across resampling runs |
| **ARI standard deviation** | 0.006 | Low variation in stability estimates |
| **Resampling design** | 10 runs, 80% samples | Stability check against the fitted four-cluster baseline |

Silhouette score measures internal cluster separation; it does not demonstrate causal impact, predictive accuracy, or financial uplift. ARI measures consistency of cluster assignments across resamples; it does not prove that segment labels will remain unchanged as customer behaviour or the underlying business changes.

## Segment Outputs

| Segment | Customers | Share | Median Recency | Median Frequency | Median Monetary | Suggested use |
|---|---:|---:|---:|---:|---:|---|
| **Champions** | 1,219 | 20.8% | 17 days | 13 orders | 5,067 | Retain, recognise, cross-sell, and upsell |
| **Recent low-value buyers** | 1,238 | 21.1% | 24 days | 3 orders | 730 | Encourage a second and subsequent purchase |
| **At-risk mid-value customers** | 1,457 | 24.8% | 185 days | 4 orders | 1,496 | Prioritise reactivation and retention |
| **Dormant low-value customers** | 1,964 | 33.4% | 405 days | 1 order | 282 | Use low-cost reactivation or reduce contact intensity |

> Segment labels are business-oriented descriptions applied after K-Means clustering. They are not natural or permanent customer types.

## Limitations

- The analysis is based on historical transactions and does not include campaign exposure, channel engagement, marketing costs, margins, customer demographics, or future outcomes.
- RFM variables describe observed activity and historical monetary value; they do not directly predict future churn, lifetime value, or response to a campaign.
- K-Means depends on Euclidean distance and can be sensitive to feature design, preprocessing choices, data drift, and the chosen number of clusters.
- The p99 cap intentionally limits the influence of extreme Frequency and Monetary values during training. Those customers may still be commercially important and should be reviewed separately where appropriate.
- The model excludes transactions without a valid customer identifier; segment results therefore describe identifiable customers only.
- Segment actions are recommendations for testing, not validated causal effects. Controlled experiments are needed before scaling interventions.

## Monitoring and Maintenance

- Recompute RFM features on a defined business cadence, such as monthly or quarterly.
- Monitor changes in segment sizes, median RFM values, and the distribution of transformed model inputs.
- Reassess silhouette score, stability, and business usefulness whenever the model is retrained.
- Compare new cluster profiles with the existing segment definitions; revise labels if their behavioural meaning changes.
- Validate segment-specific CRM actions with holdout groups or A/B tests and monitor incremental revenue, retention, conversion, and contact cost.
- Review the p99 capping threshold if transaction volume, pricing, customer mix, or extreme-value patterns materially change.

## Reproducibility

The primary modelling workflow is documented in `04_clustering.ipynb`. Data preparation and customer-level feature construction are documented in `02_data_preparation.ipynb`; exploratory analysis is documented in `03_explorative_data_analysis.ipynb`.

For reproducibility, download the source dataset from the repository README, place it locally in the expected data directory, and run the notebooks in numerical order. Raw data and customer-level output files are intentionally excluded from the public repository.

## Versioning

| Field | Value |
|---|---|
| **Model version** | 1.0 |
| **Status** | Selected portfolio model |
| **Last updated** | 2 September 2026 |
| **Owner** | Heike Thöricht |