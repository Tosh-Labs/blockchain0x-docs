---
title: Quickstart
---
# Quickstart

In about ten minutes your **AI agent** will send a real, confirmed USDC payment on
Base testnet - using nothing but our SDK. Every code block below is lifted
verbatim from a real, CI-tested example in our SDK repos; nothing here is
hand-typed.

> **Quick vocabulary** (we use these words precisely throughout the docs):
>
> - **Agent** (or **agent wallet**) - an on-chain smart wallet on Base that your
>   autonomous program owns and controls. It holds USDC, sends payments, and gets
>   paid. "Agent" and "agent wallet" mean the same thing.
> - **You / your code** - the program you write with a Blockchain0x SDK. It acts
>   *on behalf of* an agent.
> - **API key** - the credential that authenticates your code. It carries the
>   agent's identity and permissions (more in [Authentication](./authentication)).

## 1. Get a test API key (60 seconds)

Sign up at [wallet.blockchain0x.com](https://wallet.blockchain0x.com) and create a
workspace. Signing up also provisions your first **agent wallet** automatically -
you'll use it below. Then generate an API key. A key that starts with `sk_test_`
is a **test-mode** key: it talks to Base Sepolia and spends testnet USDC, so you
can run this guide with zero real funds. Keep `sk_live_` keys for production.

Export the key so the snippets below can read it:

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

Your code talks to Blockchain0x through a client. The key prefix selects the
network for you - `sk_test_*` points at testnet, so there is nothing else to
configure.

```ts
const client = createClient({ apiKey: process.env.B0X_API_KEY! }); // sk_test_* -> testnet
```

```python
client = Client(api_key=os.environ["B0X_API_KEY"])  # sk_test_* -> testnet
```

## 4. Find your agent wallet's id

You do **not** create an agent in code. Agent wallets are provisioned in the
dashboard - your first one was created when you signed up. Open
[wallet.blockchain0x.com](https://wallet.blockchain0x.com), select your agent, and
copy its id (it looks like `agw_...`). Every payment call names the agent it acts
for, by this id.

> **Do I have to pass the id every time?** Yes - your code always names the agent
> it is paying from. How the key relates to the id:
>
> - An **agent-scoped key** is bound to exactly one agent. You still pass that
>   agent's `agentId`, and it must match the key's agent (a mismatch is rejected
>   with `apikey.agent_mismatch`).
> - A **workspace key** can act for any agent in the workspace, so the `agentId`
>   you pass is what selects which agent pays.

Export your agent id alongside the key:

```bash
export B0X_AGENT_ID=agw_your_agent_id
```

## 5. Send your first payment

Now your agent sends `0.01` USDC. Amounts are integers in the token's smallest
unit - USDC has 6 decimals, so `10000` = 0.01 USDC. The call returns the on-chain
transaction hash.

```ts
// `agentId` is your agent wallet's id - copy it from the dashboard.
const payment = await client.payments.create({
  agentId: process.env.B0X_AGENT_ID!,
  to: '0x000000000000000000000000000000000000dEaD',
  amountWei: '10000', // 0.01 USDC (6 decimals)
});
console.log('payment', payment.id, payment.status, payment.txHash);
```

The same call in Python:

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

When `payment.txHash` is populated and `status` reaches `confirmed`, your agent has
made a real USDC transfer on Base Sepolia. Paste the hash into
[sepolia.basescan.org](https://sepolia.basescan.org) to see it on-chain.

> **Test agent has no balance?** A brand-new test agent needs test USDC before it
> can pay. See [Create and fund an agent](../guides/create-and-fund-an-agent) for
> the faucet step.

## Next steps

- **[Authentication](./authentication)** - scopes, key levels, key rotation.
- **[Test mode vs live mode](./test-mode-vs-live-mode)** - the faucet and `X-Network`.
- **[Receive payments](../guides/receive-payments)** - how an agent gets paid.
- **[SDK reference](../sdks/overview)** - every method, every language.