# API reference

Every method the SDK exposes. Types are in [Data model](data-model.md), numeric limits are in
[Parameters](parameters.md), failures are in [Errors](errors.md).

The SDK is a wrapper. If you are not on JavaScript, use [HTTP API](http-api.md) directly, it
carries the same surface with the same names.

## Clearing

The client. One instance per venue.

```js
new Clearing({
  venue:   string,     // venueId issued at onboarding
  signer:  string,     // private key or a Signer implementation
  chainId: number,     // 4663
  network: 'mainnet' | 'testnet',
  timeout: number,     // ms, default 8000
})
```

`signer` accepts an object with a `signTypedData` method, so a KMS or an HSM can hold the
key instead of the process environment.

## obligations

### `obligations.fromFill(params)`

Novates a matched fill. This is the only write a venue makes in the normal path.

| Param | Type | Required | Description |
| --- | --- | --- | --- |
| `fill` | `Fill` | yes | Unsigned, the SDK signs it |
| `window` | `"current" \| uint64` | no | Defaults to `"current"` |
| `idempotencyKey` | `string` | no | Defaults to `fill.fillId` |

Returns `Obligation`. Idempotent on `fillId`: the same key returns the first obligation
rather than creating a second.

```js
const ob = await clearing.obligations.fromFill({ fill, window: 'current' });
```

### `obligations.get(obligationId)`

Returns `Obligation`, or throws `SPIRE-1101` if the id is unknown.

### `obligations.list(filter)`

| Filter | Type | Description |
| --- | --- | --- |
| `member` | `address` | |
| `asset` | `address` | |
| `windowId` | `uint64` | |
| `state` | `State` | |
| `limit` | `number` | Max 500, default 100 |
| `cursor` | `string` | From the previous page |

Returns `{ items: Obligation[], cursor: string | null }`.

## positions

### `positions.get({ member, asset, windowId })`

Returns `Position`. The net, the gross and the margin actually held. A window that has not
opened yet returns zeroes rather than an error.

### `positions.list({ windowId, member })`

Returns every position in the window, one per member and asset. This is what a venue reads
after `window.finalised` to know what has to move.

## collateral

### `collateral.get(member)`

Returns `CollateralAccount`.

### `collateral.post({ member, asset, amount })`

Posts collateral. The amount counts at its value after the haircut, so posting SPIRE adds
65% of its mark to `haircutValue`. Returns the updated account.

### `collateral.withdraw({ member, asset, amount })`

Withdraws from `free` only. A withdrawal that would take `free` below zero is rejected with
`SPIRE-3002` rather than triggering a margin call.

## assets

### `assets.get(address)`

| Field | Type | Description |
| --- | --- | --- |
| `asset` | `address` | |
| `tier` | `1 \| 2 \| 3` | |
| `initialMargin` | `number` | Rate, not a percentage string |
| `maintenance` | `number` | |
| `clearable` | `boolean` | False means fills will be rejected |
| `tierEffectiveAt` | `uint64` | When a pending reclassification applies |

### `assets.list()`

Every clearable asset. Cached for one window by default.

## windows

### `windows.current()`

Returns `SettlementWindow` for the window accepting fills right now.

### `windows.get(windowId)`

Returns a historical or future window. Future windows return `state: "open"` with a zero
`obligationCount`.

## Events

The client emits over a websocket and reconnects with backoff. Every event carries
`windowId`.

| Event | Payload | When |
| --- | --- | --- |
| `window.opened` | `{ windowId, opensAt }` | New window accepts fills |
| `window.finalising` | `{ windowId }` | No further fills accepted |
| `window.finalised` | `{ windowId, positions }` | Nets are published |
| `obligation.novated` | `{ obligationId, member }` | Counterparty replaced |
| `obligation.settled` | `{ obligationId }` | Terminal |
| `margin.call` | `{ member, shortfall, curesAt }` | `free` went negative |
| `member.defaulted` | `{ member, windowId }` | Cure expired |

```js
clearing.on('margin.call', ({ member, shortfall, curesAt }) => {
  // curesAt is a unix timestamp, not a duration
});
```

Events are at-least-once. Deduplicate on `obligationId` or on the pair
`(member, windowId)` depending on the event.

## Signing

Fills are signed EIP-712 with this domain:

```js
{
  name:              'Spire Protocol',
  version:           '1',
  chainId:           4663,
  verifyingContract: '0x...'   // published per network
}
```

The SDK signs for you. Sign manually only if the key lives somewhere the SDK cannot reach,
and in that case pass the result as `fill.signature`. The typehash and the onchain verifier
are in [Contracts](contracts.md).

## The client

[spire-sdk](https://github.com/spireproto/spire-sdk) implements this surface. The
endpoint does not answer yet; signing, digests, request building, error mapping
and event handling all work offline today, which is why its test suite needs no
network and no key.
