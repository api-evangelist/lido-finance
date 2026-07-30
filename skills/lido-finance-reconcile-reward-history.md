---
name: Reconcile a wallet's stETH reward history
description: >-
  Pull every stETH interaction for an Ethereum address and its daily reward accrual, denominated in
  fiat, from the Lido Reward History Backend — for tax, accounting or portfolio reporting.
api: openapi/lido-finance-reward-history-openapi-original.json
base_url: https://reward-history-backend.lido.fi
authentication: none
operations:
  - RewardsController_rewards
generated: '2026-07-19'
method: generated
---

# Reconcile a wallet's stETH reward history

One endpoint, `RewardsController_rewards` (`GET /`), returns all stETH interactions for an address
and calculates its daily stETH rewards. Public and read-only — send no credentials.

## Steps

1. **Call with the address.** `address` is the only required query parameter and must be a valid
   Ethereum address.
2. **Choose the fiat basis.** Set `currency` to `usd`, `eur` or `gbp` (default `usd`). Set
   `archiveRate=true` (the default) to value each event at an exchange rate close to its
   transaction time — this is what you want for tax and accounting. `archiveRate=false` values
   everything at the current rate, which is only appropriate for a "what is it worth now" view.
3. **Filter if you only want yield.** Set `onlyRewards=true` to exclude transfers and stakings and
   return reward events alone.
4. **Page.** Use `skip` and `limit` for offset pagination — `skip:0 limit:100` is page 1,
   `skip:100 limit:100` is page 2. Set `sort` to `asc` or `desc` (default `desc` by `blockTime`).
5. **Read the events.** Each event carries `type` (e.g. `staking`), `shares`, `sharesBefore`,
   `sharesAfter`, `totalPooledEtherBefore/After`, `totalSharesBefore/After`, `balance`, `change`,
   `currencyChange`, `block`, `blockTime`, `transactionHash` and `epochDays`.

## Rules

- All numeric on-chain fields are wei-scale **decimal strings**. Parse them as big integers, never
  as floats — stETH rewards are lost in float rounding.
- stETH is a **rebasing** token: balance changes without a transfer. That is why reconciliation must
  go through `shares` and the `totalPooledEther`/`totalShares` pair rather than diffing balances.
- Documented failure modes (`errors/lido-finance-problem-types.yml`): `400` when `address` is
  missing/invalid or another input is invalid; `500` when subgraph limits were hit or the app hit a
  critical error. On `500`, narrow the window with `skip`/`limit` or `onlyRewards=true` and retry
  with backoff — it signals an upstream indexing limit, not a caller rate limit.
- No rate-limit policy is published. Page in bounded chunks and cache results rather than
  re-walking full history per request.
- For deeper or custom historical queries, the Lido subgraph on The Graph is the richer source
  behind this view (`graphql/lido-finance-subgraph-schema.graphql`) — it requires a The Graph API
  key, which Lido does not issue.
- Hoodi testnet equivalent: `http://reward-history-backend-hoodi.testnet.fi/?address=...`.
