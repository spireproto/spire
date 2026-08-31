# Errors and edge cases

Codes are stable. Messages are not, so branch on the code. What to alert on and what to do
about it is in [Operations](operations.md).

```js
try {
  await clearing.obligations.fromFill({ fill });
} catch (e) {
  if (e.code === 'SPIRE-3001') {
    // limit breached, this is a business condition, not an outage
  }
  throw e;
}
```

## 1xxx validation

| Code | Meaning | What to do |
| --- | --- | --- |
| `SPIRE-1001` | Malformed fill | Fix the payload, do not retry as is |
| `SPIRE-1002` | Signature does not recover to the venue key | Check the signing key and the domain |
| `SPIRE-1003` | `matchedAt` more than 60s from chain time | Fix venue clock drift |
| `SPIRE-1004` | `fillId` reused with different contents | Idempotency violation, investigate |
| `SPIRE-1101` | Unknown `obligationId` | |
| `SPIRE-1204` | Asset not clearable | Stop routing this asset, alert |
| `SPIRE-1205` | Member not onboarded | |

`SPIRE-1004` deserves attention. Reusing a `fillId` with the same contents is safe and
returns the original obligation. Reusing it with different contents means two different
trades were given one identity somewhere upstream.

## 2xxx authentication

| Code | Meaning | What to do |
| --- | --- | --- |
| `SPIRE-2001` | Unknown `venueId` | |
| `SPIRE-2002` | Venue suspended | Contact operations, do not retry |
| `SPIRE-2003` | Signature replayed | Already consumed, treat as a duplicate |

## 3xxx limits and collateral

| Code | Meaning | What to do |
| --- | --- | --- |
| `SPIRE-3001` | Position limit breached | Reject the fill upstream or post collateral |
| `SPIRE-3002` | Withdrawal exceeds `free` | Reduce the amount |
| `SPIRE-3003` | Concentration cap breached | Same asset is over 25% of the limit |
| `SPIRE-3004` | Below `minCollateral` | Member cannot clear until topped up |
| `SPIRE-3005` | Member is in a margin call | New notional refused until cured |

A limit breach is checked at novation, before anything is owed. That is deliberate: it is
much cheaper to refuse a fill than to unwind a novated obligation.

## 4xxx windows

| Code | Meaning | What to do |
| --- | --- | --- |
| `SPIRE-4001` | Window is finalising | Retry, the fill lands in the next window |
| `SPIRE-4002` | Explicit `windowId` is closed | Send with `"current"` |
| `SPIRE-4003` | Explicit `windowId` is too far ahead | Only the next window may be named |

`SPIRE-4001` is the one to handle properly. It is not a failure, it is a 30 second door
closing. Retry once and the fill is accepted into the next window.

## 5xxx settlement

| Code | Meaning | What to do |
| --- | --- | --- |
| `SPIRE-5001` | Net not delivered before deadline | Late fee applies, cure window opens |
| `SPIRE-5002` | Delivery does not match the published net | Deliver the exact net |
| `SPIRE-5003` | Settlement asset transfer failed | Check allowance on the settlement contract |

## Edge cases worth designing for

### A fill arrives during finalisation

The 30 second finalisation phase accepts nothing. The SDK retries once and the obligation
lands in the next window. The trade is not lost and the price does not change. What changes
is which net it belongs to.

### The window rolls while a member is in a margin call

The call does not reset. `curesAt` is an absolute timestamp and it survives the window
boundary. A member that cures at second 550 of a 600 second cure period keeps every position
it had.

### One side of a fill defaults

The obligations are independent from the moment of novation. The non-defaulting side settles
normally against Spire Protocol and does not learn that anything happened. The loss is
absorbed by the [default waterfall](default-waterfall.md), not by the other participant.

### A member defaults with a net of zero

It still defaults, because the default is about margin, not about the net. A member whose
net is zero but whose margin has evaporated is removed from the next window and its
collateral is released after the auction closes.

### The same asset is cleared by two venues

Netting is per member per asset, not per venue. A member that buys on one venue and sells on
another inside the same window nets across both. Neither venue can see the other side.

### A tier reclassification during an open position

The new rate applies from `tierEffectiveAt`, which is always the start of a later window.
Positions opened under the old rate keep it until the window they were novated into closes.
Nobody is margin called by a change they could not have seen.

## In the client

[spire-sdk](https://github.com/spireproto/spire-sdk) carries this table as
`src/errors.js` and adds two derived flags: `err.retryable` and `err.business`.
On chain the same codes are custom errors, listed next to each other in
[SpireErrors.sol](https://github.com/spireproto/spire-contracts/blob/main/src/SpireErrors.sol).
