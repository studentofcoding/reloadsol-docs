---
description: Complete directory of ReloadSOL's dev-wallet-gated operational hubs (/dev/*) and read-only evaluation consoles (/admin/*).
---

# Dev & Admin Console Overview

Beyond its public trading pages, ReloadSOL provides two gated operator workspaces:

* **`/dev/*` (Developer & Strategy Tools):** Interactive consoles for configuring strategies, running paper simulations, managing DLMM/CLMM liquidity, scanning arbitrage, inspecting RPC/pool health, and searching tokens across chains.
* **`/admin/*` (Read-Only Evaluation & Governance Hub):** Quantitative scorecards, ML readiness audits, lifecycle runner telemetry, rug precision/recall metrics, insert-only outcome review, and historical exit replay.

---

## Access Control & Network Gating

All `/dev/*` and `/admin/*` pages are guarded by `DevOnlyGate` and dev-wallet session checks (`DEFAULT_DEV_WALLETS` + `NEXT_PUBLIC_DEV_WALLETS`).

* **API Protection:** Dev endpoints under `DEV_API_PREFIXES` (`/api/strategies/*`, `/api/dlmm/*`, `/api/signals/*`, `/api/gmgn/*`, `/api/social/*`, `/api/sol-arb/*`, `/api/workers/*`, `/api/analytics/*`, `/api/logs/*`) require an authenticated dev-wallet session or valid service/cron secret.
* **Chain Scope:** Most strategy hubs support both `sol` and `robinhood` (with RH strategies strictly `sim_only`), while Solana-specific tools (`/dev/rpc-tester`, `/dev/arbitrage`, `/dev/ohlc-labels`, `/dev/dev-reputation`) are Solana-only (`src/config/route-network.ts`).

---

## `/dev` Workspace Directory

| Category | Route | Purpose |
| :--- | :--- | :--- |
| **Strategy** | `/dev/algo-tester` | Unified Strategy Hub: **Config** (params, activation, execution mode, notify toggles, Workers), **Open** (all-domain open positions), and **Closed** (reports, outcomes, ML review). Legacy `/dev/strategies` redirects here. |
| **Strategy** | `/dev/paper-trade` | Paper-desk readiness dashboard: ledger-rebuilt net P&L, `summary.costModel`, `peakConcurrentPositions`, and `READINESS_MIN_SAMPLE` (`≥ 30`) verdicts. |
| **Strategy** | `/dev/signals` | Signal strategy configuration, Live/Board/Tracker tabs, watch/potential/rugged labeling, and manual sim/live buys. |
| **Strategy** | `/dev/fomo` | FOMO burst detection and Robinhood/Solana trader & closed-position tracking. |
| **Strategy** | `/dev/social` | Telegram social ingest channel manager, smart-wallet pollers, and mention rollups. |
| **Signals & ML** | `/dev/rug-signal` | Calibration, soak monitoring, and class-separation console for the Rug Signal scorer. |
| **Signals & ML** | `/dev/ohlc-labels` | Visual training gallery of the first 10 minutes of 1m OHLC bars for Rising vs. Rugged tokens. |
| **Signals & ML** | `/dev/dev-reputation` | Creator/deployer provenance ("Dev DNA"), historical top-10 ATH tokens, and user-labeled rug counts. |
| **Signals & ML** | `/dev/insight` | Per-network Data-Public Scout, Macro Climate indicator (`Safe` / `Not safe` / `Unknown`), and Solana Roster Digger. |
| **Signals & ML** | `/dev/risk-compare` | Side-by-side comparison of risk scorers and shadow filters across candidate tokens. |
| **Signals & ML** | `/dev/graph-edge` | Interactive Graph Relational Edge canvas with radial expansion and 1-click inspection. |
| **Infra & LP** | `/dev/dlmm` | Meteora DLMM Hunter/Healer console (Solana) + Robinhood v3/v4 CLMM ledger tabs. |
| **Infra & LP** | `/dev/pools` & `/dev/pools-test` | Liquidity pool screening, net RR screening, and pool testing utilities. |
| **Infra & LP** | `/dev/arbitrage` | Curated triangular Solana arbitrage (`SOL → A → B → SOL`) quote scanner and executor. |
| **Infra & LP** | `/dev/rpc-tester` | Live RPC endpoint latency, slot health, index health, and send-telemetry tester. |
| **Search** | `/dev/search-token` | Cross-chain token search (`/solana`, `/robinhood`) + `TokenLocateHub` map (`/dev/search-token/detail?address=&view=`). |

---

## `/admin` Evaluation Directory (Read-Only)

| Route | What It Evaluates |
| :--- | :--- |
| `/admin/graph-edge` | Multi-relational knowledge graph, cabal ring detector, deployer motifs, and strategy consensus cliques. |
| `/admin/ml` | Live ML model status, artifact provenance, canary traffic %, and every unmet readiness bar preventing enforcement. |
| `/admin/system` | Container health, DB/PgBouncer pool health, Go cron worker status, entry-context coverage, re-entry guards, and stake compliance. |
| `/admin/strategy` | Per-strategy net/gross P&L, win rate, payoff ratio, stop-loss hit distribution, and stake compliance. |
| `/admin/lifecycle` | 4-Phase Viral Runners candidate queue, live Macro Market Regime banner, KG badges, bundler %, and granular in-flight exit reasons. |
| `/admin/rug` | Ban precision and recall by layer, 1m OHLC coverage, verdict coverage, and close-reason breakdown. |
| `/admin/review` | Insert-only human review queue for severe losses (`≤ -15%` net) and missed rugs (`tb_v1`), feeding `ml_label` back into training exports. |
| `/admin/replay` | Historical exit-rule replay engine over ReloadSOL's own 1m `token_ohlc_bars` (net of fees, paired scoring). |
| `/admin/runners` & `/admin/transactions` | Daily runners telemetry and platform-wide transaction / portfolio activity diagnostics. |
