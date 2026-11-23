What this repo does: It fine-tunes COCO-pretrained YOLOv8s to locate MILCO and NonMILCO contacts in sonar images. The reference notebook contains training, validation, test evaluation, test visualizations and training-box analysis. Expected load is one researcher running one GPU experiment.

## Must fix

1. **Test settings and checkpoint identity are not controlled** (`src/side-scan-sonar-mine-detection-project-training.ipynb`, cells 2 and 4).
   - **What this is:** Training uses 960 and validation-selected best.pt; testing uses 640 and a separately uploaded best.pt.
   - **Problem:** The uploaded checkpoint cannot be traced to the run from available evidence. The observed 0.481 MILCO test recall is also at a different image size; it does not establish what validation-selected 960 inference would do.
   - **Fix:** Pin checkpoint hashes, select inference settings on validation and freeze them before any future test run. v01 implements this protocol.
   - **If we skip it:** Results cannot be attributed to a reproducible model and operating setting.

2. **The supplied splits contain visibly related scenes** (`reports/v01/related_images.png`, `data_audit.json`).
   - **What this is:** Byte hashes found no exact duplicates across 1,170 images. A centered, normalized 32x32 grayscale thumbnail comparison found 225 cross-split pairs above cosine similarity 0.98; these are candidates, not 225 proven leaks.
   - **Problem:** Visual inspection confirms nearly identical scene content in train 0072_2010 / val 0071_2010, train 0119_2015 / val 0120_2015, and train 0315_2010 / val 0316_2010. Shared year suffixes across all splits are clues, not collection metadata.
   - **Fix:** Preserve the supplied benchmark and qualify its scores. Obtain collection/track grouping before a separately authorized independent split; do not re-split to improve a score.
   - **If we skip it:** A high benchmark score may overstate performance on independent sonar surveys. v01 documents this limitation; independence is unresolved.

## Should fix

3. **Precision/recall summaries do not provide the requested operating counts** (reference cells 2 and 4).
   - **What this is:** Ultralytics reports precision and recall from its confidence sweep; example predictions separately use confidence 0.25.
   - **Problem:** The saved summaries do not supply integer TP/FN, false MILCO predictions per image, or one common frozen confidence for all reported outputs. Multiplying rounded/interpolated recall by object count is not a valid reconstruction.
   - **Fix:** Count class-correct, confidence-ordered one-to-one matches at IoU >= 0.50, including empty images and duplicate false positives. v01 includes explicit counts and checks for wrong classes, duplicates, empties and the IoU boundary.
   - **If we skip it:** We cannot verify the >75% criterion or its false-alarm cost.

4. **A defective training row passes the native verifier** (`dataset/train/train/labels/0367_2018.txt`, row 2).
   - **What this is:** Its box is about 0.00488 x 0.03125 original pixels and extends left of the image.
   - **Problem:** The installed verifier retains it, while native random-perspective processing clips boxes and filters transformed boxes with width or height <=2 pixels. Training's effective sample labels can therefore differ from raw row counts.
   - **Fix:** Preserve and flag the row, document the baseline native transform behavior, and seek source annotation evidence before any future correction. v01 does this; no annotation repair is claimed.
   - **If we skip it:** Training/EDA counts may be described inconsistently, or a later cleanup may silently change the benchmark.

5. **Project documentation describes an obsolete state** (`docs/01_project_charter.md`, `02_methodology_and_roadmap.md`, `04_experiment_protocol.md`, `05_decisions_and_progress.md`, YAML files).
   - **What this is:** Older documents name another project root, a provisional 95% target and training not started. Dataset YAML points to Kaggle and reports/sonar_data.yaml points to an obsolete local root.
   - **Problem:** Running with these paths fails locally; treating these documents as current hides completed test exposure.
   - **Fix:** Root AGENTS.md records current authoritative instructions; v01 creates a separate local data-only YAML. This audit and handoff supersede stale factual descriptions without rewriting historical documents.
   - **If we skip it:** Future work can use wrong paths, target or evaluation claims.

Verdict: v01 addresses reproducibility and operating metrics; supplied-split independence and source annotation policy remain unresolved. Do not claim independent-survey performance.
Not checked: No new test-label contents were opened. Dataset license, acquisition tracks, annotation completeness and the separately uploaded test checkpoint could not be verified. No full training was run during this audit; the existing report builder was inspected as a saved-evidence consumer, not re-rendered.
