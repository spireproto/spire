# Collateral instead of prefunding

## The trade being made

Prefunding is collateral at 100% with a haircut of zero and a limit of exactly
what you posted. It is the safest possible arrangement and the most expensive
one, and on a chain it is the default because there was never anything else.

Clearing replaces it with margin against exposure. A member posts value, the
value is haircut by kind, and what remains buys a limit on how much novated
notional it may carry:

```
limit = haircut value / initial margin rate
```

A member holding 1,000,000 USDC against a tier 1 asset may carry 12,500,000 of
novated notional. That is the whole mechanism. Everything else is the detail of
how the numbers are chosen and what happens when they are breached.

## Haircuts

Posted collateral counts at its value after the haircut.

| Collateral | Haircut | Counts as |
| --- | --- | --- |
| USDC | 0% | 100% |
| Tokenised T-bills, under 90 days | 2% | 98% |
| Tokenised equities, tier 1 | 15% | 85% |
| SPIRE | 35% | 65% |

The protocol token is haircut hardest on purpose. A clearing layer that accepts
its own token at face value is underwriting itself, and the day it needs the
collateral is the day the token is worth least.

Collateral nobody prices is not accepted at zero, it is refused (`SPIRE-1204`).
An asset that cannot be valued at the moment of a default is not collateral, it
is a hope.

## Margin

Initial margin is a percentage of novated notional and depends on the asset tier.

| Tier | Assets | Initial | Maintenance |
| --- | --- | --- | --- |
| 1 | Tokenised equities in the top 100 by traded value | 8% | 6% |
| 2 | Everything else with a continuous on-chain price | 12% | 9% |
| 3 | Illiquid or newly listed, under 30 days | 20% | 15% |

A tier is a property of the asset, not of the member. Reclassifications are
published one window ahead and take effect at `tierEffectiveAt`, so nobody is
margin called by a change they could not have seen. Positions opened under the
old rate keep it until the window they were novated into closes.

## The account

| Field | Meaning |
| --- | --- |
| `haircutValue` | Posted value after haircuts, in the settlement asset |
| `initialMargin` | Held against open positions |
| `defaultFund` | Contribution, posted but not available as margin |
| `free` | `haircutValue - initialMargin - defaultFund`, signed |
| `limit` | Maximum novated notional |

`free` going negative is a margin call, not an error and not a default. The
member has 600 seconds to post more or reduce. `curesAt` is an absolute
timestamp, so it survives a window boundary: a member that cures at second 550 of
600 keeps every position it had.

A withdrawal comes out of `free` only. One that would take it below zero is
refused with `SPIRE-3002` rather than quietly opening a call.

## Limits, and where they are checked

At novation, before anything is owed. Three ways to fail:

| Code | Condition |
| --- | --- |
| `SPIRE-3001` | The fill would take novated notional past the limit |
| `SPIRE-3003` | One asset would exceed 25% of the limit |
| `SPIRE-3004` | Posted collateral is below the 250,000 USDC floor |
| `SPIRE-3005` | The member is already in a margin call, so no new notional |

The concentration cap is the one members argue with. It exists because a limit is
computed from a haircut that assumes the position can be auctioned, and a
position that is a quarter of the member's book in a single illiquid asset cannot
be auctioned at anything like its mark.

## What this costs a member

Run the numbers yourself:

```bash
node checks/collateral.mjs USDC=400000 SPIRE=1200000
```

from [spire-checks](https://github.com/spireproto/spire-checks). The figure that
surprises people is what the protocol token is worth as collateral. Better to be
surprised there than on a call.

Every number on this page is in [Parameters](parameters.md).
