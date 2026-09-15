---
description: The prepare → sign → execute pattern behind every Mimic action.
icon: signature
---

# Signed intents

Every action in the Mimic API follows one pattern — a transfer, a swap, a DCA strategy, even creating a Safe wallet. Learn it once and every endpoint becomes predictable.

```
  prepare  ──►  sign  ──►  execute
```

1. **Prepare.** You call `POST /{action}/prepare` with what you want to happen. The API registers the action, returns its `id`, and gives you EIP-712 typed data describing exactly what will be done.
2. **Sign.** The wallet owner signs it with their own key. This is off-chain: no gas, no funds moved.
3. **Execute.** You submit the signature to `POST /{action}/{id}/execute`. Mimic executes within the limits that were signed.

The signature is the authorization. Mimic cannot act outside what it describes, which is what makes the API non-custodial — you never hold your users' keys, and neither does Mimic.

## The shape is the same everywhere

```
POST /{action}/prepare        ──►  id, signBefore, signaturePayload,
                                   executionCost, requiredApprovals, policies
POST /{action}/{id}/execute   ──►  204 No Content
GET  /{action}/{id}           ──►  status
```

The `id` is minted by `prepare`, not by the submission — so you can record it before the user has signed anything, and the execute call is always addressed to it.

## What comes back from prepare

Every prepare response carries the same six fields:

| Field | Meaning |
|---|---|
| `id` | Identifies the action from here on. Used by `/execute` and every read endpoint |
| `signBefore` | Deadline for submitting the signature. Past it the payload is refused |
| `signaturePayload` | The EIP-712 typed data to sign |
| `executionCost` | Estimated fee per execution, and the ceiling above which an execution is skipped |
| `requiredApprovals` | Raw `approve()` transactions letting the settler spend the token. Empty when the allowance already exists |
| `policies` | EIP-712 safeguards restricting what Mimic may do with the wallet. **Required when not empty** |

See [Approvals and safeguards](../concepts/approvals-and-safeguards.md) for what the last two actually authorize on chain.

## Signing

The main payload is typed data:

```javascript
const { domain, types, values } = res.signaturePayload;
const sig = await signer.signTypedData(domain, types, values);
```

Approvals are raw transactions, so they are signed differently — `signTransaction`, producing RLP-encoded hex:

```javascript
const signedApprovals = await Promise.all(
  res.requiredApprovals.map((tx) => signer.signTransaction(tx))
);
```

Policies are typed data again, like the main payload:

```javascript
const signedPolicies = await Promise.all(
  res.policies.map((p) => signer.signTypedData(p.domain, p.types, p.values))
);
```

All three go in the same submission:

```json
{
  "signature": { "typedData": { "...": "..." }, "sig": "0x...", "signer": "0x1234..." },
  "signedApprovals": ["0x02f86c01..."],
  "signedPolicies": ["0x..."]
}
```

{% hint style="warning" %}
Prepare responses expire. Submit after `signBefore` and the payload is refused — for a strategy, its status becomes `stale` and it can never execute. Call `prepare` again for a fresh payload rather than holding one while a user decides.
{% endhint %}

## What differs: the lifecycle, not the flow

Creation is identical everywhere. What changes is how much happens afterwards.

{% tabs %}
{% tab title="One-time actions" %}
Transfers, swaps, bridges, contract calls, lending deposits and withdrawals, staking, Safe wallet creation.

They execute once. `GET /{action}/{id}` reports the outcome — status, transaction hash, and whatever that action produces, like `amountOut` for a swap.

There is nothing to stop and no history to read: the action is its own single execution.
{% endtab %}

{% tab title="Strategies" %}
DCA, stop-loss, take-profit, limit orders, lending rebalancing, deposit splitting, portfolio rebalancing, stablecoin consolidation.

They keep executing on a schedule or trigger, so they carry a lifecycle on top:

```
GET  /{action}                     ──►  list your strategies
GET  /{action}/{id}                ──►  status, portfolio, nextExecution
GET  /{action}/{id}/executions     ──►  execution history
POST /{action}/{id}/stop/prepare   ──►  deactivation payload + revocations
POST /{action}/{id}/stop/execute   ──►  outcome of each revocation
```

Stopping mirrors creation exactly: prepare, sign, submit.
{% endtab %}
{% endtabs %}

## Retries

Submitting the same signature twice is safe and returns `204` again. Submitting a *different* signature for the same `id` returns `409` — Mimic will not silently replace an authorization your user already gave.
