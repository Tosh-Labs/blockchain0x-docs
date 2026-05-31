---
title: Quickstart
---
# Quickstart

Ten minutes from `install` to a **confirmed on-chain USDC payment** on Base
testnet. Every code block below is lifted verbatim from a real, CI-tested
example in our SDK repos - nothing here is hand-typed.

## 1. Get a test API key (60 seconds)

Sign up at [wallet.blockchain0x.com](https://wallet.blockchain0x.com), create a
workspace, and generate an API key. A key that starts with `sk_test_` is a
**test-mode** key - it talks to Base Sepolia and spends testnet USDC, so you can
run this guide with zero real funds. Keep `sk_live_` keys for production.

Export it so the snippets below can read it:

```bash
export B0X_API_KEY=sk_test_your_key_here
```

## 2. Install the SDK

```bash
npm install @blockchain0x/node
```

```bash
pip install blockchain0x
```

## 3. Construct the client

The key prefix decides the network for you - `sk_test_*` points at testnet.

```ts
const client = createClient({ apiKey: process.env.B0X_API_KEY! }); // sk_test_* -> testnet
```

```python
client = Client(api_key=os.environ["B0X_API_KEY"])  # sk_test_* -> testnet
```

## 4. Create an agent

An **agent** is the wallet that holds funds and signs payments. Test-mode agents
are auto-funded from the testnet faucet, so a fresh agent can pay immediately.

```ts
const agent = await client.agents.create({ name: 'My first agent', slug: `agent-${Date.now()}` });
```

## 5. Make your first payment

Send `0.01` USDC (amounts are in wei - USDC has 6 decimals, so `10000` = 0.01)
and read back the on-chain transaction hash.

```ts
const payment = await client.payments.create({
  agentId: agent.id,
  to: '0x000000000000000000000000000000000000dEaD',
  amountWei: '10000', // 0.01 USDC (6 decimals)
});
console.log('payment', payment.id, payment.status, payment.txHash);
```

In Python the agent-create and payment are the same two calls:

```python
agent = client.agents.create({"name": "My first agent", "slug": f"agent-{os.getpid()}"})
payment = client.payments.create(
    body={
        "agentId": agent["id"],
        "to": "0x000000000000000000000000000000000000dEaD",
        "amountWei": "10000",  # 0.01 USDC
    }
)
print("payment", payment["id"], payment["status"])
```

When `payment.txHash` is populated and `status` reaches `confirmed`, you have a
real USDC transfer on Base Sepolia. Paste the hash into
[sepolia.basescan.org](https://sepolia.basescan.org) to see it on-chain.

## Next steps

- **[Authentication](./authentication)** - scopes, key rotation, test vs live keys.
- **[Test mode vs live mode](./test-mode-vs-live-mode)** - the faucet and `X-Network`.
- **[Receive payments](../guides/receive-payments)** - PaymentRequests + webhooks.
- **[SDK reference](../sdks/overview)** - every method, every language.