# Arquitectura de Influencia en Ecosistemas de Consumo

Analysis of Amazon product reviews combining **principal component analysis**, **social network analysis** and **link-based classification**, written as a reproducible [Quarto](https://quarto.org) book in R.

Course project for *Modelización de Computación Predictiva* (Universidad de Málaga). The report itself is written in Spanish.

## Research question

Do the *attributes* of a product (how it is rated and reviewed) and its *position in the market* (which other products its buyers also review) describe the same thing? The project segments products from both angles and measures how much the two segmentations agree.

## Pipeline

| Chapter | File | What it does |
|---|---|---|
| Introducción | `index.qmd` | Context, objectives and variable definitions. |
| Preprocesado | `preprocesado.qmd` | Samples ~50,000 reviews from products with at least 4 reviews, derives per-product metrics (helpfulness ratio, lifespan, review length, AFINN polarity, rating entropy) and builds the co-review edge list. |
| Bloque I: PCA | `pca.qmd` | Bartlett and KMO adequacy tests, IQR winsorisation, component retention (scree plot, Kaiser) and k-means clustering on the first three components. |
| Bloque II: SNA | `sna.qmd` | Product co-review graph (with a sensitivity analysis of the activity threshold), centralities (degree, betweenness, PageRank, closeness, coreness), Louvain vs Walktrap communities, contracted and k-core visualisations. |
| Bloque III: Enlaces | `enlaces.qmd` | Homophily of the PCA clusters over the graph (nominal assortativity and a neighbour majority-vote classifier) and link prediction with Common Neighbors, Jaccard and Adamic-Adar, evaluated by AUC-ROC. |
| Conclusiones | `conclusiones.qmd` | Cross-validates the blocks (Adjusted Rand Index, Kruskal-Wallis) and summarises the findings. |
| Anexo | `prompts.qmd` | Log of how AI assistance was used, and which of its suggestions were rejected and why. |

Each chapter writes its results to `.rds` files that later chapters read, so the chapters must run in book order.

### Graph definition

Nodes are products. Only *active* users, those who reviewed at least N = 5 distinct products, create edges: every pair of products reviewed by the same active user is linked, and the edge weight is the number of active users they share. Isolated nodes are removed and the analysis keeps the giant component.

## Main results

- **PCA**: after excluding the two variables with KMO < 0.5 (`total_resenas`, `lifespan`), three components retain 79.2 % of the variance: perceived quality (PC1), review elaboration (PC2) and helpfulness (PC3).
- **SNA**: the giant component has 993 products and 8,583 edges. Centrality distributions are heavy-tailed, and Louvain beats Walktrap on modularity.
- **Homophily**: assortativity of the PCA clusters over the graph is close to zero (0.033), and the neighbour classifier barely beats a proportional baseline (0.39 vs 0.34 accuracy).
- **Link prediction**: all three topological scores reach an AUC above 0.97, with Adamic-Adar the best (0.99).
- **Cross-validation**: the Adjusted Rand Index between PCA clusters and Louvain communities is about 0. The attributive and relational segmentations are essentially orthogonal, although the PCA cluster does shift the distribution of centrality (Kruskal-Wallis, p < 0.001). The most central products tend to be the ones with long reviews of varied tone.

## Reproducing the book

### 1. Data

Download `Reviews.csv` from the [Amazon Fine Food Reviews](https://www.kaggle.com/datasets/snap/amazon-fine-food-reviews) dataset on Kaggle (~290 MB, 568,454 reviews) and place it in the project root. It is not tracked in git because of its size.

### 2. R packages

Tested with R 4.3.2.

```r
install.packages(c(
  "tidyverse", "data.table", "tidytext", "skimr", "syuzhet",
  "factoextra", "corrplot", "psych",
  "igraph", "tidygraph", "ggraph", "RColorBrewer", "cowplot",
  "knitr", "kableExtra", "pROC", "aricode"
))
```

### 3. Render

```bash
quarto render
```

The HTML book is written to `_book/`. Execution results are cached in `_freeze/` (committed), so the book renders from the cache without the raw data. A chapter re-executes only when its source changes, and that requires the `.rds` files produced by the earlier chapters. To regenerate everything from scratch, delete `_freeze/` and render again with `Reviews.csv` in place.

All random steps use `set.seed(230205)`.

## Repository layout

```
├── _quarto.yml            # Book configuration and chapter order
├── index.qmd              # Introduction
├── preprocesado.qmd       # Sampling, metrics, edge list
├── pca.qmd                # Block I
├── sna.qmd                # Block II
├── enlaces.qmd            # Block III
├── conclusiones.qmd       # Cross-validation and conclusions
├── prompts.qmd            # AI usage annex
└── _freeze/               # Cached execution results
```

## Author

William Lu Bjornestad
