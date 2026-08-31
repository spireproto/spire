# Integration

A venue integrates at one point in its flow: the moment a trade is matched. Everything
before that point stays yours, and everything after it becomes an obligation against Spire
Protocol.

For a runnable path, start with [Quickstart](quickstart.md). This page is what to decide
before going live. The transport is described in [HTTP API](http-api.md).

## What you send

A signed fill: the two participants, the asset, size, price and the venue's own identifier.
The signature is what lets us attribute the fill and reject replays.

## What you get back

A signed obligation: the novated claim, the settlement window it belongs to, and the
current net for that participant in that asset.

## Sketch

```js
import { Clearing } from '@spireproto/sdk';

const clearing = new Clearing({
  venue: process.env['SPIRE_VENUE_ID'],
  signer: process.env['SPIRE_SIGNING_KEY'],
  chainId: 4663,
});

async function main() {
  const obligation = await clearing.obligations.fromFill({
    fill: matchedFill,
    window: 'current',
  });

  console.log(obligation.net);
}

main();
```

The package name above is the intended one. It is not published yet, and this repository
does not ship an implementation.

## Lifecycle states

`matched` -> `novated` -> `netted` -> `settled`

`defaulted` is reachable from `novated` and `netted`, and routes into the
[default waterfall](default-waterfall.md).

## What a venue has to decide

1. Which assets to clear. Clearing is per asset, not per venue.
2. Whether to clear every fill or only fills above a size threshold.
3. Who posts collateral: the venue on behalf of its users, or the users directly.

That third decision is the one with consequences. It is covered below.

## Who posts the collateral

| | Venue posts | Users post |
| --- | --- | --- |
| Members from the protocol's view | One, the venue | Many |
| Netting happens across | The venue's whole book | Each user separately |
| Capital efficiency | Higher, one net per asset | Lower, one net per user per asset |
| Who is margin called | The venue | The user |
| Who carries user credit risk | The venue | Nobody, it is novated away |

A venue that posts on behalf of its users gets the best netting in the system, because every
user's flow collapses into one position. It also takes back exactly the counterparty risk it
came here to shed, this time against its own users.

Most venues should start with users posting directly, and move to venue-posted collateral
only for a known set of professional counterparties.

## Failure handling worth building

Three cases are worth explicit code rather than a generic retry.

| Case | Code | Handling |
| --- | --- | --- |
| Window finalising | `SPIRE-4001` | Retry once, lands in the next window |
| Limit breached | `SPIRE-3001` | Refuse the fill at your matcher, not after |
| Asset not clearable | `SPIRE-1204` | Stop routing the asset, alert an operator |

`SPIRE-3001` is the one that shapes your architecture. If you check limits only when the
obligation is rejected, you have already told a user their trade was matched. Read
`positions.get` before you match, and treat the limit as a matching constraint rather than
as a settlement error.

Full list in [Errors](errors.md).

## Idempotency

`fillId` is the idempotency key. Retrying with the same `fillId` and the same contents
returns the original obligation. Retrying with the same `fillId` and different contents is
`SPIRE-1004` and means two different trades share an identity somewhere upstream.

Make `fillId` unique per matched trade, not per request. A retry is not a new trade.

## Clocks

`matchedAt` is your clock, and it is compared against chain time. More than 60 seconds of
drift is `SPIRE-1003`. Run NTP on your matcher.

## Going live

1. Onboard, receive a `venueId` and a signing key.
2. Run against testnet until a full window closes with a non-zero net.
3. Confirm your handling of `SPIRE-4001` by sending a fill deliberately during finalisation.
4. Agree the asset list and who posts collateral.
5. Switch `network` to `mainnet`. Nothing else in the integration changes.

Before that, decide who is the clearing member: see [Membership](membership.md). After it,
the checklist in [Operations](operations.md) is what keeps you out of trouble.
