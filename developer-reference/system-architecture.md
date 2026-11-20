---
description: Production Docker Compose topology, Next.js + Go + Postgres/PgBouncer + Redis architecture, core database schema, caching, and SSE data flows.
---

# System Architecture & Data Layer

reloadSol runs as a self-contained **Docker Compose stack** on a single production VPS (`flowey-vps`) behind Cloudflare Full (Strict) TLS and an Nginx edge proxy. All application, strategy, and telemetry data lives in Docker Postgres (`reloadsol_db`).

---

## 1. Production System Topology

```mermaid
flowchart TB
    Browser["Browser Clients<br/>(Phantom / Rabby · EIP-6963)"] -->|HTTPS / Full Strict TLS| CF["Cloudflare Edge<br/>(Real-IP · Cache Rules)"]
    CF -->|Port 80/443 · Host: reloadsol.app| Nginx["reloadsol-nginx<br/>(nginx:1.27 · 64M · 444 on direct IP)"]
    Nginx --> Web["reloadsol-web<br/>(Next.js 16 App Router · 768M)<br/>UI · API · ONNX ML (ro mount)"]
    Cron["reloadsol-cron<br/>(Go Scheduler · 128M · :8080)"] -->|POST /api/* (Service Auth)| Web
    Social["reloadsol-social-ingest<br/>(Python Telethon · 256M)"] -->|POST /api/social/ingest| Web
    Web --> Bouncer["reloadsol-bouncer<br/>(PgBouncer · Transaction Pool · 64M)"]
    Bouncer --> DB[("reloadsol-db<br/>Postgres 16-alpine · 768M")]
    Web --> Redis[("reloadsol-redis<br/>Redis 7-alpine · 96M LRU")]
    Web -->|RPC / Swaps| Chains["Solana Mainnet & Robinhood Chain 4663"]
```

### Container Inventory

| Service | Image / Build | Memory Cap | Role |
| :--- | :--- | :--- | :--- |
| **`reloadsol-nginx`** | `nginx:1.27-alpine` | `64 MB` | Public edge (`:80` / `:443`), Cloudflare Origin CA TLS, real-IP restoration (`01-cloudflare-realip.conf`), drops direct-to-IP requests with `HTTP 444`, `/api/` `proxy_read_timeout 120s`. |
| **`reloadsol-web`** | `Dockerfile.web` (`node:20-slim` runner) | `768 MB` | Serves pre-built `.next/standalone` SSR UI, 200+ API routes, server actions, and `onnxruntime-node` ML inference (`./ml/artifacts:/app/ml/artifacts:ro`). |
| **`reloadsol-cron`** | `Dockerfile.cron` (`golang:1.22` → `alpine:3.19`) | `128 MB` | Go scheduler driving **30** registered background workers; exposes `:8080` (`/health`, `/status`, `/workers`, and `X-Trigger-Secret`-guarded `/trigger/*`). |
| **`reloadsol-bouncer`** | `edoburu/pgbouncer:latest` | `64 MB` | Transaction-mode connection pool (`scram-sha-256`, max 200 clients). App connects via `DATABASE_URL` → `reloadsol-bouncer:5432/reloadsol_db`. |
| **`reloadsol-db`** | `postgres:16-alpine` | `768 MB` | Primary relational store (`reloadsol_db`). No public host port published in production. |
| **`reloadsol-redis`** | `redis:7-alpine` | `128 MB` (`96 MB` LRU) | Shared API caches, `records:<wallet>:*` cache, job locks, alert throttles, RH CLMM pool-state (15s TTL). Automatic in-memory fallback when absent. |
| **`reloadsol-social-ingest`** | `social-ingest/Dockerfile` (Python Telethon) | `256 MB` | Real-time Telegram channel listener extracting CAs and signal cards → `POST /api/social/ingest` and `/api/social/crosscheck`. |

---

## 2. Core Postgres Schema (`db/init/*.sql`)

Schema migrations are numbered, additive, and idempotent (`00-extensions.sql` through `97-forward-first-touch.sql` — **101** files today), verified by `npm run db:check-migrations` (double-apply on a throwaway DB) and `npm run verify:sql` (`PREPARE` sweep across 550+ SQL statements; live result 2026-10-27 **557 statements · prepared 557 · failed 0**).

| Domain | Key Tables | Notes |
| :--- | :--- | :--- |
| **User Trading & Portfolio** | `trading_records` (JSONB `data` + `chain` index), `token_operations`, `wallet_watchlist` | Strictly `wallet:chain` scoped (`23-app-network-chain.sql`). |
| **Strategies & Outcomes** | `strategy_definitions`, `strategy_outcomes` (incl. `fill_price`, `slippage_bps`, `fees_sol`), `sl_tp_positions`, `strategy_report_precompute` | `strategy_outcomes` written only on full close; `pnl_pct` is net of fees on Solana sims. |
| **Bot Safety & Locks** | `bot_job_locks`, `bot_trade_locks`, `bot_trading_state` | Supports container-scoped lock reclamation (`LOCK_INSTANCE_SCOPE`) and global circuit breaker. |
| **Market Data & OHLC** | `token_mcap_tracking`, `token_ohlc_bars` (1m), `token_anchor_bars` (95), `token_metrics_history` (hourly 1m arrays), `market_candles` (5m BTC/SOL), `market_regime_tags` | `token_ohlc_bars` preserves provenance (`source`) to distinguish `no_trade` windows from real flat candles (migration `96`). |
| **Risk, Graph & Social** | `token_info_detect`, `token_risk_features`, `dev_reputation`, `rug_v2_shadow`, `social_token_events`, `social_token_rollups`, `tracked_wallets`, `bot_call_context`, `token_forward_tracking` | Social & `token_info_detect` are Solana-scoped by design. |
| **Liquidity & Telemetry** | `dlmm_agent_config`, `dlmm_candidates`, `dlmm_positions`, `swap_tca` (94), `analytics_events`, `counterfactual_book` (86) | Dual-dispatch UI analytics (`analytics_events` + zero-PII Simple Analytics `sa_event`). |

---

## 3. Caching, Price Integrity & Real-Time SSE

* **Single Live Native Price Source:** `getSolPriceUSDCore()` (`Bybit → CoinGecko / Jupiter`, 30s cache + 5m stale) exposed via `/api/solprice` and `useSolPrice()`. Returns `price: 0, source: 'unavailable'` (HTTP `503`) if all upstreams fail—hardcoded fallback prices are forbidden and enforced in CI (`npm run verify:no-hardcoded-sol-price`).
* **Trading Records Cache Invalidation:** Every insert/update/delete calls `invalidateTradingRecordsCache(wallet)` (clears in-memory LRU + `DEL records:<wallet>:*` in Redis) and `updateTag(CACHE_TAGS.records(wallet))`, followed by `broadcastTradeUpdateServer` (`POST /api/trading/subscribe`).
* **SSE Hub (`GET /api/trading/subscribe?wallet=<addr>`):** Holds one live stream per wallet (15s keepalive, 30s cleanup sweep) broadcasting `trade_update`, `pnl_update`, and `balance_update`.
