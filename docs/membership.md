# Membership

## Who is the member

The member is whoever posts collateral and whoever gets the margin call. Those
two are the same address by construction, and deciding which one it is shapes a
venue's product more than any technical choice in the integration.

| Model | Collateral | Margin call reaches | Fits |
| --- | --- | --- | --- |
| Direct | The end participant | The end participant | Desks, market makers, funds |
| Venue as member | The venue | The venue | Retail flow |
| Hybrid | Both, per counterparty | Whoever posted | Most venues in practice |

A retail user woken at three in the morning by a margin call they did not know
they could receive is a product failure, not a clearing one. Venues that carry
retail flow should be the member for it and manage the exposure internally, which
is exactly what a broker does off chain.

## Requirements

| | |
| --- | --- |
| Minimum collateral | 250,000 USDC after haircuts |
| Default fund contribution | 15% of required margin, floor 50,000 USDC |
| Settlement address | Able to deliver a net inside 120 seconds |
| Operational contact | Reachable inside the 600 second cure period |

The last one is not paperwork. A member whose only contact is an email address
monitored on weekdays has no way to use the cure period, and the cure period is
the difference between a margin call and a default.

## Onboarding

1. **Register the address.** The member address, and the venue key if the member
   is a venue.
2. **Post collateral.** It counts after haircuts, so post the mix you intend to
   keep rather than the one that looks largest.
3. **Fund the default fund contribution.** Posted, not usable as margin.
4. **Agree the assessment cap.** 2x the contribution per event. This is the only
   number in the arrangement that is about somebody else's behaviour.
5. **Test on testnet through a full window**, including a margin call and a
   delivery, before the first mainnet fill.

## Sub-accounts

A member may run sub-accounts, and they share the parent's collateral and limit.
Netting runs through them: a member long in one sub-account and short in another
in the same asset and window nets to the difference, which is the point of having
them.

What sub-accounts do not do is isolate risk. A default is a member-level event.
If separation matters, that is two members, not one member with two accounts, and
the collateral floor applies twice.

## What membership costs

| Item | Amount | Returned |
| --- | --- | --- |
| Collateral | From 250,000 USDC | Yes, on exit |
| Default fund contribution | 15% of margin, floor 50,000 | Yes, five days after exit |
| Clearing fee | 0.35 bps of novated notional | No |
| Assessment, if somebody defaults badly | Up to 2x the contribution | No |

The five day delay on returning a contribution exists because a default the
member was part of can surface after it leaves. A member that could exit on the
morning of a stress event would be a member whose contribution meant nothing.

## Exit

Stop novating, let open windows settle, withdraw collateral once `free` covers
it, and the fund contribution returns five days later. There is no notice period
and no penalty: a clearing layer that makes leaving difficult is compensating for
something else.
