---
title: Python SDK reference
---
# Python SDK reference

`blockchain0x` for Python. Install with `pip install blockchain0x`, then construct
a `Client`:

```python
client = Client(api_key=os.environ["B0X_API_KEY"])  # sk_test_* -> testnet
```

Python idioms differ from the Node reference in three ways: methods are
`snake_case`, request bodies are plain `dict`s with the canonical camelCase wire
keys (or the provided dataclasses), and resources are reached as
`client.agents`, `client.api_keys`, and so on. A non-2xx response raises
`Blockchain0xError` (or `ApiKeyError`).

## `client.agents`

| Method                                 | Returns | Description          |
| -------------------------------------- | ------- | -------------------- |
| `agents.get(agent_id)`                 | `dict`  | Fetch one agent.     |
| `agents.list(cursor=None, limit=None)` | `dict`  | Page through agents. |

> Creating agents is a dashboard action, not an API-key call - provision them at
> [wallet.blockchain0x.com](https://wallet.blockchain0x.com) and reference them by
> `agentId`.

## `client.payments`

`payments.create` is keyword-only: pass `body` as a `dict` (camelCase wire keys) or
a `PaymentCreateBody(agent_id=..., to=..., amount_wei=..., token=None, metadata=None)`.
An `Idempotency-Key` is minted automatically unless you pass `idempotency_key`.

| Method                                           | Returns | Description                 |
| ------------------------------------------------ | ------- | --------------------------- |
| `payments.create(*, body, idempotency_key=None)` | `dict`  | Create an outbound payment. |

```python
# `agentId` is your agent wallet's id - copy it from the dashboard.
payment = client.payments.create(
    body={
        "agentId": os.environ["B0X_AGENT_ID"],
        "to": "0x000000000000000000000000000000000000dEaD",
        "amountWei": "10000",  # 0.01 USDC
    }
)
print("payment", payment["id"], payment["status"])
```

## `client.api_keys`

Key creation is split by level. Workspace keys and agent (wallet) keys are minted
by separate methods so the scope sets stay type-safe.

| Method                                                                | Returns | Description                 |
| --------------------------------------------------------------------- | ------- | --------------------------- |
| `api_keys.create_workspace_key(*, label, workspace_scopes=None, ...)` | `dict`  | Mint a workspace-level key. |
| `api_keys.create_agent_key(*, agent_id, label, scopes)`               | `dict`  | Mint an agent (wallet) key. |
| `api_keys.list()`                                                     | `dict`  | List the workspace's keys.  |
| `api_keys.get(api_key_id)`                                            | `dict`  | Fetch one key.              |
| `api_keys.revoke(api_key_id)`                                         | `None`  | Revoke a key.               |

The minted secret is on the returned dict and is shown **once**. See
[Authentication](../getting-started/authentication) for the scope sets.

## `client.payment_requests`

| Method                                                 | Returns | Description                   |
| ------------------------------------------------------ | ------- | ----------------------------- |
| `payment_requests.settle(*, payment_request_id, body)` | `dict`  | Settle an invoice with proof. |

`body` is a `dict` (or `PaymentRequestSettleBody`) carrying the on-chain proof
tuple `tx_hash` / `payer_address` / `amount_usdc_verified`; the backend rejects a
tampered tuple with `payment_request.settle_proof_invalid`.

## `client.transactions`

| Method                             | Returns | Description            |
| ---------------------------------- | ------- | ---------------------- |
| `transactions.get(transaction_id)` | `dict`  | Fetch one transaction. |

## `webhooks.verify()`

The webhook signature verifier is a standalone module - import `webhooks` from the
package. It does constant-time HMAC verification and never logs the secret. It
returns a `VerifyResult` with `.ok` and a `.code`; pass `raise_on_fail=True` to
raise `WebhookSignatureError` instead:

```python
result = webhooks.verify(headers=headers, raw_body=raw_body, secret=os.environ["B0X_WEBHOOK_SECRET"])
if not result.ok:
    raise ValueError(f"bad signature: {result.code}")
```

`verify(headers, raw_body, secret, *, tolerance_sec=..., now=None, raise_on_fail=False)` -
always pass the raw body exactly as it arrived (do not pre-parse JSON).

## Next

- **[Node](./node)**, **[Go](./go)**, **[Ruby](./ruby)** - the same surface.
- **[x402](./x402)** - HTTP-402 pay-per-call.