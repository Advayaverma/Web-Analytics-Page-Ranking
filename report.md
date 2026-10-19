# Web Analytics: PageRank on the Wikipedia Link Graph (100K Pages)

**Author:** CS Final-Year Student  
**Project:** Web Analytics: Page Ranking (PageRank Algorithm from Scratch)  
**Date:** October 2026  
**Repository:** [Web-Analytics-Page-Ranking](https://github.com/Advayaverma/Web-Analytics-Page-Ranking)  

---

## 1. Introduction
The World Wide Web is naturally modeled as a directed graph where documents represent vertices and hyperlinks represent directed edges. Ranking documents in such large-scale networks presents a fundamental challenge: lexical search alone is easily manipulated by keyword stuffing and fails to capture structural importance. In 1998, Sergey Brin and Larry Page introduced **PageRank**, an algorithm that models web navigation as a Markov chain (the *random surfer model*) to compute an objective measure of structural prestige.

This project delivers an end-to-end implementation of the **PageRank algorithm from scratch** using sparse linear algebra in Python. The implementation is executed on the **Wikipedia Link Graph Dataset (100,000 Pages)**, validated against NetworkX, analyzed across multiple structural and parameter dimensions, and benchmarked for computational efficiency.

---

## 2. Dataset
The project utilizes the **Wikipedia Link Graph Dataset (100K Pages)**, collected via Breadth-First Search (BFS) crawling up to depth 5 on English Wikipedia.

### Raw Data Files (`./data/`):
- **`pages_export.csv`** (71.54 MB): Catalog of 100,000 unique Wikipedia articles containing page `id`, `title`, word counts, and canonical URLs.
- **`links_export.csv`** (4,886.73 MB): 28,855,738 raw directed hyperlink records (`source_title` $\to$ `target_title`).
- **`graph.json`** (2,717.78 MB): JSON network representation of nodes and edges.

### Cleaning and Graph Statistics:
Outbound hyperlinks were filtered to retain internal edges within the 100,000-page catalog. Self-loops ($src = dst$) and duplicate directed edges were eliminated.

| Metric | Measured Value |
|:---|:---|
| **Total Nodes ($N$)** | 100,000 |
| **Clean Directed Edges ($E$)** | 8,572,718 |
| **Average Out-Degree** | 85.73 links/page |
| **Maximum In-Degree** | 51,534 links (*ISBN (identifier)*) |
| **Dangling Nodes ($C_j = 0$)** | 205 nodes (0.21%) |
| **Directed Graph Density** | $8.572804 \times 10^{-4}$ (0.0857%) |

---

## 3. Methodology

### 3.1 Random Surfer Model
The algorithm models a user browsing the web:
- With probability $d \in (0, 1)$ (the **damping factor**, typically 0.85), the surfer clicks uniformly at random on one of the outbound links available on the current page.
- With probability $1 - d$, the surfer teleports uniformly at random to any page in the entire graph.

### 3.2 Transition Matrix and Formula
Let $N = 100,000$. The column-substochastic transition matrix $M \in \mathbb{R}^{N \times N}$ is defined by:
$$
M_{ij} = \begin{cases}
\frac{1}{C_j} & \text{if } (j \to i) \in E \\
0 & \text{otherwise}
\end{cases}
$$
where $C_j = \text{out-degree}(j)$.

### 3.3 Handling Dangling Nodes
Nodes with out-degree zero ($C_j = 0$, dangling nodes) act as probability sinks. If untreated, probability mass leaks out, and the total rank converges to 0. At iteration $k$, the total trapped probability mass is:
$$
\text{dangling\_mass}^{(k)} = \sum_{j \in \mathcal{D}} PR_j^{(k)}, \quad \mathcal{D} = \{j \mid C_j = 0\}
$$
This mass is dynamically redistributed uniformly across all $N$ nodes.

### 3.4 Power Iteration Update Equation
Combining hyperlink transitions, dangling node redistribution, and random teleportation yields the exact per-iteration vector update:
$$
PR^{(k+1)} = d \left( M \cdot PR^{(k)} + \frac{\text{dangling\_mass}^{(k)}}{N} \mathbf{1} \right) + \frac{1-d}{N} \mathbf{1}
$$
Factoring uniform terms for optimal vectorized computation:
$$
\text{teleport}^{(k)} = \frac{(1-d) + d \cdot \text{dangling\_mass}^{(k)}}{N}
$$
$$
PR^{(k+1)} = d \cdot (M \cdot PR^{(k)}) + \text{teleport}^{(k)} \mathbf{1}
$$

### 3.5 Convergence Guarantee
By the **Perron-Frobenius Theorem**, adding the rank-one teleportation perturbation makes the transition matrix primitive (irreducible and aperiodic). The spectral gap is $1 - d = 0.15$, guaranteeing that the power iteration converges to a unique stationary distribution with contraction factor bounded by $d^k = 0.85^k$. Convergence is declared when:
$$
\| PR^{(k+1)} - PR^{(k)} \|_1 = \sum_{i=1}^N |PR_i^{(k+1)} - PR_i^{(k)}| < 10^{-6}
$$

---

## 4. Implementation
The solution is implemented using Python 3, `scipy.sparse`, and `numpy`:
- **Sparse CSR Matrix**: A dense matrix of dimension $100,000 \times 100,000$ requires $10^{10} \times 8 \text{ bytes} \approx 80\text{ GB}$ of RAM. In contrast, `scipy.sparse.csr_matrix` stores only the 8,572,718 non-zero entries, requiring just **98.49 MB** of RAM (~800× reduction).
- **Fast Vectorized Iteration**: Matrix-vector multiplication is computed using C-level CSR routines (`M.dot(PR)`), taking ~10 ms per iteration.
- **Caching**: The cleaned edge list is cached in `data/clean_edges.npz` (18.03 MB), reducing edge loading time from 78 seconds to under 0.25 seconds.

---

## 5. Results

### 5.1 Power Iteration Execution
- **Iterations to Convergence**: **42 iterations**
- **Final $L_1$ Error**: $8.759634 \times 10^{-7}$ (below $1.00 \times 10^{-6}$ tolerance)
- **Total Runtime**: **0.422 seconds** (~10.05 ms/iteration)
- **Score Conservation**: $\sum_{i=1}^N PR_i = 1.0000000000$ (error $< 10^{-12}$)
- **Score Range**: Minimum $1.5074 \times 10^{-6}$, Maximum $9.8064 \times 10^{-3}$

### 5.2 Top-20 Pages Ranking (`top20_pagerank.csv`)

| Rank | Page Title | PageRank Score | In-Degree | Out-Degree |
|:---:|:---|:---:|:---:|:---:|
| 1 | ISBN (identifier) | $9.806377 \times 10^{-3}$ | 51,534 | 185 |
| 2 | Doi (identifier) | $6.265567 \times 10^{-3}$ | 31,461 | 152 |
| 3 | ISSN (identifier) | $5.016162 \times 10^{-3}$ | 23,311 | 139 |
| 4 | Wayback Machine | $4.715811 \times 10^{-3}$ | 24,295 | 67 |
| 5 | S2CID (identifier) | $3.285555 \times 10^{-3}$ | 17,819 | 34 |
| 6 | OCLC (identifier) | $2.310606 \times 10^{-3}$ | 11,887 | 26 |
| 7 | PMID (identifier) | $2.257968 \times 10^{-3}$ | 9,911 | 36 |
| 8 | Bibcode (identifier) | $2.046279 \times 10^{-3}$ | 12,396 | 7 |
| 9 | JSTOR (identifier) | $1.815155 \times 10^{-3}$ | 11,253 | 26 |
| 10 | United States | $1.673477 \times 10^{-3}$ | 9,016 | 381 |
| 11 | The New York Times | $1.621103 \times 10^{-3}$ | 8,003 | 227 |
| 12 | PMC (identifier) | $1.470765 \times 10^{-3}$ | 6,368 | 39 |
| 13 | Time zone | $1.293277 \times 10^{-3}$ | 9,293 | 331 |
| 14 | Hdl (identifier) | $1.110218 \times 10^{-3}$ | 5,974 | 14 |
| 15 | World War II | $1.090814 \times 10^{-3}$ | 6,186 | 389 |
| 16 | International Organization for Standardization | $1.039938 \times 10^{-3}$ | 428 | 150 |
| 17 | United Kingdom | $1.037448 \times 10^{-3}$ | 6,053 | 564 |
| 18 | The Guardian | $9.530512 \times 10^{-4}$ | 5,298 | 174 |
| 19 | Daylight saving time | $9.362922 \times 10^{-4}$ | 6,899 | 118 |
| 20 | France | $8.878654 \times 10^{-4}$ | 6,089 | 545 |

### 5.3 Correlation Analysis: PageRank vs. In-Degree
- **Pearson Correlation ($r$)**: **0.8841**
- **Spearman Rank Correlation ($\rho$)**: **0.9025**
- **Key Insight**: While high in-degree strongly correlates with high PageRank, significant vertical dispersion exists (up to 2 orders of magnitude). Most notably, **'International Organization for Standardization'** ranks #16 globally despite having only **428 incoming links**, because its in-links originate from top-authority pages like *ISBN* and *ISSN*.

---

## 6. Validation against NetworkX

| Metric | From-Scratch Implementation | NetworkX (`nx.pagerank`) |
|:---|:---:|:---:|
| **Runtime** | **0.422 seconds** | **7.220 seconds** (17.11× slower) |
| **Max Absolute Difference** | Baseline | $4.259644 \times 10^{-4}$ (default tol) |
| **Mean Absolute Difference** | Baseline | $1.319186 \times 10^{-6}$ |
| **Top-20 Overlap** | 20 / 20 | 19 / 20 (95.0%) (default tol) |
| **Top-20 Overlap (Strict $L_1 < 10^{-6}$)** | **20 / 20** | **20 / 20 (100.0%)** |
| **Max Abs Diff (Strict $L_1 < 10^{-6}$)** | Baseline | **$1.7347 \times 10^{-18}$** (machine precision) |

### Algorithmic Nuance:
NetworkX's internal convergence condition checks `err < N * tol`. For $N = 100,000$, NetworkX halts early when total error is $< 0.1$. When NetworkX is set to match the true mathematical $L_1 < 10^{-6}$ stopping condition, both implementations yield **identical scores down to floating-point machine precision ($10^{-18}$)** with **100% top-20 overlap**. Our custom CSR solver runs **17.11× faster** by bypassing Python dictionary allocations.

---

## 7. Damping Factor Analysis
We tested $d \in [0.50, 0.70, 0.85, 0.95, 0.99]$ with $\text{tol} = 10^{-6}$:

| Damping Factor ($d$) | Iterations to Converge | Top Page Title | Top PageRank Score | Runtime (s) |
|:---:|:---:|:---|:---:|:---:|
| **0.50** | 14 | ISBN (identifier) | $7.621252 \times 10^{-3}$ | 0.142 |
| **0.70** | 24 | ISBN (identifier) | $9.246473 \times 10^{-3}$ | 0.244 |
| **0.85** | 42 | ISBN (identifier) | $9.806377 \times 10^{-3}$ | 0.422 |
| **0.95** | 76 | ISBN (identifier) | $9.693121 \times 10^{-3}$ | 0.771 |
| **0.99** | 114 | ISBN (identifier) | $9.412085 \times 10^{-3}$ | 1.158 |

### Findings:
1. **Convergence Scaling**: Iteration count scales inversely with the spectral gap $(1 - d)$. As $d \to 1$, convergence slows significantly (from 14 iterations at $d=0.5$ to 114 iterations at $d=0.99$).
2. **Authority Propagation**: Lower $d$ values dilute link prestige by forcing surfers to teleport frequently. Higher $d$ values allow authority to circulate through deeper paths. The canonical $d = 0.85$ provides the optimal trade-off between structural fidelity and computational efficiency.

---

## 8. Conclusion, Limitations, and Future Work

### Conclusion:
This project demonstrated that PageRank can be efficiently computed from scratch on real-world graphs containing 100,000 nodes and 8.57 million edges in under half a second. The implementation matches NetworkX to machine precision while outperforming it by over 17× in computational speed. The structural findings reveal that Wikipedia's link graph is heavily organized around bibliographic citation templates and geopolitical entities.

### Limitations:
1. **Sub-network Boundary**: The graph is restricted to the 100,000 crawled pages; hyperlinks pointing to uncrawled Wikipedia articles were pruned, creating synthetic dangling nodes.
2. **Static Link Weighting**: All outbound links on a page are treated as equally likely ($\frac{1}{C_j}$), ignoring link location, anchor text relevance, or user traffic patterns.

### Future Work:
1. **Topic-Sensitive PageRank**: Biasing the teleportation vector toward specific categories (e.g., science, history) to produce personalized rankings.
2. **TrustRank / Anti-Spam Extensions**: Teleporting only to human-verified authoritative seed pages to resist link-farming.
3. **Distributed Implementation**: Adapting the CSR matrix multiplication to Apache Spark (GraphX) or GPU acceleration (CuPy/PyTorch) for web-scale graphs exceeding billions of vertices.

---

## 9. References
1. **Brin, S. and Page, L. (1998)**. *The anatomy of a large-scale hypertextual Web search engine*. Computer Networks and ISDN Systems, 30(1-7), pp. 107–117.
2. **Wikipedia Link Graph Dataset (100K Pages)**: [https://www.kaggle.com/datasets/kutayahin/wikipedia-link-graph-100k](https://www.kaggle.com/datasets/kutayahin/wikipedia-link-graph-100k)
3. **PageRank Reference Approach**: [https://www.kaggle.com/code/vinitkp/page-rank-algorithm](https://www.kaggle.com/code/vinitkp/page-rank-algorithm)
4. **Langville, A. N. and Meyer, C. D. (2006)**. *Google's PageRank and Beyond: The Science of Search Engine Rankings*. Princeton University Press.
