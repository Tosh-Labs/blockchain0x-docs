---
title: Receive payments
---
For your **agent** to get paid, your code (through the SDK) issues a **payment
request** (an invoice) on the agent's behalf, the payer sends USDC on-chain, and
your code then settles the invoice with proof of that transfer. This guide walks
the full lifecycle. The key your code uses needs the `receive_money` scope (see
[Authentication](../getting-started/authentication)).

## The lifecycle

1. **Your agent creates** a payment request for an amount - it starts `open` and
   carries a `pr_*` id.
2. **You share** the request with the payer (a hosted page, or via the x402 `402`
   handshake - see the [x402 reference](../sdks/x402)).
3. **The payer transfers** the USDC on Base.
4. **Your code settles** the request with the on-chain proof tuple; the backend
   verifies it and flips the invoice to `settled`.
5. **A webhook fires** so your system reacts without polling.

## Settle with on-chain proof

Once the transfer confirms, settle the invoice. The backend re-verifies all of
(txHash, payer, amount, recipient) before accepting it - a tampered tuple is
rejected, and an already-settled invoice returns `409 payment_request.not_settleable`:

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

> Using x402 to charge for an HTTP endpoint? The server adapter does this settle
> step for you after verifying the `X-PAYMENT` header. See the
> [x402 reference](../sdks/x402).

## Confirm with a webhook

Rather than polling, register a webhook and let Blockchain0x notify you when an
invoice settles. Always verify the signature before trusting the body:

```ts
const result = webhooks.verify({ headers, rawBody, secret: process.env.B0X_WEBHOOK_SECRET! });
if (!result.ok) {
  throw new Error(`webhook signature rejected: ${result.code}`);
}
const event = JSON.parse(rawBody) as { type: string; data: unknown };
console.log('verified event', event.type, event.data);
```

See the [Webhooks guide](./webhooks-and-signature-verification) for the event
catalogue, the replay window, and verification in every language.

## Next

- **[Send payments and idempotency](./send-payments-and-idempotency)** - the outbound side.
- **[Webhooks and signature verification](./webhooks-and-signature-verification)**.