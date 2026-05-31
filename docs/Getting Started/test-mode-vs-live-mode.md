---
title: Test mode vs live mode
---
# Test mode vs live mode

Blockchain0x runs two fully separated worlds. **Test mode** settles on Base
Sepolia with valueless test USDC; **live mode** settles on Base mainnet with real
USDC. Agents, payments, transactions, and webhooks never cross between them.

The same code targets both - you switch worlds by switching credentials, not by
rewriting calls.

## What picks the mode

Two signals, and they must agree:

1. **Your API key prefix.** `sk_test_*` is a test-mode key; `sk_live_*` is a
   live-mode key. See [Authentication](./authentication).
2. **The `X-Network` header.** Workspace-scoped requests carry `X-Network`, whose
   value is `testnet` or `mainnet`. It scopes reads (responses are filtered to
   that network) and stamps chain-bound writes.

The SDK manages the header for you. Pass `network` when you construct the client
and it is attached to every call that needs it:

```ts
const client = createClient({ apiKey: process.env.B0X_API_KEY! }); // sk_test_* -> testnet
```

You can also set it explicitly with `createClient({ apiKey, network: 'testnet' })`
(or `'mainnet'`). If a workspace is locked to a single network for compliance, a
mismatched header is rejected with `403 network_locked`.

## Getting test USDC

A fresh test agent has no balance, and a test agent needs test USDC before it can
pay. Blockchain0x does **not** run its own faucet - fund the agent's address from
the public Base Sepolia faucets:

- [Coinbase Base Sepolia faucet](https://portal.cdp.coinbase.com/products/faucet)
- [Circle USDC faucet](https://faucet.circle.com)

In the dashboard, a test-mode agent shows a **Get test USDC** button that links
straight to these. Live mode has no faucet - you fund mainnet agents with real
USDC.

## Telling them apart

- Test-mode chain notifications (email subject, push title) are prefixed
  `[TEST]`, so a test event is never mistaken for a real one.
- Listing agents or transactions returns only the rows for the `X-Network` you
  asked for. Switch the network and the list switches with it.

## Going live

When you are ready for real money, swap the test key for an `sk_live_` key and set
`network: 'mainnet'`. Nothing else in your integration changes. Walk the
[Going live](../guides/going-live) checklist first - idempotency, live-mode
webhook verification, and rate-limit handling all matter more once funds are real.