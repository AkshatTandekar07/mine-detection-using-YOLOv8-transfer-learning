# v01 validation result — 2026-11-17

Training completed all 100 epochs. The validation-fitness-selected checkpoint is
`runs/yolov8s_v01/weights/best.pt`, SHA256
`6059aac2d77cbc9cbb560969d00d72d4ed39b7482c84f4686ccaa977747869d9`.

The predeclared validation grid selected imgsz=960, confidence=0.05,
class-aware NMS IoU=0.70, max_det=300. Both classes share these settings.
Correct detections require class agreement and IoU >=0.50 using confidence-ordered,
one-to-one matching. Evaluation includes all 176 validation images, including
133 empty-label images. Matching self-checks passed before evaluation.

| Class | TP | FP | FN | Precision | Recall | False predictions/image |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| MILCO | 47 | 32 | 12 | 59.49% | 79.66% | 0.18182 |
| NonMILCO | 19 | 58 | 15 | 24.68% | 55.88% | 0.32955 |

The >75% MILCO recall target is met **on validation**, at the frozen settings.
This does not establish test performance or independent-survey generalization.
The false-alarm tradeoff is material: 32 of 79 MILCO predictions are unmatched,
and NonMILCO precision is low. No new training or test evaluation was launched.

At confidence .05, 640 gave MILCO TP=43, FP=37, FN=16, recall=72.88%; 960 gave
TP=47, FP=32, FN=12. This is a validation comparison, not a guarantee that higher
resolution always improves performance.

Separate confidence-sweep AP summaries at the selected size:
MILCO AP50=.79521 and AP50-95=.50586; NonMILCO AP50=.41991 and AP50-95=.24192.
The precision/recall columns in validation_ap.csv are library sweep summaries;
they are not the operating-point counts in the table above.

Artifacts: validation_grid.csv, selected_settings.json, validation_ap.csv,
validation/selected_ap/, and executed outputs in src/v01.ipynb. Training and
validation switches in the saved notebook remain false to prevent accidental
reruns. Recorded notebook training-cell output now clarifies that the cell itself
did not launch the externally started training job.

Next: retain the frozen model/settings; any test evaluation must disclose prior
reference-notebook exposure and use these settings without tuning. Related-scene
leakage and unknown annotation completeness still limit interpretation. Do not
start another training experiment automatically.
