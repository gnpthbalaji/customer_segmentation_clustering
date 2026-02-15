# Customer Segmentation with Clustering

## Notebook Links

- [EDA](./notebooks/eda.ipynb)
- [Clustering Models](./notebooks/clustering_models.ipynb)

## What is Customer Segmentation

Customer segmentation is grouping customers into meaningful categories based on behavior or attributes. In an unsupervised setting, you do not have labels like “premium” or “churn risk.” Instead, you let patterns in the data suggest segments.

A clustering model does not automatically create business meaning. The clustering step produces groups. Interpretation comes from what the features represent, how stable the clusters are, and whether the segments map to actions you can take.

## Project Instructions

- Clean and preprocess the data by handling missing values, applying transformations if needed, and scaling features so no single variable dominates the clustering.

- Apply one or more clustering algorithms (for example K-Means, hierarchical clustering, or DBSCAN) and choose model parameters using appropriate diagnostics and metrics.

- Assign each customer to a cluster and evaluate whether the resulting segments are stable and meaningfully separated.

- Profile each cluster using summary statistics and visualizations to understand how the segments differ from one another.

- Describe each segment in plain language and propose at least one concrete business action or insight that could be taken based on the segmentation.

---

## Dataset Overview

The dataset contains **440 wholesale customers** with **8 features**: two categorical (Channel and Region) and six numerical spending categories (Fresh, Milk, Grocery, Frozen, Detergents_Paper, Delicassen). There were no missing values or duplicate records.

| Feature          | Mean    | Median | Std Dev |
| ---------------- | ------- | ------ | ------- |
| Fresh            | $12,000 | $8,504 | $12,647 |
| Milk             | $5,796  | $3,627 | $7,380  |
| Grocery          | $7,951  | $4,756 | $9,503  |
| Frozen           | $3,072  | $1,526 | $4,855  |
| Detergents_Paper | $3,072  | $1,286 | $4,768  |
| Delicassen       | $1,525  | $962   | $2,820  |

**Channel Distribution:** Channel 1 (Hotel/Restaurant/Café) — 298 customers (67.7%); Channel 2 (Retail) — 142 customers (32.3%).

**Region Distribution:** Region 3 dominates with 316 customers (71.8%), followed by Region 1 (77) and Region 2 (47).

---

## Exploratory Data Analysis

### Distribution & Skewness

All six spending features are heavily right-skewed, indicating most customers spend modestly while a small number of high-value customers drive up the averages. Delicassen has the highest skewness (11.15), followed by Frozen (5.91) and Milk (4.05). This motivated the use of log-transformations before clustering.

### Outliers

Outliers were identified using the IQR method. Frozen had the most outliers at 9.8% of customers, followed by Detergents_Paper (6.8%), Milk (6.4%), Delicassen (6.1%), Grocery (5.5%), and Fresh (4.5%). About 6% of customers were flagged as multivariate outliers via Mahalanobis distance — these likely represent VIP or high-volume buyers.

![Outlier Behavior Analysis](./results/analysis_1_outliers_behavior.png)

### Correlation Analysis

Three strong correlations emerged across all customers:

- **Grocery ↔ Detergents_Paper: 0.92** — nearly co-moving, suggesting retail supply bundles
- **Milk ↔ Grocery: 0.73**
- **Milk ↔ Detergents_Paper: 0.66**

Channel-level analysis revealed important differences. Channel 1 (Horeca) shows a weaker Grocery–Detergents correlation (0.55) but a unique Milk–Delicassen link (0.63). Channel 2 (Retail) shows very strong Grocery–Detergents correlation (0.92), consistent with a retail product bundle.

![Overall Correlation Matrix](./results/correlation_overall.png)

![Correlation by Channel](./results/correlation_by_channel.png)

### Channel & Region Spending Patterns

