---
title: Set up and fund an agent
---
An **agent** (agent wallet) is the on-chain wallet your AI agent controls - it
holds USDC, signs payments, and gets paid. This guide gets one ready to transact.

## 1. Provision the agent

Agent wallets are created in the **dashboard**, not through the API (agent
creation is an account action, so it is not exposed to API keys). You already have
one from signup; to add more, open
[wallet.blockchain0x.com](https://wallet.blockchain0x.com) and create an agent
there. Use a test-mode workspace so the agent lives on Base Sepolia.

Each agent has:

- an **id** (`agw_...`) - how your code names the agent in every call;
- an **on-chain address** - where its USDC actually lives on Base.

Copy the id; your code will pass it as `agentId` (see
[Quickstart, step 4](../getting-started/quickstart)).

## 2. Fund it with test USDC

A fresh agent has a zero balance, so it needs test USDC before it can pay.
Blockchain0x does not operate a faucet - fund the agent's on-chain address from
the public Base Sepolia faucets:

- [Coinbase Base Sepolia faucet](https://portal.cdp.coinbase.com/products/faucet)
- [Circle USDC faucet](https://faucet.circle.com)

In the dashboard, a test-mode agent shows a **Get test USDC** button that links
straight to these. Send USDC to the agent's address, wait for the transfer to
confirm, and the balance appears on the agent.

> Funding works the same way in live mode, except you send **real** USDC to the
> agent's mainnet address. See [Test mode vs live mode](../getting-started/test-mode-vs-live-mode).

## 3. Confirm it is ready

From your code, read the agent back with `client.agents.get(agentId)` to confirm
its network and balance. Once it holds USDC, your agent can pay bills and receive
money - continue with:

- **[Send payments and idempotency](./send-payments-and-idempotency)** - your agent pays.
- **[Receive payments](./receive-payments)** - your agent gets paid.