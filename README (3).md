# Social Network Analysis — *Friends* Season 1
### Course Project · Weeks 1–3 & 5–7

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
├── friends_sna_weeks1_3.ipynb   # Jupyter notebook — Weeks 1–3
├── friends_sna_weeks5_7.ipynb   # Jupyter notebook — Weeks 5–7
├── friends_episodes.txt         # Raw dataset of character interactions
├── friends_communities.gexf     # Network export for Gephi Lite visualization
├── friends_network.png          # Output: network visualisation
├── friends_degree_bar.png       # Output: weighted degree bar chart
├── cdf_cc.png                   # Output: CDF of clustering coefficients
├── cdf_nb_cc.png                # Output: CDF of neighbour clustering
├── cdf_comparison.png           # Output: side-by-side CDF comparison
├── week5_centrality.png         # Output: centrality bar plots
├── week5_scatter.png            # Output: betweenness vs PageRank scatter
├── week6_community_sizes.png    # Output: community size distributions
├── week6_community_graph.png    # Output: network coloured by community
├── week7_link_prediction.png    # Output: top predicted links bar plots
├── week7_cn_vs_aa_scatter.png   # Output: CN vs AA scatter plot
└── README.md                    # This file
```

---

## Requirements

```
python >= 3.9
networkx
numpy
pandas
matplotlib
jupyter
```

Install dependencies:

```bash
pip install networkx numpy pandas matplotlib jupyter
```

Run the notebooks:

```bash
jupyter notebook friends_sna_weeks1_3.ipynb
jupyter notebook friends_sna_weeks5_7.ipynb
```

Both notebooks run **end-to-end without modification** as long as `friends_episodes.txt` is in the same directory.

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

## Week 5 — Centrality Analysis

**Goal:** Identify the most structurally important characters in the network using two complementary centrality measures.

For Weeks 5–7 the network is treated as **undirected and unweighted** (edges are binary — a pair either interacted or did not), self-loops are removed, and all analysis is performed on the **Largest Connected Component**.

### Measures computed

**Betweenness Centrality**

Betweenness centrality measures the fraction of all shortest paths in the network that pass through a given node:

$$BC(v) = \sum_{s \neq v \neq t} \frac{\sigma_{st}(v)}{\sigma_{st}}$$

where $\sigma_{st}$ is the total number of shortest paths from node $s$ to node $t$, and $\sigma_{st}(v)$ is the number of those paths that pass through $v$. Values are normalised to $[0, 1]$ by dividing by $(n-1)(n-2)/2$. In the Friends network, a character with high betweenness acts as a **structural bridge** between otherwise disconnected social circles — removing them would most disrupt the flow of information across the network.

**PageRank**

PageRank assigns importance to a node based on how many important nodes link to it, propagating prestige recursively through the graph. Originally developed to rank web pages, it captures **recursive influence**: a character is important not just because they know many people, but because the people they know are themselves influential. The standard damping factor $\alpha = 0.85$ is used.

### What the notebook computes

- Betweenness and PageRank scores for all 747 nodes in the LCC
- A **table of the top 10 most central characters** for each measure
- **Horizontal bar plots** showing the rankings visually, with the six main characters colour-coded by role
- A **scatter plot** of betweenness vs PageRank for all nodes, making the structural separation between the main cast and guest characters immediately visible

### Results & interpretation

| Rank | Betweenness | PageRank |
|------|------------|----------|
| 1 | **Joey** (0.3238) | **Joey** (0.0806) |
| 2 | **Ross** (0.2718) | **Ross** (0.0705) |
| 3 | **Rachel** (0.2382) | **Chandler** (0.0668) |
| 4 | **Chandler** (0.2312) | **Rachel** (0.0642) |
| 5 | **Phoebe** (0.1907) | **Phoebe** (0.0572) |
| 6 | **Monica** | **Monica** |

Both metrics unanimously place the six main characters at the top, with an order-of-magnitude gap separating them from all 741 guest characters. This confirms the **hub-and-spoke topology** observed in Week 1: guest characters connect to the network almost exclusively through one or two main characters and have almost no direct links to each other.

The two measures capture meaningfully different notions of importance. Betweenness highlights **structural brokers**: Joey ranks first because his character straddles the broadest range of social worlds (the entertainment industry, romantic storylines, the central apartment group), making him the most frequent intermediary on shortest paths between unconnected guests. PageRank instead captures **recursive prestige**: the six main characters are densely interconnected among themselves, so they mutually reinforce each other's scores in a way no isolated guest character can replicate.

---

## Week 6 — Community Detection

**Goal:** Partition the network into groups of characters that interact more densely with each other than with the rest of the network — revealing the underlying social clusters of the show.

The graph is prepared as **undirected, unweighted, without self-loops**, and the analysis is restricted to the **Largest Connected Component**.

### What is a community in this context?

In a TV show network, a community corresponds to a **cluster of characters who predominantly appear in the same storylines or social world**. We would expect communities to reflect the distinct environments each main character inhabits — Joey's acting career, Ross's academic and family life, Rachel's fashion industry connections, Phoebe's massage clients and musical contacts, and so on. Community detection is therefore simultaneously a structural and a narrative analysis tool.

Partition quality is measured with **modularity** $Q \in (-1, 1)$:

$$Q = \frac{1}{2m} \sum_{u,v} \left[ A_{uv} - \frac{k_u k_v}{2m} \right] \delta(c_u, c_v)$$

where $A$ is the adjacency matrix, $k_u$ is the degree of node $u$, $m$ is the total number of edges, and $\delta(c_u, c_v) = 1$ only when $u$ and $v$ belong to the same community. Values of $Q > 0.3$ are generally considered indicative of meaningful community structure.

### Methods compared

**Greedy Modularity Optimisation** (`greedy_modularity_communities`)

An agglomerative algorithm that starts with each node in its own community and iteratively merges the pair of communities whose union produces the greatest gain in $Q$. Because it directly optimises the modularity objective, it tends to produce stable, well-balanced partitions and is well-suited to networks with genuine mesoscale structure.

**Label Propagation** (`label_propagation_communities`)

Each node is initialised with a unique label. In every iteration, each node adopts the most frequent label among its neighbours. The process repeats until convergence. The algorithm is fast and parameter-free but stochastic, and its behaviour can degrade on hub-dominated networks where popular nodes quickly spread their label across the graph.

### Results

| Method | Communities | Modularity $Q$ | Assessment |
|--------|:-----------:|:--------------:|------------|
| **Greedy Modularity** | **11** | **0.4038** | ✓ Strong, interpretable structure |
| Label Propagation | 21 | 0.0495 | ✗ Near-degenerate partition |

Greedy modularity significantly outperforms label propagation. Its 11 communities include six large clusters (109–138 nodes each), each anchored by **exactly one main cast member** — a result that maps directly onto the narrative structure of the show:

| Community | Main character | Storyline cluster |
|-----------|---------------|-------------------|
| C1 (138 nodes) | Joey | Acting career, co-stars, romantic partners |
| C2 (126 nodes) | Chandler | Office colleagues, Janice's circle |
| C3 (121 nodes) | Ross | University, ex-wives, academic contacts |
| C4 (120 nodes) | Monica | Chef career, culinary and family contacts |
| C5 (118 nodes) | Rachel | Fashion industry, family, Barry's circle |
| C6 (109 nodes) | Phoebe | Massage clients, musical contacts, spiritual world |

Label propagation fails on this network. Hub dominance causes the six main characters to propagate their labels to nearly all guest characters in the first iteration, collapsing 694 of 747 nodes into a single giant community and producing a near-zero modularity.

### What the notebook produces

- Community size distribution bar charts for both methods, displayed side by side
- A spring-layout network graph of the top-40 characters, coloured by community (greedy partition), with node size proportional to degree
- A modularity comparison table
- A detailed written interpretation linking each community to the show's narrative structure

### Gephi Lite export

The full graph is exported to **`friends_communities.gexf`** for interactive exploration in [Gephi Lite](https://gephi.org/gephi-lite/) (File → Open → select the file). Each node carries the following attributes for use in colouring, sizing and filtering:

| Attribute | Description |
|-----------|-------------|
| `community_greedy` | Integer community ID — Greedy Modularity partition |
| `community_lp` | Integer community ID — Label Propagation partition |
| `degree` | Node degree (recommended for node sizing) |
| `betweenness` | Normalised betweenness centrality (from Week 5) |
| `pagerank` | PageRank score (from Week 5) |
| `is_main_cast` | `1` if one of the six main characters, `0` otherwise |

---

## Week 7 — Link Prediction

**Goal:** Identify pairs of characters who have never directly interacted but are structurally the most likely candidates to do so — predicting missing or future edges in the network using topological similarity.

The graph is treated as **undirected, unweighted**, with self-loops removed and analysis restricted to the **Largest Connected Component**.

### Topological similarity indices

Link prediction rests on the intuition that two nodes with similar neighbourhoods are more likely to form a connection in the future. Two indices are computed for every non-existing pair $(u, v)$:

**Common Neighbors (CN)**

$$CN(u, v) = |\mathcal{N}(u) \cap \mathcal{N}(v)|$$

The simplest index: count the number of mutual acquaintances. Two characters who share many common contacts already move in the same social circle and are natural candidates for a future on-screen interaction.

**Adamic–Adar (AA)**

$$AA(u, v) = \sum_{w \,\in\, \mathcal{N}(u) \cap \mathcal{N}(v)} \frac{1}{\log k_w}$$

A weighted refinement of CN: each shared neighbour $w$ contributes $1/\log k_w$ instead of 1, **down-weighting shared neighbours who are popular hubs**. The intuition is that a mutual contact who knows everyone (e.g. Monica or Joey) provides weaker evidence of a meaningful link than a niche shared contact. AA is therefore more discriminating in hub-dominated networks like Friends, where virtually every pair of guest characters shares multiple main-cast neighbours.

### What the notebook computes

- All **277,021 non-existing edges** in the LCC are enumerated and evaluated
- CN and AA scores are computed for every candidate pair using the NetworkX generators `common_neighbor_centrality` and `adamic_adar_index`
- Both score columns are **min-max normalised** to $[0, 1]$ independently
- A **combined score** is computed as `score_sum = CN_norm + AA_norm`, giving equal weight to both indices
- The **top 10 most likely missing links** are tabulated and visualised for each of the three scores (CN, AA, score_sum)
- A **scatter plot** of normalised CN vs AA (for all pairs with CN > 0) shows the correlation between the two indices and highlights where the top predictions sit

### Results & interpretation

The strongest predicted missing link according to the combined score is **Rachel ↔ Susan** (score_sum = 2.000, the theoretical maximum), followed by **Joshua ↔ Joey** (1.91) and **Ross ↔ doctor_5_8** (1.90).

These predictions are structurally and narratively plausible. Rachel and Susan share over 81 common neighbours — the highest value observed — because both are deeply embedded in Ross's social world. Yet they are narratively positioned as antagonists (Susan is Carol's partner), which explains why they never appear in a direct friendly interaction despite their structural proximity. Joshua is Rachel's boyfriend for part of the season and shares many of her social contacts with Joey, yet the two characters never share a scene. These are **narratively motivated non-edges**: pairs whose social circles have already merged in the background of the show but who happen never to have had a direct on-screen encounter.

The divergence between CN and AA rankings is itself informative. Pairs that score high on CN but low on AA share their mutual contacts almost exclusively with the main-cast hubs — the shared context is broad but shallow. AA surfaces pairs where at least some shared contacts are less universally connected, making the structural overlap more exclusive and the prediction more credible. The combined score rewards agreement between both metrics and therefore identifies the most robust candidates.

---

## Authors

Course project submitted for the Social Network Analysis module.  
Dataset: Friends Season 1 interaction network.