Channel 1 customers spend significantly more on Fresh ($13,476 vs $8,904) and Frozen ($3,748 vs $1,653), while Channel 2 customers dominate in Grocery ($16,323 vs $3,962), Milk ($10,717 vs $3,452), and Detergents_Paper ($7,270 vs $791).

![Average Spending by Channel](./results/stacked_spending_channel.png)

![Average Spending by Region](./results/stacked_spending_region.png)

### Spending Breakdown

On average, customers allocate their budgets as follows: Fresh (37.5%), Grocery (23.0%), Milk (16.8%), Frozen (10.6%), Detergents_Paper (7.4%), and Delicassen (4.8%). Total spending ranges from $904 to $199,891, with a mean of $33,226 and a median of $27,492.

![Feature Engineering](./results/analysis_4_feature_engineering.png)

### PCA (Principal Component Analysis)

PCA on log-transformed and scaled features revealed that the first two components explain 71.3% of variance, and three components capture 82.0%. Four components reach 92.1%.

- **PC1** loads heavily on Milk, Grocery, and Detergents_Paper (the "retail" axis)
- **PC2** loads on Fresh and Frozen (the "perishables" axis)

![PCA Analysis](./results/analysis_3_pca.png)

---

## Clustering Methodology

### Feature Preparation

Log-transformed features were used to address the heavy right-skew in the original data. RobustScaler was applied to further reduce outlier influence while preserving the relative structure of the data.

![Log Transform Comparison](./results/log_transform_comparison.png)

![Scaling Method Comparison](./results/scale_comparison.png)

### Determining Optimal k

Multiple methods were used to select the number of clusters:

- **Elbow Method:** Suggested k = 3–4 based on diminishing returns in inertia
- **Silhouette Score:** Highest at k = 2 (0.253), but k = 3 (0.244) was nearly as strong
- **Dendrogram:** Hierarchical analysis suggested 3–4 main clusters

Based on these diagnostics, k = 3, 4, and 5 were tested with both K-Means and hierarchical clustering.

![Elbow Method](./results/elbow_method.png)

![Silhouette Analysis](./results/silhouette_analysis.png)

![Dendrogram](./results/dendrogram.png)

![K-Distance Plot for DBSCAN](./results/k_distance_plot_dbscan.png)

### Algorithms Tested

**K-Means** (k = 3, 4, 5), **Agglomerative Hierarchical Clustering** with Ward linkage (k = 3, 4, 5), and **DBSCAN** (eps = 0.8–1.2) were all evaluated.

DBSCAN consistently found only one cluster with significant noise (8–25%), making it unsuitable for this dataset. The high dimensionality and varying cluster densities caused DBSCAN to fail.

### Model Comparison

| Algorithm    | k     | Silhouette | Davies-Bouldin | Calinski-Harabasz |
| ------------ | ----- | ---------- | -------------- | ----------------- |
| **K-Means**  | **3** | **0.245**  | **1.320**      | **138.18**        |
| K-Means      | 4     | 0.202      | 1.384          | 121.02            |
| K-Means      | 5     | 0.190      | 1.443          | 108.63            |
| Hierarchical | 3     | 0.243      | 1.392          | 112.78            |
| Hierarchical | 4     | 0.161      | 1.591          | 100.12            |
| Hierarchical | 5     | 0.138      | 1.586          | 91.76             |

**Best Model: K-Means with k = 3** — highest Silhouette score (0.245), lowest Davies-Bouldin index (1.320), and highest Calinski-Harabasz score (138.18).

![Model Comparison](./results/model_comparison.png)

---

## Cluster Evaluation

### Silhouette Analysis

The detailed silhouette plot shows that all three clusters have positive average silhouette values, indicating reasonable cohesion. While the overall score of 0.245 is moderate (below the 0.5 "good" threshold), this is common for real-world customer data with continuous features and overlapping segments.

![Detailed Silhouette Analysis](./results/silhouette_analysis_detailed.png)

### PCA Visualization

