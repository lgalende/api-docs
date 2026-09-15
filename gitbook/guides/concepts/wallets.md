---
description: Safe wallets, custodial EOAs, and which actions each supports.
icon: wallet
---

# Wallets

Every action pulls funds from a `wallet` address. Mimic works with wallets you bring, and can also create them for you.

## Creating a Safe wallet

A Safe is a smart contract wallet owned by an address your user controls. Creation follows the same [signed-intent flow](../getting-started/signed-intents.md) as any other action:

```http
POST /wallets/safe/prepare
X-Api-Key: <your-api-key>
Content-Type: application/json

{
  "chain": "ethereum",
  "owner": "0x1234567890abcdef1234567890abcdef12345678"
}
```

Then submit the signature to `POST /wallets/safe`.

Two optional fields are worth knowing:

* **`expectedAddress`** — reuse an address already deployed on another chain, so one user has the same wallet address everywhere. If the wallet already exists for that `owner` and `chain`, creation is skipped.
* **`externalId`** — your own identifier for the wallet, so you can reference it by an id from your system rather than storing addresses.

## Creating a custodial EOA

```http
POST /wallets/eoa
X-Api-Key: <your-api-key>
Content-Type: application/json

{ "chain": "ethereum" }
```

Mimic generates and stores the keypair. There is no prepare step and nothing to sign, because there is no existing owner to authorize it.

{% hint style="warning" %}
A custodial EOA means Mimic holds the key. That is a different trust model from the rest of the API — choose it deliberately.
{% endhint %}

## Which wallet for which action

| Action | Safe | EOA |
|---|---|---|
| Transfers, swaps, bridges | ✅ | ✅ |
| Lending, staking | ✅ | ✅ |
| Contract calls | ✅ | ❌ unless using EIP-7702 delegation |
| Strategies (DCA, stop-loss, …) | ✅ | ✅ |

Contract calls are the exception: they execute arbitrary calldata from the wallet, which a plain EOA cannot do without delegating its code via EIP-7702.

## Balances

`GET /wallets/{wallet}/balances` returns what a wallet holds, on a given chain, with USD values.
