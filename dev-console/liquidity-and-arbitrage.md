---
description: Concentrated liquidity provisioning on Meteora DLMM (Solana), alert-only CLMM monitoring on Robinhood Chain, pool screening, and triangular SOL arbitrage.
---

# Meteora DLMM, Robinhood CLMM & Arbitrage

Beyond directional spot trading, ReloadSOL includes non-directional yield harvesting and market-neutral arbitrage engines under `/dev/dlmm`, `/dev/pools`, and `/dev/arbitrage`.

---

## 1. Meteora DLMM Agent (`/dev/dlmm` — Solana)

ReloadSOL operates an automated **Hunter + Healer** liquidity agent for Meteora Dynamic Liquidity Market Maker (DLMM) pools:

### Hunter (`dlmm_screen`, every `300s`)
* Queries Meteora pools **sorted by `tvl:desc`** across paginated responses (avoiding dust pools with inflated fee/TVL denominators).
* Scores each pool via `scorePool × solDlmmConfidence` against minimum TVL (`$50,000` default) and `fee/TVL` thresholds (`0.1`), persisting top candidates to `dlmm_candidates` (`chain = 'sol'`) with Redis last-good fallback on empty screens.

### Auto-Deploy (`dlmm_sim_track`, every `300s`)
* Automatically deploys paper positions (`dry_run` mode) onto top-ranked candidates while enforcing outcome-keyed reopen cooldowns (`60m` pool / `1,440m` token).
* Supports the `dlmm_range_fee_accumulator` playbook (`±15%` concentrated range targeting `≥ 15%` daily fee yield on range-bound mid-cap chop).

### Healer (`dlmm_manage`, every `60s`)
* Evaluates every open, out-of-range (OOR), or pending position through `src/utils/dlmm/reasoner.ts`.
* Decides whether to hold in-range, claim fees, redeploy, or close based on configured guardrails:
  * **Take-Profit:** `+5%`
  * **Stop-Loss:** `-10%`
  * **Out-of-Range Timeout:** `16 minutes`
* Dispatches decision alerts via `sendDlmmDecisionAlert` (Telegram) and writes closed outcomes via `recordDlmmOutcome`.
* Can be paused independently via Telegram `/pause` (`dlmm_agent_config.enabled`).

---

## 2. Robinhood Chain CLMM Monitor (`/dev/dlmm` — RH Tabs)

For Uniswap v3 and v4-fork Concentrated Liquidity Market Maker (CLMM) positions on **Robinhood Chain (`4663`)**, ReloadSOL runs `rh_clmm_manage` every `300s`:

* **Strict Alert-Only Mode:** Uses batched **Multicall3** RPC reads to check position range status (in-range vs. out-of-range) and claimable fees across the `dlmm_positions` RH ledger (`pool_key`, `fee`, `tick_spacing`).
* **Zero Autonomous Signing:** Sends Telegram OOR and fee-claim alerts only—it performs no on-chain writes or automated rebalancing.

---

## 3. Pool Screening & Testing (`/dev/pools` & `/dev/pools-test`)

* **Pool Screener (`/dev/pools`):** Inspects raw candidate pools, TVL depth, 24h fee generation, net risk/reward screening (`SPEC-lp-risk-reward-gate-v1`), and staleness indicators (`candidate-freshness.ts`).
* **Pool Utilities (`/dev/pools-test`):** Diagnostic sandbox for testing pool quote adapters and liquidity reads.

---

## 4. Triangular SOL Arbitrage (`/dev/arbitrage` — Solana Only)

ReloadSOL includes a curated triangular arbitrage scanner (`SOL → Token A → Token B → SOL`) backed by SolanaTracker Raptor (`maxHops` set) and `/api/sol-arb/*`:

* **Scanner (`sol_arb_scan`, every `60s`):** Evaluates curated pairs via `GET /api/sol-arb/scan` and fires a Telegram alert whenever net profit exceeds `SOL_ARB_MIN_EDGE_LAMPORTS` after fees and slippage.
* **Execution:** Manual or automated execution via `/api/sol-arb/quote`, `/api/sol-arb/execute`, and `/api/sol-arb/execute-atomic` is strictly gated behind `SOL_ARB_LIVE_ENABLED` (default OFF).

{% hint style="warning" %}
**No Jito Bundle Support on Standard RPC:** Measured production benchmarks confirm that standard SolanaTracker/Shyft/Helius endpoints return `method not found` for `sendBundle`. Any tip sent through standard RPC is a plain transfer, not private MEV protection—factor public mempool competition into arbitrage edge thresholds.
{% endhint %}
