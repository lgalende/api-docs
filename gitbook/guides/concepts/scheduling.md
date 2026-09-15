---
description: Cron schedules, trigger types, and signing deadlines.
icon: clock
---

# Scheduling

Strategies execute when a trigger fires. Three trigger types exist:

| Type | Fires |
|---|---|
| Cron | On a recurring schedule |
| Event | On an on-chain event |
| Once | Exactly once, at a specific time |

Most strategies you configure directly — DCA, rebalancing, deposit splitting — use a cron schedule.

## Cron expressions

Standard 5-field cron, **evaluated in UTC**:

```
┌───────── minute (0-59)
│ ┌─────── hour (0-23)
│ │ ┌───── day of month (1-31)
│ │ │ ┌─── month (1-12)
│ │ │ │ ┌─ day of week (0-6, Sunday = 0)
│ │ │ │ │
0 0 * * *
```

| Expression | Meaning |
|---|---|
| `0 0 * * *` | Every day at midnight |
| `0 9 * * 1` | Every Monday at 09:00 |
| `0 */6 * * *` | Every 6 hours |
| `30 14 1 * *` | 14:30 on the 1st of each month |

{% hint style="info" %}
UTC, always. A "daily at midnight" schedule fires at midnight UTC regardless of where your user is — worth surfacing in your UI if you show them a local time.
{% endhint %}

## Signing deadlines

A strategy's prepare response includes `signBefore`, a Unix timestamp in milliseconds. Submit the signature after it and the strategy is marked `stale` and can never execute — call `prepare` again for a fresh payload.

This matters most when a human is in the signing loop. If your user might leave the tab open, re-prepare rather than holding a payload.

## When the next execution happens

`GET /{action}/{id}` returns `nextExecution`, a timestamp, while the strategy is `active`. It is omitted for any other status — so its absence is a signal, not a gap.

## Executions that do not happen

A scheduled execution can be skipped rather than run:

* **Fee ceiling.** Each strategy carries `maxUsdPerExecution`, derived at prepare time. If an execution's fee would exceed it, that execution is skipped. This protects a long-running strategy from gas spikes.
* **Spend cap.** `maxTotalAmount` stops the strategy after the last execution that fits within it. The status becomes `ended`.
