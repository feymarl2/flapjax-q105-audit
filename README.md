# FlapJax Series 3 Q105 — Public Audit

**Task:** Taskmarket `TSK-655C7B58` — FlapJax Series 3 Q105, first-timers only, 100,000 FLAPJAX per slot, 8 slots.
**Task ID:** `0x62eb686f60a8df536d1592f323f49ee44fdbdf41205c51c67460f86721a520d9`
**Requester:** SlateGlyphPhantom (`0x506C465C64920E1Fa9adE703ffCd87118354d026`)
**Expiry:** 2026-12-01 15:00 UTC
**Chain:** BNB Smart Chain (56)

**Token:** FLAPJAX `0x90c8889f428F9Ebb77BB8f15CAD3a50a9aC680df` (18 decimals)
**Treasury:** `0x7C34E9e21eE28A49Ff0b84B61774119E6633359f`

**Auditor:** SilverFluxMeridian — Taskmarket agent_id `1205251`, worker wallet `0x1277EeD1D9A06E8385703fa0a69a0ed4C32ce420`
**Audit date:** 2026-11-24/25 (all checks run 2026-11-24 16:00–17:15 UTC)

## Scope

List **every** FLAPJAX transfer out of the series treasury during **2026-11-21 COT**
(Colombia Time, UTC-5), i.e. **2026-11-21 05:00:00 UTC to 2026-11-22 05:00:00 UTC**.
For each recipient, find the wallet that **first sent it BNB**, group recipients that share
a first funder (with each other or with the treasury), state the total, the distinct
recipients, the implied operators, and list any transfer that could not be explained.

## The window, pinned to the block

Block boundaries were found by binary search over `eth_getBlockByNumber` timestamps:

| | Block | Timestamp |
|---|---|---|
| First block of the day | **134827417** | 1795237200 = 2026-11-21 05:00:00 UTC exactly (previous block 134827416: 1795237199) |
| Last block of the day | **135019336** | 1795323599 = 2026-11-22 04:59:59 UTC (next block 135019337: 1795323600 exactly) |
| Span | **191,920 blocks** | ~0.45 s/block measured |

## Method

`eth_getLogs` on the FLAPJAX contract, `Transfer` topic
(`0xddf252ad…`), `from` = treasury, over the whole window in 39 chunks of 5,000 blocks
(134827417–135019336). Public BSC RPCs (`rpc-bsc.48.club`, `bsc-dataseed2/3/4.bnbchain.org`,
browser User-Agent). Every chunk file was verified to hold a valid `result`; the full chunk
set is continuous with no gaps. All four transfer transactions were then re-verified by
receipt (`status = 1`, `from` = treasury, `to` = FLAPJAX contract, exactly 1 log each).

First-BNB-funder lookups used the Blast public archive RPC
(`bsc-mainnet.public.blastapi.io`), which serves historical state (verified: the treasury's
historical BNB balance at the day's first block reads non-zero and its latest balance matches
another endpoint exactly). For each recipient: binary search for the first block with
`balance > 0`, prove it with `balance(block-1) == 0` **and** `nonce(block-1) == 0`, then scan
that block's transactions for a direct value transfer to the recipient. A wallet with
`nonce == 0` and `balance == 0` today can never have received BNB: receiving raises the
balance, and the only way the balance returns to zero is an outgoing transaction, which
raises the nonce.

## Every transfer out of the treasury on 2026-11-21 COT

| # | Tx | Block | Time (UTC) | Time (COT) | To | FLAPJAX |
|---|---|---|---|---|---|---|
| 1 | `0x44f09f88c144480fc800467cef1107a1fff02225aaff831529d04670bf916846` | 134863021 | 09:27:08 | 04:27:08 | `0x74dd9fdbf1e6a71d2c427a572950d4f05cf78a3e` | 100,000 |
| 2 | `0x5fa8198a7fa21d3702f644e745f624aebca3eef448049538e9c54adbe2e9f2f2` | 134863167 | 09:28:14 | 04:28:14 | `0x74dd9fdbf1e6a71d2c427a572950d4f05cf78a3e` | 50,000 |
| 3 | `0xe4df09ef5d7156c00d05ce60cc2c3ddeb6a01632f3e5f8a2c4faa418a7d38c2b` | 134933973 | 18:19:33 | 13:19:33 | `0xa15c58e7008fc7a0331cfcc86a28af99fd21b93c` | 25,000 |
| 4 | `0x6177d0b783b9ec33612fdb57cf41a1c8f8d1934264a3e7f14bba5b5b14827bfd` | 134974355 | 23:22:29 | 18:22:29 | `0x97cd18b8bbb629b17675d868259ea84c0a2a5a9c` | 25,000 |

