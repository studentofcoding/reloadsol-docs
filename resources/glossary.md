---
description: Authoritative A–Z glossary of financial trading, DeFi execution, risk management, and reloadSol platform terminology.
---

# Trading & Platform Glossary

Clear definitions of every trading, DeFi, and quantitative term used across reloadSol.

---

### A

* **ATA (Associated Token Account):** A deterministic Solana account holding a specific SPL or Token-2022 token for your wallet. Each ATA locks `2,039,280 lamports` (`~0.00203928 SOL`) in rent-exempt reserve, which reloadSol reclaims when you close empty or burned accounts.
* **Atomic Batching:** Executing multiple on-chain operations (e.g., ETH wrap + Permit2 pull + 5 swaps) inside one transaction so either every step succeeds together or the entire batch reverts cleanly.

### B

* **Basis Points (bps):** One hundredth of one percent (`1 bps = 0.01%`, `100 bps = 1.00%`). Used for swap fees (`12 bps`), platform fees (`25 bps`), and slippage (`50–1,000 bps`).
* **BatchExecutor (`BatchExecutor.sol`):** reloadSol's owner-scoped, immutable smart contract on Robinhood Chain (`4663`) that executes multi-token Kyber swaps in a single signature.
* **Bound Wallet:** A server-configured EVM wallet (using `GMGN_PRIVATE_KEY`) that executes GMGN Trade API orders for dev/operator flows, as opposed to a browser-signed **Parent Wallet**.
* **Break-Even Win Rate:** The minimum win rate required for a strategy to break even given its Payoff Ratio ($R$): $\text{BEWR} = \frac{1}{1 + R}$.
* **Bundler Concentration (`bundlers_hold_pct`):** Percentage of a token's supply acquired by wallets coordinated inside the same launch block/slot. Empirical reloadSol data shows `≥ 35%` bundler concentration carries a `73.65%` probability of a `≥ 50%` dump within 30 minutes.

### C

* **Circuit Breaker (`bot_trading_state`):** Automated live-trading kill switch that halts real capital execution after `N` consecutive failures (default `3–5`) for a cooldown period (`20–30 min`).
* **Climate Gate:** Market-wide posture (`Hype`, `Range`, `Mixed`, `De-risk`, `Cash`) published by reloadSol's Market Brain to scale position sizes from `1.00×` down to `0.00×`.
* **CLMM / DLMM:** Concentrated Liquidity Market Maker (Uniswap v3/v4 on Robinhood Chain) and Dynamic Liquidity Market Maker (Meteora on Solana)—liquidity pools where capital is concentrated within discrete price ticks or bins to earn higher swap fees, at the risk of going **Out-of-Range (OOR)**.
* **Counterfactual Book:** Ledger tracking forward 1m OHLC outcomes of *rejected* trade candidates to measure whether a risk filter prevented real losses or missed winners.

### D

* **Dev DNA (`dev_reputation`):** Historical provenance score of a token's deployer wallet, including past launch count, graduation ratio, top-10 ATH tokens, and confirmed rugs.
* **Drawdown:** Peak-to-trough percentage decline in a position or portfolio's value.
* **Dust:** Small-value token remainders left in a wallet after partial fills or price declines that are tedious to swap individually.

### E

* **EIP-5792 (`wallet_sendCalls`):** Ethereum wallet standard allowing dApps to submit multiple calls in one atomic wallet request when supported by the user's wallet.
* **Enforce vs. Shadow Mode:** In **Shadow Mode**, an ML classifier or risk gate scores every candidate and logs its decision without blocking trades. In **Enforce Mode** (enabled only after readiness bars and canary validation pass), the gate actively rejects failing candidates.

### I

* **In-Flight OHLC 3-Pattern Defense:** reloadSol's active exit protection combining Pattern A (3-bar structural breakdown early cut), Pattern B (asymmetric warn clamp), and Pattern C (3-stage dynamic ratchet to breakeven and 2-bar low runner trail).

### K

* **Knowledge Graph (SHACL Gate):** Relational graph linking deployers, funders, holders, pools, and social callers to detect 2-hop sybil rings, recycled rug LP, and alpha wallet cliques.

### M

* **MAE / MFE (Maximum Adverse / Favorable Excursion):** The deepest unrealized drawdown (MAE) and highest unrealized peak gain (MFE) reached during a trade's lifetime.
* **Macro Market Regime:** Classification of broader crypto tape (`MOMENTUM_RUNNER`, `NORMAL_CHOP`, `DEFENSIVE`) derived from 5-minute BTC/SOL returns and 7-day realized volatility.

### O

* **OHLC:** Open, High, Low, Close candlestick price data. reloadSol maintains its own 1-minute `token_ohlc_bars` table with provenance tracking (`source`) to separate real candles from `no_trade` windows.
* **OOR (Out-of-Range):** State of a DLMM/CLMM liquidity position when spot price moves outside the position's active bin/tick range, stopping fee accrual.

### P

* **Payoff Ratio ($R$):** Ratio of average winning trade percentage to absolute average losing trade percentage ($R = \frac{\text{Avg Win}}{\text{|Avg Loss|}}$).
* **Permit2 (`0x0000...8BA3`):** Canonical EVM token approval contract enabling single-signature swaps and batches after one-time token approval.
* **Price Impact:** The percentage shift in execution price caused by your order's size relative to the DEX pool's available liquidity (capped at `15%` by default on reloadSol).

### R

* **Realized vs. Unrealized P&L:** **Realized P&L** is locked-in profit or loss from closed legs (net of fees); **Unrealized P&L** is the live mark-to-market gain or loss on currently held tokens.
* **Receipt-Gated Confirmation:** Architecture where trades stay `pending` until an on-chain receipt confirms `status = success`, ensuring reverted transactions are never shown as confirmed.

### S

* **Slippage:** Maximum acceptable difference between quoted price and actual on-chain fill price (dynamically scaled `50–1,000 bps` on reloadSol).
* **SL/TP (Stop-Loss / Take-Profit):** Pre-defined price/percentage thresholds that automatically close a position to cap downside loss or lock in upside gain.

### T

* **TCA (Transaction Cost Analysis):** Measurement of total execution friction—AMM fees, slippage, priority fees, and landing rates (`swap_tca`).
* **Token-2022 (SPL Token Extensions):** Upgraded Solana token standard supporting transfer hooks, transfer fees, and metadata extensions; handled natively in reloadSol's burn-close packing.
