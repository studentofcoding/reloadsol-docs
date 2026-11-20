---
description: Complete catalog of reloadSol's REST and SSE API routes, authentication tiers, rate limits, and service-to-service headers.
---

# REST & SSE API Reference

reloadSol exposes 200+ Next.js App Router endpoints under `/api/*`, governed by a fail-closed, four-tier access control model (`src/config/api-access.ts`, `src/utils/api-auth.ts`). Any unclassified `/api/*` route defaults to **`wallet`** tier.

{% hint style="tip" %}
**Interactive OpenAPI Playground:** This repository ships an OpenAPI 3.1 specification at [`openapi.yaml`](../openapi.yaml). Connect it in GitBook (**Integrations → OpenAPI**) to test public endpoints (`/api/health`, `/api/solprice`, `/api/wallet/preview`, `/api/regime/climate`, `/api/rpc/diagnostics`) live from the docs.
{% endhint %}

---

## 1. Authentication Tiers

| Tier | Auth Mechanism | Example Routes |
| :--- | :--- | :--- |
| **`public`** | No session required (display payloads, rate-limited ingests, health checks). | `/api/health`, `/api/solprice`, `/api/ethprice`, `/api/tokens/*`, `/api/regime/climate`, `/api/wallet/preview`, `/api/rpc/health`, `/api/rpc/diagnostics`, `/api/operations/last-reload`, `POST /api/operations/ask-for-fund`, `POST /api/analytics/events`, `/api/analytics/knowledge-graph`, `/api/jupiter/metadata` (GET/POST). |
| **`wallet`** (Default) | Signed Solana wallet session cookie (`WALLET_SESSION_SECRET`) or verified `0x` EVM address on RH-compatible routes. | `/api/rpc`, `/api/jupiter/*`, `/api/buy`, `/api/trading/records`, `/api/trading/subscribe`, `/api/watchlist`, `/api/operations/*`, `/api/trade/compare`, `/api/prices/open`. |
| **`dev`** | Dev-wallet whitelist session (`DEFAULT_DEV_WALLETS` / `NEXT_PUBLIC_DEV_WALLETS`). | `/api/strategies/*`, `/api/dlmm/*`, `/api/signals/*`, `/api/gmgn/*`, `/api/social/*`, `/api/sol-arb/*`, `/api/workers/*`, `/api/analytics/*`, `/api/mcap-tracking/*`, `/api/pnl/*`, `/api/logs/*`, `/api/dev/reputation`. |
| **`service` / `self-auth`** | Handler-verified secret: `?key=<TRENDING_TRACKER_SECRET>`, `Authorization: Bearer`, `X-Trigger-Secret`, or route-specific token (e.g., `PNL_UPDATE_SECRET`, `SOCIAL_INGEST_SECRET`, `RELOADSOL_READONLY_TOKEN`). | `/api/sl-tp-monitor`, `/api/ohlc/sample`, `/api/metrics/copy`, `/api/ml/pattern/reload`, `/api/gmgn/activity-poll`, `/api/social/ingest`, `/api/desktop/snapshot`, `/api/evidence/archive`. |

---

## 2. Public & Trader Endpoints

### Swaps, Quotes & RPC
| Endpoint | Method | Tier | Description |
| :--- | :--- | :--- | :--- |
| `/api/swap/route-config` | `GET` | `wallet` | Delivers runtime bulk-routing flags (`mode`, `shadow`) and `platformFeeSettleMode` to the browser. Strictly `DEFAULT_API_ACCESS_TIER` — it is in no public list, so signed-out callers get `401` and keep the incumbent lane. Exists because non-`NEXT_PUBLIC_*` env flags are inlined as `undefined` in the client bundle, so a flag read inside the browser build path would never arm (DC-36); the response is cached per page load and fails open to the incumbent lane on error, so it can never block a buy. |
| `/api/jupiter/quote` · `/api/jupiter/lite/*` | `GET` / `POST` | `wallet` | Proxies Jupiter Swap V2 `/order`, `/execute`, Portfolio, and Lite fallback. |
| `/api/solanatracker/{quote,swap,send,health}` | `GET` / `POST` | `wallet` / `dev` | Proxies SolanaTracker Raptor estimates and arb routes. |
| `/api/kyber/routes` · `/api/kyber/build` | `GET` / `POST` | `wallet` | Proxies KyberSwap aggregator routes and calldata builds on Robinhood Chain `4663`. |
| `/api/gmgn/trade/{quote,swap,order}` | `GET` / `POST` | `dev` | Bound-wallet GMGN execution (`swap` enforces `from` = bound address). |
| `/api/rpc` · `/api/rh/rpc` | `POST` | `wallet` | Same-origin Solana (Shyft/Helius failover) and Robinhood `4663` JSON-RPC proxies. |
| `/api/rpc/health` · `/api/rpc/diagnostics` | `GET` | `public` | Endpoint latency, `send-only` status, and `?send=1` landing telemetry from `swap_tca`. |

