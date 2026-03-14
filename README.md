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
- The file contains data for **all 10 seasons** — the code reads **only Season 1**, stopping when it encounters the marker `#s2e1`

**Season 1 coverage:** 24 episodes · **126 unique characters** · **281 unique interactions**

---

## Project Structure

```
├── social_network_analysis_project_friends.ipynb   # Main Jupyter notebook
├── friends_episodes.txt                            # Raw dataset (all seasons)
└── README.md                                       # This file
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
jupyter notebook social_network_analysis_project_friends.ipynb
```

The notebook runs **end-to-end without modification** as long as `friends_episodes.txt` is in the same directory.

---

## Week 1 — Network Construction & Basic Statistics

**Goal:** Build the network and characterise its basic topology.

**What the code does:**
- Parses `friends_episodes.txt`, stopping at `#s2e1` to isolate Season 1 only
- Counts repeated pairs to assign edge weights
- Constructs an **undirected weighted graph** using `nx.Graph`

**Metrics computed:**

| Metric | Value | Interpretation |
|--------|-------|----------------|
| Nodes | 126 | Unique characters in Season 1 |
| Edges | 281 | Unique interacting pairs |
| Average degree | 4.46 | Each character interacts with ~4.5 others on average |
| Density | 0.0357 | Only 3.6% of all possible pairs actually interacted |

**Visualisation:** Two graphs are produced deliberately to show the network at different levels of detail:

1. **Full network graph (126 nodes):** shows the real structure of the Season 1 network, including every peripheral guest character. Labels and edge weights are visible, giving a true picture of the network's density.
2. **Focused graph (top 30 by weighted degree):** a more readable view highlighting the main cast and the most active recurring characters. Node size is proportional to weighted degree; edge width is proportional to interaction count. The six main characters are colour-coded.

**Key finding:** The network has a clear **core-periphery structure**. The six main characters form a densely connected core, while the vast majority of characters (guest stars, one-episode appearances) are loosely attached at the periphery and rarely interact with each other.

---

## Week 2 — Clustering Analysis

**Goal:** Implement the clustering coefficient from scratch and compare it to NetworkX built-in functions.

All Week 2 analysis uses the **Largest Connected Component (LCC)** — in this dataset the LCC equals the full Season 1 graph (all 126 nodes are reachable from any other node).

### Functions implemented

**`get_node_clustering(graph, node)`**
Computes the local clustering coefficient for a single node:

$$C(v) = \frac{2 \times \text{edges among neighbours of } v}{k_v \cdot (k_v - 1)}$$

where $k_v$ is the degree of node $v$. Returns 0 for nodes with degree < 2.

**`get_average_clustering(graph)`**
Returns the arithmetic mean of all local clustering coefficients across the graph.

### Comparison with NetworkX

| Method | Value | Definition |
|--------|-------|------------|
| Custom `get_average_clustering` | 0.5058 | Mean of local CCs |
| `nx.average_clustering` | 0.5058 | Mean of local CCs — **matches exactly** ✓ |
| `nx.transitivity` | 0.1593 | Global ratio: `3 × triangles / triplets` |

**Why transitivity differs from average clustering:**
`nx.average_clustering` treats every node equally regardless of degree. `nx.transitivity` counts all triangles and connected triplets globally, so highly connected nodes (the main cast) contribute disproportionately. Since the main characters interact with many guest stars who never interact with each other, they generate a large number of open triplets — which drives the global transitivity down.

---

## Week 3 — Clustering Distributions

**Goal:** Examine how clustering is distributed across nodes and their neighbourhoods, and compare the two distributions.

### Plot a — CDF of Node Clustering Coefficients

The empirical CDF of local clustering coefficients is computed by sorting all node CC values and plotting $P(CC \leq x)$.

**Finding:** The distribution is strongly bimodal. About 40% of nodes have CC = 0.0 (characters who only know one person, or whose friends never interact with each other). A second large cluster sits at exactly CC = 1.0: these are one-off guest stars whose only connections are main cast members who already know each other — giving them a perfectly closed neighbourhood.

### Plot b — CDF of Average Neighbour Clustering

For each node $v$, the average clustering of its neighbours is:

$$\bar{C}_{nbr}(v) = \frac{1}{k_v} \sum_{u \in \mathcal{N}(v)} C(u)$$

**Function implemented:** `get_avg_neighbor_clustering(graph, node)`

**Finding:** This distribution is more concentrated, with most values falling between 0.1 and 0.4. The extreme values (0 and 1) that dominate the node CC distribution disappear here because averaging over multiple neighbours smooths out individual outliers.

### Plot c — Comparison of Both Distributions

The side-by-side comparison illustrates the **Friendship Paradox**:

| | Node CC Distribution | Neighbour CC Distribution |
|--|----------------------|---------------------------|
| **Shape** | Bimodal — spikes at 0 and 1 | Concentrated between 0.1–0.4 |
| **Interpretation** | Many peripheral nodes at the extremes | Averaging smooths out extremes |

Nodes with CC = 1.0 (right end of the blue curve) are minor characters whose only contacts are main cast members. But who are those neighbours? The main cast — and the main cast has very low clustering scores (~0.1) because they are connected to dozens of disconnected guest stars. This is precisely the Friendship Paradox: your friends are, on average, less clustered than you are.

---

## Authors

Course project submitted for the Social Network Analysis module.
Dataset: Friends interaction network (all seasons), Season 1 only analysed.
