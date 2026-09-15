---
description: Every Mimic API endpoint, generated from the OpenAPI specification.
icon: code
---

# API Reference

This section is generated directly from the Mimic OpenAPI specification, so it never drifts from the running API.

**Base URL** — see `GET /chains` and `GET /settlers` for the chain and settler data every action depends on.

**Authentication** — all endpoints require an `X-Api-Key` header, except four public reference-data endpoints: `GET /chains`, `GET /settlers`, `GET /settlers/{chain}` and `GET /tokens`. See [Authentication](https://app.gitbook.com/s/FKuIxMBkxkTJ5CQ0KNYc/getting-started/authentication).

## How the endpoints are organised

Every action follows the same two calls. `POST /{action}/prepare` registers the action, returns its `id` and computes the EIP-712 data to sign; `POST /{action}/{id}/execute` submits the signature. Recurring strategies add a lifecycle on top — a list endpoint, an execution history, and a stop procedure — but create exactly the same way.

If you are integrating for the first time, start with [Signed intents](https://app.gitbook.com/s/FKuIxMBkxkTJ5CQ0KNYc/getting-started/signed-intents) in the Guides section rather than reading endpoint-by-endpoint.

## Conventions

* Amounts are decimal strings in human units, never base-unit integers
* Timestamps are Unix milliseconds
* Slippage is basis points (`50` = 0.50%)
* The native asset is addressed as `0xEeeeeEeeeEeEeeEeEeEeeEEEeeeeEeeeeeeeEEeE`
