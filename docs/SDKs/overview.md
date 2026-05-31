---
title: SDKs overview
---
Every Blockchain0x SDK wraps the same HTTP API behind the same resource surface,
so the concepts you learn in one language carry to the others. The Node SDK is
the reference implementation; the rest mirror it with idiomatic naming.

## Pick your language

| Language  | Package                                | Install                                       | Registry                                                |
| --------- | -------------------------------------- | --------------------------------------------- | ------------------------------------------------------- |
| Node / TS | `@blockchain0x/node`                   | `npm i @blockchain0x/node`                    | [npm](https://www.npmjs.com/package/@blockchain0x/node) |
| Python    | `blockchain0x`                         | `pip install blockchain0x`                    | [PyPI](https://pypi.org/project/blockchain0x/)          |
| Go        | `github.com/Tosh-Labs/blockchain0x-go` | `go get github.com/Tosh-Labs/blockchain0x-go` | Go proxy (git tags)                                     |
| Ruby      | `blockchain0x`                         | `gem install blockchain0x`                    | [RubyGems](https://rubygems.org/gems/blockchain0x)      |

For HTTP-402 pay-per-call, the **x402 family** (`@blockchain0x/x402` and its
Python/Go/Ruby siblings) adds the pay-side client and receive-side server
adapters. See the [x402 reference](./x402).

## The resource surface

Every SDK exposes the same resources, plus a standalone webhook-signature helper:

| Resource          | What it manages                                    |
| ----------------- | -------------------------------------------------- |
| `agents`          | Agent wallets - create, read, list.                |
| `apiKeys`         | API keys - create, list, rotate, revoke.           |
| `payments`        | Outbound payments.                                 |
| `paymentRequests` | Invoices - create and settle with on-chain proof.  |
| `transactions`    | On-chain transaction history and lookups.          |
| `webhooks`        | Webhook endpoint management + `webhooks.verify()`. |

## Configure the client

You construct a client with your API key; everything else has a sensible default:

```ts
const client = createClient({ apiKey: process.env.B0X_API_KEY! }); // sk_test_* -> testnet
```

The constructor options:

| Option      | Default                        | Purpose                                              |
| ----------- | ------------------------------ | ---------------------------------------------------- |
| `apiKey`    | required                       | Bearer credential; prefix selects test/live.         |
| `baseUrl`   | `https://api.blockchain0x.com` | Override for self-hosted or staging targets.         |
| `network`   | (from key)                     | `testnet` or `mainnet`; sets the `X-Network` header. |
| `timeoutMs` | `30000`                        | Per-attempt request timeout.                         |

## Retries and idempotency

The SDK retries transient failures automatically: on `429` and `5xx` it retries
up to **3 times** with exponential backoff (roughly 250ms, 500ms, 1000ms), and it
honours a `Retry-After` header when the server sends one. Pass `retry: 'off'` on a
call to disable it.

Because a retry can re-send a write, **send an `Idempotency-Key`** on
state-changing calls (payments, settlements). The server dedupes a given key for
24 hours, so a retried payment is never double-spent. See
[Send payments and idempotency](../guides/send-payments-and-idempotency).

## Error model

A non-2xx response throws a typed error you can branch on:

| Error                   | When                                                                        |
| ----------------------- | --------------------------------------------------------------------------- |
| `Blockchain0xError`     | Any API error. Carries `code`, `status`, `message`, `requestId`, `details`. |
| `ApiKeyError`           | Subclass for the `apikey.*` failure codes (scope, assignment, revocation).  |
| `WebhookSignatureError` | Raised by `webhooks.verify()` when a signature fails to validate.           |

Branch on `error.code` for exhaustive handling, and log `error.requestId` when you
contact support. The full catalogue lives in
[Errors and rate limits](../guides/errors-and-rate-limits).

## Per-language reference

- **[Node](./node)** - `@blockchain0x/node`
- **[Python](./python)** - `blockchain0x`
- **[Go](./go)** - `blockchain0x-go`
- **[Ruby](./ruby)** - `blockchain0x`
- **[x402](./x402)** - the HTTP-402 protocol family