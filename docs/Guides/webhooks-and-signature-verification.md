---
title: Webhooks and signature verification
---
# Webhooks and signature verification

Webhooks push events to your server the moment something happens - a payment
completes, a transaction confirms - so you react without polling. Every delivery is
HMAC-signed; **always verify the signature before trusting the body.**

## Events

Subscribe an endpoint (via `webhooks.create`) to the events you care about:

| Event                   | Fires when                                     |
| ----------------------- | ---------------------------------------------- |
| `payment.completed`     | An outbound payment finishes.                  |
| `transaction.confirmed` | An on-chain transaction confirms.              |
| `webhook.test`          | You trigger a test delivery (`webhooks.test`). |

## How signing works

Each delivery carries two headers - `X-Blockchain0x-Signature` (an HMAC over the
timestamp and the raw body) and `X-Blockchain0x-Timestamp`. The verifier
recomputes the HMAC in constant time and checks the timestamp against a **5-minute
replay window** (300s, configurable). Verify the **raw** body bytes - never the
re-serialised JSON.

The SDK verifiers return a coarse `ok` plus a `webhook.*` reason code; they never
log the secret.

## Verify in your language

**Node** - returns `{ ok, code }`:

```ts
const result = webhooks.verify({ headers, rawBody, secret: process.env.B0X_WEBHOOK_SECRET! });
if (!result.ok) {
  throw new Error(`webhook signature rejected: ${result.code}`);
}
const event = JSON.parse(rawBody) as { type: string; data: unknown };
console.log('verified event', event.type, event.data);
```

**Python**:

```python
result = webhooks.verify(headers=headers, raw_body=raw_body, secret=os.environ["B0X_WEBHOOK_SECRET"])
if not result.ok:
    raise ValueError(f"bad signature: {result.code}")
```

**Go**:

```go
result := webhooks.Verify(webhooks.Args{
	Headers: webhooks.HeadersFromMap(map[string]string{}),
	RawBody: []byte("{}"),
	Secret:  os.Getenv("B0X_WEBHOOK_SECRET"),
})
if !result.OK {
	fmt.Println("bad signature:", result.Code)
}
```

**Ruby**:

```ruby
result = Blockchain0x::Webhooks.verify(
  headers: {}, raw_body: '{}', secret: ENV.fetch('B0X_WEBHOOK_SECRET', '')
)
puts "bad signature: #{result.code}" unless result.ok
```

## Retries and replay

A delivery the receiver does not `2xx` is retried with backoff. Because retries (and
network mirrors) can redeliver an event, treat handling as idempotent - key on the
event id. The 5-minute window bounds replay: a delivery whose timestamp is outside
it fails with `webhook.timestamp_outside_window`.

## Next

- **[Receive payments](./receive-payments)** - the settlement flow webhooks confirm.
- **[Going live](./going-live)** - production webhook hardening.