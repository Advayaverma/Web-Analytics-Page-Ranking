# Web Analytics: PageRank on Wikipedia Link Graph (100K Pages)

A university final-year computer science project implementing the **PageRank algorithm from scratch** using power iteration on the **Wikipedia Link Graph Dataset (100,000 Pages)**. The implementation features sparse linear algebra, rigorous dangling node redistribution, NetworkX validation, damping factor sensitivity experiments, and network visualizations.

---

## 🚀 Key Results at a Glance

- **Graph Scale**: 100,000 nodes, 8,572,718 clean directed edges, 205 dangling nodes.
- **Convergence**: Achieved $L_1$ tolerance of $10^{-6}$ in **42 iterations** ($d=0.85$).
- **Computational Speed**: **0.422 seconds** (~10 ms per iteration).
- **Validation**: **17.11× faster than NetworkX** (`nx.pagerank`), with **100% top-20 overlap** and scores matching to **floating-point machine precision ($1.73 \times 10^{-18}$)** under strict tolerance.
- **Top Authority Page**: *ISBN (identifier)* with PageRank score $9.806 \times 10^{-3}$ and 51,534 incoming links.

---

## 📁 Repository Structure

```text
Web-Analytics-Page-Ranking/
├── pagerank_wikipedia.ipynb   # Complete executed Jupyter notebook with all sections & plots
├── report.md                  # Comprehensive formal project report with empirical findings
├── viva_notes.md              # Conceptual viva examination preparation notes (5 key questions)
├── requirements.txt           # Python dependency specifications
├── top20_pagerank.csv         # Top 20 pages ranking table (rank, title, score, in/out degrees)
├── all_pagerank.csv           # Complete PageRank scores for all 100,000 pages
├── .gitignore                 # Git ignore rules (ignores raw dataset, checkpoints, cache)
├── data/                      # (Ignored by git) Wikipedia dataset files:
│   ├── pages_export.csv       # 100,000 page metadata records (71.5 MB)
│   ├── links_export.csv       # 28.85 million raw directed hyperlink records (4.88 GB)
│   ├── graph.json             # Graph representation in JSON (2.71 GB)
│   └── clean_edges.npz        # Compressed clean edge cache (18.0 MB, loads in 0.2s)
└── outputs/                   # Generated high-resolution visualization figures:
    ├── convergence.png        # Semi-log L1 error decay curve across 42 iterations
    ├── pagerank_histogram.png # Log-log score distribution (power-law behavior)
    ├── pagerank_vs_indegree.png # Log-log correlation scatter plot (r = 0.884, rho = 0.903)
    └── top50_subgraph.png     # Induced network graph of top 50 pages
```

---

## 🛠️ Tech Stack & Requirements

- **Python**: 3.10+ (tested on Python 3.14)
- **Core Libraries**:
  - `pandas` $\ge$ 2.0.0
  - `numpy` $\ge$ 1.24.0
  - `scipy` $\ge$ 1.10.0 (CSR sparse matrix routines)
  - `networkx` $\ge$ 3.0 (validation benchmark)
  - `matplotlib` $\ge$ 3.7.0 (visualizations)
  - `jupyter` / `nbconvert` / `ipykernel`
  - `kaggle` $\ge$ 1.5.16 (dataset downloading)

---

## ⚙️ Installation and Setup

### 1. Clone the Repository
```bash
git clone https://github.com/Advayaverma/Web-Analytics-Page-Ranking.git
cd Web-Analytics-Page-Ranking
```

### 2. Install Dependencies
```bash
pip install -r requirements.txt
```

### 3. Configure Kaggle API Credentials (Optional if dataset already downloaded)
To download the dataset automatically, ensure your `kaggle.json` API token is placed at:
- **Windows**: `C:\Users\<Username>\.kaggle\kaggle.json`
- **Linux/macOS**: `~/.kaggle/kaggle.json`

*(To generate a token: Kaggle.com $\to$ Profile Settings $\to$ API $\to$ Create New Token).*

### 4. Download Dataset (If not already present)
```bash
kaggle datasets download -d kutayahin/wikipedia-link-graph-100k -p ./data/ --unzip
```

---

## 🏃 How to Run

### Option A: Interactive Jupyter Notebook
Launch the Jupyter Notebook server and run all cells sequentially:
```bash
jupyter notebook pagerank_wikipedia.ipynb
```

### Option B: Headless Command-Line Execution
Run the entire notebook end-to-end and re-render all outputs in place:
```bash
jupyter nbconvert --to notebook --execute --inplace pagerank_wikipedia.ipynb
```

---

## 📊 Summary of Key Findings

1. **Memory Feasibility**: Storing a dense $100,000 \times 100,000$ matrix requires **80 GB of RAM**. Constructing a `scipy.sparse.csr_matrix` uses only **98.49 MB**, achieving an **~800× reduction** in memory.
2. **Speed**: Our from-scratch power iteration converges in **0.422 seconds**, running **17.11× faster** than NetworkX (`7.220 seconds`) on the same hardware.
3. **Accuracy**: When evaluated under strict matching tolerance ($L_1 < 10^{-6}$), the difference between our implementation and NetworkX is on the order of **$10^{-18}$** (floating-point machine precision) with **100% top-20 overlap**.
4. **Authority over Volume**: *International Organization for Standardization* ranks #16 globally despite only 428 in-links, because those links originate from high-PageRank authority pages (*ISBN* and *ISSN*).
5. **Damping Factor Trade-off**: As $d$ increases from 0.50 to 0.99, convergence iterations increase from 14 to 114. The canonical choice $d = 0.85$ (42 iterations) offers the optimal balance between computational efficiency and path depth.

---

## 📖 References
- **Brin, S. and Page, L. (1998)**. *The anatomy of a large-scale hypertextual Web search engine*. Computer Networks and ISDN Systems, 30(1-7), pp. 107–117.
- **Dataset**: [Wikipedia Link Graph Dataset, 100K Pages (Kaggle)](https://www.kaggle.com/datasets/kutayahin/wikipedia-link-graph-100k)
- **Reference Code Sample**: [Kaggle PageRank Algorithm](https://www.kaggle.com/code/vinitkp/page-rank-algorithm)
- **Langville, A. N. and Meyer, C. D. (2006)**. *Google's PageRank and Beyond: The Science of Search Engine Rankings*. Princeton University Press.
