# Collateral instead of prefunding

Atomic settlement asks both sides to hold the entire asset at the instant of the trade.
That is a guarantee bought with idle money.

Spire Protocol grants the right to move volume against collateral instead. Capital works
rather than guarding a position.

## Position limits

A participant's limit is a function of the collateral it has staked.

```
limit = (collateralValue - haircut) / initialMarginRate
```

More collateral, more volume. The limit is checked at the moment an obligation is accepted,
so an obligation that would breach it is never novated in the first place. The failure is
`SPIRE-3001` and it arrives before anything is owed, which is the cheapest moment for it to
arrive.

### Worked example

A member posts 1,000,000 USDC and trades a tier 1 asset at an 8% initial margin rate.

| | Value |
| --- | --- |
| Posted | 1,000,000 USDC |
| Haircut on USDC | 0% |
| Collateral value | 1,000,000 |
| Initial margin rate | 8% |
| **Position limit** | **12,500,000** |

The same member posting 1,000,000 of SPIRE instead, at a 35% haircut, gets a limit of
8,125,000. The token is accepted, and it is accepted at a discount.

## What counts as collateral

| Collateral | Haircut | Effective value |
| --- | --- | --- |
| USDC | 0% | 100% |
| Tokenised T-bills, under 90 days | 2% | 98% |
| Tokenised equities, tier 1 | 15% | 85% |
| SPIRE | 35% | 65% |

USDC is the settlement asset and carries no haircut, so a member that wants the simplest
possible risk profile posts only USDC and never marks its collateral against a second
market during a stress event.

The other three are accepted because refusing them would push members back into holding
idle cash, which is the problem this protocol exists to remove. They are accepted at
haircuts sized so that a fall in their price does not immediately become a shortfall.

SPIRE is haircut hardest on purpose. A clearing layer that accepts its own token at face
value is underwriting itself, and a stress event would hit the collateral and the token at
the same moment.

## Margin call

`free` is what is left after margin and the default fund contribution:

```
free = haircutValue - initialMargin - defaultFund
```

`free` going negative is a margin call, not an error. The member has 600 seconds to post
more or reduce, and `curesAt` is an absolute timestamp that survives a window boundary.

During a call the member may reduce but may not add new notional. New fills are refused with
`SPIRE-3005`. If the cure period expires the member is declared in default and the
[waterfall](default-waterfall.md) runs.

The runbook for the ten minutes in between is in [Operations](operations.md).

## Concentration

No more than 25% of a member's limit may sit in one asset. A member with a 12,500,000 limit
cannot carry more than 3,125,000 in any single name.

The cap exists because the waterfall is sized against a default that can be auctioned. A
position large enough to move the price of the asset it is being liquidated into cannot be
auctioned at anything close to its mark.

## Properties

- Collateral is not lent out. It exists to absorb a default, not to earn a carry.
- Collateral is held per member, not per venue, and it backs that member's positions
  everywhere it trades.
- A participant that defaults is slashed, in the order described in the
  [default waterfall](default-waterfall.md).
- Withdrawals come out of `free` only. A withdrawal that would create a margin call is
  refused with `SPIRE-3002` rather than executed.

## The trade-off, stated plainly

Prefunding removes credit risk by freezing money. Collateral converts credit risk into a
sizing problem: how much can this participant move before its collateral stops covering it.
That is a question with a numeric answer, which is what makes clearing possible at all.

Every number in that answer is in [Parameters](parameters.md).

## Price your own basket

```bash
git clone https://github.com/spireproto/spire-checks
node checks/collateral.mjs USDC=400000 SPIRE=1200000
```

The number that surprises people is what the protocol token is worth as
collateral. Better to be surprised there than on a margin call.
