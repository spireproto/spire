# Overview

## What a clearing layer is for

Two participants agree on a price. That is a trade. Between the moment they agree
and the moment the asset and the money change hands, each of them is holding a
claim on the other, and each of them has to have formed a view on whether the
other will be there when the time comes.

Every mature market eventually decides that this view is not worth forming ten
thousand times a day, and puts one counterparty in the middle. The bilateral
trade is discharged and replaced by two obligations against that counterparty.
Nobody has to price anybody else's solvency, and the market stops being a graph
of mutual exposures and starts being a star.

That is clearing. It is not a settlement optimisation, it is a change in who
faces whom.

## What tokenised equities did instead

On chain, the answer to counterparty risk was to remove the interval. Nothing is
owed because everything is prefunded: the asset is in the account before the
trade and the cash is in the account before the trade, and the transfer is the
trade.

This works, and it is the reason on-chain venues have never had a failed
delivery. It also has a price, and the price is paid in capital that has to sit
still. Every unit of turnover requires a unit of assets already in place, at that
moment, in the right account.

A market maker quoting both sides of five assets on three venues does not need
capital equal to its net risk. It needs capital equal to the gross of everything
it might trade, on every venue where it might trade it, at all times.

## What Spire Protocol does

Four mechanisms, none of them new. What is new is that they are being written for
a chain that does not have them.

| | |
| --- | --- |
| [Novation](novation.md) | The bilateral trade is discharged. Two obligations against the clearing layer take its place, and they are independent of each other from the moment they exist. |
| [Netting in time](netting.md) | Obligations accumulate in a 300 second window and collapse against each other before anything reaches the chain. Per member, per asset, across venues. |
| [Collateral](collateral.md) | Exposure is covered by margin against the net, not by prefunding the gross. A tier 1 position costs 8% of its notional to carry. |
| [Default waterfall](default-waterfall.md) | Loss is absorbed in an order fixed before anyone defaults, and the most a stranger's failure can cost a member is bounded before it joins. |

Together they answer one question: how much capital does the market have to hold
to support a given turnover? Prefunding answers "all of it". Clearing answers
"the margin on what is left after everything cancels out".

## The lifecycle

```
fill  ->  novated  ->  netted  ->  settled
                 \
                  ->  defaulted  ->  waterfall
```

A fill is a signed statement from a venue that two participants agreed. It is
novated on arrival, which is where limits are checked and margin is held: a
breach is refused before anything is owed, because refusing a fill is cheap and
unwinding a novated obligation is not.

At the close of its window the obligation is netted against everything else that
member did in that asset. What is left is delivered inside 120 seconds. Nothing
carries across a window boundary, and no state is ever revisited.

## What this is not

It is not a venue. Spire Protocol does not match, quote, or hold an order book,
and it never sees why a fill happened. Venues compete on liquidity and product;
the clearing layer underneath them is not a place anyone wants competition,
because the whole benefit comes from everyone facing the same counterparty.

It is not custody in the sense of an account you cannot see into. Collateral sits
in a vault contract whose balances are public, and the
[solvency proof](solvency.md) exists so the layer can be checked rather than
believed.

It is not deployed. See [Status](status.md), which is deliberately blunt about
what exists and what does not.

## Where to read next

- [Novation](novation.md) if you want the mechanism.
- [Netting](netting.md) if you want the arithmetic that makes it worth doing.
- [Parameters](parameters.md) if you want every number in one table.
- [Integration](integration.md) if you are a venue and want to know what you send.
- [Status](status.md) if you want to know what is real today.
