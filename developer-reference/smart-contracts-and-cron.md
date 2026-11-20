---
description: Technical reference for BatchExecutor.sol and Permit2 on Robinhood Chain 4663, plus the dual-scheduler architecture (Go Cron 30 registered workers and VPS Host Crontab).
---

# Smart Contracts & Background Workers

reloadSol pairs on-chain smart contract batching on Robinhood Chain with a two-tier background scheduling architecture (`reloadsol-cron` in Go + repo-managed VPS host crons).

---

## 1. Smart Contracts (Robinhood Chain — EVM `4663`)

### `BatchExecutor.sol` (`contracts/src/BatchExecutor.sol`)
Enables **1-click atomic multi-token buys and sells** for browser wallets on Robinhood Chain without requiring wallet-level EIP-5792 support:

* **Security Properties:** Owner-scoped, immutable configuration, pausable emergency switch, and strictly plain `call` execution (no `delegatecall`).
* **Mint-Agnostic Design:** `executeBatch(calls[])` executes arbitrary sequences—WETH wrap, Permit2 token pulls, and `N` KyberSwap router calls—within a single atomic transaction hash. Adding custom sell output tokens requires **zero contract redeployment**.
* **Canonical Permit2 (`0x0000...8BA3`):** One-time ERC-20 approval to Permit2 + `permit2.approve(token, spender, maxUint160, maxUint48)` unlocks single-signature batch trading (`src/utils/dlmm/rh-batch-executor.ts`, `rh-permit2-readiness.ts`).

---

## 2. Scheduler #1: Go Cron Service (`reloadsol-cron`)

Defined in `main.go` and `worker_tracker.go`, `reloadsol-cron` registers **30** periodic workers (`initWorkerRegistry` in `worker_tracker.go`) that POST to `http://web:3000/api/*` with service auth. Manual triggers are exposed on `:8080/trigger/<worker>` (requiring `X-Trigger-Secret`).

### High-Frequency Market & Execution Workers

| Worker ID | Cadence | Target Route | Function |
| :--- | :--- | :--- | :--- |
| **`ohlc_sampler`** | `15s` | `/api/ohlc/sample` | Batches up to `500` mints from Jupiter spot prices into 1m `token_ohlc_bars`. |
| **`sltp_monitor`** | `60s` | `/api/sl-tp-monitor` | **Sole exit owner** for all manual & bot `sl_tp_positions` (paper + live). |
| **`signals_refresh`** | `60s` | `/api/trading/signals` | Warms Stage-1 Early Enter signal scoring. |
| **`dlmm_manage`** | `60s` | `/api/dlmm/manage` | Meteora DLMM Healer (TP/SL/OOR decisions + Telegram alerts). |
| **`sol_arb_scan`** | `60s` | `/api/sol-arb/scan` | Scans curated triangular SOL arb pairs for net edge. |
| **`signals_sim_track`** | `120s` | `/api/signals/sim-track` | Scores candidates, runs freshness/recheck/edge gates, opens paper positions. |
| **`mcap_tracker_sim_track`** | `120s` | `/api/mcap-tracking/sim-track` | MCap discovery & entry opens across `STRATEGY_CHAINS`. |
| **`gmgn_sim_track`** | `120s` | `/api/gmgn/sim-track` | Smart-money/KOL paper opens with security gate & ML soft size. |
| **`unfiltered_trending`** | `2m` | `/api/trending` | Refreshes hot trending caches and bulk mcap snapshots. |
| **`gmgn_activity_poll`** | `180s` | `/api/gmgn/activity-poll` | Scores 60m SM/KOL buys → 2h Radar cards (NEW/TRACKING/SURGE/FADING). |

### Mid- & Low-Frequency Analytics, Liquidity & Rollup Workers

| Worker ID | Cadence | Target Route | Function |
| :--- | :--- | :--- | :--- |
| **`dlmm_screen` / `dlmm_sim_track`** | `300s` | `/api/dlmm/screen`, `/api/dlmm/sim-track` | Meteora TVL-sorted pool screener & paper auto-deploy. |
| **`rh_clmm_manage`** | `300s` | `/api/dlmm/rh-clmm-manage` | Robinhood Chain v3/v4 CLMM Multicall3 monitor (**alert-only**). |
| **`social_rollup` / `social_wallet_poll`** | `300s` | `/api/social/rollup`, `/api/social/wallet-poll` | Refreshes `social_token_rollups`, `mcap_social_pattern_24h`, and tracked wallet buys. |
| **`metrics_copier`** | `15m` | `/api/metrics/copy` | Paced GMGN-web candle copy (`2 rps`) into `token_metrics_history` + liquidity snapshots. |
| **`social_sim_track`** | `15m` (`900s`) | `/api/social/sim-track` | FOMO burst paper opens (`social_only_fomo_gt7`) + Jev Noul shadow. |
| **`social_cleanup`** | `30m` | `/api/social/cleanup` | TTL retention pruning for `social_token_events`. |
| **`gmgn_wallet_digger`** | `4h` (`14400s`) | `/api/gmgn/wallet-digger` | Portfolio edge analysis to populate roster concurrence wallets. |
| **`strategy_search`** | `6h` | `/api/strategies/search-cycle` | Walk-forward grid search pruning/spawning `search_*` clones. |
| **`report_precompute`** | `6h` (`21600s`) | `/api/report-precompute/refresh` | Precomputes consensus & capital report tables (`strategy_report_precompute`). |
| **`pnl_update`** | Daily `02:00 UTC` | `/api/pnl/update` | Recomputes 30-day matched wallet P&L into `token_operations`. |
| **`pnl_daily_report`** | Daily `09:15 WIB` | `/api/pnl/daily-report` | Deterministic Telegram digest + attached HTML report (runners vs. rugs, 7d trend). |
| **`strategy_report` / `gmgn_radar_digest`** | Daily (`86400s`) | `/api/strategies/report-digest`, `/api/gmgn/radar-digest` | Pinned 24h strategy leaderboard & digest (`strategy_report` uses `180s` HTTP budget). |

---

## 3. Scheduler #2: VPS Host Crontab & Watchdogs

Jobs that require host-level filesystem access, Python ML training, or direct `docker exec` DB checks run from the VPS crontab (managed via `scripts/install-host-crons.sh` and `scripts/install-ml-pattern-cron.sh`—not visible in `:8080/workers`):

* **ML & Measurement:** `ml-pattern-daily` (`03:05`), `ml-closed-loop-weekly` (Sun `07:23`, executes inside `web` via `127.0.0.1:3000` to bypass nginx's 120s timeout), `counterfactual` (`*/5`), `rug-v2-soak` (`*/30`), `mcap entry-meta` (`*/5`), `dev-reputation` (`07:17`).
* **Hygiene:** `analytics prune` (`05:17`), `disk-prune` (Sun `06:19`, trims builder/npm/pip caches).
* **Watchdogs:** `check-copier-freshness` (`*/15`) and `check-sltp-closer-freshness` (`*/5`) read Postgres directly through `reloadsol-db` (working even if `reloadsol-web` is down) and distinguish recent lock contention from stalled sweeps using `LOCK_INSTANCE_SCOPE` and lock age.
