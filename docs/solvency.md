# Solvency, checkable

## The problem with a closed settlement

Clearing works by putting one counterparty in the middle. That is also its
failure mode: everyone's exposure is now to the same balance sheet, and if that
balance sheet is a claim rather than a fact, the market has replaced many small
questions with one large one.

Traditional clearing houses answer this with regulation, audits and disclosure on
a quarterly cadence. On chain there is a better answer available, and refusing to
use it would be strange.

## What is published

One commitment per closed window, over the collateral held and the margin
required against it:

```solidity
function proofOf(uint64 windowId)
    external
    view
    returns (bytes32 commitment, uint256 collateral, uint256 required, uint64 publishedAt);
```

The commitment is a merkle root over per member positions and collateral. A
member can prove its own line against it without the rest of the book being
disclosed, which matters: a proof that reveals every member's position is not a
transparency feature, it is a leak with a nice name.

`coverageBps` is the aggregate, collateral over required margin. Under 10,000
means the layer is short, and it is meant to be read by people who do not trust
us.

## Checking it without asking us

Three view calls, no API, no permission:

1. `SolvencyRegistry.latestProof()` and check the commitment against the window
   you care about.
2. `CollateralVault.haircutValue()` summed over members, against the aggregate
   required margin in `ClearingHouse`. These are independent contracts; if they
   disagree, one of them is wrong and that is the finding.
3. `Governor.timelockDelay()` is still 48 hours and no proposal is queued that you
   have not seen.

All three are reads. None of them requires trusting an endpoint, including ours.
When there is a deployment,
[spire-checks](https://github.com/spireproto/spire-checks) runs exactly these and
prints the answer.

## What the proof does not say

It says the layer holds enough collateral against the margin it requires. It does
not say the margin rates are high enough for the next gap, that the haircuts are
conservative enough for the next illiquid morning, or that an auction will clear
at the price the model assumed.

Those are judgements, and a merkle root cannot launder a judgement into a fact.
What the proof removes is the separate and much stupider risk of the numbers
being misreported, which is the one failure mode that should not exist on a chain.

## Why this is not optional

A clearing layer asks the market to concentrate its counterparty risk in one
place. The only honest way to ask for that is to make the concentration
inspectable at the same frequency the market trades at, not quarterly, and not
through us.

Everything else in this repository, the interfaces published before deployment,
the checks that run against our own claims, the honest [status](status.md) page,
is the same idea applied to the things a merkle root cannot cover.
