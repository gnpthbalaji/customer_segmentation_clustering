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