### Holdings, Prices, Portfolio & Real-Time
| Endpoint | Method | Tier | Description |
| :--- | :--- | :--- | :--- |
| `/api/wallet/preview?address=<sol>` | `GET` | `public` | Signed-out wallet balance, sellable token value, and reclaimable ATA rent snapshot. |
| `/api/operations/ask-for-fund` | `POST` | `public` | Server-verified gas assistance request for 0-SOL wallets holding recoverable assets. |
| `/api/shyft/wallet/all_tokens` | `GET` | `wallet` | Cached Solana token balances (`15s` fresh / `120s` stale; `fresh=1` bypasses cache). |
| `/api/rh/wallet-tokens` | `GET` | `wallet` | Robinhood Chain holdings (GMGN → Blockscout → RPC fallback + quote asset injection). |
| `/api/tokens/prices` · `/api/solprice` · `/api/ethprice` | `GET` | `public` | Batched multi-chain USD token prices (Solana + 42-char EVM) and live native prices. |
| `/api/trading/records?wallet=&chain=&limit=` | `GET` / `POST` | `wallet` | Per-wallet, per-chain trade history (`trading_records`). |
| `/api/trading/subscribe?wallet=<addr>` | `GET` / `POST` | `wallet` | Real-time SSE stream (`trade_update`, `pnl_update`, `balance_update`). |
| `/api/watchlist` | `GET` / `POST` / `DELETE` | `wallet` | Cross-chain watchlist CRUD (accepts Solana session or EVM `0x` address). |

---

## 3. Developer, Quant & Automation Endpoints

| Domain | Key Endpoints | Purpose |
| :--- | :--- | :--- |
| **Strategies** | `GET /api/strategies`, `PATCH /api/strategies/[id]`, `GET/PATCH /api/strategies/outcomes`, `GET /api/strategies/reports`, `POST /api/strategies/search-cycle`, `GET /api/strategies/positions`, `GET /api/strategies/lifecycle` | Registry merge, config updates (deactivate auto-closes positions), outcome labeling, precomputed reports, walk-forward search, lifecycle queue. |
| **Execution & Exits** | `POST /api/sl-tp-monitor`, `POST /api/signals/sim-track`, `POST /api/mcap-tracking/sim-track`, `POST /api/trending/track`, `POST /api/gmgn/sim-track`, `POST /api/social/sim-track` | Cron-driven discovery, paper/live execution, and unified SL/TP closing. |
| **DLMM / CLMM** | `POST /api/dlmm/{screen,sim-track,manage,rh-clmm-manage}`, `GET /api/dlmm/positions` | Meteora Hunter/Healer cycles and Robinhood CLMM Multicall3 alert monitor. |
| **Arbitrage** | `GET /api/sol-arb/scan`, `/api/sol-arb/{quote,execute,execute-atomic}` | Curated SOL triangular arbitrage scanner and gated executor. |
| **ML & Rug** | `GET /api/ml/{readiness,monitor-report,forward-tracking}`, `POST /api/ml/pattern/reload`, `GET /api/mcap-patterns/training-export`, `/api/rug-signal/{calibrate,separation,shadow,v2-shadow,verdicts}` | ML dataset export, ONNX hot-reload, readiness stats, and Rug v2 calibration/soak. |
| **Knowledge Graph** | `/api/analytics/knowledge-graph`, `POST /api/graph/shadow-tick` | Ontodia visual graph API and background shadow deployer/bundler evaluation. |
| **Workers & Ops** | `GET /api/workers/{status,runtime}`, `POST /api/workers/trigger`, `POST /api/pnl/{update,daily-report}`, `POST /api/ohlc/sample`, `POST /api/metrics/copy`, `POST /api/counterfactual/tick` | Go cron status proxy, manual worker triggers (`X-Trigger-Secret`), OHLC/metrics samplers, and counterfactual book. |
