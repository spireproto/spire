# HTTP API

The SDK is a wrapper over this. Anything the SDK does can be done with an HTTP client, which
is what you want if you are not on JavaScript.

| Setting | Value |
| --- | --- |
| Base URL, mainnet | `https://api.spireproto.xyz` |
| Base URL, testnet | `https://api.testnet.spireproto.xyz` |
| Version | Path prefix `/v1` |
| Content type | `application/json` |
| Streaming | `wss://api.spireproto.xyz/v1/stream` |

Breaking changes get a new prefix. `/v1` keeps working while `/v2` exists, and additive
fields are not breaking, so parse permissively. The full versioning rules are in the
[Changelog](changelog.md).

## Authentication

Two headers on every request.

| Header | Value |
| --- | --- |
| `X-Spire-Venue` | Your `venueId` |
| `X-Spire-Signature` | EIP-712 signature over the request body |

Reads accept the venue header alone. Writes need both, and the signature covers the exact
bytes you send, so sign after serialising.

```bash
curl https://api.spireproto.xyz/v1/windows/current \
  -H "X-Spire-Venue: $SPIRE_VENUE_ID"
```

## Idempotency

Every write accepts `Idempotency-Key`. For `POST /v1/fills` it defaults to `fillId`.

Replaying a key with an identical body returns the original result and
`Idempotency-Replayed: true`. Replaying with a different body is `SPIRE-1004`.

## Errors

One envelope, always.

```json
{
  "error": {
    "code": "SPIRE-3001",
    "message": "position limit exceeded",
    "detail": { "member": "0xBd4...", "limit": "12500000000000", "attempted": "13100000000000" }
  }
}
```

| HTTP | Meaning |
| --- | --- |
| 400 | `SPIRE-1xxx`, malformed or invalid |
| 401 | `SPIRE-2xxx`, authentication |
| 409 | `SPIRE-3xxx`, limits and collateral |
| 425 | `SPIRE-4001`, window is finalising, retry |
| 422 | `SPIRE-5xxx`, settlement |
| 429 | Rate limited, see `Retry-After` |

Branch on `error.code`, not on the HTTP status. Full list in [Errors](errors.md).

## Pagination

List endpoints take `limit` (max 500, default 100) and `cursor`, and return
`{ items, cursor }`. A null cursor means the last page.

## POST /v1/fills

Novates a matched fill. The only write in the normal path.

```bash
curl -X POST https://api.spireproto.xyz/v1/fills \
  -H "X-Spire-Venue: $SPIRE_VENUE_ID" \
  -H "X-Spire-Signature: $SIG" \
  -H "Content-Type: application/json" \
  -d '{
    "fillId": "v7-9f31c2",
    "asset": "0xA1c...",
    "buyer": "0xBd4...",
    "seller": "0x39e...",
    "size": "250000000000000000000",
    "price": "187420000",
    "matchedAt": 1774483200,
    "window": "current"
  }'
```

```json
{
  "obligationId": "0x8c1f...",
  "fillId": "v7-9f31c2",
  "windowId": 918342,
  "state": "novated",
  "novatedAt": 1774483201,
  "sides": [
    { "member": "0xBd4...", "direction": "receive", "size": "250000000000000000000", "cash": "-46855000000" },
    { "member": "0x39e...", "direction": "deliver", "size": "250000000000000000000", "cash": "46855000000" }
  ]
}
```

Field meanings are in [Data model](data-model.md).

## GET /v1/obligations/{obligationId}

Returns one obligation. `404` with `SPIRE-1101` if the id is unknown.

## GET /v1/obligations

Filters: `member`, `asset`, `windowId`, `state`, `limit`, `cursor`.

```bash
curl "https://api.spireproto.xyz/v1/obligations?member=0xBd4...&windowId=918342&state=netted" \
  -H "X-Spire-Venue: $SPIRE_VENUE_ID"
```

## GET /v1/positions

The net view. This is the endpoint to poll if you do not want the websocket.

```bash
curl "https://api.spireproto.xyz/v1/positions?member=0xBd4...&asset=0xA1c...&windowId=918342" \
  -H "X-Spire-Venue: $SPIRE_VENUE_ID"
```

```json
{
  "member": "0xBd4...",
  "asset": "0xA1c...",
  "windowId": 918342,
  "gross": "1000000000000000000000",
  "net": "250000000000000000000",
  "requiredMargin": "3748400000",
  "utilisation": 1840
}
```

Omit `windowId` for the open window. Omit `asset` to get every asset for that member.

## GET /v1/collateral/{member}

```json
{
  "member": "0xBd4...",
  "posted": [ { "asset": "USDC", "amount": "1000000000000" } ],
  "haircutValue": "1000000000000",
  "initialMargin": "80000000000",
  "defaultFund": "12000000000",
  "free": "908000000000",
  "limit": "12500000000000"
}
```

## POST /v1/collateral/deposits

Body `{ member, asset, amount }`. Returns the updated account. The amount counts after its
haircut, listed in [Parameters](parameters.md).

## POST /v1/collateral/withdrawals

Body `{ member, asset, amount }`. Comes out of `free` only. A withdrawal that would take
`free` below zero is `409` with `SPIRE-3002`.

## GET /v1/assets and GET /v1/assets/{address}

```json
{
  "asset": "0xA1c...",
  "tier": 1,
  "initialMargin": 0.08,
  "maintenance": 0.06,
  "clearable": true,
  "tierEffectiveAt": 0
}
```

Cache this. It changes at window boundaries, never inside one.

## GET /v1/windows/current and GET /v1/windows/{windowId}

```json
{
  "windowId": 918342,
  "opensAt": 1774483000,
  "closesAt": 1774483300,
  "finalisedAt": 0,
  "state": "open",
  "obligationCount": 4127
}
```

## GET /v1/proofs/{windowId}

The solvency proof for a closed window. Verifiable by anyone, no venue header required.

```json
{
  "windowId": 918341,
  "commitment": "0x9a3b...",
  "proof": "0x...",
  "aggregateCollateral": "48210000000000",
  "aggregateObligations": "31447000000000",
  "verified": true
}
```

The proof asserts that collateral covers obligations without opening any member's position.
See [Solvency](solvency.md).

## WSS /v1/stream

```bash
wscat -c "wss://api.spireproto.xyz/v1/stream" \
  -H "X-Spire-Venue: $SPIRE_VENUE_ID"
```

Subscribe after connecting:

```json
{ "op": "subscribe", "channels": ["window", "obligation", "margin"], "member": "0xBd4..." }
```

Each message carries `type`, `windowId` and a payload matching the events listed in
[API reference](api-reference.md). Delivery is at-least-once, so deduplicate on
`obligationId` or on `(member, windowId)`.

Heartbeat every 15 seconds. No frame for 40 seconds means reconnect.

## Rate limits

| Endpoint group | Limit |
| --- | --- |
| `POST /v1/fills` | 200 per second per venue |
| Reads | 50 per second per venue |
| Websocket connections | 8 per venue |

`429` carries `Retry-After` in seconds. The fills limit is per venue, not per member, so
size your matcher accordingly.
