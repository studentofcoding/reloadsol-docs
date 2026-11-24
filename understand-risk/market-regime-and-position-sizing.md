---
last_reviewed: 2026-11-21T00:00:00.000Z
description: >-
  How to read the header Market Climate indicator on reloadSol and adjust your
  spot position sizes for choppy or risk-off markets.
---

# Market climate & sizing

**TL;DR:** The **Climate** chip in the reloadSol header summarizes current crypto market conditions (**Hype**, **Range**, **Mixed**, **De-risk**, or **Cash**) so you can scale your spot position sizes down when the broader tape turns defensive.

[Open reloadSol →](https://reloadsol.app/)

***

## Reading the Market Climate chip

| Climate state | Typical posture                                | Suggested sizing discipline                     |
| ------------- | ---------------------------------------------- | ----------------------------------------------- |
| **Hype**      | Strong positive BTC/SOL trend and breadth      | Normal spot position size (`1.00×`).            |
| **Range**     | Sideways, range-bound tape                     | Slightly reduced size (`0.75×`).                |
| **Mixed**     | Conflicting signals, choppy follow-through     | Half size (`0.50×`), more selective entries.    |
| **De-risk**   | Risk-off drawdown or elevated crash volatility | Quarter size (`0.25×`), preserve capital.       |
| **Cash**      | Severe downside cascade                        | Stand down—avoid opening new speculative spots. |

_Explore how climate multipliers affect sizing in our illustrative lab:_ [Open the Regime & Sizing Simulator ↗](https://studentofcoding.github.io/reloadsol-docs/widgets/index.html?view=lifecycle)

{% hint style="warning" %}
**Informational indicator only:** On the public app, the Climate chip does **not** automatically resize or block your manual wallet trades. You remain in full control of every swap amount you sign.
{% endhint %}

***

## Related

* [Risk basics for spot traders](risk-basics.md)
* [Bulk buy up to 4 tokens](../trade/bulk-buy.md)
* [Risk disclosure](../help/risk-disclosure.md)
