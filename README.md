# Network Intrusion Detection on NSL-KDD — A Critical Reproduction Study

**Course:** Data Science in Cyber — Final Project
**Domain:** Intrusion Detection Systems (network intrusion detection with machine learning)

## Project description

This project reproduces and **critically evaluates** a popular machine-learning intrusion-
detection tutorial built on the NSL-KDD benchmark. Rather than simply re-running it, we
test whether the tutorial's near-perfect reported accuracy reflects real detection ability.

Main findings:

- **The ~99.8% accuracy is an evaluation artefact.** The source measures accuracy with
  cross-validation *inside* one distribution. NSL-KDD ships a dedicated test set
  (`KDDTest+`) that deliberately contains **17 attack types never seen in training**
  (16.6% of the test set). Under that intended protocol our best model (XGBoost) drops
  from **0.999 CV accuracy to 0.795 test accuracy** — a 20-point fall — with an MCC of
  only 0.64.
- **Accuracy hides the dangerous attacks.** On the official test set, recall is high for
  high-volume attacks (DoS 0.87, Probe 0.73) but collapses for the severe, rare families
  (**R2L 0.06, U2R 0.19**), and novel attacks are detected only 39% of the time.
- **Methodology gaps:** a constant feature (`num_outbound_cmds`) is fed to every model,
  one-hot encoding is not aligned between train and test, only accuracy is reported, and a
  25-year-old dataset is treated as representative of modern traffic.

**Verdict:** the "Random Forest / tree ensembles are best" ordering holds, but the near-
perfect accuracy is **not** evidence of a deployable intrusion detector.

## Selected source (tutorial)

- **Tutorial / GitHub repository:** *Mamcose — NSL-KDD Network Intrusion Detection*
  https://github.com/Mamcose/NSL-KDD-Network-Intrusion-Detection
  (loads the official NSL-KDD files, splits attacks into DoS/Probe/R2L/U2R, trains Random
  Forest, KNN and a linear SVM, and reports accuracy by 10-fold cross-validation.)

## Dataset source

- **Dataset:** NSL-KDD (`KDDTrain+.txt`, `KDDTest+.txt`), the de-duplicated successor to
  the KDD Cup 1999 dataset, originally from the Canadian Institute for Cybersecurity (UNB).
- **Files used:** `data/KDDTrain+.txt` (125,973 records) and `data/KDDTest+.txt` (22,544
  records), 41 features + label + difficulty score.
- **Direct mirror used:**
  https://github.com/defcom17/NSL_KDD

## Repository contents

| Path | Description |
|---|---|
| `NSL_KDD_Intrusion_Detection.ipynb` | Main deliverable — complete, executed, documented notebook. |
| `NSL_KDD_Report.pdf` / `NSL_KDD_Report.docx` | Written report (all required sections). |
| `data/KDDTrain+.txt`, `data/KDDTest+.txt` | The dataset. |
| `outputs/` | Saved metrics (`model_metrics.csv`, `category_recall.csv`, `key_findings.json`) and figures. |
| `requirements.txt` | Pinned dependencies (Python 3.12). |

## Execution instructions

```bash
# 1. Install dependencies (Python 3.12 recommended)
pip install -r requirements.txt

# 2a. Open and run interactively (Kernel -> Restart & Run All)
jupyter notebook NSL_KDD_Intrusion_Detection.ipynb

# 2b. ...or execute headlessly
python -m nbconvert --to notebook --execute --inplace \
    NSL_KDD_Intrusion_Detection.ipynb
```

All randomness is seeded (`RANDOM_STATE = 42`); results are deterministic. The notebook
reads only the two files in `data/` and writes its artifacts to `outputs/`. Training the
six models under both evaluation protocols takes a few minutes.

## Reproducibility note

The pipeline is fully reproducible from the public NSL-KDD files. The dataset itself,
however, is a frozen 1998–1999 traffic simulation, so reproducing the accuracy says little
about performance on a modern network — a limitation analysed in Section 4 of the report.
