---
description: What the settler is allowed to do with a wallet's funds, and how that is authorized.
icon: shield-check
---

# Approvals and safeguards

Two separate mechanisms control what Mimic can do with a wallet. They arrive together in prepare responses and are easy to confuse.

## The settler

Actions are executed on chain by Mimic's settler contract:

```
0x8be430d29a2c5692f9345a28506b849c6a99edaa
```

For it to move a token, the wallet must have granted it an ERC-20 allowance. `GET /settlers` and `GET /settlers/{chain}` return the current addresses — prefer reading them over hardcoding.

## Approvals — permission to spend

`requiredApprovals` in a prepare response holds raw `approve()` transactions granting that allowance. You never craft them yourself.

They are **raw transactions**, so they are signed with `signTransaction`, not typed-data signing:

```javascript
const signedApprovals = await Promise.all(
  res.requiredApprovals.map((tx) => signer.signTransaction(tx))
);
```

Submit them as `signedApprovals` alongside the action signature. The array is empty when the allowance already exists — a second action on the same wallet and token usually needs no approval.

## Safeguards — limits on what may be done

`policies` is the newer, stronger control. Where an approval says *"the settler may spend this token"*, a safeguard says *"…and only for these operations"*. The safeguard currently set by strategy endpoints allows swaps that pay out to the wallet itself, and denies every other kind of operation.

Policies are EIP-712 payloads, signed like the main payload:

```javascript
const signedPolicies = await Promise.all(
  res.policies.map((p) => signer.signTypedData(p.domain, p.types, p.values))
);
```

{% hint style="danger" %}
When `policies` is non-empty you **must** submit `signedPolicies`. Without them the safeguards are never set and the request is rejected — the strategy is not created.
{% endhint %}

`policies` is empty when the wallet is already safeguarded, so this is typically a one-time cost per wallet.

One consequence worth designing around: while the safeguard allows only the wallet as swap recipient, a strategy's `recipient` must equal its `wallet`.

## Revocations — taking permission back

Stopping a strategy returns `revocations`: raw `approve()` transactions setting the allowance back to zero, so no spending permission is left behind.

Signing them is **optional** — the strategy stops either way. Omitting them just leaves the existing allowance in place.

They come back empty when there is nothing to remove, and also when another strategy on the same wallet and token is still live: the allowance is shared, so zeroing it would break that one.
