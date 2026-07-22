# Netting in time

## The window

Obligations do not settle one at a time. They accumulate inside a settlement
window and collapse against each other before anything reaches the chain.

```
window N      [ intake 300s ][ finalise 30s ][ settle 120s ]
window N+1                   [ intake 300s ][ finalise 30s ][ settle 120s ]
```

Intake is contiguous: window N is finalising and settling while N+1 is already
accepting fills. That is why a day holds 288 windows and not 192, and why nothing
ever waits for the previous window to finish.

An obligation is assigned to a window at novation and never moves to another one.
A fill sent to a window that has just closed lands in the open one instead
(`SPIRE-4001`): the trade is not lost and the price does not change, only which
net it belongs to.

## The arithmetic

One member, one asset, one window, three fills:

| # | Direction | Size |
|---|---|---|
| 1 | buy | 400 |
| 2 | sell | 250 |
| 3 | buy | 350 |

Gross flow is 1,000. What has to be delivered is `400 - 250 + 350 = 500`, and
once the same collapse runs on the funding side of the book the amount that
actually moves at the close comes to 250.

Same turnover. A quarter of the capital moved.

## Across venues, not per venue

The interesting case is the same member trading the same asset in two places
inside one window.

| Venue | Direction | Size |
| --- | --- | --- |
| A | buy | 600 |
| B | sell | 450 |

Bilaterally that is two open positions on two venues, each with its own margin
and its own settlement. After [novation](novation.md) both obligations face the
same counterparty in the same asset, so they are the same instrument and they
net. The member delivers 150, and neither venue learns that the other exists.

## What margin is charged against

The net, not the gross. This is where netting in time stops being an efficiency
and becomes the product.

| | Gross | Net |
| --- | --- | --- |
| Notional in the window | 1,000 | 250 |
| Initial margin, tier 1 at 8% | 80 | 20 |

Twenty of margin against a thousand of turnover. Under prefunding the same
turnover needs a thousand of assets sitting still, on the right venue, at the
right moment.

## The rules, in full

- Netting is per member, per asset, per settlement window. Not per venue.
- Obligations from different venues net against each other, because after
  novation they are the same instrument.
- Bought and sold inside one window nets to zero. Nothing goes on chain at all.
- Nothing carries across a window boundary. Every window is closed and settled on
  its own.
- An obligation is assigned to a window at novation and never moves.

## What compression actually depends on

Flow, not protocol. The same rules give very different answers on a busy day and
a thin one, because netting needs more than one leg per member per asset per
window to have anything to cancel.

Run over 33 modelled days in
[netting-replay](https://github.com/spireproto/netting-replay), compression
ranges from 8.3% on the thinnest day to 47.8% on the busiest. Any figure quoted
without the flow behind it is a figure to distrust, including ours: run
[spire-checks](https://github.com/spireproto/spire-checks) over your own fills
and the number is yours.

## Not a saving on gas

Every unit of gross flow that reaches the chain is a unit of capital that has to
exist, at that moment, in the right place. Netting reduces the capital the market
must hold to support a given turnover. Gas is a rounding error next to that, and
a clearing layer sold on transaction fees is a clearing layer that has
misunderstood its own product.

## Finalisation

At the close of intake there are 30 seconds where nothing is accepted and the
nets are computed, then they are published and delivery is due within 120
seconds. `finalise` is permissionless: anyone can call it once the window has
closed, because a protocol whose settlement depends on an operator running a job
has an operator-shaped hole in it.

Numbers in [Parameters](parameters.md), failure codes in
[Errors](https://spireproto.xyz/docs/errors).
