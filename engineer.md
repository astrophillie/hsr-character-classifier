# ai-tutor — Engineering audit

## Tutor profile

**Name:** ai-tutor  
**Description:** Review AI engineering homework from the perspective of a senior engineer and explain improvements to the writing, code, experimental design, and presentation. Teach the reasoning behind each recommendation, prioritize concrete fixes, and preserve the student's ownership of the work.

For future reviews: cite the relevant file, notebook section or cell ID; distinguish confirmed defects from limitations and optional extensions; give a small actionable correction; and say what was actually verified. Do not invent experiments, results, personal experience, or grading requirements.

## Scope and verdict

Reviewed on 2026-11-12: [hsr-character-classifier](https://github.com/astrophillie/hsr-character-classifier), main at commit `8ae8a39df2eb1f016a851df527c368cb1ec0369c`.

Inspected the README, all 26 notebook cells, requirements lock, Git ignore rules, repository file tree, and all seven committed result/manifest files. This is a static code and artifact audit. Training, image inspection, source grouping, package installation, and checkpoint inference were not rerun. The original images and checkpoints are absent from the repository. Local shell and file-write tools failed in this session; this summary was saved through GitHub.

The project demonstrates a sensible transfer-learning workflow and unusually careful limitations for a small course exercise. The recorded headline numbers agree with the saved confusion matrices. The main submission weaknesses are inconsistent narrative scope and incomplete reproducibility. No grading rubric was supplied, so this is feedback rather than a grade.

## What is already working well

- Appropriate starting model: pretrained ResNet18 with a replacement classification head.
- Training and validation class mappings are checked; validation uses evaluation mode and no gradient tracking.
- Losses are weighted by batch size when aggregated, avoiding bias from the final smaller batch.
- Checkpoints retain class order. The two-class experiment includes a direct comparison of logits after reloading.
- The README reports correct counts as well as percentages and separates the two-class and three-class tasks.
- The write-up acknowledges validation-based selection, missing source provenance, domain differences, and the limitations of softmax on unseen characters.
- Fresh-artwork URLs are pinned to an upstream commit and accompanied by hashes.

## Findings, in priority order

### 1. Reconcile the notebook's experiment scope — fix before submission

**Evidence:** The opening markdown cell (`2a78d746`) says feature extraction was the only strategy tested and that no fine-tuning comparison is claimed. Section 10 recommends fine-tuning as an unperformed extension. Section 11 then reports an actual two-class fine-tuning experiment. Section 10 partly qualifies its statements as baseline-only, but the opening claim reads as applying to the whole notebook.

**Change:** Introduce both experiments at the start, explicitly title Section 10 as the three-class baseline conclusion, and add a final overall conclusion after Section 11. Replace the opening sentence with:

> The three-class baseline uses feature extraction for three epochs. Section 11 compares feature extraction and full fine-tuning on the Aventurine/Kafka subset; those results describe a different classification task.

Keep the distinction between epochs, labels, learning rates, and validation populations visible throughout.

### 2. Make the saved baseline artifacts reproducible — fix before submission

**Evidence:** Section 10 references `outputs/baseline/validation-predictions.csv`, which is absent from the committed tree. The notebook computes the baseline history and predictions but does not export the committed baseline `history.csv` and `results.json`. It also does not reproduce the baseline checkpoint-logit comparison recorded as true in that JSON. Section 11 writes additional histories, predictions, figures, and a conclusion that are absent from the current tree.

**Change:** Add an explicit baseline export cell using the metrics already computed. Export per-image relative paths, true labels, predicted labels, and scores. Add the reload comparison for the baseline, following the existing two-class check. Commit lightweight CSV/JSON evidence or clearly label each omitted artifact as generated locally and remove unavailable references.

All 13 code cells have null execution counts and empty outputs. An executed HTML export or selected saved figures would make the homework easier to assess without private assets. Do not imply that absent evidence proves the original run was incorrect.

### 3. Preserve the split and preparation evidence — important validity limitation

**Evidence:** The notebook describes screening 270 images down to 241, grouping related shots, and checking duplicates. The repository contains no dataset-wide manifest, grouping assignments, preparation script, or split-generation code. The path-disjointness assertions in Section 11 cannot establish that two different files contain duplicate or related images.

**Change:** Save a manifest with relative path, class, split, file/decoded-pixel hashes, group ID, medium, and source ID where known. Include the grouping/splitting procedure and duplicate checks. Explicitly mark unavailable source provenance rather than filling it in by guesswork.

For a future evaluation, split by source or related-image group and reserve an independent test set before tuning. With the existing small dataset, repeated group-aware validation is also useful, but cannot recover unknown source independence. Current evidence supports a result on this curated split, not a reliable general recognition rate.

### 4. Make BatchNorm behavior an explicit experimental choice

**Evidence:** `train_model` calls `model.train()` each epoch, including when backbone parameters have `requires_grad=False`. The notebook correctly acknowledges that BatchNorm running statistics still change.

**Change:** Describe the existing method as a frozen-parameter backbone with BatchNorm-statistic adaptation. If testing a strictly fixed feature extractor, set the frozen backbone to evaluation mode after each call to `model.train()`, while leaving the new head in training mode. Reapply this every epoch.

The current behavior is documented and is not an undisclosed bug. A change would create a new experiment and require new results; preserve the existing baseline.

### 5. Report precision alongside recall and keep conclusions modest

Derived directly from the saved baseline confusion matrix:

| Class | Precision | Recall | F1 |
| --- | ---: | ---: | ---: |
| Aventurine | 90.0% | 64.3% | 75.0% |
| Kafka | 58.3% | 93.3% | 71.8% |
| Phainon/Khaslana | 100.0% | 75.0% | 85.7% |

Macro F1 is 77.5%. Kafka's high recall coexists with many false positives: 10 of 24 Kafka predictions are wrong. Add precision, recall, F1, and support to the report.

The selected two-class run gets 28/29 correct versus 27/29 for ten-epoch feature extraction: one image, or 3.45 percentage points. Both strategy and learning rate differ. Report the observed comparison without claiming a reliable causal advantage for fine-tuning. Repeat seeds and use source-separated evaluation before making a stronger claim.

### 6. Make setup and reruns easier to follow

**Evidence:** README gives the environment file and required folders but no installation or kernel commands. The recorded environment uses Python 3.14.3 and CUDA-specific Torch/Torchvision builds. The notebook depends on its working directory, private data, demo images, and sequential state.

**Change:** Document the tested OS, Python version, installation command and package source for those builds, kernel registration, project-root launch command, and CPU alternative if verified. Distinguish the recorded environment from environments actually tested. Do not present the CUDA lock as universally portable.

Provide a read-only route to inspect committed results and a separate route to retrain with local assets. Use experiment-specific output folders so running fresh demonstrations does not overwrite the recorded CSVs. Store seed, preprocessing, pretrained weight identifier, and configuration beside each checkpoint. Seeds help repeatability; they alone do not guarantee identical results across hardware and versions.

### 7. Treat image preprocessing changes as separate experiments

**Evidence:** The notebook documents hidden RGB content becoming visible when transparent PNGs are converted directly to RGB. Training applies a random crop to 224 pixels and then the pretrained transform, which resizes to 256 and center-crops back to 224.

**Change:** In a new experiment, composite transparency onto a declared background and handle EXIF orientation consistently at ingestion. Inspect the final transformed images. Consider using random crop, flip, tensor conversion, and normalization directly for training, while retaining the weights' standard evaluation transform for validation. The current extra resize/crop is executable, but may remove additional character detail.

Compare preprocessing choices on a fixed development protocol, record the change, and preserve the original results. Expand the illustration sample before attributing errors to art style: the README records only two illustrations among 241 dataset images.

## Suggested final wording

> The three-class baseline classified 38 of 49 validation images correctly (77.6%). On the Aventurine/Kafka subset, ten-epoch full fine-tuning classified 28 of 29 correctly, compared with 27 of 29 for ten-epoch feature extraction. These are observations on small validation sets used during development, not independent test results. The unseen-character examples illustrate forced assignment to known labels and do not establish unknown-character detection.

This wording retains the achievement while matching the available evidence.

## Verification performed

- Parsed the notebook and inspected all cell sources.
- Recomputed confusion-matrix totals and diagonals: 38/49 for the baseline and 28/29 for the selected two-class model.
- Checked the recorded run accuracies against the README: 18/29, 27/29, and 28/29.
- Compared baseline epoch metrics with the committed history and fresh-example observations with the prediction CSV.
- Checked the unseen Phainon/Khaslana summary: 94 Aventurine assignments plus 5 Kafka assignments equals 99.
- Derived the precision/recall/F1 values above from the saved matrix.
- Checked artifact references against the complete repository tree.

These checks establish internal consistency of the committed records, not independent reproduction of training.

## Next submission checklist

1. Reconcile the opening and conclusion with Section 11.
2. Export baseline metrics and per-image predictions from the notebook; fix missing references.
3. Save split/group metadata and document remaining provenance gaps.
4. Add exact setup instructions and an inspectable execution artifact.
5. Add per-class precision/F1 and keep the one-image comparison appropriately qualified.
