# Settlement is closed. Solvency is not.

No serious participant settles where its book is visible to a competitor. Positions and
flows are not published.

What leaves the system is a proof that collateral covers obligations, without revealing
whose collateral it is or how large any position is.

## The two properties, separately

- **Confidentiality.** Individual positions, sizes and counterparties stay closed.
- **Verifiability.** The aggregate claim, that posted collateral covers outstanding
  obligations, is checkable by anyone at any time.

These are not in tension. The proof is about the aggregate, not about the members.

## What the proof actually asserts

One inequality, published every window, and readable onchain from `SolvencyRegistry`
(see [Contracts](contracts.md)):

```
sum(haircutCollateral) >= sum(requiredMargin) + sum(defaultFund)
```

Both sides are sums over every member. The proof commits to each member's contribution
without opening it, then shows that the totals satisfy the inequality. An observer learns
that the system is covered. They do not learn by how much any single member is covered, or
how many members there are on either side of an asset.

## What is published and what is not

| Published every window | Never published |
| --- | --- |
| The solvency proof | Individual positions |
| Aggregate collateral value | Which member holds what |
| Number of settled windows | Counterparty pairs |
| Asset tiers and margin rates | A member's utilisation |
| Default events, after the fact | The order book of any venue |

A member's own `utilisation` is readable by that member through
[`positions.get`](api-reference.md), and by nobody else.

## Against the traditional baseline

A traditional clearing house publishes solvency on a quarterly cadence, audited after the
fact, and participants take it on trust between reports. Continuous verifiable proof is a
strictly stronger guarantee than a quarterly attestation, and it is the thing onchain
infrastructure can offer that the incumbent structure cannot.

The difference is not that the numbers are better. It is that the gap between claims shrinks
from three months to five minutes, and the claim stops requiring trust in the party making
it.

## What a failed proof means

If the inequality does not hold at a window close, the window does not settle. New novation
stops, open obligations stay open, and the shortfall is resolved through margin calls before
anything moves.

A protocol that settles anyway and reconciles later is a protocol that has decided its own
solvency claim is advisory.

## What is deliberately not claimed

- Not a claim that a proof removes the need for collateral. It does not. It shows the
  collateral is there.
- Not a claim of privacy against the protocol itself. The clearing layer necessarily knows
  the obligations it novates.
- Not a claim that confidentiality survives a default. A defaulting member's position
  becomes visible when it reaches auction, because it has to be biddable.
