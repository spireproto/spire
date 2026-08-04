# Default waterfall

When a member fails to meet an obligation, Spire Protocol does not improvise. The
loss is absorbed in an order decided long before anyone defaults.

## The order

1. **Margin of the defaulting member.** Its own collateral goes first.
2. **Its contribution to the default fund.** Still its own money.
3. **The protocol insurance layer.** Capital set aside by the protocol itself.
4. **The mutualised fund of the other members.** Only here does anyone else's
   money move, and it is capped.
5. **Auction.** The position is auctioned to other clearing members.

The order is fixed in advance and no parameter can reorder it, because a
discretionary waterfall is not a waterfall. Whatever a governance process decides
in the middle of a stress event, it will not be decided in the interest of the
member who is not in the room.

## The layers, with numbers

| Layer | Source | Size at launch |
| --- | --- | --- |
| 1 | Defaulter's initial margin | 8% to 20% of notional, by tier |
| 2 | Defaulter's fund contribution | 15% of required margin, floor 50,000 USDC |
| 3 | Protocol insurance | 2,500,000 USDC |
| 4 | Mutualised fund | Capped at 2x each member's contribution per event |
| 5 | Auction | The position itself |

Layer 4 is the only one that touches capital belonging to members who did nothing
wrong. The cap on it is the number to read before joining: it is the most that
somebody else's failure can cost, and it is bounded before anything is signed.

## Worked, twice

A tier 1 member carrying 10,000,000 of notional stops meeting calls. Its margin
is 800,000 and its fund contribution is 120,000.

**Six percent underwater at the cure deadline:**

| Step | Amount | Left |
| --- | --- | --- |
| Loss | 600,000 | 600,000 |
| Layer 1, its margin | 600,000 | 0 |

Absorbed at layer one. No other member is touched, no insurance is drawn, no
auction happens, and nobody outside the operations channel learns that it
occurred.

**Fourteen percent underwater, a gap rather than a drift:**

| Step | Amount | Left |
| --- | --- | --- |
| Loss | 1,400,000 | 1,400,000 |
| Layer 1, margin | 800,000 | 600,000 |
| Layer 2, own contribution | 120,000 | 480,000 |
| Layer 3, protocol insurance | 480,000 | 0 |

Still nobody else's money. The mutualised layer is reached only when a single
default burns through 2,500,000 of protocol capital after the defaulter's own is
gone.

Both examples run as tests in
[spire-core](https://github.com/spireproto/spire-core):

```bash
node examples/default.js
```

## The timeline

| Event | When |
| --- | --- |
| `free` goes negative | Margin call issued, `curesAt` set |
| Cure period | 600s |
| Default declared | Immediately after cure expires |
| Auction opens | 300s after the default |
| Auction runs | 900s |

Ten minutes of cure is the entire budget for noticing, deciding and getting a
confirmed transaction on chain. If alerting eats four minutes, six are left. That
is a staffing fact, not a protocol one, and it is why
[Operations](operations.md) is written around these numbers rather than around a
monitoring product.

## Observable after the fact

`absorb` returns four sums rather than one, and `Absorbed` fires once per layer
touched. A default that stops at layer one emits one event, one that reaches the
mutualised fund emits four. The order of absorption is therefore something you
can check against the chain afterwards rather than something we assert in a
document like this one.

## What a member should take from this

If somebody else defaults, the answer is almost always that nothing happens to
you. The layers above the mutualised fund are sized so that an ordinary default
never reaches it, and if it does, your exposure was bounded before you joined.

That is the product. Not the absence of defaults, which nobody can promise, but a
default that is somebody else's problem in a way you could verify in advance.
