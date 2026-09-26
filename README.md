# Manufacturing Defect Discovery - Clustering & Association Rules

##Overview

Manufacturing defects are often influenced by combinations of operating and production conditions rather than by a single factor. This project analyzes manufacturing production data to identify distinct production environments and investigate the conditions associated with different types of defects.

The analysis follows a two-stage approach:

1) K-Means clustering is used to segment production batches into distinct operating environments.
2) Apriori association rule mining is applied separately within each cluster to identify recurring combinations of production conditions associated with specific defect types.

This allows the analysis to move from identifying what types of production environments exist to understanding which condition combinations are associated with defects within each environment.

## Problem

A manufacturing facility may operate under many combinations of temperature, pressure, vibration, power consumption, material properties, machine age, production speed, and other conditions.

Simply looking at the overall defect rate may hide important patterns because the same defect can occur under different production environments.

The objective of this project is to:

Segment production batches into meaningful operating-condition clusters.
Compare defect rates across the identified clusters.
Profile the characteristics of each production environment.
Identify recurring combinations of production conditions associated with specific defect types.
Provide interpretable patterns that can be investigated for process monitoring and quality improvement.

## Dataset

The dataset contains 4,000 production records and 35 columns.

Continuous Production Variables:

Operating Temperature
Pressure
Vibration
Power Consumption
Material Hardness
Humidity
Cycle Time
Machine Age
Production Speed
Material Thickness
Tool Age
Batch Size

Missing values in these numerical variables are handled using median imputation before clustering.

Categorical Variables:

Machine Type
Material Type
Shift
Operator Experience
Production Line

These variables are not used to determine the K-Means clusters. They are encoded and used for post-clustering profiling to understand the composition of each production environment.

Binary Production Conditions:

Old Machine
High Speed
High Temperature
High Pressure
High Vibration
Long Cycle
High Power Consumption
Thick Material
Hard Material
Old Tool
Large Batch
High Humidity
Night Shift
New Operator
High Workload

These indicators are used to construct transactions for Apriori association rule mining.

Target Information:

Defect — indicates whether a production batch contains a defect.
Defect_Type — identifies the type of defect.

Defect categories include:

Assembly Defect
Dimensional Defect
Material Defect
Structural Defect
Surface Defect
No Defect

## Approach

The analysis is split into two stages:

1. **Clustering** — group batches into production environments using only the
   continuous, standardized process variables. Categorical fields (machine type,
   material type, shift, etc.) and the binary flags were deliberately excluded from
   the clustering step and used afterward only to describe and interpret the
   resulting clusters.
2. **Association rule mining** — within each cluster, run Apriori on the binary
   condition flags plus `Defect_Type` to find which condition combinations recur
   alongside specific defect types.

K-Means was used for clustering, with the number of clusters (k=4) chosen using the
elbow method and cross-checked against a hierarchical dendrogram. Apriori was run
separately per cluster (rather than on the whole dataset at once) so that the rules
reflect what's actually happening within each environment, not patterns that only
exist because different environments are being averaged together.

## Cluster profiles

| Cluster | Size | Defect rate | Profile |
|---|---|---|---|
| 0 | 961 | 21.4% | Fast production lines - highest speed and operating temperature, moderately high power consumption, frequent large batches and high workload |
| 1 | 1514 | 9.2% | Standard operating conditions - lowest vibration and power consumption of the four clusters, few elevated-condition flags |
| 2 | 701 | 44.9% | Aging equipment - oldest machines and tools, highest vibration, longest cycle times |
| 3 | 824 | 35.6% | Heavy material processing - highest pressure and power consumption, hardest/thickest material, relatively old tooling |

Cluster 2 and Cluster 3 stand out as the higher-risk environments, with defect rates
roughly 4–5x that of Cluster 1. Cluster 1 behaves as a baseline: process conditions
stay close to normal and defects are comparatively rare.

## What Apriori found in each cluster

**Cluster 0 (fast lines).** Assembly defects were tied to new operators working
night shifts on fast, large-batch lines. Material defects showed up when hard
material was combined with high pressure, temperature, or power draw. A smaller
set of rules linked high vibration and pressure to dimensional defects, and night
shift plus thick material to structural defects.

**Cluster 1 (standard conditions).** No rules cleared the support/confidence/lift
thresholds. This is consistent with the low defect rate in this cluster — there
simply isn't a strong recurring pattern to detect here, which is itself a useful
result: conditions in this cluster don't need targeted intervention.

**Cluster 2 (aging equipment).** The strongest rules in the entire analysis came
from this cluster. Several combinations of high pressure, high temperature, and long
cycle time reached 100% confidence for surface defects. Old machines and tools
combined with high vibration were linked to structural and material defects, and
night shift plus old equipment showed up repeatedly in assembly defect rules.

**Cluster 3 (heavy material).** Surface defects were the dominant pattern, driven by
high humidity together with hard or thick material, often alongside high
temperature and older machines. Assembly defects were associated with long cycle
times on old machines, and dimensional defects with high temperature and large
batch sizes on worn tooling.

## Reading these results

These are association rules, not causal relationships — a rule with high lift means
a condition combination and a defect type occur together far more often than chance
would suggest, not that one directly causes the other. They're best used as a
starting point for investigation (e.g. checking Cluster 2's oldest machines first,
or reviewing new-operator onboarding on Cluster 0's fast lines) rather than as a
final diagnosis.

## Tools

Python, pandas, scikit-learn (KMeans, StandardScaler, SimpleImputer,
AgglomerativeClustering, ColumnTransformer/OneHotEncoder), scipy (dendrogram), and
apyori for association rule mining.
