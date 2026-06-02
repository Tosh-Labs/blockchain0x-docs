---
title: Ruby SDK reference
---
The `blockchain0x` gem. Install with `gem install blockchain0x`, then construct a
client:

```ruby
client = Blockchain0x::Client.new(api_key: ENV.fetch('B0X_API_KEY'))  # sk_test_* -> testnet
```

`Client.new` takes `api_key:`, `base_url:` (default `https://api.blockchain0x.com`),
`network:` (`:mainnet`/`:testnet`, inferred from the key prefix when omitted), and
`timeout:`. Resources are `client.payments`, `client.payment_requests`,
`client.transactions`, and `client.api_keys`. API failures raise
`Blockchain0x::Error` (or `Blockchain0x::APIKeyError`).

> Like the Go SDK, the Ruby gem does not expose an `agents` resource; create
> agents from the dashboard. Ruby covers the spend-and-settle path.

## `client.payments`

`create(body:, idempotency_key: nil)`. Pass `body:` as a Hash with the canonical
camelCase wire keys (or a `PaymentCreateBody`). An `Idempotency-Key` is minted
automatically unless you pass `idempotency_key:`.

```ruby
payment = client.payments.create(
  body: {
    agentId: 'agw_demo',
    to: '0x000000000000000000000000000000000000dEaD',
    amountWei: '10000' # 0.01 USDC
  }
)
puts "payment #{payment['id']} #{payment['status']}"
```

## `client.payment_requests`

`settle(payment_request_id, body)` - `body` is a Hash (or `PaymentRequestSettleBody`)
with the on-chain proof tuple `txHash` / `payerAddress` / `amountUsdcVerified`.

```ruby
settled = client.payment_requests.settle(
  'pr_demo',
  { txHash: '0x' + 'a', payerAddress: '0x' + 'b', amountUsdcVerified: '25.00' }
)
puts "settled #{settled['settledTxHash']}"
```

## `client.transactions`

`get(transaction_id)` - read on-chain transaction state.

## `client.api_keys`

| Method                               | Description                 |
| ------------------------------------ | --------------------------- |
| `api_keys.create_workspace_key(...)` | Mint a workspace-level key. |
| `api_keys.create_agent_key(...)`     | Mint an agent (wallet) key. |
| `api_keys.list`                      | List the workspace's keys.  |
| `api_keys.get(api_key_id)`           | Fetch one key.              |
| `api_keys.revoke(api_key_id)`        | Revoke a key.               |

## `Blockchain0x::Webhooks.verify`

The verifier is a module method. It returns a result with `.ok` and `.code`; pass
`raise_on_fail: true` to raise `Blockchain0x::WebhookSignatureError` instead.
Always pass the raw body exactly as it arrived:

```ruby
result = Blockchain0x::Webhooks.verify(
  headers: {}, raw_body: '{}', secret: ENV.fetch('B0X_WEBHOOK_SECRET', '')
)
puts "bad signature: #{result.code}" unless result.ok
```

## Next

- **[Node](./node)**, **[Python](./python)**, **[Go](./go)** - the same surface.
- **[x402](./x402)** - HTTP-402 pay-per-call.