---
description: Authenticate every request with your API key.
icon: key
---

# Authentication

Every endpoint requires your API key in the `X-Api-Key` header:

```http
X-Api-Key: <your-api-key>
```

Your key scopes everything you can see. Read endpoints only ever return resources created under your own key — `GET /dca` lists your strategies, never anyone else's.

{% hint style="danger" %}
Your API key is a server-side credential. Never ship it in a browser, mobile app, or any client your users control.
{% endhint %}

## The public endpoints

Four reference-data endpoints need no key at all:

* `GET /chains`
* `GET /settlers`
* `GET /settlers/{chain}`
* `GET /tokens`

Useful for populating a chain or token picker before your user has authenticated with you.

## When authentication fails

A missing or invalid key returns `401`. A valid key that doesn't own the resource returns `403` — worth distinguishing in your error handling, because `403` usually means a bug in which key you're sending, not an expired one.
