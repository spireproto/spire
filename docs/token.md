# Token

Three roles, and they should not be collapsed into one word like "utility".

## 1. Collateral

Clearing members stake for the right to position limits. More stake, more volume they may
move. A member that defaults is slashed. This is the role that ties the token to the
mechanics rather than to sentiment.

SPIRE is accepted as collateral at a 35% haircut, so a million of staked SPIRE carries the
same limit as 650,000 USDC. The discount is deliberate. A clearing layer that accepts its
own token at face value is underwriting itself, and a stress event would hit the collateral
and the token in the same moment.

| Collateral | Haircut | Limit per 1,000,000 posted, tier 1 |
| --- | --- | --- |
| USDC | 0% | 12,500,000 |
| SPIRE | 35% | 8,125,000 |

## 2. Accepted risk

Part of the stake sits in the default fund, layer four of the
[waterfall](default-waterfall.md). Holders earn fees precisely because they carry tail
risk. The yield is explainable: it is payment for standing behind other members' failures.
It is not emitted out of thin air.

The exposure is bounded before anyone commits to it. A member's contribution is 15% of its
required initial margin, with a floor of 50,000 USDC, and the most it can be assessed in a
single default event is twice that contribution.

That cap is what makes the role underwritable. An uncapped mutualised layer is not a risk
position, it is an open-ended guarantee.

## 3. A share of the flow

Fees are taken in the settlement asset, not in the token. The clearing fee is 0.35 basis
points of novated notional and it is paid in USDC.

The token is not a toll paid to enter. It is a claim on the clearing business.

## The thesis

Price follows clearing turnover, not the number of holders.

Every mechanism above is denominated in notional cleared. Limits scale with it, fund
contributions scale with it, fees scale with it. None of them scale with how many addresses
hold the token.

## Not claimed here

No supply schedule, no price target, no yield figure. Those belong in a published
tokenomics document, and until that exists, quoting numbers would be inventing them.

The parameters on this page are protocol mechanics, published in
[Parameters](parameters.md). They are not a return.
