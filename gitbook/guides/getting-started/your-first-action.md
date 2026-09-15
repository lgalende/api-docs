---
description: Send tokens from a wallet, end to end.
icon: rocket
---

# Your first action

A transfer is the smallest complete example of the [signed-intent flow](signed-intents.md): one prepare, one signature, one submission.

{% stepper %}
{% step %}
## Prepare

```http
POST /transfers/prepare
X-Api-Key: <your-api-key>
Content-Type: application/json

{
  "chain": "ethereum",
  "token": "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48",
  "amount": "25.0",
  "wallet": "0x1234567890abcdef1234567890abcdef12345678",
  "recipient": "0xabcdef1234567890abcdef1234567890abcdef12",
  "maxFeeUsd": "0.50"
}
```

All six fields are required. `maxFeeUsd` caps what you will pay for the on-chain transaction — the transfer does not execute if the fee would exceed it.

{% hint style="info" %}
Use `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE` as `token` to transfer the chain's native asset.
{% endhint %}

**Response:**

```json
{
  "signaturePayload": {
    "domain": { "name": "Mimic Protocol - Registry", "version": "1" },
    "primaryType": "Trigger",
    "types": { "...": "..." },
    "values": { "...": "..." }
  },
  "requiredApprovals": []
}
```
{% endstep %}

{% step %}
## Sign

```javascript
const { domain, types, values } = res.signaturePayload;
const sig = await signer.signTypedData(domain, types, values);
```

If `requiredApprovals` came back non-empty, sign each entry as a raw transaction too — see [Approvals and safeguards](../concepts/approvals-and-safeguards.md).
{% endstep %}

{% step %}
## Execute

```http
POST /transfers
X-Api-Key: <your-api-key>
Content-Type: application/json

{
  "signature": {
    "typedData": { "types": { "...": "..." }, "values": { "...": "..." } },
    "sig": "0x...",
    "signer": "0x1234567890abcdef1234567890abcdef12345678"
  },
  "signedApprovals": []
}
```

You get back an `id`. Poll `GET /transfers/{id}` for status.
{% endstep %}
{% endstepper %}

## Next

That is the whole model. Swaps, bridges and contract calls differ only in their prepare body. For recurring behaviour, continue to [DCA](../integrations/dca.md), which adds a strategy lifecycle on top of the same three steps.
