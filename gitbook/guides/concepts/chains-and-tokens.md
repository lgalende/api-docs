---
description: Supported chains, token addressing, and amount formatting.
icon: link
---

# Chains and tokens

## Supported chains

Chains are identified by lowercase ids:

`ethereum` · `optimism` · `arbitrum` · `base` · `gnosis` · `sonic` · `polygon` · `avalanche` · `bnb`

`GET /chains` returns the current list at runtime. Prefer it over hardcoding — it is a public endpoint and needs no API key, so you can populate a chain picker before your user signs in.

## Token addressing

Tokens are identified by contract address on their chain. The native asset uses a sentinel address:

```
0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE
```

`GET /tokens` lists supported tokens, optionally filtered by chain; `GET /tokens/{address}` returns one with its current USD price. Both are public.

## Amounts are decimal strings

Every amount in the API is a **decimal string in human units**, not an integer in base units:

```json
{ "amount": "100.0" }
```

Not `100000000` for 100 USDC. This avoids the precision loss that comes with JSON numbers on 18-decimal tokens.

{% hint style="warning" %}
Amounts with more decimals than the token supports are **truncated**, not rejected. Passing `"0.0000001"` for a 6-decimal token like USDC silently becomes `"0.0"`.
{% endhint %}

## Related conventions

| Field | Format |
|---|---|
| Timestamps | Unix milliseconds, integer |
| Slippage | Basis points, integer — `50` = 0.50% |
| Fees | USD decimal string — `"0.42"` |
| Addresses | 0x-prefixed 20-byte hex on EVM chains |
| Signatures | 65-byte hex, 130 chars plus `0x` |
