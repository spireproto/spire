# Novation

## The step

A fill arrives. Buyer owes seller an amount of the settlement asset, seller owes
buyer an amount of a tokenised equity. Both sides are exposed to each other for
as long as that stands.

Novation discharges it. The original obligation is extinguished and two new ones
are written in its place, both against Spire Protocol:

```
before        buyer  <------------------>  seller

after         buyer  <----->  Spire  <----->  seller
```

This is not a guarantee sitting on top of a bilateral trade. It is a replacement
of the trade. There is no residual claim between the two participants, and if one
of them defaults the other never finds out, because there is nothing left
connecting them.

## Why it has to be a replacement

A guarantee is a promise to pay if somebody else does not. It leaves the original
claim intact, which means it leaves the original question intact: is the
guarantor good for it, and for how many of these at once?

Replacement removes the question rather than answering it. After novation there
is exactly one counterparty in the market, and its solvency is
[checkable](solvency.md) rather than assessable. That is the difference between a
clearing house and an insurance policy, and it is why the first thing a clearing
layer does is discharge, not promise.

## What happens at the moment of novation

Four things, in this order, before any state is written:

1. **The signature is checked.** A fill is signed EIP-712 by the venue over the
   domain `Spire Protocol / 1 / 4663`. If it does not recover to the venue key on
   file, nothing happens: `SPIRE-1002`.
2. **The clock is checked.** `matchedAt` more than 60 seconds from chain time is
   `SPIRE-1003`. A venue with a drifting clock stops clearing before it notices
   anything else is wrong.
3. **The limit is checked.** The member's collateral buys a limit; if this fill
   would exceed it the fill is refused with `SPIRE-3001` and no obligation
   exists. This is the important one: it is much cheaper to refuse a fill than to
   unwind a novated obligation.
4. **The window is assigned.** The obligation belongs to exactly one settlement
   [window](netting.md), decided here and never moved.

Only then are the two obligations written, margin held, and `Novated` emitted
once per side.

## Independence

The two obligations are independent from the moment they exist. The buyer's does
not fail because the seller's does. Concretely:

- One side defaults. The other settles normally against Spire Protocol and is
  told nothing, because nothing about its position has changed. The loss goes to
  the [waterfall](default-waterfall.md), not to the other participant.
- One side's venue is suspended. The other's obligation is unaffected: the venue
  was the messenger, not the counterparty.
- Both sides are the same member, on two venues. The obligations still exist
  separately and then net against each other, which is the ordinary case for a
  market maker, not a special one.

## Idempotency

`fillId` is the idempotency key. Sending the same fill twice returns the
obligation created the first time rather than creating a second one, and the
signature is deterministic so a retry produces the same bytes.

Reusing a `fillId` with *different* contents is `SPIRE-1004` and it is worth
alerting on rather than retrying: it means two different trades were given one
identity somewhere upstream, and no clearing layer can decide which one was meant.

## What a venue gives up

Visibility into the other side. After novation a venue can see its own members'
obligations and nothing else, because the netting that follows crosses venue
boundaries. A member that buys on one venue and sells the same asset on another
inside one window delivers the difference, and neither venue can see that the
other side exists.

That is a feature for the member and a constraint for the venue, and it is worth
saying out loud before an integration rather than after.

## In the interfaces

```solidity
function novate(Fill calldata fill, bytes calldata signature)
    external
    returns (bytes32 obligationId);
```

`novate` reverts rather than returning an error, and each revert maps onto one
published code: `SPIRE-3001` surfaces as `LimitExceeded(member, limit,
attempted)`. Full surface in
[spire-contracts](https://github.com/spireproto/spire-contracts).
