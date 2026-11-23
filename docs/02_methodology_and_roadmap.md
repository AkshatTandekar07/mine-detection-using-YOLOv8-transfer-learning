# Methodology and Lifecycle Execution

Execution framework under CRISP-DM and BADIR methodologies for sonar target detection.

---

## 1. CRISP-DM Lifecycle Summary

| Lifecycle Phase | Scope & Strategy | Final Execution Evidence |
| :--- | :--- | :--- |
| **1. Business Understanding** | Frame naval MCM operational requirements: prioritize MILCO recall (>75%), one-to-one IoU $\ge 0.50$ matching, and track false alarm burden. | Defined in [Project Charter](01_project_charter.md) and [`Machine Learning Project Synopsis.docx`](Machine%20Learning%20Project%20Synopsis.docx). |
| **2. Data Understanding** | Inspect class mapping, bounding box geometry, empty-label distributions, and acoustic highlight-shadow properties. | Executed in `src/v02.ipynb` (Cells 6–26). Identified survey-track temporal autocorrelation. |
| **3. Data Preparation** | Verify label integrity, maintain non-destructive annotation loading, and stage clean dataset YAML manifests. | Tracked via `dataset/mine.yaml`. Source files preserved intact without clipping. |
| **4. Modeling** | Fine-tune pretrained YOLOv8s across resolution regimes: 960px baseline (v01) and 1280px high-resolution comparison (v02). | Completed 100 epochs per run with SGD optimizer (`seed=42`). Artifacts preserved in `reports/`. |
| **5. Evaluation** | Systematic threshold sweep across validation set; freeze parameters before test evaluation; generate visual comparisons. | Selected `conf=0.05`. Baseline achieved **79.66% MILCO recall**. 3-sample test visual comparisons generated. |
| **6. Deployment & Packaging** | Clean intermediate files, export release weights, sanitize documentation, and publish GitHub repository. | Checkpoints and dataset hosted on GitHub Releases `v1.0.0`; clean repo state prepared. |

---

## 2. BADIR Analytical Framework

- **Business Question:** Can transfer-learned YOLOv8s achieve >75% MILCO recall in side-scan sonar imagery while keeping false alarms operationally bounded?
- **Analysis Plan:** Train baseline detector at 960px (v01), conduct controlled high-resolution comparison at 1280px (v02), sweep confidence thresholds on validation data, and evaluate false alarms per image.
- **Data Collection:** 1,170 annotated Gavia AUV sonar images across 819 train, 176 validation, and 175 test frames.
- **Insights:** The 960px baseline achieved 79.66% recall with 0.181 FPPI. Scaling to 1280px degraded recall to 76.27% and doubled false alarms (0.390 FPPI) due to acoustic speckle amplification.
- **Recommendations:** Deploy 960px baseline at `conf = 0.05` within a two-tier MCM triage architecture.

---

## 3. Directory & Code Architecture

```
.
├── dataset/
│   └── mine.yaml             # Dataset paths and class mapping (0: MILCO, 1: NonMILCO)
├── docs/
│   ├── 01_project_charter.md          # System goal, 5W1H, and audited distributions
│   ├── 02_methodology_and_roadmap.md  # CRISP-DM / BADIR lifecycle summary
│   ├── 03_experiment_protocol.md      # Evaluation framework and resolution benchmark
│   └── Machine Learning Project Synopsis.docx  # Comprehensive technical project document
├── reports/
│   ├── v01/                  # Baseline (960px): evaluation metrics, grid sweep, visual samples
│   └── v02/                  # High-res (1280px): metrics and comparative evaluation
├── src/
│   ├── v01.ipynb             # Executable 960px baseline notebook
│   └── v02.ipynb             # Executable 1280px notebook & comprehensive EDA
├── pyproject.toml            # Dependencies and virtual environment configuration
└── README.md                 # Project overview, benchmark tables, and release links
```
