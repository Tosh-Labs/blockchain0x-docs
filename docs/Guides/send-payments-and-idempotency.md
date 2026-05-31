---
title: Send payments and idempotency
---
# Send payments and idempotency

Your **agent** sends money when your code calls `payments.create` (naming the
agent by its `agentId`). The key needs the `pay_bills` scope, and every payment is
capped by the agent's **spend permission** - the key can spend within that
allowance but can never raise it. That two-layer gate is what makes it safe to
hand a key to an autonomous agent.

## Your agent sends a payment

`agentId` is the agent wallet the funds come from; `amountWei` is the integer USDC
amount (6 decimals, so `'10000'` = 0.01 USDC):

```ts
const payment = await client.payments.create(
  { agentId, to: '0x000000000000000000000000000000000000dEaD', amountWei: '10000' },
  { idempotencyKey: `pay-${agentId}-001` }
);
```

## Why idempotency matters

A payment moves real money, so a duplicate is the worst kind of bug. Two things
protect you:

- **The SDK mints an `Idempotency-Key` automatically** for every `payments.create`,
  and the server collapses any retry carrying the same key into the **same** chain
  transfer for 24 hours - a retried request never double-spends.
- **Retries are off by default** for payments specifically. A lost `5xx` response
  could otherwise resubmit, so the SDK does not auto-retry payments unless you opt
  in with `retry: 'default'`.

Pass your own deterministic `idempotencyKey` (as above) when you want to dedupe
across process restarts - reuse the same key for the same logical payment, and the
server guarantees at-most-once execution.

## Reconcile

After sending, read the transaction with `client.transactions.get(...)` to watch it
reach `confirmed`, or subscribe to a webhook so your system reacts without polling
(see [Webhooks](./webhooks-and-signature-verification)).

## Next

- **[Receive payments](./receive-payments)** - the inbound side.
- **[Errors and rate limits](./errors-and-rate-limits)** - handling failures.
- **[Going live](./going-live)** - the production checklist.