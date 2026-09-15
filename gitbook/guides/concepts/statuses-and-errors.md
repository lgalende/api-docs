---
description: Every status value the API reports, and what each HTTP error means.
icon: list-check
---

# Statuses and errors

## Strategy status

Reported by `GET /{action}/{id}` for any recurring strategy.

| Status | Meaning |
|---|---|
| `pending` | Prepared but the signature has not been submitted, so it has never executed |
| `stale` | The signature was not submitted before `signBefore`, so it will never execute |
| `active` | Running on schedule. `nextExecution` tells you when the next one fires |
| `stopped` | Deactivated via `POST /{action}/{id}/stop/execute` |
| `ended` | Reached its end condition and will not execute again |

`pending`, `stale` and `ended` are all terminal-ish in different ways: `stale` and `ended` can never become `active` again, while `pending` still can — until its deadline passes.

{% hint style="info" %}
A stopped or ended strategy cannot be reactivated. Create a new one to resume.
{% endhint %}

## Execution status

Reported per execution by `GET /{action}/{id}/executions`.

| Status | Meaning |
|---|---|
| `pending` | Still in flight — queued or submitted, not yet resolved on chain |
| `succeeded` | Every intent in the execution completed successfully |
| `failed` | The execution was invalid, or at least one intent was discarded, expired or failed |
| `skipped` | The trigger fired but produced no intents, so nothing was submitted on chain |

`skipped` is the one to handle deliberately — it is not an error. A fee ceiling exceeded, or nothing to do this round, both surface here.

`costUsd` is the fee actually paid, and is omitted while `pending`.

## Revocation status

Returned by `POST /{action}/{id}/stop/execute` for each submitted revocation.

| Status | Meaning |
|---|---|
| `succeeded` | Confirmed on chain, the allowance is now zero |
| `failed` | Rejected when broadcast, reverted, or not confirmed in time |
| `skipped` | Not broadcast, because another strategy on the same wallet and token still needs the allowance |

The strategy stops regardless of what these report. A `failed` or `skipped` entry leaves an allowance in place without affecting the stop itself.

## HTTP errors

| Code | Meaning |
|---|---|
| `400` | Invalid parameters or body |
| `401` | Missing or invalid `X-Api-Key` |
| `403` | The key is valid but does not own this resource |
| `404` | Unknown id |
| `409` | Conflict — see below |

`409` is the one with real semantics behind it:

* **On execute** — a *different* signature was already submitted for this id, or the signing window closed and the strategy is now `stale`. Resubmitting the *same* signature is always safe and succeeds.
* **On stop** — the strategy has never executed, so there is nothing to stop.
* **On any endpoint taking `Idempotency-Key`** — the key was reused with a different body, or a request with that key is still in flight.
