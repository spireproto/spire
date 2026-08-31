# Glossary

**Assessment cap** — the most a member can be charged in one default event, set at twice its
default fund contribution. See [Default waterfall](default-waterfall.md).

**Clearing** — everything that happens between a matched trade and final settlement.

**Clearing member** — an address with posted collateral and a position limit. Members are
onboarded once and trade across every venue.

**Collateral** — assets posted to secure obligations, sized against position limits. Counted
after a haircut.

**Concentration cap** — the maximum share of a member's limit that may sit in one asset,
25%.

**Counterparty** — after novation, always Spire Protocol.

**Cure period** — the 600 seconds a member has to resolve a margin call before default is
declared.

**Default fund** — mutualised capital that absorbs losses after a defaulter's own margin.

**Default waterfall** — the fixed order in which capital is consumed after a default.

**Finalisation** — the 30 second phase at the end of a window during which no fills are
accepted and nets are computed.

**Finality** — the point after which a settled obligation cannot be reversed.

**Fill** — a venue's signed statement that two participants agreed on a price. The input to
[novation](novation.md).

**Free collateral** — collateral value after haircuts, less initial margin and the default
fund contribution. Negative `free` is a margin call.

**Gross** — the sum of absolute sizes before netting. The number that would have to move
under prefunding.

**Haircut** — the discount applied to posted collateral before it counts toward a limit.

**Initial margin** — collateral held against a novated position, set by asset tier.

**Margin call** — the state a member enters when `free` goes negative. New notional is
refused until it is cured.

**Net** — what actually moves at the close of a window, after offsetting obligations
collapse.

**Netting** — collapsing offsetting obligations so that only the difference settles.

**Novation** — discharging a bilateral trade and replacing it with two obligations facing
the clearing layer.

**Obligation** — one side of a novated trade. One fill produces two, and they are
independent.

**Position limit** — the maximum exposure a member may hold, derived from its collateral.

**Prefunding** — holding the whole asset before trading it. What clearing exists to remove.

**Settlement window** — the interval inside which obligations accumulate and net, 300
seconds.

**Solvency proof** — evidence that posted collateral covers outstanding obligations,
without revealing individual positions.

**Tier** — an asset's risk classification, 1 to 3, which sets its margin rate.

**Utilisation** — the share of a member's limit currently in use, in basis points.

**Variation margin** — the mark to market applied at every window close.

**Venue** — a place where trades are matched. Sends fills, never holds counterparty risk
after novation.
