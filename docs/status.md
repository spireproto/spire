# Status

Written to be read by somebody deciding whether to spend a week on an
integration. Nothing on this page is a roadmap promise.

## What exists

| | |
| --- | --- |
| Specification | ✅ twenty pages, the same in this repository and at [spireproto.xyz/docs](https://spireproto.xyz/docs) |
| Clearing arithmetic, tested | ✅ [spire-core](https://github.com/spireproto/spire-core), 55 tests, zero dependencies |
| Client and EIP-712 signing | ✅ [spire-sdk](https://github.com/spireproto/spire-sdk), 27 tests, works offline |
| On-chain interfaces | ✅ [spire-contracts](https://github.com/spireproto/spire-contracts), published ahead of deployment |
| Checks against our own claims | ✅ [spire-checks](https://github.com/spireproto/spire-checks) |
| Netting measured over modelled flow | ✅ [netting-replay](https://github.com/spireproto/netting-replay), 33 days |

## What does not exist

| | |
| --- | --- |
| Deployed contracts | ❌ nothing on mainnet or testnet. `node checks/deployment.mjs` says so |
| A live API | ❌ `api.spireproto.xyz` does not answer |
| Published SDK package | ❌ `@spireproto/sdk` is not on npm; install from the repository |
| Audits | ❌ none. When there are, with the commit they cover and the findings, comfortable or not |
| Members | ❌ nobody has posted collateral, because there is nowhere to post it |
| Token | ❌ no emission schedule, no price, no yield. Deliberately not invented |

## What is measured rather than claimed

Two things on this whole site come from observation instead of design:

- **Block time on chain 4663 is 0.101 seconds**, measured over the last million
  blocks. The parameter table said 2 seconds until 30 August 2026; it was wrong,
  and it is corrected in [Parameters](parameters.md) with the measurement linked.
- **Compression over 33 modelled days ranges from 8.3% to 47.8%**, and the model
  behind it is documented rather than described. Modelled flow is not observed
  trading, and the repository says so on every file.

## What would change this page

A deployment. When there is one, the addresses go in
[spire-contracts](https://github.com/spireproto/spire-contracts), the checks stop
saying "not published", and this table gets shorter on the right.

Until then the honest summary is: the mechanism is specified, the arithmetic is
tested and runnable today, and nothing holds anyone's money because there is
nothing deployed to hold it with.
