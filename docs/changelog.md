# Changelog

What changed, when, and whether it can break you.

## How versions work

Three things version separately, and conflating them is how integrations break.

| What | Versioned by | Breaking change means |
| --- | --- | --- |
| HTTP and SDK surface | Path prefix, `/v1` | A new prefix appears, the old one keeps working |
| Protocol parameters | Governor proposal | A value in [Parameters](parameters.md) changes at a window boundary |
| Contracts | Proxy upgrade behind a 48h timelock | Storage or interface changes, announced before it lands |

Adding a field to a response is not breaking. Parse permissively and ignore what you do not
recognise, because new fields will appear without a version bump.

Removing a field, renaming one, changing a type, or changing the meaning of an existing
value is breaking, and it only happens behind a new prefix.

## Parameter changes

Every change to a number in [Parameters](parameters.md) is listed here with the window it
took effect in. This is the log to watch if you size positions against margin rates.

Nothing has changed yet. The values published are the launch values.

## Spec history

### 0.3 — 30 August 2026

- Added the onchain surface: contract interfaces, events, EIP-712 domain, upgrade and pause
  rules. See [Contracts](contracts.md).
- Added the transport surface: [HTTP API](http-api.md) with the same names as the SDK, so a
  non-JavaScript integration no longer has to read the SDK to learn the protocol.
- Added this changelog and the versioning rules above.
- Added diagrams for novation, the obligation state machine, netting and the waterfall.

### 0.2 — 30 August 2026

- Published concrete values for every parameter: window length, margin by tier, haircuts,
  fund sizing, fees and the default timeline.
- Added [Data model](data-model.md), [API reference](api-reference.md),
  [Errors](errors.md) and [Quickstart](quickstart.md).
- Reworked the mechanics pages to carry worked examples with numbers rather than prose
  alone.
- Set the assessment cap at 2x a member's default fund contribution and stated it in the
  places a member would look before joining.

### 0.1 — 26 August 2026

- First public description of the clearing layer: novation, collateral instead of
  prefunding, netting inside a settlement window, the default waterfall, and closed
  settlement with a public solvency proof.

## What is not built

The docs describe a design. As of this entry there is no deployed contract, the SDK is not
published, and the API is not serving.

That is stated here rather than only in passing, because a specification that reads like a
running system is the easiest way to waste somebody's week.

When any of it ships, the entry above will say so and will carry the commit it corresponds
to.