The 2D PCA projection (69.1% variance explained) shows the three clusters occupy distinct regions of the feature space, with some overlap at boundaries. The 3D projection (82.2% variance) provides clearer separation.

![PCA Cluster Visualization](./results/pca_visualization.png)

![3D PCA Visualization](./results/pca_3d_visualization.png)

---

## Cluster Profiles

![Cluster Heatmap](./results/cluster_heatmap.png)

![Spending by Cluster](./results/spending_by_cluster.png)

![Radar Chart — Cluster Profiles](./results/radar_chart.png)

![Channel & Region Distribution by Cluster](./results/channel_region_distribution.png)

### Cluster 0 — "High-Value Diversified Buyers" (151 customers, 34.3%)

| Category         | Avg Spending |
| ---------------- | ------------ |
| Fresh            | $16,720      |
| Grocery          | $13,224      |
| Milk             | $10,435      |
| Detergents_Paper | $4,956       |
| Frozen           | $4,037       |
| Delicassen       | $2,811       |
| **Total**        | **$52,182**  |

These are the highest-spending customers, buying heavily across all categories. The channel mix is 60.9% Retail and 39.1% Horeca, making this the most balanced cluster. They represent the top revenue tier and warrant the most attention for retention.

### Cluster 1 — "Horeca-Focused Fresh Buyers" (209 customers, 47.5%)

| Category         | Avg Spending |
| ---------------- | ------------ |
| Fresh            | $12,196      |
| Frozen           | $3,249       |
| Grocery          | $2,490       |
| Milk             | $1,980       |
| Delicassen       | $892         |
| Detergents_Paper | $445         |
| **Total**        | **$21,253**  |

The largest segment, dominated almost entirely by Channel 1 / Horeca (98.1%). Spending concentrates on Fresh and Frozen — perishable items consistent with restaurants and cafés. Grocery, Milk, and Detergents_Paper spending is minimal, reflecting their business needs.

### Cluster 2 — "Retail Grocery-Centric Buyers" (80 customers, 18.2%)

| Category         | Avg Spending |
| ---------------- | ------------ |
| Grocery          | $12,266      |
| Milk             | $7,010       |
| Detergents_Paper | $5,331       |
| Fresh            | $2,582       |
| Frozen           | $788         |
| Delicassen       | $750         |
| **Total**        | **$28,727**  |

The smallest cluster, with 57.5% Retail customers. Spending is heavily concentrated in Grocery, Milk, and Detergents_Paper — the classic retail product triad that showed the strongest correlations in the EDA. Fresh and Frozen spending is very low.

---

## Business Recommendations

### Cluster 0 — High-Value Diversified Buyers ($52,182/yr)

1. **VIP Loyalty Program** — Implement a tiered rewards program to retain these top spenders and increase lifetime value.
2. **Dedicated Account Management** — Assign account managers for personalized service, cross-sell opportunities, and early churn detection.
3. **Volume-Based Discounts** — Offer progressive pricing to incentivize even larger orders.
4. **Priority Service** — Provide faster delivery, extended credit terms, or exclusive product access.

### Cluster 1 — Horeca Fresh Buyers ($21,253/yr)

1. **Upselling Campaigns** — Target this large segment with promotions on Grocery and Milk products to broaden their purchasing mix.
2. **Introductory Promotions** — Offer trial pricing on underutilized categories (Detergents_Paper, Grocery) to shift behavior.
3. **Product Education** — Showcase how expanding product categories can benefit their restaurant or café operations.
4. **Bundle Offers** — Create Fresh + complementary category bundles at a discount.

### Cluster 2 — Retail Grocery Buyers ($28,727/yr)

1. **Cross-Sell Fresh & Frozen** — This cluster underspends on perishables; targeted promotions could grow basket size.
2. **Retail Partnership Programs** — Strengthen relationships with these retail-heavy customers through co-marketing or shelf placement support.
3. **Seasonal Campaigns** — Leverage their strong Grocery affinity with seasonal product bundles.

---
