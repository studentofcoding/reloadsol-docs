---
description: reloadSol's LightGBM + ONNX machine learning pipeline, shadow-vs-enforce governance, 4-class OHLC head, and the /admin evaluation suite.
---

# Machine Learning Pipeline & Admin Evaluation

reloadSol uses **supervised tabular machine learning (LightGBM exported to ONNX)** to score and rank token entries inside the strategy runtime. Models are trained on the host in Python (`ml/`) and served at sub-millisecond latency inside `reloadsol-web` via `onnxruntime-node` from a read-only bind mount (`/app/ml/artifacts:ro`).

---

## 1. Active ML Tracks

| Track | Target / Classes | Runtime Status | Artifact Directory |
| :--- | :--- | :--- | :--- |
| **Pattern-Gate (Binary)** | 24h MCap + social cohort (`winner = 1` ≥ 120% growth, `loser = 0` < 80%) | **Served in Shadow** (`attachMlEntryShadow` stamps `ml_pattern_*`; displayed on Stage-1 Signals early alerts) | `$ML_PATTERN_ARTIFACT_DIR` (`ml/artifacts/pattern-gate-rug`, fallback `pattern-gate/`) |
| **4-Class OHLC Growth Head** | Sol `token_mcap_tracking` growth: **loser** (`< 0%`), **bep** (`0–20%`), **winner** (`20–120%`), **moonbag** (`≥ 120%`) | **Labels + Train/Eval Only** (not yet wired to live size or enforce) | `ml/artifacts/growth4-ohlc/` |
| **Closed-Loop Weekly Eval** | Weekly evaluation scan over live closed outcomes | **Shadow Weekly** (Sunday host cron `run-ml-closed-loop-weekly-on-vps.sh`) | `/api/strategies/ml/eval-scan` |
| **Legacy `v2-gate` / `v2-potential`** | Retired `2026-10-16` | **Removed** — never stamped | Retired |

---

## 2. Anti-Leakage & Provenance Rules

reloadSol enforces institutional ML hygiene (`SPEC-ml-hardening-v1`, `SPEC-feature-registry-v1`):

1. **Entry-Time Features Only:** Models only see features knowable at entry (`log_first_mcap`, 30m social mentions/channels, smart-wallet flags, 60m GMGN activity window around first-seen). Exit prices, monitor snapshots, and post-entry candles are strictly forbidden.
2. **Time-Ordered 3-Way Splits:** `train_pattern.py` uses `three_way_split()` (train → valid → test in chronological order). Decision thresholds (`0.05–0.95`) are tuned **strictly on validation**; the test holdout is untouched until final reporting.
3. **Honest Metrics (Never Accuracy Alone):** Because winners are a minority class (~15%), a trivial all-loser classifier scores `~88%` accuracy with `0.00` winner recall. Readiness gates require **macro-F1**, **PR-AUC**, and **minimum class support** (`≥ 30` per class, `≥ 60` rows, `macro_f1 ≥ 0.60` for Pattern-Gate).
4. **Feature-Registry Parity at Load:** `ml/validate_artifact.py` verifies that `model.meta.json`'s `feature_columns` match `ml/feature-schema.json` in exact **names and order**, and checks numeric `feature_schema_version` provenance (`git_sha`, `training_data_cutoff`, `eval{auc, ci_low, brier}`).
5. **Canary & Rollback:** Hot-swapping an artifact uses `scripts/ml-artifact-swap.sh validate`, preserves `<live>.prev`, and rolls back automatically if `POST /api/ml/pattern/reload` reports `schema_ok = false`. Enforcement is additionally gated by `ML_ENFORCE_TRAFFIC_PCT` (FNV-1a deterministic canary bucketing).

---

## 3. Daily & Weekly Automation

* **Nightly Pattern Pipeline (`03:05` host cron → `scripts/ml-pattern-daily.sh`):** Exports `mcap_social_pattern_24h` to parquet → validates dataset readiness → trains only if `train_ready = true` → hot-reloads ONNX via `POST /api/ml/pattern/reload` with zero web downtime.
* **Counterfactual Book (`*/5` host cron → `/api/counterfactual/tick`):** Records forward outcomes on *rejected* candidates so operators can measure whether a filter prevented real dumps or vetoed real winners.

---

## 4. Admin Evaluation Suite (`/admin/*`)

| Console | Primary Use |
| :--- | :--- |
| **`/admin/ml`** | Inspects every model's live mode (`shadow` vs. `enforce`), readiness bar checklist, feature coverage (`non_null_rate` / `non_zero_rate`), and canary holdout logs. |
| **`/admin/strategy`** | Per-strategy scorecard: net & gross P&L, win rate, payoff ratio, stop-loss hit rates, and stake compliance. |
| **`/admin/rug`** | Layer-by-layer rug ban precision/recall, 1m OHLC coverage, and close-reason distributions. |
| **`/admin/review`** | Insert-only human audit queue for trades closing `≤ -15%` net and missed rugs (`tb_v1`). Assigning `ml_label` (`skip` / `interesting` / `anomaly`) persists into `strategy_outcomes.features` for future training exports. |
| **`/admin/replay`** | Replays alternative SL/TP and trailing-stop geometries over reloadSol's own 1m `token_ohlc_bars` (net of fees) with paired per-token clustered confidence intervals. |
| **`/admin/graph-edge`** | Interactive relational graph audit for cabal rings, deployer motifs, and multi-strategy consensus cliques. |
| **`/dev/rug-signal` & `/dev/ohlc-labels`** | Calibration soak console (`rug_v2_shadow`) and visual gallery of first-10-minute Rising vs. Rugged OHLC snapshots. |
| **`/dev/insight`** | Data-Public Scout strip, Macro Climate indicator (`Safe` / `Not safe` / `Unknown`), and Solana Roster Digger. |
