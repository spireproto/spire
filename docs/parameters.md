# Parameters

Every number the protocol runs on. These are the launch values. Changes go
through governance and take effect at a window boundary, never inside one.

The same table is code in three places: `src/params.js` in
[spire-core](https://github.com/spireproto/spire-core), `SpireParameters.sol` in
[spire-contracts](https://github.com/spireproto/spire-contracts), and
`lib/params.mjs` in [spire-checks](https://github.com/spireproto/spire-checks).
If any of them disagrees with this page, that is a bug and the test suites are
written to catch it.

## Network

| Parameter | Value |
| --- | --- |
| Chain | Robinhood Chain |
| Chain ID | `4663` |
| Settlement asset | USDC, six decimals |
| Block time | 0.101s, measured |
| Finality assumed at | 120 blocks |

> **Corrected 30 August 2026.** This table previously said two seconds, carried
> over from an early note. Measured over the last million blocks the chain runs
> at 0.101s, twenty times faster, which puts about 2,959 blocks inside a
> settlement window rather than 150. Windows are wall clock, so nothing about
> netting or margin changes; what changes is any confirmation depth sized off the
> old figure. The measurement is in
> [spire-checks/results](https://github.com/spireproto/spire-checks/tree/main/results)
> and reruns on a schedule.

## Settlement window

| Parameter | Value | Note |
| --- | --- | --- |
| `windowLength` | 300s | Obligations accumulate here |
| `finalisationLag` | 30s | Nothing accepted, netting computed |
| `settlementDeadline` | 120s after finalisation | Net must be delivered inside this |
| Windows per day | 288 | Intake is contiguous, so windows overlap |

## Margin

| Tier | Assets | Initial | Maintenance |
| --- | --- | --- | --- |
| 1 | Top 100 by traded value | 8% | 6% |
| 2 | Continuous on-chain price | 12% | 9% |
| 3 | Illiquid or under 30 days | 20% | 15% |

## Collateral haircuts

| Collateral | Haircut | Effective |
| --- | --- | --- |
| USDC | 0% | 100% |
| Tokenised T-bills, under 90 days | 2% | 98% |
| Tokenised equities, tier 1 | 15% | 85% |
| SPIRE | 35% | 65% |

## Position limits

```
limit = haircut value / initial margin rate
```

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

## Fees

| Fee | Value | Taken in |
| --- | --- | --- |
| Clearing fee | 0.35 bps of novated notional | USDC |
| Late settlement | 25 bps of the undelivered net | USDC |
| Auction shortfall | Charged to the waterfall, not to the member | |

Fees are taken in USDC, not in SPIRE. The token is a claim on the clearing
business, not a toll on using it.

## Timings, collected

| Event | Deadline |
| --- | --- |
| Fill accepted before window close | 300s |
| Net published after finalisation | 30s |
| Net delivered | 120s |
| Margin call cured | 600s |
| Default declared | Immediately after cure expires |
| Auction opens | 300s after default |
| Auction runs | 900s |
| Governance timelock | 48h |
