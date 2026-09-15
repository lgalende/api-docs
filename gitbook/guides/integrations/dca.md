---
description: Create, monitor and stop a recurring dollar-cost averaging strategy.
icon: chart-line-up
---

# DCA

A DCA strategy swaps a fixed amount of one token for another on a recurring schedule. The wallet keeps custody throughout — Mimic executes only within the authorization its owner signed.

This guide assumes you know the [signed-intent flow](../getting-started/signed-intents.md). DCA follows it, with a strategy lifecycle on top:

1. `POST /dca/prepare` — returns the strategy `id` and the payload to sign
2. Sign the payload
3. `POST /dca/{id}/execute` — activates the strategy
4. `GET /dca`, `GET /dca/{id}`, `GET /dca/{id}/executions` — monitor it
5. `POST /dca/{id}/stop/prepare` + `/stop/execute` — stop it

## Creating a strategy

{% stepper %}
{% step %}
## Prepare

| Field | Required | Description |
|---|---|---|
| `chain` | yes | Chain where the swaps execute |
| `tokenIn` | yes | Token to spend on each execution |
| `tokenOut` | yes | Token to accumulate |
| `amount` | yes | Amount of `tokenIn` per execution, as a decimal string |
| `wallet` | yes | Address the funds are pulled from |
| `schedule` | yes | [Cron expression](../concepts/scheduling.md) (UTC) |
| `slippageBps` | yes | Max slippage in basis points (50 = 0.50%) |
| `maxTotalAmount` | no | Cap on cumulative `tokenIn` spend. The strategy `ended`s after the last execution that fits. Omit for no limit |
| `recipient` | no | Address receiving `tokenOut`. Defaults to `wallet` |

Swap 100 USDC into WETH daily at midnight UTC, up to 1,200 USDC total:

```http
POST /dca/prepare
X-Api-Key: <your-api-key>
Content-Type: application/json

{
  "chain": "ethereum",
  "tokenIn": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  "tokenOut": "0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2",
  "amount": "100.0",
  "wallet": "0x1234567890abcdef1234567890abcdef12345678",
  "schedule": "0 0 * * *",
  "slippageBps": 50,
  "maxTotalAmount": "1200.0"
}
```

**Response:**

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "signBefore": 1700003600000,
  "signaturePayload": { "...": "..." },
  "executionCost": {
    "estimatedUsdPerExecution": "0.42",
    "maxUsdPerExecution": "0.6"
  },
  "requiredApprovals": [ { "...": "..." } ],
  "policies": [ { "...": "..." } ]
}
```

`executionCost` comes back from every action, but it earns its keep here: `maxUsdPerExecution` is a ceiling derived from the estimate, and any execution whose fee would exceed it is skipped. Over a long-running schedule that is what protects the strategy from gas spikes.

All subsequent calls use the returned `id`, and the payload must be signed before `signBefore`.
{% endstep %}

{% step %}
## Sign

Sign `signaturePayload` as typed data, plus each `requiredApprovals` entry as a raw transaction and each `policies` entry as typed data — the mechanics are identical for every action and are covered in [Signed intents](../getting-started/signed-intents.md).
{% endstep %}

{% step %}
## Execute

```http
POST /dca/550e8400-e29b-41d4-a716-446655440000/execute
X-Api-Key: <your-api-key>
Content-Type: application/json

{
  "signature": { "typedData": { "...": "..." }, "sig": "0x...", "signer": "0x1234..." },
  "signedApprovals": ["0x02f86c01..."],
  "signedPolicies": ["0x..."]
}
```

Returns `204 No Content`. The strategy is now `active` and will execute on schedule.
{% endstep %}
{% endstepper %}

## Monitoring

### List your strategies

`GET /dca`, optionally `?wallet=0x...`, returns every strategy created under your API key.

### Status and portfolio

`GET /dca/{id}` returns the strategy plus a portfolio snapshot:

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "active",
  "inputs": { "...": "..." },
  "nextExecution": 1700086400000,
  "maxUsdPerExecution": "0.6",
  "portfolio": {
    "timestamp": 1700000000000,
    "totalValueUsd": "1198.34",
    "balances": [
      { "token": "0xA0b8...eB48", "amount": "1100.0", "valueUsd": "1100.00" },
      { "token": "0xC02a...6Cc2", "amount": "0.031", "valueUsd": "98.34" }
    ],
    "totalSell": "100.0",
    "totalBuy": "0.031",
    "averagePrice": "3225.80"
  }
}
```

Three portfolio fields are DCA-specific and are the ones worth surfacing to a user:

* **`totalSell`** — total `tokenIn` spent so far
* **`totalBuy`** — total `tokenOut` accumulated
* **`averagePrice`** — average USD price paid per unit of `tokenOut`, which is the number that actually tells them whether averaging in worked

For `status` values see [Statuses and errors](../concepts/statuses-and-errors.md).

### Execution history

`GET /dca/{id}/executions` returns the most recent executions, newest first, each with its swaps:

```json
{
  "executions": [
    {
      "status": "succeeded",
      "timestamp": 1700000000000,
      "costUsd": "0.41",
      "swaps": [
        {
          "sourceChain": "ethereum",
          "destinationChain": "ethereum",
          "tokenIn": "0xA0b8...eB48",
          "tokenOut": "0xC02a...6Cc2",
          "amountIn": "100.0",
          "amountOut": "0.031",
          "minAmountOut": "0.0308",
          "txHash": "0xabc..."
        }
      ],
      "logs": []
    }
  ]
}
```

`minAmountOut` is the floor derived from `slippageBps` at signing time and is always present. `amountOut` is read from the settlement transaction, so it is absent until the swap settles.

## Stopping

Stopping mirrors creation: prepare, sign, submit.

```http
POST /dca/{id}/stop/prepare
```

returns a `signaturePayload` authorizing the deactivation, plus `revocations` — optional transactions that zero the settler's allowance. Submit to:

```http
POST /dca/{id}/stop/execute
```

with `signedRevocations` if the owner wants the allowance removed. The strategy stops either way; see [Approvals and safeguards](../concepts/approvals-and-safeguards.md) for when revocations come back empty and what their result statuses mean.

{% hint style="danger" %}
A stopped strategy cannot be reactivated. Create a new one to resume.
{% endhint %}

Both stop endpoints return `409` if the strategy has never executed — there is nothing to stop yet.
