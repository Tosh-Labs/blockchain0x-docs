---
title: Go SDK reference
---
# Go SDK reference

`github.com/Tosh-Labs/blockchain0x-go`. Install with
`go get github.com/Tosh-Labs/blockchain0x-go`. Every call takes a
`context.Context` and returns a typed value plus an `error`; API failures come
back as `*blockchain0x.Error` (or `*blockchain0x.APIKeyError`), so match with
`errors.As`.

Construct a client - `NewClient` returns an error when the API key is missing:

```go
client, err := blockchain0x.NewClient(blockchain0x.Options{APIKey: os.Getenv("B0X_API_KEY")})
if err != nil {
	panic(err)
}
```

`Options` carries `APIKey`, `BaseURL` (default `https://api.blockchain0x.com`),
`Network` (`"mainnet"`/`"testnet"`, inferred from the key prefix when empty),
`Timeout` (default 30s), and `MaxRetries` (default 3).

> The Go SDK does not expose an `Agents` resource; create and manage agents from
> the dashboard or another SDK. Go covers the spend-and-settle path.

## `client.Payments`

`Create(ctx, PaymentCreateRequest, RequestOptions) (*Payment, error)`. The request
struct is `{ AgentID, To, AmountWei, Token?, Metadata? }`; `AmountWei` is the
integer USDC amount (6 decimals, so `"10000"` = 0.01 USDC).

```go
payment, err := client.Payments.Create(ctx, blockchain0x.PaymentCreateRequest{
	AgentID:   "agw_demo",
	To:        "0x000000000000000000000000000000000000dEaD",
	AmountWei: "10000", // 0.01 USDC
}, blockchain0x.RequestOptions{})
if err != nil {
	panic(err)
}
fmt.Println("payment", payment.ID, payment.Status)
```

## `client.PaymentRequests`

`Settle(ctx, paymentRequestID, PaymentRequestSettleBody) (*PaymentRequestSettled, error)`.
The body is the on-chain proof tuple `{ TxHash, PayerAddress, AmountUsdcVerified }`.

```go
settled, err := client.PaymentRequests.Settle(ctx, "pr_demo", blockchain0x.PaymentRequestSettleBody{
	TxHash:             "0x" + "a",
	PayerAddress:       "0x" + "b",
	AmountUsdcVerified: "25.00",
})
if err != nil {
	panic(err)
}
fmt.Println("settled", settled.SettledTxHash)
```

## `client.Transactions`

`Get(ctx, transactionID) (*Transaction, error)` - read on-chain transaction state.

## `client.APIKeys`

| Method                                 | Returns                |
| -------------------------------------- | ---------------------- |
| `APIKeys.CreateWorkspaceKey(ctx, ...)` | `(*APIKey..., error)`  |
| `APIKeys.CreateAgentKey(ctx, ...)`     | `(*APIKey..., error)`  |
| `APIKeys.List(ctx)`                    | `(*APIKeyPage, error)` |
| `APIKeys.Get(ctx, apiKeyID)`           | `(*APIKey, error)`     |
| `APIKeys.Revoke(ctx, apiKeyID)`        | `error`                |

## `webhooks.Verify`

The verifier lives in the `webhooks` subpackage:
`Verify(Args) VerifyResult`, where `Args` is `{ Headers, RawBody, Secret }` and the
result is `{ OK, Code, ... }`. Pass the raw body exactly as it arrived. A
`VerifyHTTPRequest(r, secret, body)` helper wraps `*http.Request`.

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

## Next

- **[Node](./node)**, **[Python](./python)**, **[Ruby](./ruby)** - the same surface.
- **[x402](./x402)** - HTTP-402 pay-per-call.