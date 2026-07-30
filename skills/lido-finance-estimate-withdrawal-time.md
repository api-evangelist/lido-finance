---
name: Estimate a Lido withdrawal wait time
description: >-
  Tell a staker how long a Lido stETH withdrawal will take — before they place the request, and
  after — using the unauthenticated Lido Withdrawals API, plus the withdrawal-request NFT metadata.
api: openapi/lido-finance-withdrawal-queue-openapi-original.json
base_url: https://wq-api.lido.fi
authentication: none
operations:
  - RequestTimeController_calculateTime
  - RequestTimeController_requestsTime
  - NFTController_nftMetaV1
  - NFTController_nftImageV1
  - EstimateController_requestTimeV1
  - ValidatorsController_validatorsV1
generated: '2026-07-19'
method: generated
---

# Estimate a Lido withdrawal wait time

Withdrawing stETH puts a request in the Lido withdrawal queue; it becomes claimable only after
finalization. This API estimates that wait. It is public and read-only — send no credentials.

## Steps

1. **Before the user requests.** Call `RequestTimeController_calculateTime`
   (`GET /v2/request-time/calculate`) with `amount` set to the stETH the user intends to withdraw,
   e.g. `?amount=32`. Omit `amount` to get the wait time for the current queue as it stands.
2. **After the request exists.** Call `RequestTimeController_requestsTime`
   (`GET /v2/request-time`) with the withdrawal request ids as **repeated** query parameters:
   `?ids=1&ids=2`. The response returns a per-request estimate keyed by request id, each with the
   underlying request info.
3. **Show the position as an object.** Each queue position is an ERC-721 issued by
   `WithdrawalQueueERC721`, where `tokenId` equals the request id. Call `NFTController_nftMetaV1`
   (`GET /v1/nft/{tokenId}`) for metadata and `NFTController_nftImageV1`
   (`GET /v1/nft/{tokenId}/image`) for the image.
4. **Price the transaction.** Call `EstimateController_requestTimeV1` (`GET /v1/estimate-gas`) for a
   gas estimate on the withdrawal-related transaction.
5. **Explain a long queue.** Call `ValidatorsController_validatorsV1` (`GET /v1/validators-info`)
   for the validator-set information the wait-time model is built on.

## Rules

- Use the **v2** request-time endpoints. `RequestTimeController_requestTimeV1`
  (`GET /v1/request-time`) still resolves but v2 is what the documentation points at.
- Errors use the NestJS envelope `{ statusCode, message[], error }`, not RFC 9457. Two documented
  `400`s matter here: `ids` must be an array of string request ids, and `amount` must be a valid ETH
  value (`errors/lido-finance-problem-types.yml`).
- The optional `WQ-Request-Source` header accepts `widget`, `sdk` or `unknown`. It is attribution
  telemetry, not auth — set it to `sdk` when you are an integrator, or omit it.
- These are estimates over a live queue, not commitments. Re-query rather than caching a wait time
  across a session, and never present the number as a guaranteed finalization date.
- Placing or claiming a withdrawal is an **on-chain** action — this API cannot do it. Use
  `@lidofinance/lido-ethereum-sdk` (`LidoSDKWithdraw`) for `requestWithdrawal` and `claim`
  (`packages/lido-finance-packages.yml`).
- Test against `https://wq-api-hoodi.testnet.fi` on the Hoodi testnet.
