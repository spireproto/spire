# Data model

Five objects carry the whole lifecycle. Everything else in the API is a view over them.

## Fill

What a venue sends. A fill is a statement that two participants agreed on a price, signed by
the venue so that the claim can be attributed and replays can be rejected.

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `fillId` | `string` | yes | Unique inside the venue. Used for idempotency |
| `venueId` | `string` | yes | Issued at onboarding |
| `asset` | `address` | yes | Contract address of the tokenised asset |
| `buyer` | `address` | yes | Member or member sub-account |
| `seller` | `address` | yes | Member or member sub-account |
| `size` | `uint256` | yes | Base units of the asset |
| `price` | `uint256` | yes | Settlement asset per whole unit, 6 decimals |
| `matchedAt` | `uint64` | yes | Unix seconds, venue clock |
| `signature` | `bytes` | yes | EIP-712 over the fields above |

`fillId` is the idempotency key. Sending the same `fillId` twice returns the obligation
created the first time, it does not create a second one.

## Obligation

What the protocol returns. The bilateral trade is gone and two obligations against Spire
Protocol exist in its place. See [Novation](novation.md).

| Field | Type | Description |
| --- | --- | --- |
| `obligationId` | `bytes32` | Assigned at novation |
| `fillId` | `string` | The fill it came from |
| `member` | `address` | Who owes or is owed |
| `asset` | `address` | Tokenised asset |
| `direction` | `"deliver" \| "receive"` | Which side of the obligation |
| `size` | `uint256` | Base units |
| `cash` | `int256` | Settlement asset leg, signed |
| `windowId` | `uint64` | Window it settles in, fixed at novation |
| `state` | `State` | See below |
| `novatedAt` | `uint64` | Unix seconds, chain clock |

One fill produces two obligations, one per side. They are independent from the moment they
exist: the buyer's obligation does not fail because the seller's does.

## State

```
matched -> novated -> netted -> settled
```

`defaulted` is reachable from `novated` and `netted` and routes into the
[default waterfall](default-waterfall.md). No other transition exists, and no state is ever
revisited.

| State | Meaning |
| --- | --- |
| `matched` | Fill received, signature not yet verified |
| `novated` | Spire Protocol is the counterparty, margin is held |
| `netted` | Window finalised, this obligation is inside a net |
| `settled` | Delivered on chain, terminal |
| `defaulted` | Cure period expired, terminal for the member |

## SettlementWindow

| Field | Type | Description |
| --- | --- | --- |
| `windowId` | `uint64` | Monotonic, one per `windowLength` |
| `opensAt` | `uint64` | Unix seconds |
| `closesAt` | `uint64` | `opensAt + windowLength` |
| `finalisedAt` | `uint64` | Zero until finalisation completes |
| `state` | `"open" \| "finalising" \| "settling" \| "closed"` | |
| `obligationCount` | `uint32` | Obligations assigned to this window |

A window is the unit of netting. Nothing settles inside one, and nothing carries across one.
See [Netting](netting.md).

## Position

The net view of a member in one asset inside one window. This is derived, not stored as an
independent truth.

| Field | Type | Description |
| --- | --- | --- |
| `member` | `address` | |
| `asset` | `address` | |
| `windowId` | `uint64` | |
| `gross` | `uint256` | Sum of absolute sizes before netting |
| `net` | `int256` | What actually moves, signed |
| `requiredMargin` | `uint256` | Against the net, not the gross |
| `utilisation` | `uint16` | Basis points of the member's limit in use |

Margin is charged against the net. A member that buys and sells the same asset inside one
window carries margin on the difference, which is the whole point of netting in time.

## CollateralAccount

| Field | Type | Description |
| --- | --- | --- |
| `member` | `address` | |
| `posted` | `Balance[]` | Raw balances by collateral asset |
| `haircutValue` | `uint256` | Value after haircuts, in settlement asset |
| `initialMargin` | `uint256` | Currently held against open positions |
| `defaultFund` | `uint256` | Contribution, not available as margin |
| `free` | `uint256` | `haircutValue - initialMargin - defaultFund` |
| `limit` | `uint256` | Maximum novated notional |

`free` going negative is a margin call, not an error. The member has `marginCallCure`
seconds to post more or reduce. Values are in [Parameters](parameters.md).

## In code

These five objects are the ones
[spire-core](https://github.com/spireproto/spire-core) operates on, with the same
field names. Amounts are BigInt throughout, because a notional does not fit in a
double and losing the last three digits is not an option anyone would pick.
