# Integration

What a venue actually builds. The short version: you send one signed message per
fill and you read one event per window.

## What you send

```js
const { Clearing } = require('@spireproto/sdk');

const clearing = new Clearing({
  venue:   process.env.SPIRE_VENUE_ID,
  signer:  process.env.SPIRE_SIGNING_KEY,   // or an object with signTypedData
  chainId: 4663,
  network: 'testnet',
});

const obligation = await clearing.obligations.fromFill({ fill, window: 'current' });
```

A fill is `{ fillId, asset, buyer, seller, size, price, matchedAt }`. That is the
whole write path in the normal case: one call, per fill, at match time. You are
not asked what your order book looked like, why the trade happened, or who your
users are.

Or without the SDK, since everything it sends is a request you could make
yourself:

```bash
curl -X POST https://api.spireproto.xyz/v1/fills \
  -H "X-Spire-Venue: $SPIRE_VENUE_ID" \
  -H "X-Spire-Signature: $SIG" \
  -H "Content-Type: application/json" \
  -d '{"fillId":"v7-9f31c2","asset":"0xA1c...","buyer":"0xBd4...","seller":"0x39e...",
       "size":"250000000000000000000","price":"187420000","matchedAt":1774483200,"window":"current"}'
```

## What you read

| Event | Why you care |
| --- | --- |
| `window.finalised` | The nets are published. This is what your members have to deliver. |
| `margin.call` | One of your members has 600 seconds. If you carry the member relationship, this is your alert, not theirs. |
| `obligation.settled` | Terminal. Close the row. |
| `member.defaulted` | Somebody's cure expired. If it is not your member, the answer is to do nothing. |

Delivery is at-least-once, so deduplicate on `obligationId` or on
`(member, windowId)`.

## The three failures worth designing for

**`SPIRE-4001`, window finalising.** Not an outage: a thirty second door closing.
Retry once and the fill lands in the next window at the same price. Any venue
that treats this as an error will page somebody 288 times a day.

**`SPIRE-3001`, limit breached.** A business condition. The right handling is
usually upstream: refuse the order at the matcher rather than discovering it at
novation. `err.business` is true for exactly this class.

**`SPIRE-1003`, clock drift.** Your `matchedAt` is more than 60 seconds from
chain time. This is a monitoring problem that presents as a clearing problem, and
`node checks/chain.mjs` in
[spire-checks](https://github.com/spireproto/spire-checks) tells you before it
matters.

## Who is the member

Three models, and this is the decision that shapes your integration more than
anything technical:

| Model | Who posts collateral | Who gets the margin call |
| --- | --- | --- |
| Direct | Each end user | The end user |
| Venue as member | The venue | The venue |
| Hybrid | Venue for retail, direct for desks | Whoever posted |

Most venues start as the member for their retail flow and let professional
counterparties clear directly. It is the only model where a retail user is never
woken up by a margin call they do not understand.

## Testing before there is anything to test against

The endpoint does not answer yet. What works today, offline and without a key:

```bash
node examples/sign.js        # the exact digest and signature a fill produces
node examples/integrate.js   # the whole path against a stub, including the retry
node checks/netting.mjs my-fills.json   # what your own flow would net to
```

An integration can be written, signed and tested end to end against the published
surface before a single contract is deployed. That is the reason for publishing
the surface first.

## Checklist

- [ ] Sign fills with a key the protocol has on file, ideally held by a KMS
- [ ] Handle `SPIRE-4001` with a single retry, not an alert
- [ ] Reject over-limit orders at the matcher, not at novation
- [ ] Monitor clock drift against chain time, not against NTP alone
- [ ] Decide the membership model before you write the collateral UI
- [ ] Subscribe to `window.finalised` and reconcile every window, not every day
