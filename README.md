# SIREN: SIgnal-to-REgulon Network

SIREN links extracellular microenvironmental signals to chromatin-defined TF regulons in recipient cells through a **Ligand → Receptor → TF → Regulon** cascade, integrating CellPhoneDB (LR), NicheNet/CellCall/scMLnet (Receptor–TF), and FigR-derived regulons (TF–Target).

This repository provides the analysis code for the **medulloblastoma (MB)** example in our manuscript.

> ⚠️ **Data Availability**: Single-cell raw/processed data are **NOT** included in this repository.
> Raw FASTQ files are deposited in the Genome Sequence Archive (GSA) under **PRJCA068087**. Processed MuData/AnnData files are available from the authors upon reasonable request.

---

## 📥 Reference Database

The curated receptor–TF relationship database is **not** hosted in this repository. Please download it from the link below and place it into the `database/` folder before running the pipeline:

| File | Description | Download |
|------|-------------|----------|
| `RTF_human.csv` | Curated receptor–transcription factor (RTF) relationships | https://cloud.tsinghua.edu.cn/f/7008b7b3f0aa455ca60c/?dl=1 |

```bash
# After downloading, place the file as follows:
# SIREN/database/RTF_human.csv
```

The CellPhoneDB interaction reference file (`interaction_input.csv`), used to assign ligand/receptor directionality, is **not** provided here. Please obtain it directly from the [CellPhoneDB](https://github.com/ventolab/CellphoneDB) database distribution and place it in the `database/` folder.

---

## 📂 Repository Structure

```
SIREN/
├── README.md
├── environment.yml
├── database/                      # Reference databases (NOT included — see download above)
│   ├── RTF_human.csv              # Curated receptor–TF relationships (download)
│   └── interaction_input.csv      # CellPhoneDB interaction reference (from CellPhoneDB)
├── notebooks/
│   ├── T00_Data_pre.ipynb         # Data preprocessing: build CellPhoneDB inputs
│   ├── T01_Method2.ipynb          # CellPhoneDB statistical analysis
│   ├── T02_Analysis.ipynb         # CCI exploration and plotting
│   └── T03_SIREN.ipynb            # SIREN signal-to-regulon inference (core)
└── figures/                       # Output figures
```

---

## ⚙️ Installation

> **🚨 Crucial Note on CellPhoneDB & Python Version:**
> This pipeline strictly requires **Python 3.8** and a specific version of **CellPhoneDB (v4.1.0)**.
> The downstream visualization library (`ktplotspy`) parses CellPhoneDB's output files (`pvalues.txt`, `means.txt`). Using newer versions of CellPhoneDB or Python 3.9+ will cause parsing errors or dependency conflicts (e.g., with `pandas` and `anndata`).
> You must pre-download the CellPhoneDB v4.1.0 database zip file locally and update the `cpdb_file_path` variable in `T01_Method2.ipynb`.

```bash
git clone https://github.com/zhangqf-lab/SIREN.git
cd SIREN

# 1. Create a Python 3.8 environment
conda create -n siren python=3.8.5 -y
conda activate siren

# 2. Install CellPhoneDB (v4.1.0)
pip install cellphonedb==4.1.0

# 3. Install core dependencies
pip install scanpy==1.9.8 anndata==0.9.2 muon mudata
pip install ktplotspy statsmodels seaborn plotnine scipy
```

---

## 🔧 Workflow

**Step 0 — Preprocessing** (`T00_Data_pre.ipynb`)
Annotate MB subtypes (Group3, Group4, SHH) and TME subtypes, merge metadata, and export CellPhoneDB inputs: `data/metadata.tsv` and `data/normalised_log_counts.h5ad`.

**Step 1 — CellPhoneDB statistical analysis** (`T01_Method2.ipynb`)
Run 1000 permutations to identify significant Ligand–Receptor interactions between tumor and TME cells.

**Step 2 — CCI exploration & visualization** (`T02_Analysis.ipynb`)
Query specific interactions and generate heatmaps/dot plots using `ktplotspy`.

**Step 3 — SIREN inference** (`T03_SIREN.ipynb`, core)
1. Assign ligand/receptor directionality using `database/interaction_input.csv`.
2. Map receptors to TFs via `database/RTF_human.csv` (RTF edges).
3. Map TFs to targets via FigR associations (score > 1) (TFT edges).
4. Compute Pearson correlation between receptor expression and TF regulon (AUCell) activity across single cells; apply BH-FDR correction.
5. Retain significant signaling-to-regulon modules (`r > 0.05`, FDR < 0.05).

---

## 📄 Key Outputs

| File | Content |
|------|---------|
| `results/LR_results.csv` | Directionally annotated LR interactions |
| `results/RTF_{Group}.csv` / `TTF_{Group}.csv` | Significant RTF / TFT edges |
| `results/sig_pathway_{Group}.csv` | Receptor → regulon modules (`r`, `p_corr_adj`, `regulon_score`) |
| `results/sig_pathway_group.csv` | Subtype-specific modules |

---

## 📝 Citation

If you use SIREN or this code, please cite our preprint:

**A comparative single-cell multi-omic atlas links intrinsic regulatory programs to microenvironmental signaling in pediatric brain tumors**
*Yuzhe Li, Jinsong Zhang, Yifan Wei, Kaiyu Fan, Ruiyun Yang, Xu Han, Yanqiu Shao, Lei Tang, Weiwei Zhai, Wenze Huang, Jian Gong, Qiangfeng Cliff Zhang*
**Lang Tao Sha (Preprint)**
🔗 [https://langtaosha.org.cn/lts/en/preprint/view/351](https://langtaosha.org.cn/lts/en/preprint/view/351)

```bibtex
@article{li2026siren,
  title={A comparative single-cell multi-omic atlas links intrinsic regulatory programs to microenvironmental signaling in pediatric brain tumors},
  author={Li, Yuzhe and Zhang, Jinsong and Wei, Yifan and Fan, Kaiyu and Yang, Ruiyun and Han, Xu and Shao, Yanqiu and Tang, Lei and Zhai, Weiwei and Huang, Wenze and Gong, Jian and Zhang, Qiangfeng Cliff},
  journal={Lang Tao Sha Preprint},
  year={2026},
  url={https://langtaosha.org.cn/lts/en/preprint/view/351}
}
```

---

## 📮 Contact

- **Qiangfeng Cliff Zhang** — qczhang@tsinghua.edu.cn
- **Jian Gong** — gongjian88@vip.163.com
- **Wenze Huang** — hwz16@tsinghua.org.cn