# Operations

The protocol's timings decide the staffing, not the other way round. This page is
what that means in practice.

## The budget

| Event | Clock |
| --- | --- |
| `free` goes negative, call issued | 0s |
| Cure period ends | 600s |
| Default declared | 600s |
| Auction opens | 900s |
| Auction closes | 1800s |

Ten minutes from the call to the default. That is the entire budget for
detection, decision and a confirmed transaction. If alerting takes four minutes
to reach a human, six are left, and a human who has to look something up in a
runbook they have never opened will use all six.

Every operational decision below follows from that one number.

## What to monitor

| Signal | Threshold | Severity |
| --- | --- | --- |
| Clock drift against chain time | > 20s | page |
| Fills rejected `SPIRE-1002` | any | page |
| Fills rejected `SPIRE-4001` | > 5% of a window | warn |
| Fills rejected `SPIRE-3001` | > 1% of a window | warn |
| Member utilisation | > 85% of limit | warn |
| Member `free` | < 0 | page |
| Window not finalised | 60s past close | page |
| Net not delivered | 30s before deadline | page |
| Our own reconciliation delta | any non-zero | page |
| Coverage ratio | < 10,500 bps | warn |
| Insurance layer balance | < 2,000,000 USDC | warn |

Two of those are unusual and deliberate. `SPIRE-1002` at any rate is a page
because a signature failure is either a key problem or an impersonation attempt,
and neither improves by waiting. A non-zero reconciliation delta is a page even
when it is small, because the size of the first delta says nothing about the size
of the second.

## Reconciliation

Per window, not per day. For each closed window:

1. Sum the obligations you sent against the positions the protocol published.
2. Compare your expected net per member and asset against `netOf`.
3. Compare the delivered amount against the published net.

Three ordinary causes of a delta, in the order you should check them:

- **A fill you sent during finalisation.** It is in the next window, not lost.
  Your reconciliation is off by one window, not off by an amount.
- **A member trading the same asset on another venue.** Netting crosses venues,
  so the published net is smaller than the one you computed from your own book.
  This is the system working.
- **A cancelled order that your matcher counted and never sent.** This one is
  yours, and it is the reason to reconcile against what you sent rather than
  against what you intended to send.

## Runbooks

**Your member is in a margin call.** Notify, then watch `curesAt`. Do not novate
new notional for them: it is refused with `SPIRE-3005` anyway, and a queue of
refused fills is a worse alert than the call itself.

**A window is not finalising.** `finalise` is permissionless. Call it yourself.
That is not a workaround, it is the design.

**Somebody else's member defaults.** Do nothing. Your obligations are independent
and settle normally. If your monitoring pages you for this, tune it: an alert
that fires when the system works as intended will be ignored when it does not.

**A venue is suspended, and it is yours.** Existing obligations stand and settle.
Stop sending, work the suspension, and do not attempt to unwind positions the
protocol still considers live.

**Clock drift.** Fix the clock, not the fills. Retrying a fill with a corrected
`matchedAt` is falsifying a trade record.

## Before going live

- [ ] Reconcile every window for two weeks on testnet with zero unexplained deltas
- [ ] Alert on clock drift and prove the alert reaches a human in under 90 seconds
- [ ] Rehearse a margin call end to end, including the part where somebody posts
- [ ] Confirm nobody is paged for another member's default
- [ ] Have the key custody answer written down before the first mainnet fill
