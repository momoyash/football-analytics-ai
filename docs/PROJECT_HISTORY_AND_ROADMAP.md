# Project history & roadmap

This document records **what has been built** in this repository and **what you plan or are willing to do** next. Update it as the project evolves.

---

## 1. Completed work

### 1.1 Repository & environment

- [x] **Project skeleton** — Folders: `config/`, `data/{raw,interim,processed,external}/`, `models/{artifacts,metrics}/`, `notebooks/`, `reports/`, `src/football_ai/{io,preprocessing,modeling,evaluation,pipeline}/`, `tests/`.
- [x] **`requirements.txt`** — Core stack: pandas, numpy, scikit-learn, statsbombpy, mplsoccer, streamlit, torch; plus matplotlib, seaborn, jupyter, tqdm, plotly.

### 1.2 Data ingestion

- [x] **`src/football_ai/io/statsbomb_loader.py`** — Uses `statsbombpy` to fetch matches for a competition/season and save **one CSV per match** (default under `data/raw/comp_<id>_season_<id>/`).

### 1.3 Notebooks

- [x] **`notebooks/01_exploration_statsbomb.ipynb`** — Explore StatsBomb-style data: imports, competitions/matches/events (as designed for Open Data workflow), EDA, mplsoccer visuals.
- [x] **`notebooks/02_feature_prototyping.ipynb`** — Glob-load all `data/raw/**/events_*.csv`, concatenate events, compute **match-level** aggregates (shots, passes, pressures, xG-style columns where present).
- [x] **`notebooks/03_modeling_experiments.ipynb`** — Placeholder for further experiments (extend as needed).

### 1.4 Utilities & features

- [x] **`src/football_ai/evaluation/match_summary.py`** — CLI: load a **single** event CSV; print total passes, shots, pressures, xG, possession-style breakdown, per-team tables.
- [x] **`src/football_ai/preprocessing/feature_engineering.py`** —  
  - Team-level: `compute_team_features` — possession %, passes/min, pressures/min, average shot distance, total xG.  
  - Shot-level: `build_shot_dataset` — coordinates, distance, time, optional body part/technique/type, binary **goal** target for xG-style models.

### 1.5 Machine learning pipelines

- [x] **`src/football_ai/modeling/train.py`** — **Match outcome** classifier: reads a CSV with **team-level features** and a **`result`** column (e.g. win/draw/loss); `StandardScaler` + `RandomForestClassifier`; saves `joblib` artifact.
- [x] **`src/football_ai/modeling/xg_model.py`** — **Expected goals (xG)** pipeline: globs event CSVs, builds shot rows via `build_shot_dataset`, sklearn `Pipeline` (numeric scale + one-hot categoricals + `GradientBoostingClassifier`); reports ROC-AUC / Brier; saves model.
- [x] **`src/football_ai/modeling/tactical_state.py`** — **Tactical state** from **event sequences**: sliding windows over events (per match/team); features = counts of pass/shot/pressure + mean location; label = `tactical_state` on the last event in the window. **Requires labeled data** — see §3.

### 1.6 Operational lessons (documented for you)

- [x] **Windows paths** — Raw strings (`r"..."`) or forward slashes avoid `OSError: [Errno 22]` from escape sequences (`\f`, `\t`, etc.).
- [x] **PowerShell** — Line continuation uses **backtick** `` ` ``, not `\`.
- [x] **`football_ai` import** — Run notebooks with CWD = project root and `sys.path` including `.../football-analytics-ai/src`, or install package editable.
- [x] **`train.py` FileNotFoundError** — `data/team_features.csv` must exist or pass a real path; that file is **not** auto-generated from raw events yet.

---

## 2. What you are willing to do next (roadmap)

Use the checkboxes to track intent. Reorder by priority.

### 2.1 Data & labels

- [ ] **Build `team_features.csv`** — Script or notebook: join match results to per-team features from `compute_team_features` for many matches → feed `train.py`.
- [ ] **Tactical state labels** — Rule-based or manual/human-in-the-loop labels on events; write `tactical_state` column to CSV → enable `tactical_state.py` training.
- [ ] **Versioned datasets** — Document naming under `data/processed/` (e.g. `shots_xg_v1.parquet`).

### 2.2 Modeling

- [ ] **Calibrate xG** — Reliability curves, hold-out competitions, compare to `shot_statsbomb_xg` where available.
- [ ] **Hyperparameter tuning** — `RandomizedSearchCV` / `GridSearchCV` for xG and outcome models.
- [ ] **Sequences / deep learning** — Optional PyTorch model over event sequences (if you want to go beyond windowed sklearn features).

### 2.3 Product & UX

- [ ] **Streamlit app** — Upload or select match CSV → show `match_summary`, team features, optional xG predictions.
- [ ] **Configuration** — Centralize paths and IDs in `config/config.yaml` + small loader in `config.py`.

### 2.4 Engineering

- [ ] **`pyproject.toml`** — Editable install `pip install -e .` so `from football_ai...` works everywhere.
- [ ] **Tests** — `pytest` for `feature_engineering`, `match_summary`, and small synthetic event frames.
- [ ] **CI** — Lint + tests on push (optional).

### 2.5 Documentation & sharing

- [ ] **Notebook narrative** — Short methodology section in each notebook (data source, assumptions).
- [ ] **Public write-up** — Blog or report linking this repo to findings.

---

## 3. Dependency matrix (quick reference)

| Component | Input | Output / artifact |
|-----------|--------|-------------------|
| `statsbomb_loader` | competition_id, season_id | CSVs under `data/raw/...` |
| `match_summary` | One `events_*.csv` | Printed summary |
| `compute_team_features` | Events DataFrame | Per-team feature table |
| `build_shot_dataset` | Events DataFrame | Shot-level table |
| `xg_model` | Glob of event CSVs | `models/artifacts/xg_model.joblib` |
| `train` (match outcome) | CSV with `result` + features | `models/artifacts/match_outcome_model.joblib` |
| `tactical_state` | Event CSVs with `tactical_state` | `models/artifacts/tactical_state_model.joblib` |

---

## 4. How to update this doc

After each milestone, add a bullet under **§1 Completed work** and tick or add items under **§2 Roadmap**. Keep **§3** in sync if you add new scripts or change CLI flags.
