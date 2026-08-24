# Glossary

Terms as this protocol uses them. Where market usage differs, the difference is
noted, because most arguments about clearing are arguments about words.

**Assessment.** A call on the other members' capital after a default has burned
through the layers above it. Capped at twice each member's contribution per
event, which is the number to read before joining rather than after.

**Clearing member.** An address with collateral posted and a limit against it.
Not necessarily an end user: a venue can be the member for its own flow, and for
retail it usually should be. See [Membership](membership.md).

**Compression.** The share of gross flow that never reaches the chain because it
netted against something else. A property of flow, not of the protocol: 8% on a
thin day and 48% on a busy one, with the same rules.

**Cure period.** 600 seconds between a margin call and a default. `curesAt` is an
absolute timestamp and survives a window boundary.

**Default fund.** Capital posted by every member against somebody else's failure.
15% of required margin, floored at 50,000 USDC. Posted, but not available as
margin.

**Fill.** A signed statement from a venue that two participants agreed on a
price. The only thing a venue sends.

**Finalisation.** The 30 seconds after a window closes, when nothing is accepted
and the nets are computed. Permissionless to trigger.

**Free collateral.** Haircut value minus initial margin minus the default fund
contribution. Signed. Negative is a margin call, not an error.

**Gross.** The sum of absolute sizes before netting. What prefunding requires you
to hold.

**Haircut.** The discount applied to posted collateral before it counts. Zero for
USDC, 35% for the protocol's own token.

**Initial margin.** Held at novation against the notional of a position, at the
rate of the asset's tier: 8%, 12% or 20%.

**Maintenance margin.** The level below which an account is in a call. Lower than
initial, so ordinary marking does not trigger one.

**Net.** What actually moves at the close of a window, per member per asset.
Signed: negative is a delivery, positive a receipt, zero means nothing goes on
chain at all.

**Novation.** Discharging a bilateral trade and writing two obligations against
the clearing layer in its place. Not a guarantee: a replacement.

**Obligation.** One side of a novated fill. Independent of its pair from the
moment it exists.

**Position.** The netted view of one member in one asset inside one window.
Derived, not stored as an independent truth.

**Prefunding.** Holding the asset and the cash before the trade so that nothing
is ever owed. The current on-chain default, and the thing this protocol replaces.

**Settlement window.** 300 seconds of intake, the unit of netting. 288 in a day,
and they overlap: window N settles while N+1 accepts.

**SPIRE.** The protocol token. Acceptable as collateral at 65% of its mark, and
not accepted for fees, which are taken in USDC.

**Tier.** The margin class of an asset, 1 to 3. A property of the asset, never of
the member holding it.

**Variation margin.** The mark to market at a window close, charged against the
net.

**Waterfall.** The fixed order in which a default is absorbed. Fixed in advance,
because a discretionary waterfall is not a waterfall.

**Window id.** `floor(unix seconds / 300)`. Derived from the clock, so anyone can
compute which window a moment belongs to without asking.
