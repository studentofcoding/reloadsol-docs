---
description: How reloadSol adapts strategy timing, volatility gates, and position sizing to macro crypto regimes, brain climate states, and probabilistic ML sizing.
---

# Macro Market Regime & Position Sizing

A momentum strategy that thrives during a high-volume Solana bull sprint will bleed out during a choppy or risk-off drawdown if it trades with fixed parameters and full size. reloadSol conditions **entry timing**, **consolidation tightness**, **stop-loss clamps**, and **position sizing** on live macro market conditions.

---

## 1. Tier 1: Macro Market Regime (`market-regime.ts`)

Computed from stored 5-minute BTC and SOL `market_candles` (1h/24h returns, 7-day realized volatility terciles) plus the daily market brain climate—strictly using candles **before** entry time to prevent lookahead bias:

| Macro Regime | Market Character | Lifecycle Delay Scaling | Compression Gate (`post_pump_base`) | Guard Stop-Loss Clamp |
| :--- | :--- | :--- | :--- | :--- |
| **`MOMENTUM_RUNNER`** | Strong positive BTC/SOL trend, healthy breadth | **`+45m` to `+60m`** ($0.5\times$ faster maturation) | Standard ($\le 25\%$ range) | Standard strategy stop |
| **`NORMAL_CHOP`** | Sideways / range-bound tape | **`+120m`** ($1.0\times$ baseline) | Standard ($\le 25\%$ range) | Standard / warn clamp |
| **`DEFENSIVE`** | Negative BTC/SOL impulse or elevated crash volatility | **`+180m`** ($1.5\times$ patient filter) | Tightened (**$\le 18\%$** range) | Tightened: **`-10%` (`pass`)** / **`-7%` (`warn`)** + **`0.50×`** size |

Every closed `strategy_outcomes` row preserves **two** distinct regime timestamps:
* **Entry-time regime (`features.market_regime_at_entry`):** What was knowable at the exact moment the trade opened.
* **Close-time regime (`market_regime_tags`):** The market climate when the position exited.

---

## 2. Climate Gate & Stake Scaling (`/api/regime/climate`)

reloadSol's Market Brain publishes a real-time climate state consumed by the header indicator, `/dev/insight`, and automated paper/live sizing:

| Climate State | `sizeScale` Multiplier | Trading Posture |
| :--- | :--- | :--- |
| **Hype** | **`1.00×`** | Full base position size |
| **Range** | **`0.75×`** | Moderately reduced size |
| **Mixed** | **`0.50×`** | Half size; selective entries |
| **De-risk** | **`0.25×`** | Quarter size; capital preservation |
| **Cash / Cascade Veto** | **`0.00×`** | Stand down — block new entries |

{% hint style="warning" %}
**DLMM Climate Gate (`CLIMATE_GATE`):** Optional flag (default OFF). When enabled, stand-down/cascade states block new Meteora DLMM paper opens and scale size by `sizeHint`. Kill switches, daily loss limits, and capital caps always take precedence. Live DLMM capital is never climate-gated unless `CLIMATE_GATE_LIVE=1` is explicitly enabled.
{% endhint %}

---

## 3. Position Sizing & Capital Caps

### Soft ML Sizing (`ml-soft-size.ts`)
For `mcap_tracker`, `signals`, and `gmgn` lanes, reloadSol applies **soft sizing** rather than hard binary blocking while classifiers remain in shadow calibration:

$$\text{Effective Size} = \text{Base Stake} \times \text{Climate Scale} \times \max\Big(\text{SOL\_ML\_SIZE\_FLOOR},\ (1 - p_{\text{bad}}) \times \text{confidence}\Big)$$

* Default `SOL_ML_SIZE_FLOOR = 0.25` (scales size down on weak setups without dropping the trade from shadow telemetry).

### Portfolio & Live Capital Safeguards
Before any automated live order is signed via server keypair, reloadSol enforces:

1. **`MAX_SOL_AT_RISK`:** Caps total simultaneous live SOL deployed across all open positions (evaluated against `peakConcurrentPositions` in `/dev/paper-trade` before arming a strategy).
2. **`MIN_SOL_BALANCE`:** Preserves a native SOL buffer for gas and emergency exits.
3. **Fixed-Cost Sizing Floor:** Because a Solana round-trip costs ~`0.00006`–`0.0003 SOL` in flat network/priority fees, live stakes should sit at **`≥ 0.006 SOL`** (ideally `≥ 0.01 SOL`) so fixed transaction costs stay under `1%–3%` of position value.
4. **Global Circuit Breaker (`bot_trading_state`):** Automatically sets `real_trading_halted = true` after `BOT_TRADING_FAILURE_THRESHOLD` (default `3–5`) consecutive execution failures and holds for `BOT_TRADING_HALT_MINUTES` (default `20–30 min`), auto-resetting on timer expiry or next confirmed success.
