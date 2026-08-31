# Clearing membership

A member is an address with posted collateral and a position limit. Members are onboarded
once and clear across every venue, because after [novation](novation.md) an obligation from
one venue is the same instrument as an obligation from another.

## Who needs to be a member

Not everybody who trades. It depends on the decision described in
[Integration](integration.md).

| Model | Who is the member | Who is margin called |
| --- | --- | --- |
| Users post directly | Each end user | The user |
| Venue posts for its users | The venue | The venue |
| Hybrid | Venue for retail flow, professionals direct | Both, separately |

A venue that posts on behalf of its users gets the best netting in the system and takes back
its users' credit risk in exchange. Most start hybrid: users direct, and the venue's own
market making desk as a separate member.

## Requirements

| Requirement | Value | Note |
| --- | --- | --- |
| Minimum collateral | 250,000 USDC | After haircuts |
| Default fund contribution | 15% of required margin | Floor 50,000 USDC |
| Address | One per member | Sub-accounts are derived, not separate members |
| Clock | NTP synced | Fills more than 60s off chain time are rejected |

The default fund contribution is not a fee. It is capital at risk that is returned when the
member exits, less anything consumed by a default. See [Default waterfall](default-waterfall.md).

## Onboarding

1. Submit the member address and the assets you intend to clear.
2. Receive a `venueId` and a signing key if you are also a venue. A member that only trades
   does not need one.
3. Post collateral through `collateral.post` or `POST /v1/collateral/deposits`.
4. Read `collateral.get(member)` and confirm `limit` is what you expect.
5. Trade.

Steps 3 to 5 are self-service. Step 1 is not, because somebody has to decide which assets a
new member may clear.

## Sub-accounts

A member may derive sub-accounts to separate strategies or desks.

| Property | Behaviour |
| --- | --- |
| Collateral | Held at the member level, not per sub-account |
| Position limit | Shared across sub-accounts |
| Netting | Across the member, so sub-accounts net against each other |
| Reporting | Per sub-account through the `member` filter |

Two desks that trade opposite sides of the same asset in the same window will net to a
smaller number than either of them expects. That is the intended behaviour and it is worth
telling both desks before they see it in a report.

## What membership costs

| | Amount | Recoverable |
| --- | --- | --- |
| Collateral | From 250,000 USDC | Yes, on withdrawal from `free` |
| Default fund contribution | 15% of required margin | Yes, on exit, less losses absorbed |
| Clearing fee | 0.35 bps of novated notional | No, it is a fee |
| Worst case from somebody else's default | 2x your contribution | No |

That last row is the one to model before joining. It is bounded, it is knowable in advance,
and it is the entire economic argument for the assessment cap existing at all.

## Exiting

1. Stop sending fills and let open obligations settle. There is no way to cancel a novated
   obligation, only to let its window close.
2. Withdraw collateral down to the level that supports remaining positions.
3. Request removal. The default fund contribution is released after the replenishment
   period, five days, so that a default occurring on your last day is still covered by the
   capital that was standing behind it.

Nothing about exiting is instant, and that is deliberate. A member that could withdraw its
fund contribution the moment it smelled trouble would make the mutualised layer worthless.
