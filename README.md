# Social Network Analysis — *Friends* Season 1
### Course Project · Weeks 1–3

---

## Overview

This project analyses the social network of the TV series **Friends (Season 1)** using Python and NetworkX. Characters are modelled as nodes and their interactions during episodes as weighted, undirected edges. The weight of each edge equals the number of times that pair of characters interacted across the season.

---

## Dataset

**File:** `friends_episodes.txt`

**Format:**
- Lines starting with `#` are episode markers (e.g. `# s1e1`) and are **ignored**
- Every other non-empty line contains exactly two character names: `CharacterA CharacterB`
- Repeated pairs accumulate as edge weight

**Coverage:** Season 1 of Friends — 24 episodes, 747 unique characters, 1,610 unique interactions

---

## Project Structure

```
├── friends_sna.ipynb        # Main Jupyter notebook (all three weeks)
├── friends_episodes.txt     # Raw dataset
├── friends_network.png      # Output: network visualisation
├── friends_degree_bar.png   # Output: weighted degree bar chart
├── cdf_cc.png               # Output: CDF of clustering coefficients
├── cdf_nb_cc.png            # Output: CDF of neighbour clustering
├── cdf_comparison.png       # Output: side-by-side CDF comparison
└── README.md                # This file
```

---

## Requirements

```
python >= 3.9
networkx
numpy
matplotlib
jupyter
```

Install dependencies:

```bash
pip install networkx numpy matplotlib jupyter
```

Run the notebook:

```bash
jupyter notebook friends_sna.ipynb
```

The notebook runs **end-to-end without modification** as long as `friends_episodes.txt` is in the same directory.

---

## Week 1 — Network Construction & Basic Statistics

**Goal:** Build the network and characterise its basic topology.

**What the code does:**
- Parses `friends_episodes.txt`, skipping comment lines
- Counts repeated pairs to assign edge weights
- Constructs an **undirected weighted graph** using `nx.Graph`

**Metrics computed:**

| Metric | Value | Interpretation |
|--------|-------|----------------|
| Nodes | 747 | Unique characters across all episodes |
| Edges | 1,610 | Unique interacting pairs |
| Average degree | 4.31 | Each character interacts with ~4 others on average |
| Density | 0.0058 | Very sparse — only 0.6% of all possible pairs interact |

**Visualisation:** Two figures are produced for clarity:
1. **Network graph** — top 35 characters by weighted degree, edges filtered to weight ≥ 3. Node size encodes weighted degree; edge width and opacity encode interaction count. The six main characters are colour-coded.
2. **Bar chart** — ranked weighted degrees of the same 35 characters, showing the sharp drop-off between the main cast and recurring guests.

**Key finding:** The network has a clear hub-and-spoke structure. The six main characters (Monica, Chandler, Ross, Joey, Rachel, Phoebe) each have weighted degrees of ~4,400–5,000, while the next highest (Judy, Mike, Jack) reach only ~180. This reflects the ensemble nature of the show.

---

## Week 2 — Clustering Analysis

**Goal:** Implement clustering coefficient functions from scratch and compare them to NetworkX built-ins.

All analysis in this week uses the **Largest Connected Component (LCC)** — in this dataset the LCC equals the full graph (all 747 nodes are connected).

### Functions implemented

**`clustering_coefficient(G, node)`**
Computes the local clustering coefficient for a single node:

$$C(v) = \frac{\text{number of edges among neighbours of } v}{\binom{k_v}{2}}$$

where $k_v$ is the degree of node $v$. Returns 0 for nodes with degree < 2.

**`all_clustering_coefficients(G)`**
Returns a dictionary `{node: CC}` for every node in the graph.

**`average_clustering_coefficient(G)`**
Returns the arithmetic mean of all local clustering coefficients.

### Comparison with NetworkX

| Method | Value | Definition |
|--------|-------|------------|
| Custom `average_clustering_coefficient` | 0.5003 | Mean of local CCs |
| `nx.average_clustering` | 0.5003 | Mean of local CCs — **matches exactly** ✓ |
| `nx.transitivity` | 0.0335 | Global ratio: `3 × triangles / triplets` |

**Why transitivity differs from average clustering:**
`nx.average_clustering` treats every node equally regardless of degree. `nx.transitivity` counts triangles and connected triplets globally, so high-degree hub nodes (the main cast) contribute disproportionately. Since hubs are only moderately clustered — they connect to many guest characters who don't connect to each other — transitivity is much lower than the node-averaged metric.

---

## Week 3 — Clustering Distributions

**Goal:** Examine how clustering is distributed across nodes and their neighbourhoods.

### 3a — Cumulative Distribution of Clustering Coefficients

The CDF of local clustering coefficients is computed by sorting all node CC values and plotting the empirical CDF: $P(CC \leq x)$.

**Finding:** The distribution is strongly bimodal. A large fraction of nodes have CC = 1 (peripheral characters with only 2 neighbours who also know each other), while the hub nodes cluster around intermediate values (0.3–0.7).

### 3b — Average Clustering of Neighbours

For each node $v$, the average clustering of its neighbours is:

$$\bar{C}_{nbr}(v) = \frac{1}{k_v} \sum_{u \in \mathcal{N}(v)} C(u)$$

**Function implemented:** `neighbor_avg_clustering(G, cc_dict)`

### 3c — Cumulative Distribution of Neighbour Clustering

The same CDF procedure is applied to the neighbour-averaged CC values.

### 3d — Comparison of the Two Distributions

| | Node CC Distribution | Neighbour CC Distribution |
|--|----------------------|---------------------------|
| **Mean** | 0.50 | Higher (shifted right) |
| **Shape** | Bimodal, high variance | Smoother, more concentrated |
| **Driven by** | Many peripheral nodes with CC = 1 | Averaging smooths out extremes |

**Interpretation:** The neighbour CC distribution is shifted right compared to the node CC distribution. This is because peripheral characters (CC = 1, tiny fully-connected neighbourhoods) tend to be neighbours of the hubs, pulling the neighbour average upward for those hubs. Meanwhile, hubs themselves average over many neighbours with varying CC, producing intermediate values. This asymmetry is a signature of the **friendship paradox**: your friends are, on average, more "clustered" than you are.

---

## Authors

Course project submitted for the Social Network Analysis module.  
Dataset: Friends Season 1 interaction network.
