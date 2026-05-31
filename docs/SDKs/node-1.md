---
title: Node SDK reference
---
# Node SDK reference

`@blockchain0x/node` - the reference implementation. Install with
`npm i @blockchain0x/node`, then construct a client (see
[SDKs overview](./overview) for all options):

```ts
const client = createClient({ apiKey: process.env.B0X_API_KEY! }); // sk_test_* -> testnet
```

Every method throws a typed [`Blockchain0xError`](./overview) on a non-2xx
response - branch on `error.code`.

## `agents`

Agent wallets that hold funds and sign payments. Read them by id or list them in
the workspace.

| Method                             | Returns                  | Description                          |
| ---------------------------------- | ------------------------ | ------------------------------------ |
| `agents.get(id)`                   | `Promise<AgentSummary>`  | Fetch one agent by id.               |
| `agents.list({ cursor?, limit? })` | `Promise<AgentListPage>` | Page through the workspace's agents. |

`AgentSummary` carries `id`, `name`, and `network` (`mainnet` | `testnet`).

> **Creating agents** is a dashboard/account action and is not callable with an
> API key. Provision agents at [wallet.blockchain0x.com](https://wallet.blockchain0x.com),
> then reference them in code by `agentId` (see
> [Set up and fund an agent](../guides/create-and-fund-an-agent)).

## `apiKeys`

Manage the API keys that authenticate calls. See
[Authentication](../getting-started/authentication) for scopes and key levels.

| Method                              | Returns                     | Description                                |
| ----------------------------------- | --------------------------- | ------------------------------------------ |
| `apiKeys.create(body)`              | `Promise<ApiKeyWithSecret>` | Mint a key; `secret` is returned **once**. |
| `apiKeys.list({ cursor?, limit? })` | `Promise<ApiKeyListPage>`   | Page through the workspace's keys.         |
| `apiKeys.rotate(apiKeyId)`          | `Promise<ApiKeyRotation>`   | Issue a successor with an overlap window.  |
| `apiKeys.revoke(apiKeyId)`          | `Promise<void>`             | Immediately revoke a key.                  |

Failures throw `ApiKeyError` with an `apikey.*` code.

```ts
// Rotate a key: the response carries the NEW secret once + an overlap window
// during which BOTH the old and new secrets are accepted.
const rotated = await client.apiKeys.rotate(apiKeyId);
console.log('new secret (shown once):', rotated.successor.secret);
```

## `payments`

Send outbound USDC. Requires the `pay_bills` scope, and each payment is capped by
the agent's spend permission.

| Method                         | Returns            | Description                 |
| ------------------------------ | ------------------ | --------------------------- |
| `payments.create(body, opts?)` | `Promise<Payment>` | Create an outbound payment. |

`PaymentCreate` is `{ agentId, to, amountWei, token?, metadata? }` - `amountWei` is
the integer amount (USDC has 6 decimals, so `'10000'` = 0.01 USDC).

Two payment-specific behaviours: the SDK mints an `Idempotency-Key` automatically,
and **retries are off by default** (a lost 5xx response must not double-submit).
Pass your own key via `opts.idempotencyKey`, and opt back into retries with
`opts.retry = 'default'`:

```ts
const payment = await client.payments.create(
  { agentId, to: '0x000000000000000000000000000000000000dEaD', amountWei: '10000' },
  { idempotencyKey: `pay-${agentId}-001` }
);
```

## `paymentRequests`

Settle an invoice once the payer's on-chain USDC transfer confirms. Requires
`receive_money`. Settle is naturally idempotent server-side (an already-settled
invoice returns `409 payment_request.not_settleable`).

| Method                                               | Returns                          | Description                   |
| ---------------------------------------------------- | -------------------------------- | ----------------------------- |
| `paymentRequests.settle({ paymentRequestId, body })` | `Promise<PaymentRequestSettled>` | Settle an invoice with proof. |

`body` is `{ txHash, payerAddress, amountUsdcVerified }`; the backend re-verifies
all of (txHash, payer, amount, recipient) before flipping the invoice to
`settled`.

```ts
const settled = await client.paymentRequests.settle({
  paymentRequestId,
  body: {
    txHash: '0x' + 'a'.repeat(64),
    payerAddress: '0x' + 'b'.repeat(40),
    amountUsdcVerified: '25.00',
  },
});
```

## `transactions`

Read on-chain transaction state - for example, to poll a freshly broadcast
payment until it confirms.

| Method                            | Returns                | Description            |
| --------------------------------- | ---------------------- | ---------------------- |
| `transactions.get(transactionId)` | `Promise<Transaction>` | Fetch one transaction. |

```ts
const tx = await client.transactions.get(transactionId);
```

## `webhooks`

Manage webhook endpoints. See the
[Webhooks guide](../guides/webhooks-and-signature-verification) for the event
catalogue and the receive-side flow.

| Method                                              | Returns                        | Description                                       |
| --------------------------------------------------- | ------------------------------ | ------------------------------------------------- |
| `webhooks.create(body)`                             | `Promise<WebhookEndpoint>`     | Register an endpoint; returns the signing secret. |
| `webhooks.list({ cursor?, limit?, agentId? })`      | `Promise<WebhookEndpointPage>` | Page through endpoints.                           |
| `webhooks.update(webhookId, patch)`                 | `Promise<WebhookEndpoint>`     | Patch an endpoint.                                |
| `webhooks.rotateSecret(webhookId)`                  | `Promise<WebhookEndpoint>`     | Roll the signing secret.                          |
| `webhooks.delete(webhookId)`                        | `Promise<void>`                | Remove an endpoint.                               |
| `webhooks.test(webhookId, { eventType, payload? })` | `Promise<WebhookDelivery>`     | Enqueue a test delivery.                          |

### `webhooks.verify()`

A standalone, dependency-free HMAC verifier for your receiver. It never throws and
never logs the secret - it returns `{ ok: true }` or `{ ok: false, code }` where
`code` is a `webhook.*` reason (`signature_missing`, `signature_mismatch`,
`timestamp_outside_window`, and so on). Always verify before trusting a payload:

```ts
const result = webhooks.verify({ headers, rawBody, secret: process.env.B0X_WEBHOOK_SECRET! });
if (!result.ok) {
  throw new Error(`webhook signature rejected: ${result.code}`);
}
const event = JSON.parse(rawBody) as { type: string; data: unknown };
console.log('verified event', event.type, event.data);
```

## Next

- **[Python](./python)**, **[Go](./go)**, **[Ruby](./ruby)** - the same surface.
- **[x402](./x402)** - HTTP-402 pay-per-call.
- **[Guides](../guides/receive-payments)** - end-to-end task walkthroughs.