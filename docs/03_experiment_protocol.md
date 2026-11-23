# Experiment Protocol and Evaluation Framework

Formal evaluation protocol, matching convention, and comparative resolution benchmark.

---

## 1. Controlled Experimental Design

Two controlled transfer-learning configurations were executed under identical training parameters:

1. **v01 Baseline:** Pretrained YOLOv8s fine-tuned at **imgsz = 960** (batch 8).
2. **v02 High-Resolution:** Pretrained YOLOv8s fine-tuned under identical settings at **imgsz = 1280** (batch 4).

### Fixed Hyperparameters
- **Epochs:** 100 (patience = 20)
- **Optimizer:** SGD (`lr0 = 0.01`, `momentum = 0.937`, `weight_decay = 0.0005`)
- **Random Seed:** 42 (fully deterministic)
- **Base Checkpoint:** Generic COCO-pretrained `yolov8s.pt` (349 / 355 weights transferred)

---

## 2. Evaluation Convention & Selection Rules

- **Matching Rule:** Strict one-to-one matching between predictions and ground-truth boxes requiring **class agreement** and $\text{IoU} \ge 0.50$.
- **Operating Grid Sweep:** Evaluated across confidence thresholds:
  $$[0.001, 0.01, 0.025, 0.05, 0.10, 0.15, 0.25, 0.40, 0.60]$$
- **Non-Maximum Suppression (NMS):** Class-aware NMS at $\text{IoU} = 0.70$, `max_det = 300`.
- **Validation Selection Rule:**
  1. Filter all threshold settings satisfying $\text{MILCO Recall} > 75.0\%$ on validation data.
  2. Select the threshold minimizing false MILCO alarms per image (FPPI).
  3. Freeze the selected threshold (`conf = 0.05`) before held-out test evaluation.

---

## 3. Quantitative Results Across Input Resolutions

Evaluated on the 176-image validation split at `conf = 0.05`:

| Model Pipeline | Input Resolution | MILCO Recall (IoU $\ge$ 0.50) | MILCO Precision | False Alarms / Image | mAP50 | Latency (ms) | Operational Target |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **v01 Baseline (Selected)** | **960 × 960** | **79.66%** (47 / 59 TP) | **59.49%** | **0.181** (32 FP) | **0.6142** | **~11.1** | **Target Exceeded (>75%)** |
| **v02 High-Resolution** | 1280 × 1280 | **76.27%** (45 / 59 TP) | 39.47% | 0.390 (69 FP) | 0.5786 | ~23.4 | Target Met (High FP) |
| **v3 Reference** | 640 × 640 | 64.00% | 68.20% | N/A | 0.7440 | ~7.1 | Target Failed (<75%) |

---

## 4. Empirical Resolution Findings

1. **Optimality of 960px:** Resolves small acoustic highlight-shadow pairs sufficiently to exceed the 75% recall requirement while maintaining tight false alarm bounds (1 false alarm every 5.5 images).
2. **Speckle Amplification at 1280px:** Scaling resolution to 1280px reduced recall by 3.39 percentage points and more than doubled false alarms (1 false alarm every 2.6 images). Receptive fields at higher resolution amplify acoustic reverberation and seabed sand ripples, inducing spurious detections.
3. **Loss of Detail at 640px:** Fails to reach 75% recall due to downsampling small mine highlights into background clutter.

---

## 5. Critical Generalization Limitations

- **Survey Track Autocorrelation:** Benchmark frames were partitioned randomly across splits rather than by continuous AUV survey track. Consecutive pings share sediment texture, vehicle altitude, and grazing angles. Real-world deployment in unmapped maritime areas will exhibit performance drop.
- **Uncalibrated Confidence:** Raw detector scores reflect relative ranking rather than calibrated Bayesian probabilities.
- **MILCO Designation:** Labels denote Mine-Like Contacts, not confirmed explosive ordnance; final clearance requires secondary optical/diver verification.
