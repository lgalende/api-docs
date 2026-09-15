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
  "recipient": "0xabcdef1234567890abcdef1234567890abcdef12"
}
```

All five fields are required.

{% hint style="info" %}
Use `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE` as `token` to transfer the chain's native asset.
{% endhint %}

**Response:**

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "signBefore": 1700003600000,
  "signaturePayload": {
    "domain": { "name": "Mimic Protocol - Registry", "version": "1" },
    "primaryType": "Trigger",
    "types": { "...": "..." },
    "values": { "...": "..." }
  },
  "executionCost": {
    "estimatedUsdPerExecution": "0.42",
    "maxUsdPerExecution": "0.6"
  },
  "requiredApprovals": [],
  "policies": []
}
```

Keep the `id` — every later call uses it. You do not choose the fee ceiling: `maxUsdPerExecution` is derived from the estimate, and the transfer is skipped rather than executed if the actual fee would exceed it.

Both arrays are empty here, meaning the settler already has the allowance it needs and the wallet is already safeguarded. When they are not, see the next step.
{% endstep %}

{% step %}
## Sign

```javascript
const { domain, types, values } = res.signaturePayload;
const sig = await signer.signTypedData(domain, types, values);
```

If `requiredApprovals` or `policies` came back non-empty, each entry needs signing too — approvals as raw transactions, policies as typed data. See [Signed intents](signed-intents.md#signing) for both.
{% endstep %}

{% step %}
## Execute

```http
POST /transfers/550e8400-e29b-41d4-a716-446655440000/execute
X-Api-Key: <your-api-key>
Content-Type: application/json

{
  "signature": {
    "typedData": { "types": { "...": "..." }, "values": { "...": "..." } },
    "sig": "0x...",
    "signer": "0x1234567890abcdef1234567890abcdef12345678"
  },
  "signedApprovals": [],
  "signedPolicies": []
}
```

Returns `204 No Content`. Submit before the `signBefore` deadline, or the payload is refused.

Poll `GET /transfers/{id}` for the outcome — it reports `status` and, once settled, `txHash`.
{% endstep %}
{% endstepper %}

## Next

That is the whole model, and it does not change. Swaps, bridges and contract calls differ only in their prepare body; even creating a Safe wallet follows the same three steps.

For recurring behaviour, continue to [DCA](../integrations/dca.md), which adds a strategy lifecycle on top of this same flow.
