# Manufacturing Defect Discovery — Clustering & Association Rules

## Problem

A manufacturing company runs thousands of production batches, but defects don't occur
uniformly across the plant. Some combinations of machine condition, material, process
settings, and shift factors are riskier than others. The goal of this project is to
segment production batches into distinct operating environments and identify which
condition combinations within each environment are associated with specific defect
types.

## Dataset

4,000 production batches with 30+ recorded variables, covering:

- Continuous process readings: operating temperature, pressure, vibration, power
  consumption, material hardness/thickness, humidity, cycle time, machine age,
  production speed, tool age, batch size
- Pre-thresholded binary flags for the same variables (e.g. `High_Temperature`,
  `Old_Machine`, `Large_Batch`)
- Categorical fields: machine type, material type, shift, operator experience,
  production line
- Target fields: `Defect` (yes/no) and `Defect_Type` (Assembly, Material, Dimensional,
  Structural, Surface, or none)

A small percentage of rows had missing values across the continuous columns; these
were imputed using the median of each column before clustering.

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
| 0 | 961 | 21.4% | Fast production lines — highest speed and operating temperature, moderately high power consumption, frequent large batches and high workload |
| 1 | 1514 | 9.2% | Standard operating conditions — lowest vibration and power consumption of the four clusters, few elevated-condition flags |
| 2 | 701 | 44.9% | Aging equipment — oldest machines and tools, highest vibration, longest cycle times |
| 3 | 824 | 35.6% | Heavy material processing — highest pressure and power consumption, hardest/thickest material, relatively old tooling |

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
