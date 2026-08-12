# Flavor Network Analysis

A network science project analysing the **Flavor Network**, a graph connecting ingredients that share chemical flavor compounds, to uncover structural patterns in how humans combine foods, using centrality analysis, small-world testing, community detection, robustness/attack simulation, and k-core role analysis.

This project was completed as part of a Network Analysis course project. It builds on the dataset and findings of Ahn et al.'s *"Flavor network and the principles of food pairing"* (Scientific Reports, 2011), extending the original paper into areas it did not cover.

---

## Project Overview

The task was to analyse the backbone of the flavor network, a graph where nodes are ingredients and edges represent shared flavor compounds, and combine it with real-world recipe data to study both its structure and its relationship to actual cooking practice.

The dataset, sourced from Zenodo, contained:

- **376 ingredient nodes** in the flavor network backbone
- **917 weighted edges** (shared flavor compounds)
- **57,320 recipes** across three cuisine datasets, used to measure real-world ingredient co-occurrence
- Ingredient category metadata (Dairy, Meats, Seafoods, Spices, etc.)

The project went beyond replicating the original paper's findings, focusing on four extensions:

- Community detection and its alignment with culinary food groups
- Network robustness and vulnerability to targeted attacks
- Ingredient role analysis (core vs. periphery via k-core decomposition)
- Discovery of "surprise pairings" — ingredients with high flavor similarity but low real-world usage together

---

## Key Results

| Property | Value |
|---|---:|
| Nodes | 376 |
| Edges | 917 |
| Average degree | 4.88 |
| Network density | 0.0130 |
| Average clustering coefficient | 0.3122 |
| Degree assortativity | -0.1563 (disassortative) |
| Small-world coefficient (σ) | **18.45** |

A small-world coefficient of σ ≈ 18.5 confirms the network is a **small world**: high clustering combined with short average path lengths (≈4.27), meaning most ingredients are only a few steps apart.

---

## Network Topology

### Degree Distribution & Hubs

The network follows a heavy-tailed, hub-dominated structure. The most connected ingredients were:

| Rank | Ingredient | Degree |
|---|---|---:|
| 1 | Roasted beef | 49 |
| 2 | Black tea | 47 |
| 3 | Strawberry | 42 |
| 4 | Beer | 33 |
| 5 | White wine | 25 |
| 6 | Apple | 25 |
| 7 | Blue cheese | 24 |

These hubs also dominated betweenness and closeness centrality, indicating they act as bridges connecting otherwise distant parts of the network.

### Assortativity

The network is **disassortative** (coefficient ≈ -0.156): high-degree hub ingredients tend to connect to low-degree, more specialized ingredients rather than to each other, a pattern common in biological and food networks.

---

## Small-World Test

The real network was compared against 10 Erdős–Rényi random graphs of matching size and density (restricted to the largest connected component: 362 nodes, 902 edges).

| Metric | Real Graph | Random Graph |
|---|---:|---:|
| Avg. clustering coefficient | 0.3104 | 0.0152 |
| Avg. shortest path length | 4.2685 | 3.8469 |

This gave a clustering ratio of ~20.5x and a path-length ratio of ~1.1x, producing **σ = 18.45**, confirming the small-world property.

---

## Community Detection

Three community detection algorithms were compared to test whether the network structure "rediscovers" traditional food categories.

| Method | Communities Found | Modularity |
|---|---:|---:|
| **Louvain** | 18 | **0.7834** |
| Label Propagation | 45 | 0.7713 |
| Girvan-Newman | 14 | 0.7187 |

Louvain gave the best balance of modularity and interpretability and was used for further analysis.

### Alignment with Culinary Logic

Cross-tabulating Louvain communities against ingredient category labels showed strong alignment with real food groups:

- **Community 5** — dominated by **Dairy** (71% of ingredients)
- **Community 11** — dominated by **Seafood** (93.3%)
- **Community 4** — dominated by **Fruits** (71.7%)

Other communities showed mixed pairings (e.g. vegetables and meats sharing a community), reflecting common culinary bases such as mirepoix or sofrito.

---

## Unexpected Pairings

An "unexpectedness" score was computed to find ingredient pairs with strong chemical similarity but rare real-world use together:

```text
unexpectedness = flavor_weight / (recipe_cooccurrence + 1)
```

Using 57,320 recipes across three datasets, 30,358 unique ingredient pairs and 508 candidate pairings (ingredients appearing in ≥10 recipes) were evaluated.

**Top unexpected pairings discovered:**

| Ingredient 1 | Ingredient 2 | Flavor Weight | Recipe Co-occurrence |
|---|---|---:|---:|
| Porcini | Enokidake / Matsutake | 117 | 0 |
| Sherry | Beer | 103 | 0 |
| Beer | Cognac | 99 | 0 |
| Guava | Strawberry | 83 | 0 |
| Whiskey | Cognac | 71 | 0 |

These pairs share a large number of flavor compounds but rarely appear together in recipes, suggesting untapped combinations for food pairing innovation.

---

## Network Robustness

Robustness was tested under two node-removal strategies: random failure and targeted "degree attack" (removing highest-degree hubs first), tracking the giant connected component fraction *S(f)* as a function of the fraction of nodes removed *f*.

