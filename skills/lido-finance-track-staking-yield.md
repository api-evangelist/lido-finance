---
name: Track Lido staking yield
description: >-
  Read Lido's current and historical Ethereum staking yield — latest stETH APR, the 7-day simple
  moving average, ETH/stETH prices and protocol stats — from the unauthenticated Lido Ethereum API.
api: openapi/lido-finance-eth-api-openapi-original.json
base_url: https://eth-api.lido.fi
authentication: none
operations:
  - ProtocolController_findLastAPRforSTETH
  - ProtocolController_findSmaAPRforSTETH
  - ProtocolController_findAPRforSTETH
  - ProtocolController_findStEthPrice
  - ProtocolController_findEthPrice
  - ProtocolController_findStethStats
generated: '2026-07-19'
method: generated
---

# Track Lido staking yield

The Lido Ethereum API is public and read-only. Send no credentials — there are no API keys, no
OAuth, and no scopes (`authentication/lido-finance-authentication.yml`).

## Steps

1. **Get the headline yield.** Call `ProtocolController_findLastAPRforSTETH`
   (`GET /v1/protocol/steth/apr/last`) for the most recent stETH APR datapoint. For Lido V2+ this
   value is computed from `TokenRebased` events rather than periodic oracle polls, so it moves once
   per oracle report frame.
2. **Prefer the smoothed number for anything user-facing.** Call
   `ProtocolController_findSmaAPRforSTETH` (`GET /v1/protocol/steth/apr/sma`) for the 7-day simple
   moving average. Single-report APR is noisy; the SMA is what Lido's own surfaces quote.
3. **Pull the series for charts.** Call `ProtocolController_findAPRforSTETH`
   (`GET /v1/protocol/steth/apr`) for the historical series. The response carries `data`, `meta`
   and a `pagination` block — page through it rather than requesting an unbounded window.
4. **Denominate it.** Call `ProtocolController_findStEthPrice` and `ProtocolController_findEthPrice`
   for current stETH and ETH prices, and `ProtocolController_findStethStats` for short-form
   protocol statistics.

## Rules

- Every operation is a `GET`. Nothing here changes state, so retries are always safe — there is no
  idempotency-key contract because none is needed (`conventions/lido-finance-conventions.yml`).
- The spec declares only `200` responses for this service; treat any non-2xx as transient and back
  off. No rate-limit policy is published, so be conservative: cache the APR for at least one oracle
  frame rather than polling per page view.
- Amounts in responses are wei-scale decimal strings. Do not parse them as floats.
- To test against a testnet, swap the host for `https://eth-api-hoodi.testnet.fi` — same paths
  (`sandbox/lido-finance-sandbox.yml`).
- Do not confuse the APR of ETH staking (`/v1/protocol/eth/apr`) with the stETH APR
  (`/v1/protocol/steth/apr`): the latter is net of Lido protocol fees.
