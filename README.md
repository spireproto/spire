<div align="center">

# Spire Protocol

**Clearing for tokenised assets on Robinhood Chain.**

Novation, netting in time, collateral instead of prefunding, and a default
waterfall fixed before anyone defaults.

[![License](https://img.shields.io/badge/license-MIT-CCE624?style=flat-square&labelColor=050403)](LICENSE)
[![Chain](https://img.shields.io/badge/chain%20id-4663-CCE624?style=flat-square&labelColor=050403)](docs/parameters.md)
[![Window](https://img.shields.io/badge/window-300s-CCE624?style=flat-square&labelColor=050403)](docs/netting.md)
[![Deployed](https://img.shields.io/badge/deployed-nothing%20yet-CCE624?style=flat-square&labelColor=050403)](docs/status.md)
[![Custody](https://img.shields.io/badge/custody-none-CCE624?style=flat-square&labelColor=050403)](docs/solvency.md)

[spireproto.xyz](https://spireproto.xyz) · [Docs](https://spireproto.xyz/docs) · [@spire_proto](https://x.com/spire_proto)

</div>

---

## There will be many venues. There are one or two clearing houses.

Every market that grew up eventually put one counterparty between the two sides
of a trade. Not because the trades were bad, but because a market where ten
thousand participants each have to form a view on ten thousand others spends its
capital on that question instead of on liquidity.

Tokenised equities skipped the step. On chain, counterparty risk was solved by
removing the interval: everything is prefunded, so nothing is ever owed. It
works, and nobody has failed to deliver. The bill arrives somewhere else, as
capital that has to sit still.

A market maker quoting both sides of five assets on three venues does not need
capital equal to its net risk. It needs capital equal to the gross of everything
it might trade, everywhere it might trade it, at all times.

## What that costs, in one table

One member, one asset, one window: buy 400, sell 250, buy 350.

| | Prefunded | Cleared |
| --- | --- | --- |
| Turnover | 1,000 | 1,000 |
| Capital that has to be in place | 1,000 | 20 |
| What reaches the chain | 1,000 | 250 |

The 20 is initial margin on the net at tier 1. The mechanism that turns the first
column into the second is four things, none of them new, none of them present on
this chain today.

| | |
| --- | --- |
| [Novation](docs/novation.md) | The bilateral trade is discharged. Two obligations against the clearing layer replace it, independent from the moment they exist. |
| [Netting in time](docs/netting.md) | Obligations collapse inside a 300 second window, per member and per asset, across venues. |
| [Collateral](docs/collateral.md) | Margin against the net instead of prefunding the gross. Limits are checked at novation, before anything is owed. |
| [Default waterfall](docs/default-waterfall.md) | Loss absorbed in a fixed order, and the most a stranger's failure can cost you is bounded before you join. |

## Run the arithmetic yourself

```bash
git clone https://github.com/spireproto/spire-core
cd spire-core
node --test test/*.test.js     # 55 tests, zero dependencies
node examples/netting.js       # the window above, priced
node examples/default.js       # a default, absorbed layer by layer
```

Every worked example in these docs is one of those tests. If the published
numbers and the code ever drift apart, the suite fails before the docs do.

## Documentation

The same twenty pages as [spireproto.xyz/docs](https://spireproto.xyz/docs), from
the same source. The site renders them with search and navigation; here they are
markdown you can read in a diff, and the figures are mermaid rather than SVG.

**Start**

| | |
| --- | --- |
| [Overview](docs/overview.md) | What a clearing layer is for, and what this one is not |
| [Quickstart](docs/quickstart.md) | Zero to a novated obligation |
| [Integration](docs/integration.md) | What a venue sends and reads |

**Mechanics**

| | |
| --- | --- |
| [Novation](docs/novation.md) | The step, and the four checks that precede it |
| [Netting](docs/netting.md) | The window, the arithmetic, and what compression really depends on |
| [Collateral](docs/collateral.md) | Haircuts, limits, margin calls |
| [Default waterfall](docs/default-waterfall.md) | Five layers, two worked defaults, the timeline |
| [Solvency](docs/solvency.md) | Checking the layer without asking us |

**Reference**

| | |
| --- | --- |
| [Parameters](docs/parameters.md) | Every number, in one table |
| [Data model](docs/data-model.md) | The five objects that carry the lifecycle |
| [API reference](docs/api-reference.md) | Every method the SDK exposes |
| [HTTP API](docs/http-api.md) | The same surface without the SDK |
| [Contracts](docs/contracts.md) | Interfaces, events, the EIP-712 domain |
| [Errors](docs/errors.md) | Codes, and the edge cases worth designing for |
| [Glossary](docs/glossary.md) | Terms as this protocol uses them |
| [Changelog](docs/changelog.md) | What can change, and what breaks when it does |

**Operate**

| | |
| --- | --- |
| [Membership](docs/membership.md) | Who the member is, and what it costs |
| [Operations](docs/operations.md) | What the ten minute cure period means for staffing |

**Other**

| | |
| --- | --- |
| [Token](docs/token.md) | What SPIRE is a claim on, and what it is not |
| [Status](docs/status.md) | What exists and what does not |

## Repositories

| | |
| --- | --- |
| [spire-core](https://github.com/spireproto/spire-core) | Windows, novation, netting, margin, collateral, waterfall. BigInt, zero dependencies |
| [spire-sdk](https://github.com/spireproto/spire-sdk) | Client and EIP-712 signing. Keccak and secp256k1 written out, not pulled in |
| [spire-contracts](https://github.com/spireproto/spire-contracts) | The on-chain surface, published ahead of deployment |
| [spire-checks](https://github.com/spireproto/spire-checks) | Every number we publish, as a script you can run against us |
| [netting-replay](https://github.com/spireproto/netting-replay) | 33 modelled days through the netting rules, each reproducible from its date |
| [awesome-tokenized-equities](https://github.com/spireproto/awesome-tokenized-equities) | The landscape this sits in |

## Where this stands

Nothing is deployed. No contract address is claimed, nothing here holds an asset,
and the API does not answer. The specification is complete, the arithmetic is
tested and runnable today, and the checks that would embarrass us are in the same
account as the claims.

One number on this site is measured rather than designed: the chain runs at 0.101
seconds a block, not the two seconds the parameter table used to say. That was
our error, it was found by our own check, and the correction is in
[Parameters](docs/parameters.md) with the measurement linked.

Full accounting in [Status](docs/status.md).

## License

MIT