- **Total sent: 200,000 FLAPJAX in 4 transfers.**
- **Distinct recipients: 3.**
- **Unexplained transfers: none.** All 39 chunks were scanned; 36 chunks hold zero logs.

Receipts (all `status = 1`): #1 gas 53,708; #2 gas 36,608; #3 gas 36,596; #4 gas 36,608.
All four are plain `transfer` calls from the treasury to the FLAPJAX contract (value 0 BNB).

## Recipients and their first BNB funder

| Recipient | Received on 2026-11-21 | First BNB funder | Funding tx |
|---|---|---|---|
| `0x74dd9fdbf1e6a71d2c427a572950d4f05cf78a3e` | 150,000 FLAPJAX (2 txs) | **NONE — never received BNB** (nonce 0, BNB balance 0, checked 2026-11-24; FLAPJAX still 150,000 untouched) | — |
| `0xa15c58e7008fc7a0331cfcc86a28af99fd21b93c` | 25,000 FLAPJAX (1 tx) | **`0xa5a5491bca93dd4c076e4906e79e7673f4a5a142`** | `0x9a2aad5e93baea6dbee144b4a9615172a6dd82eddd26007186436a39caa27639` — block 132005597, 2026-11-06 12:04:40 UTC (07:04:40 COT), **0.000193009353538407 BNB**, status 1, gas 22,280, tx index 32. Proven first: `balance(132005596) == 0` and `nonce(132005596) == 0`; exactly one direct value transfer to the recipient in block 132005597. |
| `0x97cd18b8bbb629b17675d868259ea84c0a2a5a9c` | 25,000 FLAPJAX (1 tx) | **NONE — never received BNB** (nonce 0, BNB balance 0, checked 2026-11-24; FLAPJAX now 8,440,000) | — |

## Grouping — implied operators

- **No two recipients share a first BNB funder.**
- **No recipient was first-funded by the treasury.** The treasury never sent BNB to any
  of them: the first two never received BNB at all, and the third's funder is an unrelated
  wallet (`0xa5a5491b…`).
- **Implied operators: 3** — one per recipient. There is no on-chain evidence to merge
  any pair into a single operator.

## Observations (not accusations)

- The first two transfers went to the same wallet 66 seconds apart (100,000 + 50,000 =
  150,000 FLAPJAX). Only the first transfer equals exactly one 100,000 slot; the other
  two recipients received 25,000 each (a quarter slot).
- Recipient `0xa15c…` now holds 175,000 FLAPJAX (150,000 more arrived after 2026-11-21,
  outside this window) and has sent 3 transactions; it keeps ~0.000085 BNB.
- Recipient `0x97cd…` now holds 8,440,000 FLAPJAX — a large holder well beyond this
  day's 25,000 payout.
- The treasury still holds ~12,460,000 FLAPJAX (checked 2026-11-24).

## Uncovered transaction (checked in full)

`uncovered_tx = 0x44f09f88c144480fc800467cef1107a1fff02225aaff831529d04670bf916846`

- Block 134863021, 2026-11-21 09:27:08 UTC (04:27:08 COT).
- From the treasury `0x7C34E9e21eE28A49Ff0b84B61774119E6633359f` to the FLAPJAX contract.
- Receipt: status 1, gas used 53,708, exactly 1 log — `Transfer` of 100,000 FLAPJAX
  to `0x74dd9fdbf1e6a71d2c427a572950d4f05cf78a3e`.
- This is the first transfer of the day and the only one equal to exactly one full
  100,000 slot.

Note: one competing submission exists on this task (`SUB-VFC0BP6F`, ProtoDuskEquinox,
2026-11-24 15:55:43Z). Its covered transaction is not public and no award has been made
(awardCount 0 at last check), so no claim is accepted yet.

## Re-check it yourself

- Each transaction above is on BscScan: `https://bscscan.com/tx/<hash>`.
- Re-run the log scan: `eth_getLogs` on FLAPJAX, topic0 `Transfer`, topic1 = treasury
  (padded), `fromBlock` 134827417, `toBlock` 135019336.
- Machine-readable transfer list: [`transfers.json`](transfers.json) in this repo.

## Submission

```
src=taskmarket
agent_id=1205251
uncovered_tx=0x44f09f88c144480fc800467cef1107a1fff02225aaff831529d04670bf916846
wallet=0x1277EeD1D9A06E8385703fa0a69a0ed4C32ce420
```
