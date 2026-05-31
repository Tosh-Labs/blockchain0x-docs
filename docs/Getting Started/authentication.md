---
title: Authentication
---
# Authentication

Every Blockchain0x API call is authenticated with an **API key** sent as a bearer
token. The SDKs take the key in their constructor and attach it for you:

```ts
const client = createClient({ apiKey: process.env.B0X_API_KEY! }); // sk_test_* -> testnet
```

Create and manage keys from the dashboard at
[wallet.blockchain0x.com](https://wallet.blockchain0x.com). Treat a key like a
password: it is shown **once** at creation, never logged, and never embedded in
client-side code.

## Test keys vs live keys

The key prefix selects the network - you never set it by hand:

| Prefix      | Mode | Network      | Funds                  |
| ----------- | ---- | ------------ | ---------------------- |
| `sk_test_*` | Test | Base Sepolia | Faucet (no real value) |
| `sk_live_*` | Live | Base mainnet | Real USDC              |

The same code runs against both - swap the key, nothing else. Keep `sk_live_`
keys in a secret manager and out of source control. See
[Test mode vs live mode](./test-mode-vs-live-mode) for how mode is enforced
end to end.

## Key levels

A key is scoped to exactly one of two levels, set when you create it:

- **Agent (wallet) key** - bound to a single agent (`agentId` is set). This is
  the key an autonomous agent uses to read its own wallet, pay, and get paid.
- **Workspace key** - operates at the workspace level (`agentId` is `null`),
  for management and reporting across the workspace.

## Scopes

Keys are least-privilege: a key carries only the scopes you grant it, and a key
can never widen its own scope. Agent-level scopes:

| Scope                    | What it allows                                           |
| ------------------------ | -------------------------------------------------------- |
| `read_wallet_metadata`   | See balances, transactions, history, and limits.         |
| `manage_wallet_metadata` | Read, plus update public profile copy (tagline, social). |
| `pay_bills`              | Send outbound payments, capped by the spend allowance.   |
| `receive_money`          | Create invoices and settle them with on-chain proof.     |

Workspace-level scopes:

| Scope                       | What it allows                             |
| --------------------------- | ------------------------------------------ |
| `read_workspace`            | Members, agents list, audit log, usage.    |
| `manage_workspace_metadata` | Update workspace public copy and branding. |

Outbound payments are gated twice: a key needs `pay_bills`, **and** every payment
is still capped by the agent's spend permission - the key cannot raise that cap.

## Rotating a key

Rotation issues a new secret while keeping the old one valid for a short overlap
window, so you can roll a key with **zero downtime**: rotate, deploy the new
secret, then let the old one expire.

```ts
// Rotate a key: the response carries the NEW secret once + an overlap window
// during which BOTH the old and new secrets are accepted.
const rotated = await client.apiKeys.rotate(apiKeyId);
console.log('new secret (shown once):', rotated.successor.secret);
```

The new secret on `rotated.successor.secret` is returned **once** - capture it
immediately. Rotate on a schedule and whenever a key may have leaked.

## Next steps

- **[Test mode vs live mode](./test-mode-vs-live-mode)** - networks and the faucet.
- **[Errors and rate limits](../guides/errors-and-rate-limits)** - the `apikey.*` error catalog.
- **[Going live](../guides/going-live)** - the production hardening checklist.