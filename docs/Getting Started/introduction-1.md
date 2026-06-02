---
title: Introduction
---
Blockchain0x is the payments API for autonomous software. It gives an AI agent,
a backend service, or a script its own on-chain wallet and a clean HTTP API to
**send and receive USDC on Base** - test mode on Base Sepolia, live mode on Base
mainnet, the same code for both.

If you can call an HTTP endpoint, you can move money programmatically. No smart
contracts to write, no key-management to build, no chain-specific plumbing.

## What you can build

- **Agentic spend.** Give an AI agent a scoped, revocable key and let it pay for
  APIs, compute, or data on your behalf, within limits you set.
- **Pay-per-call APIs.** Charge for your own endpoints with the
  [x402](https://www.x402.org) HTTP 402 standard - a caller gets a `402`, pays in
  USDC, and retries with proof of payment. Our SDKs implement both sides.
- **Programmatic invoicing.** Create a payment request, hand the payer a hosted
  amount, and settle it the moment the on-chain transfer confirms.
- **Wallet-as-a-service.** Spin up a funded agent wallet, read its balance and
  transaction history, and rotate its keys - all over the API.

## The SDKs

Every SDK exposes the same resource surface (`agents`, `apiKeys`, `payments`,
`paymentRequests`, `transactions`, `webhooks`) plus a standalone
`webhooks.verify()` signature helper. Pick your language:

| Language  | Package                                | Install                                       |
| --------- | -------------------------------------- | --------------------------------------------- |
| Node / TS | `@blockchain0x/node`                   | `npm i @blockchain0x/node`                    |
| Python    | `blockchain0x`                         | `pip install blockchain0x`                    |
| Go        | `github.com/Tosh-Labs/blockchain0x-go` | `go get github.com/Tosh-Labs/blockchain0x-go` |
| Ruby      | `blockchain0x`                         | `gem install blockchain0x`                    |

Plus the **x402 protocol family** (`@blockchain0x/x402`, `blockchain0x-x402`, and
the Go / Ruby equivalents) for the HTTP-402 pay-side client and receive-side
server adapters.

## Get going

1. **[Quickstart](./quickstart)** - go from zero to a confirmed USDC payment on
   Base testnet in under 10 minutes.
2. **[Authentication](./authentication)** - API keys, scopes, and key rotation.
3. **[Test mode vs live mode](./test-mode-vs-live-mode)** - networks, the faucet,
   and how mode is enforced.

Prefer to read the surface first? The full **[API reference](/reference)** is
generated from our OpenAPI spec, and every SDK method links to a runnable
example.