# Quickstart

From nothing to a novated obligation. The path below runs against the testnet, where
collateral is faucet-funded and no real asset moves.

## 1. Get a venue identifier

Onboarding issues two things: a `venueId` and a signing key. The key signs fills, and the
identifier tells the protocol which venue is making the claim.

```bash
export SPIRE_VENUE_ID="vn_7Q2K..."
export SPIRE_SIGNING_KEY="0x..."
```

The signing key never leaves your infrastructure. The protocol only ever sees signatures.

## 2. Install

```bash
npm install @spireproto/sdk
```

The package is not published yet. This page describes the intended surface so that an
integration can be designed against it now.

## 3. Connect

```js
import { Clearing } from '@spireproto/sdk';

const clearing = new Clearing({
  venue:   process.env['SPIRE_VENUE_ID'],
  signer:  process.env['SPIRE_SIGNING_KEY'],
  chainId: 4663,
  network: 'testnet',
});
```

## 4. Check that you can clear the asset

Clearing is granted per asset, not per venue. Ask before you send.

```js
const asset = await clearing.assets.get('0xA1c...');

console.log(asset.tier);          // 1
console.log(asset.initialMargin); // 0.08
console.log(asset.clearable);     // true
```

If `clearable` is false the fill will be rejected with `SPIRE-1204`. Handle it as a venue
level condition, not as a per-trade error.

## 5. Send a matched fill

```js
const obligation = await clearing.obligations.fromFill({
  fill: {
    fillId:    'v7-9f31c2',
    asset:     '0xA1c...',
    buyer:     '0xBd4...',
    seller:    '0x39e...',
    size:      '250000000000000000000',   // 250 units, 18 decimals
    price:     '187420000',               // 187.42 USDC, 6 decimals
    matchedAt: Math.floor(Date.now() / 1000),
  },
  window: 'current',
});

console.log(obligation.obligationId);  // 0x8c1f...
console.log(obligation.state);         // "novated"
console.log(obligation.windowId);      // 918342
```

The call returns when novation is done. At that point the bilateral link between buyer and
seller no longer exists and both face Spire Protocol.

## 6. Read the net

The number that matters is not the fill, it is what the member actually has to deliver at
the close of the window.

```js
const pos = await clearing.positions.get({
  member:   '0xBd4...',
  asset:    '0xA1c...',
  windowId: obligation.windowId,
});

console.log(pos.gross);       // 1000000000000000000000
console.log(pos.net);         // 250000000000000000000
console.log(pos.utilisation); // 1840  (18.4% of the limit)
```

Gross four times the net is normal inside an active window. That difference is the capital
that never has to move. See [Netting](netting.md).

## 7. Watch it settle

```js
clearing.on('window.finalised', async ({ windowId }) => {
  const net = await clearing.positions.list({ windowId });
  // deliver against net, or let the settlement contract pull it
});

clearing.on('obligation.settled', ({ obligationId }) => {
  // terminal, nothing further happens to this obligation
});
```

## What you have not had to do

You did not prefund the trade, you did not price the counterparty, and you did not write
anything to the chain per fill. One write happens per member per asset per window, against
the net.

## Next

Read [Integration](integration.md) for what a venue has to decide before going live, and
[Errors](errors.md) for the failure modes worth handling explicitly.

Not on JavaScript? Every step above is a plain HTTP call. See [HTTP API](http-api.md).

## Before the endpoint answers

```bash
git clone https://github.com/spireproto/spire-sdk
node examples/sign.js        # the exact digest and signature a fill produces
node examples/integrate.js   # the whole path against a stub, including the retry
```

An integration can be written, signed and tested end to end against this surface
before a single contract is deployed. That is the reason for publishing it first.
