---
title: Errors and rate limits
---
Every API error comes back in one shape, and every SDK turns it into a typed
exception you can branch on. This guide covers the envelope, the API-key error
catalog, and rate limiting.

## The error envelope

A non-2xx response carries a single JSON object:

```json
{
  "error": {
    "code": "request.invalid",
    "message": "Request body failed validation.",
    "requestId": "01HQX5NXY2P3W2YGJ8ZQK5V4G6"
  }
}
```

- **`code`** - a stable, dot-namespaced `snake_case` identifier. Branch on this;
  never parse `message`.
- **`message`** - human-readable English, safe to log, never localised.
- **`requestId`** - matches the `x-request-id` response header. Quote it to support.

The SDKs raise this as `Blockchain0xError` (with `.code` / `.status` /
`.message` / `.requestId`), or the `ApiKeyError` subclass for `apikey.*` codes. See
the [SDKs overview](../sdks/overview) for the per-language error types.

## API-key error catalog

The closed set of `apikey.*` codes - branch on these for auth failures:

| Code                                  | Meaning                                              |
| ------------------------------------- | ---------------------------------------------------- |
| `apikey.invalid`                      | The key is malformed or unknown.                     |
| `apikey.expired`                      | The key has passed its expiry.                       |
| `apikey.revoked`                      | The key was revoked.                                 |
| `apikey.agent_revoked`                | The owning agent was revoked.                        |
| `apikey.scope_insufficient`           | The key lacks the agent scope this call needs.       |
| `apikey.workspace_scope_insufficient` | The key lacks the workspace scope this call needs.   |
| `apikey.network_mismatch`             | The `X-Network` does not match the key's mode.       |
| `apikey.agent_mismatch`               | The key is bound to a different agent.               |
| `apikey.wallet_not_assigned`          | No wallet is assigned to the key.                    |
| `apikey.unsupported_endpoint`         | This endpoint is not callable with an agent key.     |
| `apikey.workspace_endpoint_blocked`   | This endpoint is dashboard-only for the key's level. |
| `apikey.no_grants_remaining`          | No wallet grants remain to assign.                   |
| `apikey.role_insufficient_for_grants` | The caller's role cannot issue these grants.         |

The canonical catalog (every code across the API, not just `apikey.*`) lives in
[05-api-design.md](../../plan/05-api-design.md) and
`packages/shared/src/types/errors.ts`.

## Rate limits

When you exceed a rate limit the API responds `429` with a `Retry-After` header
(integer seconds to wait). The SDKs **honour it automatically**: on a `429` (and
on `5xx`) they retry up to 3 times with exponential backoff, waiting the
`Retry-After` value when present. If you build raw HTTP calls yourself, read
`Retry-After` and back off rather than hammering.

Note that `payments.create` does **not** auto-retry by default (a lost response
must not double-spend) - see
[Send payments and idempotency](./send-payments-and-idempotency).

## Next

- **[Going live](./going-live)** - the production checklist.
- **[Authentication](../getting-started/authentication)** - scopes behind the `apikey.*` codes.