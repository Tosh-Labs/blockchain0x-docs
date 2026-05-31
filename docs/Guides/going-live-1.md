---
title: Going live
---
Test mode and live mode share the same code - going live is a matter of swapping
credentials and hardening a handful of operational details. Work this checklist
before you move real USDC.

## 1. Switch to a live key

Replace the `sk_test_*` key with an `sk_live_*` key and set `network: 'mainnet'`.
Nothing else in your integration changes. Keep live keys in a secret manager,
never in source control or client-side code. See
[Authentication](../getting-started/authentication).

## 2. Rotate on a schedule

Rotate keys periodically and immediately if one may have leaked. Rotation keeps the
old secret valid through a short overlap window, so you can roll with zero downtime:

```ts
// Rotate a key: the response carries the NEW secret once + an overlap window
// during which BOTH the old and new secrets are accepted.
const rotated = await client.apiKeys.rotate(apiKeyId);
console.log('new secret (shown once):', rotated.successor.secret);
```

## 3. Verify live-mode webhooks

Register your **production** webhook endpoint and confirm your receiver verifies the
HMAC signature on every delivery (within the 5-minute replay window) before acting
on it. Test mode and live mode use different signing secrets - make sure production
holds the live secret. See
[Webhooks and signature verification](./webhooks-and-signature-verification).

## 4. Make writes idempotent

Send an `Idempotency-Key` on every state-changing call so a retry can never
double-spend. The SDK does this for `payments.create` automatically; supply your own
deterministic key when you need dedupe across restarts. See
[Send payments and idempotency](./send-payments-and-idempotency).

## 5. Honour rate limits

On a `429`, back off for the `Retry-After` interval. The SDKs do this automatically
(3 retries with backoff); if you call the API directly, implement the same. See
[Errors and rate limits](./errors-and-rate-limits).

## 6. Keep X-Network correct

Every workspace-scoped call carries `X-Network`. In production it must be `mainnet`,
and it must match your key's mode - a mismatch is rejected (`apikey.network_mismatch`
or `403 network_locked`). Let the SDK set it from the key, or pass `network:
'mainnet'` explicitly. See [Test mode vs live mode](../getting-started/test-mode-vs-live-mode).

## Production checklist

- [ ] Live `sk_live_*` key in a secret manager; no keys in source.
- [ ] Key rotation scheduled.
- [ ] Production webhook endpoint registered; signatures verified with the live secret.
- [ ] `Idempotency-Key` on every payment / settlement.
- [ ] `429` / `Retry-After` handled (or SDK retries left on).
- [ ] `X-Network: mainnet` consistent with the live key.

Need the underlying contracts? The full [API reference](/reference) shows every
operation with SDK code samples.