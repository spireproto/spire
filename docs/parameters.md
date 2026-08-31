# Parameters

Every number the protocol runs on, in one place. These are the launch values. Changes go
through governance and take effect at a window boundary, never inside one. The governance
path and its 48 hour timelock are in [Contracts](contracts.md), and every change is logged
in the [Changelog](changelog.md).

## Network

| Parameter | Value |
| --- | --- |
| Chain | Robinhood Chain |
| Chain ID | `4663` |
| Settlement asset | USDC |
| Block time | 0.101s, measured |
| Finality assumed at | 120 blocks |

> **Corrected 30 August 2026.** This table said two seconds until a check
> measured it. Over the last million blocks the chain runs at 0.101s, twenty
> times faster, which puts about 2,959 blocks inside a settlement window rather
> than 150. Windows are wall clock, so netting, margin and the settlement
> deadline are unchanged; what changes is any confirmation depth sized off the
> old figure. The measurement is in
> [spire-checks/results](https://github.com/spireproto/spire-checks/tree/main/results)
> and reruns on a schedule.

Venues on other chains connect through bridged assets. The clearing layer itself is native
and does not run a second deployment elsewhere.

## Settlement window

| Parameter | Value | Note |
| --- | --- | --- |
| `windowLength` | 300s | Obligations accumulate here |
| `finalisationLag` | 30s | No new obligations accepted, netting is computed |
| `settlementDeadline` | 120s after finalisation | Net must be delivered inside this |
| Windows per day | 288 | |

An obligation belongs to exactly one window, decided at novation and never moved. A fill
that arrives during finalisation is assigned to the next window, not rejected.

## Margin

Initial margin is a percentage of novated notional and depends on the asset tier. Variation
margin is marked at every window close.

| Tier | Assets | Initial margin | Maintenance |
| --- | --- | --- | --- |
| 1 | Tokenised equities in the top 100 by traded value | 8% | 6% |
| 2 | Everything else with a continuous on-chain price | 12% | 9% |
| 3 | Illiquid or newly listed, under 30 days | 20% | 15% |

A tier is a property of the asset, not of the member. Tier changes are published one window
ahead so that nobody is margin called by a reclassification they could not see coming.

## Collateral haircuts

Posted collateral counts at its value after the haircut.

| Collateral | Haircut | Effective value |
| --- | --- | --- |
| USDC | 0% | 100% |
| Tokenised T-bills, under 90 days | 2% | 98% |
| Tokenised equities, tier 1 | 15% | 85% |
| SPIRE | 35% | 65% |

SPIRE is deliberately haircut hardest. A clearing layer that accepts its own token at face
value is underwriting itself.

## Position limits

```
limit = (collateralValue - haircut) / initialMarginRate
```

A member holding 1,000,000 USDC against a tier 1 asset may carry 12,500,000 of novated
notional. The limit is checked at novation, not at settlement, so a fill that would breach
it is rejected while it is still cheap to reject.

| Parameter | Value |
| --- | --- |
| `minCollateral` | 250,000 USDC |
| `concentrationCap` | 25% of a member's limit in one asset |
| `marginCallCure` | 600s |

## Default fund

| Parameter | Value |
| --- | --- |
| Member contribution | 15% of required initial margin |
| Contribution floor | 50,000 USDC |
| Protocol insurance layer | 2,500,000 USDC at launch |
| Assessment cap | 2x a member's contribution per event |
| Replenishment period | 5 days |

The assessment cap is the number that matters to a member deciding whether to join. It is
the most a default by somebody else can cost, and it is bounded before the member signs
anything.

## Fees

| Fee | Value | Taken in |
| --- | --- | --- |
| Clearing fee | 0.35 bps of novated notional | Settlement asset |
| Late settlement | 25 bps of the undelivered net | Settlement asset |
| Auction shortfall | Charged to the waterfall, not to the member | |

Fees are taken in USDC, not in SPIRE. The token is a claim on the clearing business, not a
toll on using it. See [Token](token.md).

## Timings, collected

| Event | Deadline |
| --- | --- |
| Fill accepted before window close | `windowLength` |
| Net published after finalisation | 30s |
| Net delivered | 120s |
| Margin call cured | 600s |
| Default declared | Immediately after cure expires |
| Auction opens | 300s after default |
| Auction runs | 900s |

## The same table, as code

These numbers are `src/params.js` in
[spire-core](https://github.com/spireproto/spire-core), `SpireParameters.sol` in
[spire-contracts](https://github.com/spireproto/spire-contracts) and
`lib/params.mjs` in [spire-checks](https://github.com/spireproto/spire-checks).
If any of them disagrees with this page, that is a bug, and the test suites are
written to catch it.
