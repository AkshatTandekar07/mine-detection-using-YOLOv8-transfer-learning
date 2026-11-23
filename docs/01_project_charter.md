# Project Charter, 5W1H, and Data Audit

Technical specification and verified baseline audit for automated underwater mine detection.

---

## 1. System Goal & Target Classes

Given a side-scan sonar (SSS) image, automatically detect, locate, and classify contacts into:
- **MILCO (Class 0):** Mine-Like Contact requiring operational verification.
- **NonMILCO (Class 1):** Non-hazardous seabed clutter or debris.
- **Empty Seabed:** Preserved as-is with "No objects detected".

Labels are predicted autonomously without human-in-the-loop dependencies during inference.

---

## 2. Five Ws and How (5W1H)

| Dimension | Specification |
| :--- | :--- |
| **What?** | Autonomous detection and localization of MILCO and NonMILCO targets in side-scan sonar imagery. |
| **Why?** | Naval Mine Countermeasures (MCM) demand high recall: minimizing missed mines (False Negatives) is critical to fleet safety. |
| **Who?** | Project engineering team; runtime inference is fully autonomous. |
| **Where?** | Developed in PyTorch with Ultralytics YOLOv8. Hosted at [GitHub Repository](https://github.com/AkshatTandekar07/mine-detection-using-YOLOv8-transfer-learning). |
| **When?** | Completed baseline training (v01), high-resolution comparison (v02), model cleanup, and release packaging (v1.0.0). |
| **How?** | Transfer learning from COCO-pretrained YOLOv8s weights (349/355 layers transferred), fine-tuned for 100 epochs with SGD. |

---

## 3. Repository Structure & Deliverables

- **Working Notebooks:** `src/v01.ipynb` (960px baseline) and `src/v02.ipynb` (1280px experiment and full EDA).
- **Dataset Configuration:** `dataset/mine.yaml` managing train/val/test splits.
- **Release Artifacts:** Pretrained checkpoints (`best_960.pt`, `best_1280.pt`) and dataset archive (`sonar_dataset.zip`) hosted on [GitHub Release v1.0.0](https://github.com/AkshatTandekar07/mine-detection-using-YOLOv8-transfer-learning/releases/tag/v1.0.0).
- **Project Report:** Professional specification in [`Machine Learning Project Synopsis.docx`](Machine%20Learning%20Project%20Synopsis.docx).

---

## 4. Target Criteria & Achieved Results

| Evaluation Metric | Target Criterion | Achieved Result (v01 Baseline) | Status |
| :--- | :--- | :---: | :---: |
| **MILCO Recall** | **Strictly > 75.0%** at $\text{IoU} \ge 0.50$ (one-to-one matching) | **79.66%** (47 / 59 TP) at `conf = 0.05` | **Target Exceeded** |
| **MILCO Precision** | Balance operational sensitivity against false alarms | **59.49%** | Verified |
| **False Alarms / Image** | Bound false positives across all evaluated frames | **0.181 FP / image** (32 FP across 176 frames) | Controlled |
| **NonMILCO Performance** | Detect seabed clutter alongside mines | Recall 55.88%, Precision 24.68% | Verified |
| **Inference Latency** | Near-real-time AUV stream processing | **~11.1 ms / image** | Real-Time Capable |

---

## 5. Verified Dataset Audit & Distributions

Audited across 1,170 real acoustic images collected between 2010 and 2021 by a Teledyne Marine Gavia AUV:

| Metric / Check | Training Set | Validation Set | Test Set | Total Dataset |
| :--- | :---: | :---: | :---: | :---: |
| **Total Images** | 819 (70.0%) | 176 (15.0%) | 175 (15.0%) | 1,170 (100%) |
| **Empty Label Files (Background)** | 491 (60.0%) | 107 (60.8%) | 107 (61.1%) | 705 (60.3%) |
| **MILCO Instances (Class 0)** | 275 | 59 | 63 | 397 |
| **NonMILCO Instances (Class 1)** | 218 | 48 | 49 | 315 |
| **Corrupted / Zero-Byte Files** | 0 | 0 | 0 | 0 |

> **Data Integrity:** All original coordinates were preserved without destructive clipping. The known boundary annotation in `train/labels/0367_2018.txt` was validated and preserved intact.

---

## 6. Operational Recommendations

1. **Operating Point Selection:** Standard `conf = 0.25` causes severe false negatives (recall < 55%). The model must operate at `conf = 0.05` to guarantee >75% MILCO recall.
2. **Two-Tier MCM Triage:** The automated YOLOv8s detector acts as Tier 1 high-sensitivity alert generation. Flagged regions (0.181 FPPI) are queued for Tier 2 human operator or secondary classifier review before dispatching neutralization assets.
