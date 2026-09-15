---
description: The prepare → sign → execute pattern behind every Mimic action.
icon: signature
---

# Signed intents

Every action in the Mimic API follows one pattern. Learn it once and every endpoint becomes predictable.

```
  prepare  ──►  sign  ──►  execute
```

1. **Prepare.** You call `POST /{action}/prepare` with what you want to happen. The API returns EIP-712 typed data describing exactly that, plus any approvals it needs.
2. **Sign.** The wallet owner signs it with their own key. This is off-chain: no gas, no funds moved.
3. **Execute.** You submit the signature back. Mimic executes within the limits that were signed.

The signature is the authorization. Mimic cannot act outside what it describes, which is what makes the API non-custodial — you never hold your users' keys, and neither does Mimic.

## Two shapes

Where the signature is submitted depends on whether the action runs once or repeatedly.

{% tabs %}
{% tab title="One-shot actions" %}
Transfers, swaps, bridges, contract calls, lending deposits and withdrawals.

```
POST /transfers/prepare   ──►  signaturePayload, requiredApprovals
POST /transfers           ──►  id
GET  /transfers/{id}      ──►  status
```

`prepare` returns the payload directly. You submit to the bare action endpoint and get an `id` back to track it.
{% endtab %}

{% tab title="Strategies" %}
DCA, stop-loss, take-profit, limit orders, rebalancing, deposit splitting.

```
POST /dca/prepare         ──►  id, signBefore, signaturePayload,
                               requiredApprovals, policies
POST /dca/{id}/execute    ──►  204
GET  /dca/{id}            ──►  status, portfolio
POST /dca/{id}/stop/...   ──►  deactivation
```

`prepare` registers the strategy up front and returns its `id`, so the execute call is addressed to that id. Strategies also carry a lifecycle: a status, an execution history, and a stop procedure.
{% endtab %}
{% endtabs %}

## What comes back from prepare

Beyond `signaturePayload`, a prepare response can include work you must do alongside the signature:

| Field | Meaning |
|---|---|
| `requiredApprovals` | Raw `approve()` transactions letting the settler spend the token. Sign each as a raw transaction. Empty when the allowance already exists |
| `policies` | EIP-712 safeguards restricting what Mimic may do with the wallet. **Required** when present |
| `signBefore` | Deadline for submitting. Strategies only |

See [Approvals and safeguards](../concepts/approvals-and-safeguards.md) for what these actually authorize on chain.

## Signing

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

{% hint style="warning" %}
Prepare responses are not indefinitely valid. Strategies return an explicit `signBefore` timestamp; submit after it and the strategy is marked `stale` and can never execute. Call `prepare` again for a fresh payload.
{% endhint %}

## Retries

Submitting the same signature twice is safe and returns the same success. Submitting a *different* signature for the same `id` returns `409` — Mimic will not silently replace an authorization your user already gave.
