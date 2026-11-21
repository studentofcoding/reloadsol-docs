---
description: How to verify you are on the real reloadSol app, read wallet signature prompts safely, and avoid crypto phishing scams.
last_reviewed: 2026-11-21
---

# Stay safe on reloadSol

**TL;DR:** Bookmark `https://reloadsol.app`. reloadSol is non-custodial—we never DM first, never ask for your seed phrase or private key, and never ask you to "validate" a wallet.

[Open official reloadSol →](https://reloadsol.app/)

---

## Verify you are on the real site

{% stepper %}
{% step %}
### Check the exact domain

Confirm your browser address bar reads **`https://reloadsol.app`** (with HTTPS lock). Watch out for lookalike domains or sponsored search ads.
{% endstep %}

{% step %}
### Bookmark the official URL

Save `https://reloadsol.app` as a browser bookmark and open it directly instead of clicking links in unsolicited DMs or replies.
{% endstep %}

{% step %}
### Read every wallet prompt before signing

Check the simulated balance changes in Phantom, Solflare, or Rabby before clicking **Approve**. If a prompt shows an unexpected transfer of all your SOL or tokens, click **Reject**.
{% endstep %}
{% endstepper %}

---

## Official rules reloadSol will never break

| Claim | True or Fake? |
| :--- | :--- |
| "Send us your 12/24-word recovery phrase or private key" | **100% Fake — scam.** |
| "Support DM'd you first to fix a stuck swap" | **100% Fake — scam.** |
| "Sign a message to validate / synchronize your wallet" | **100% Fake — scam.** |
| "Practice first using Paper mode with no wallet connected" | **Real** — see [Paper mode](paper-mode.md). |

---

## Approval hygiene (Robinhood Chain)

When using Robinhood Chain with a browser wallet, one-time Permit2 setup grants allowance so multi-token batches execute in one click. Setup does not move funds, and every trade still requires your signature. You can revoke any token approval at any time from your wallet's approval manager.

{% hint style="warning" %}
**On-chain transactions are irreversible.** Once you sign and a transaction confirms on Solana or Robinhood Chain, neither reloadSol nor any support team can reverse it. Always verify token addresses and amounts before approving.
{% endhint %}

---

## Related

* [What is checked before you sign](paper-trading-and-safety.md)
* [Practice with Paper mode](paper-mode.md)
* [Risk disclosure](../help/risk-disclosure.md)
