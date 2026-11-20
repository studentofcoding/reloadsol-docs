---
description: Multi-layer rug detection, deployer provenance ("Dev DNA"), bundler concentration vetoes, Brand Impersonation filters, and Semantic Knowledge Graph SHACL gates.
---

# Rug Detection, Dev Reputation & Knowledge Graph

On-chain micro-cap markets experience extreme adversarial launch rates (empirical data shows >`90%` of raw bonding-curve launches fail). ReloadSOL defends both manual traders and automated strategies with a layered risk stack that combines **single-token telemetry** with **multi-hop relational graph intelligence**.

---

## 1. Multi-Layer Rug Defense Stack

| Layer | Mechanism | Scope | Mode |
| :--- | :--- | :--- | :--- |
| **Layer 1: Hard Security & Liquidity Gate** | Checks mint/freeze authority, top-holder concentration, pool TVL ($\ge \$5,000$), and LP burn/lock ($\ge 90\%$). | GMGN, Signals, Robinhood, Bulk Buy | **Active** |
| **Layer 2: Bundler Concentration Veto (M2)** | Pre-registered empirical rule on `token_info_detect.bundlers_hold_pct`. Tokens with **`≥ 35%` bundler holdings** suffer a `73.65%` ≥50%-dump rate within 30 minutes (vs. `33.65%` below—a `2.19×` relative risk). | Front-door candidate queue (`enqueueCandidateWithGraphGate`) & Gate 4 | **Active (`veto` ≥ 35%, `warn` 20%–35%)** |
| **Layer 3: Brand Impersonation Veto (Rule 5 / P1.5)** | Deterministic filter blocking toxic corporate/trademark impersonation tokens (e.g., fake `AMAZON`, `Meta AI`, `TESLA SI`, `Coca Cola` clones that empirically round-trip `-92%` to `-99%`). | Mid-cap strategies (`signals_shelf_breakout`, Mid-Cap Advanced Trio) | **Active** |
| **Layer 4: Rug Signal & Rug v2 Composite** | Scores tokens using RugCheck features, OHLC drop patterns, and shadow calibration (`rug_v2_shadow`). Audited on `/dev/rug-signal` and `/admin/rug`. | All tracked Solana tokens | **Shadow / Calibrating** |
| **Layer 5: Semantic Knowledge Graph (SHACL Gates)** | Evaluates multi-hop deployer funding lineage, recycled rug LP, cabal sybil rings, and pre-call insider frontrunning. | `/analytics/knowledge-graph`, `/admin/graph-edge`, Lifecycle & Scalp strategies | **Shadow + Pre-Computed Fast-Path Tags** |

---

## 2. Semantic Knowledge Graph & SHACL Pre-Trade Circuit Breakers

Single-token metrics have a critical blind spot: a serial rugger can deploy from a brand-new burner wallet funded two hops away by a tainted gas master. ReloadSOL's Semantic Knowledge Graph (`SPEC-knowledge-graph-edge-v1` and `SPEC-semantic-knowledge-graph-cross-token-intelligence-v1`) closes this gap across four relational dimensions:

1. **Cabal Sybil Ring & Atomic Bundle Detection (`CabalConcentrationShape`):** Identifies wallets funded from a shared root or buying in the same slot; flags cluster supply concentration and bundlers (`≥ 35%` veto).
2. **Serial Deployer "Dev DNA" Provenance (`DevReputationShape`):** Tracks each deployer's historical launches, graduation ratio ($\ge 10\%$ clean threshold), top-10 ATH tokens, and user-labeled rugs (`dev_reputation`, inspectable at `/dev/dev-reputation`).
   * **2-Hop Funder Lineage Veto:** Flags `lineage_risk_score ≥ 0.70` or `recycled_lp_pct ≥ 30%` at Block 0.
   * **Alpha Acceleration:** Conversely, identifies serial-winner deployers (`deployer_pedigree_score ≥ 0.60`) and Alpha Wallet "Winning Clique" convergence (`alpha_clique_co_buys ≥ 3`) to enter legitimate runners 5–15 minutes earlier.
3. **AMM Pool Liquidity Routing (`LiquidityDepthShape`):** Verifies pool TVL depth and LP lock/burn status.
4. **Pre-Call Frontrunning Defense (`PreCallInsiderShape`):** Flags wallets systematically accumulating `30s–300s` before public Telegram/X calls.

{% hint style="info" %}
**Sub-Millisecond Runtime Latency:** Graph traversal never blocks a live trade tick. An asynchronous background materialization pipeline pre-computes compact graph risk and pedigree tags into `token_entry_context.features` for $O(1)$ lookup during execution.
{% endhint %}

---

## 3. Graph Truthfulness Guarantee

ReloadSOL strictly forbids fabricating graph confidence when upstream data is missing (`SPEC-graph-edge-truthfulness-v1`, Defect Class `DC-66`):

* If holder or deployer history is unavailable for a token, the engine returns `verdict: 'unknown'` and `lineageScore: null` (`UNKNOWN_DEPLOYER`) with explicit `coverageGaps`—never falsely labeling an unseen deployer `CLEAN_DEPLOYER` or returning `pass` on an empty holder array.
* In both `/knowledge-graph` and `/admin/graph-edge`, synthetic demo rings are only rendered when an operator explicitly selects a demo preset.
