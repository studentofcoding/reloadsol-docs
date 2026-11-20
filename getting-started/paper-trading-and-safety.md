---
description: How reloadSol protects traders before signing transactions and how paper trading models real-world execution costs.
---

# Paper Trading & Transaction Safety

Trading on-chain carries irreversible execution risk. reloadSol implements strict pre-signature verification for live trades and an empirically calibrated cost model for paper (simulated) trading so that test performance reflects real-world execution.

---

## Pre-Signature Transaction Safety (Solana)

reloadSol enforces five non-negotiable invariants on every Solana transaction handed to your wallet (codified in `SPEC-phantom-simulation-hardening-v1`):

1. **Exact-Bytes Simulation (`simulateExact`):** Every swap leg, close-account chunk, rent reclaim, and fee transfer is simulated using the **exact serialized bytes** and the **real recent blockhash** (`sigVerify: false`, never `replaceRecentBlockhash: true`). What our pre-flight check simulates is byte-for-byte what Phantom or Solflare simulates.
2. **Stale Blockhash Auto-Rebuild:** If a simulation fails due to a stale blockhash, reloadSol rebuilds the transaction once automatically—it never hands a stale transaction to your wallet.
3. **Zero `skipPreflight: true` Bypasses:** Every RPC broadcast runs with preflight simulation enabled.
4. **Split-on-Revert Before Signing:** When closing multiple dust or zero-balance accounts, if a packed chunk fails simulation, reloadSol splits and isolates the reverting account *before* calling `signAllTransactions`, preserving a single clean wallet approval for all healthy accounts.
5. **Message-Equality Guard:** After wallet signing, reloadSol verifies whether the wallet modified the transaction message (for example, refreshing the blockhash) before deciding whether to route through Jupiter's managed `/execute` endpoint or direct RPC broadcast.

### Balance Reserve Protection (Bulk Buy)

Before preparing a Solana bulk buy, reloadSol checks `requiredBuySol`, reserving not only your swap spend and priority fees, but also **ATA rent (`2,039,280 lamports` ≈ `0.00204 SOL` per new token account)** and **base transaction fees** (`5,000 lamports` per signature). This prevents partial batch failures mid-execution caused by running out of SOL for account creation.

{% hint style="success" %}
**Fail-Open Platform Fee Protection:** If a platform fee instruction fails simulation or cannot be built, reloadSol drops the fee leg and proceeds with your trade rather than blocking or reverting your swap.
{% endhint %}

---

## Pre-Signature Safety (Robinhood Chain / EVM)

* **Atomic `BatchExecutor.sol`:** The batch executor contract is owner-scoped, immutable, pausable, and uses plain `call` (no `delegatecall`), ensuring arbitrary code cannot execute in the contract's storage context.
* **One-Time Permit2 Setup:** Approving tokens to canonical Permit2 (`0x0000...8BA3`) does not move funds; every trade still requires your explicit signature.
* **Strict Receipt Verification:** Both EIP-5792 (`wallet_sendCalls`) and sequential fallback paths inspect every mined receipt's `status`. A mined-but-reverted transaction is always recorded as `failed`, never `confirmed`.

---

## Realistic Paper Trading & Execution Cost Model

reloadSol supports paper trading (`is_simulation: true`) both for manual experimentation and for automated strategy validation (`/dev/paper-trade`, `/dev/algo-tester`).

Unlike naive simulators that assume zero-friction mid-price fills, reloadSol applies a calibrated execution cost model (`src/strategies/execution-model.ts` and `src/strategies/paper-fee.ts`):

| Cost Component | Default Value | Empirical Basis |
| :--- | :--- | :--- |
| **AMM / Swap Fee (`SIM_FEE_BPS`)** | `12 bps` per side (~`24–26 bps` round trip) | Measured against live Solana DEX round-trip fills |
| **Spread (`SIM_SPREAD_BPS`)** | `0 bps` | On AMM routes, pool fee + price impact captures execution cost without double-counting |
| **Priority Fee (`SIM_PRIORITY_FEE_QUOTE`)** | `0.00003 SOL` (30,000 lamports) | Matches actual production priority fee sent on-chain |
| **Price Impact (`SIM_IMPACT_COEFF`)** | Constant-product average-price impact (`coeff = 1, exp = 1`) | Scales impact realistically with trade size vs. pool liquidity |
| **Flat Paper Position Fee (`PAPER_FEE_SOL_PER_POSITION`)** | `0.0003 SOL` per round-trip close | Applied once on Solana sim closes inside `insertStrategyOutcome` |

### Why Stake Size Matters (Fixed-Cost Drag)

Because network fees and priority tips are **flat per transaction** regardless of position size, tiny trades face severe percentage drag:

$$\text{Fee Drag (\%)} = \frac{\text{Round-Trip Fixed Fee (SOL)}}{\text{Position Stake (SOL)}} \times 100$$

| Position Stake | Flat Fee (`0.0003 SOL`) | Break-Even Hurdle |
| :--- | :--- | :--- |
| `0.0025 SOL` | `0.0003 SOL` | **12.0%** just to break even |
| `0.0100 SOL` | `0.0003 SOL` | **3.0%** |
| `0.0500 SOL` | `0.0003 SOL` | **0.6%** |

{% hint style="info" %}
**Net vs. Gross Reporting:** In `strategy_outcomes`, `pnl_pct` is stored **net of fees**, while `gross_pnl_pct`, `fee_sol`, `fee_pct`, and `fee_model` are preserved inside `features` so analysts can audit both raw signal quality and net-of-cost expectancy.
{% endhint %}

---

## Interactive Fee-Drag & Payoff-Ratio Calculator

Adjust **Position Stake (SOL)**, **Average Win %**, and **Average Loss %** to see how flat network/paper fees (`0.0003 SOL`) affect your net payoff ratio ($R$) and break-even win rate:

{% embed url="https://studentofcoding.github.io/reloadsol-docs/widgets/index.html?view=calculator" %}
Interactive reloadSol Fee-Drag & Payoff-Ratio Calculator
{% endembed %}
