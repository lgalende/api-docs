---
description: Execute on-chain wallet actions via signed intents, without taking custody.
icon: hand-wave
---

# Overview

The Mimic API lets you execute on-chain wallet actions — transfers, swaps, bridges, lending, staking — and run automated strategies like DCA, stop-loss and rebalancing, **without ever taking custody of your users' funds**.

Nothing moves unless the wallet owner has signed an explicit authorization. The API computes EIP-712 typed data, your user signs it with their own key, and you submit the signature back. Mimic executes on their behalf, within the limits they signed.

## Where to start

| If you want to… | Go to |
|---|---|
| Understand the model before writing code | [Signed intents](getting-started/signed-intents.md) |
| Make your first API call | [Authentication](getting-started/authentication.md) |
| See a complete action end to end | [Your first action](getting-started/your-first-action.md) |
| Integrate a specific action | [Integrations](integrations/dca.md) |
| Look up an endpoint | The **API Reference** section |

## Two kinds of action

Every action is created the same way. What differs is what happens afterwards:

* **One-time actions** — a transfer, a swap, a bridge, a contract call. Prepared, signed, executed once.
* **Strategies** — DCA, stop-loss, take-profit, rebalancing. Prepared and signed once, then executed repeatedly by Mimic on a schedule or trigger until they end or you stop them.

The creation flow is identical for both. Strategies add a lifecycle on top: a list endpoint, an execution history, and a stop procedure.