| Removal Strategy | Fraction removed to break S below 0.5 | Nodes removed |
|---|---:|---:|
| Random failure | f ≈ 0.407 | ~153 of 376 |
| Degree attack (hubs first) | f ≈ 0.020 | ~8 of 376 |

- Removing the **top 10 highest-degree hubs** dropped the giant component to 44% of remaining nodes (160 of 366 nodes left in the largest component).
- Removing just **1 hub** (roasted beef) still left 84% of the network connected.
- The network is **robust to random noise** but **highly fragile to targeted attacks** — a classic scale-free network signature.

### Bridge Edges

Cutting the 10 edges with the highest betweenness centrality (e.g. *beer — roasted beef*, *black tea — soybean*, *apple — beer*) only reduced the giant component to 96% of the original cast, showing that while individual hub *nodes* are critical, no small set of edges alone controls global connectivity to the same degree.

---

## Ingredient Roles: Core vs. Periphery

K-core decomposition was used to classify ingredients by structural centrality:

| Role | Criterion | Count |
|---|---|---:|
| **Core** | Maximum coreness (k = 17) | 18 |
| **Mantle** | Coreness ≥ 50% of max | 25 |
| **Periphery** | Coreness < 50% of max | 333 |

**The Dairy Core:** The innermost core (k = 17) consists almost entirely of cheeses — Roquefort, Romano, Camembert, Cheddar, Parmesan, Gruyère, and others — reflecting how densely cheeses share flavor compounds with one another. This core aligned tightly with Louvain Community 5, which had a 58.1% core-ingredient share, far higher than any other community.

The remaining 88% of ingredients occupy the periphery, reflecting more specialized flavor profiles with fewer strong connections.

---

## Project Workflow

```text
Flavor Network Backbone (nodes + edges) + Recipe Datasets
   ↓
Graph Construction (NetworkX)
   ↓
Degree, Centrality & Assortativity Analysis
   ↓
Small-World Test (vs. Erdős–Rényi random graphs)
   ↓
Community Detection (Louvain, Label Propagation, Girvan-Newman)
   ↓
Community vs. Food Category Alignment
   ↓
Recipe Co-occurrence Analysis → Unexpected Pairings
   ↓
Network Robustness Simulation (random failure vs. degree attack)
   ↓
Bridge Edge (Betweenness) Analysis
   ↓
K-Core Decomposition → Ingredient Role Analysis
```

---

## Technologies Used

- Python
- NetworkX
- python-louvain (community detection)
- Pandas
- NumPy
- Matplotlib
- Seaborn
- Zenodo (dataset retrieval)

---

## Dataset

The flavor network backbone and recipe co-occurrence data were sourced from Zenodo (DOI: `10.5281/zenodo.11449658`), originally compiled for Ahn et al.'s flavor-pairing research. The dataset includes:

- `flavor_network_backbone_nodes.tsv` — ingredients with category and color metadata
- `flavor_network_backbone_edges.tsv` — weighted edges (shared compound counts)
- `ingr_comp.tsv`, `ingr_info.tsv`, `comp_info.tsv` — ingredient-to-compound mappings
- `allr_recipes.txt`, `epic_recipes.txt`, `menu_recipes.txt` — recipe datasets across cuisines, used for co-occurrence analysis

---

## What the Original Paper Did Not Cover

This project extended Ahn et al.'s original analysis into four areas:

1. Community detection
2. Network robustness and attack sensitivity
3. Ingredient role analysis (core vs. periphery)
4. Surprise pairings — ingredients highly connected by shared compounds but rarely used together

---

## Skills Demonstrated

This project demonstrates experience with:

- Graph construction and analysis with NetworkX
- Centrality measures (degree, betweenness, closeness)
- Small-world network testing against null models
- Community detection algorithm comparison and modularity evaluation
- Network robustness and percolation-style attack simulation
- K-core decomposition and structural role classification
- Real-world dataset integration (recipes + chemical compound network)
- Data visualization for network structure and statistics
- Scientific interpretation of network topology

---

## Future Improvements

Potential improvements include:

- Incorporating ingredient compound-level data (`ingr_comp.tsv`) for a weighted chemical similarity network beyond the pre-computed backbone
- Cuisine-specific sub-network analysis to compare flavor pairing patterns across regions
- Temporal or trend analysis of ingredient pairing adoption
- Validating "surprise pairings" through expert or sensory panel review
- Extending robustness analysis to weighted/targeted edge-removal attacks
- Applying graph neural networks for ingredient/category link prediction

---

## Authors

Lowie De Wever
Darija Avramoska
Nina Grozina
Sofiya Vorobyeva

---

## Project Status

Completed as part of a Network Analysis course project. Confirmed the flavor network's small-world property (σ ≈ 18.5), identified 18 Louvain communities that largely align with traditional food categories (modularity 0.78), showed high vulnerability to targeted hub attacks despite robustness to random failure, and surfaced a dairy-dominated structural core alongside several unexpected high-similarity ingredient pairings.

---

## License

This repository is intended for educational and portfolio purposes.
