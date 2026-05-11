# Whisker Detection Decoding — Project 1

Analysis of NWB sessions from a mouse whisker deflection detection task. The pipeline extracts a neural population vector score and evaluates its ability to discriminate stimulated from non-stimulated trials.

---

## Installation

```bash
conda create -n NEURO_PROJECT1 python=3.10
conda activate NEURO_PROJECT1
pip install -r requirements.txt
```

---

## Data structure

```
data/
├── brut/
│   ├── WR+/      ← raw WR+ NWB files (includes MH* files)
│   └── WR-/      ← raw WR- NWB files
├── valid/
│   ├── WR+/      ← validated WR+ sessions (includes MH*)
│   └── WR-/      ← validated WR- sessions
└── non_valid/    ← rejected sessions
```

> **Note:** `.nwb` files are too large for GitHub and are excluded via `.gitignore`.

---

## Active notebooks

### `test_sess_valid.ipynb` — Session validation

Scans `data/brut/WR+/` and `data/brut/WR-/` and decides which sessions are usable.

**Validation criteria:**
- At least 10 units in the target area (wS1 / SSp-bfd)
- Required columns present in the NWB file

**Output:** valid sessions are copied to `data/valid/WR+/` or `data/valid/WR-/` based on their group. A summary table (with a `Grp` column) is displayed. A second block scans the `valid/` folders to verify the `EngagedTrials` time window.

---

### `results.ipynb` — Main analysis pipeline

#### Session types

| Type | Paradigm | Stimulation | Piezo | Catch trials |
|------|----------|-------------|-------|--------------|
| **WR+** | Water restriction, task positive | 4 amplitude levels (1–4) | ✓ | ✓ |
| **WR-** | Water restriction, task negative | 4 amplitude levels (1–4) | ✗ | ✗ |
| **MH\*** | MH files (different NWB format) | Single amplitude → mapped to amp 4 | ✓ | ✗ |

#### Pipeline

1. **Scan** (`SESSION_PATHS`): glob over `data/valid/WR+/*.nwb` and `data/valid/WR-/*.nwb`
2. **`_areas_for_path(path)`**: for MH files, detects available SSp-bfd variants (`SSp-bfd`, `SSp-bfd-C2`, `SSp-bfd-C3`) and returns a list — each variant is run separately
3. **`run_session(path, target_area=None)`**: analyses one session
   - Unit selection in the target area (supported columns: `Target_area`, `ccf_parent_acronym`, `brain_area`, `location`)
   - PSTH and population vector score computation
   - Threshold sweep (`pct_axis = linspace(1, 99.9, 200)` + extension up to `score.max()`)
   - TP / FP / FP-catch computed at each threshold
   - Best threshold via Youden index (with TP4 ≥ 0.5 constraint)
   - P(lick | ITI detection) computed at each threshold (WR+ only)
   - `EngagedTrials` filter: spikes and trials are trimmed to the last engaged timestamp
4. **Split**: `results_wrp` (WR+ + MH), `results_wrm` (WR-)

#### Plots

| Cell | Content |
|------|---------|
| **Plot 1** — PSTH | Mean PSTH per amplitude; for MH files: single "Stim" curve |
| **Plot 2** — TP/FP | Mean TP/FP + P(lick\|ITI) curves; F1 score matrix (sessions × threshold); d-prime ranking; 4 correlation scatter plots (n_units, n_bursts, peak PSTH stim4) |
| **Plot 3** — P(lick) | Lick probability by amplitude (WR+ only) |
| **Stats** | Wilcoxon, Fisher, binomial null tests (WR+ only) — interactive Plotly scatter |
| **Plot 5** — Lick bouts | Lick bout analysis (WR+ only) |
| **Vis** | CD score / firing rate / piezo for a single selected session |

#### Session labels

`_result_label(r)` generates a unique label: `[WR+] filename.nwb` or `[WR+] filename_SSp-bfd-C2.nwb` for MH sub-populations.

---

## Other files

The notebooks `CDMatrix.ipynb`, `CDMatrix_MH029.ipynb`, `CDMatrix_MH031.ipynb`, `results_cd_methods.ipynb`, and `results_mPFC.ipynb` are exploratory or secondary analyses — **they are not part of the main pipeline**.
