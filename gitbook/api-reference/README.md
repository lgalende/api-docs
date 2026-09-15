---
description: Every Mimic API endpoint, generated from the OpenAPI specification.
icon: code
---

# API Reference

This section is generated directly from the Mimic OpenAPI specification, so it never drifts from the running API.

**Base URL** — see `GET /chains` and `GET /settlers` for the chain and settler data every action depends on.

**Authentication** — all endpoints require an `X-Api-Key` header, except four public reference-data endpoints: `GET /chains`, `GET /settlers`, `GET /settlers/{chain}` and `GET /tokens`. See [Authentication](https://app.gitbook.com/s/FKuIxMBkxkTJ5CQ0KNYc/getting-started/authentication).

## How the endpoints are organised

Most actions come in pairs. `POST /{action}/prepare` computes the EIP-712 data to sign; `POST /{action}` submits the signature. Recurring strategies differ slightly — `prepare` returns an `id` and the signature goes to `POST /{action}/{id}/execute`.

If you are integrating for the first time, start with [Signed intents](https://app.gitbook.com/s/FKuIxMBkxkTJ5CQ0KNYc/getting-started/signed-intents) in the Guides section rather than reading endpoint-by-endpoint.

## Conventions

* Amounts are decimal strings in human units, never base-unit integers
* Timestamps are Unix milliseconds
* Slippage is basis points (`50` = 0.50%)
* The native asset is addressed as `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE`
