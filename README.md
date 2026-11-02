# Build on CKB: Campaign #7 — Send Your First Multi-Hop Payment with Fiber

Proof of completion for **Build on CKB Campaign #7** on CKBoost.
I ran two browser Fiber nodes (Sender A and Receiver C) and sent a real
Testnet payment between them through **Bottle B**, a public intermediary
node. Then I reversed it to restore C's receiving liquidity.

- **Tutorial:** [Send a Multi-Hop Invoice Payment](https://www.fiber.world/docs/build/multi-hop-invoice)
- **Version used:** Hosted live demo (browser)
- **Network:** CKB Testnet only
- **Date completed:** 2 November 2026

## Results

| | Route | Amount | Actual fee | Payment hash |
|---|---|---|---|---|
| Quest 1 | A → Bottle B → C | 1 CKB | 0.005 CKB | `0xdfb7a216b86358d3fba0605ba992826fc063956bdcf8d5e60df888d66a571109` |
| Quest 2 | C → Bottle B → A | 1 CKB | 0.005 CKB | `0x18b46e98fd9af8a1a17b9d3d964f1db941de16232eab29c2e7c7668849843892` |

## My process

### 1. Set up two browser nodes
I started Sender A and Receiver C in the tutorial. Each node has its own
identity, keys and storage, and its own Testnet funding address.

### 2. Funded both nodes and opened channels with Bottle
I funded each node from the CKB Testnet faucet, then opened a 499 CKB public
channel from each node to Bottle and waited for both to reach CHANNEL_READY.

![Channel opening](setup-channel-opening.png)

### 3. Starting liquidity
With both channels ready, the maximum invoice was **151 CKB**.

![Starting liquidity](start-liquidity-channels-ready.png)

### 4. Quest 1: A → Bottle B → C
Receiver C created a 1 CKB invoice, and Sender A paid it through Bottle.

![Quest 1 outcome](quest1-outcome-payment-completed.png)

### 5. Quest 2: Restore receiving liquidity (C → Bottle B → A)

**Before the reverse payment:**

![Before return](quest2-liquidity-before-return.png)

**Liquidity restored:**

![Liquidity restored](quest2-liquidity-restored-confirmation.png)

**After the reverse payment:**

![After return](quest2-liquidity-after-return.png)

## Liquidity ledger (from my screenshots)

| Channel side | Start | After Quest 1 | After Quest 2 |
|---|---|---|---|
| A–B: Outbound from A (A's side) | 400 | 398.995 | 399.995 |
| A–B: Inbound to A (Bottle's side) | 151 | 152.005 | 151.005 |
| B–C: Inbound to C (Bottle's side) | 151 | 150 | 151.005 |
| B–C: Outbound from C (C's side) | 400 | 401 | 399.995 |
| **Maximum invoice** | 151 | 150 | 151.005 |

Each channel's total is 551 CKB in every column.

## How the reverse payment changed liquidity

### B–C channel during Quest 2
Before the reverse payment, C's side (Outbound from C) was 401 CKB and Bottle's
side (Inbound to C) was 150 CKB. After it, C's side became 399.995 CKB and
Bottle's side became 151.005 CKB. C's side dropped by 1.005 CKB because C was
now the sender. C paid the 1 CKB going back to A plus Bottle's 0.005 CKB
routing fee, and all 1.005 CKB moved from C's side of the channel to Bottle's side.

### A–B channel during Quest 2
Bottle's side (Inbound to A) went from 152.005 to 151.005 CKB, and A's side
(Outbound from A) went from 398.995 to 399.995 CKB. This happened because Bottle
forwarded the payment on the next hop. It moved exactly 1 CKB from its side of
the A–B channel to A and kept the 0.005 CKB fee it had picked up in the B–C
channel. Each channel only shifts balance between its own two sides, and
Bottle's two channels move in opposite directions.

### Why "Inbound to C" matters
Even though C held 401 CKB on its own side, the amount C can receive depends on
Bottle's side of the B–C channel ("Inbound to C"). C's own balance is what C can
send. To receive, the peer on the other end needs enough balance to move over
to C. I saw this when the maximum invoice changed from 150 CKB (before) to
151.005 CKB (after). The same idea showed up in the restore panel: the maximum
return was 152.005 CKB, which matched "Inbound to A", because A was now the
receiver and could only get as much as Bottle held on its side of the A–B channel.

### Fees
Each payment cost 0.005 CKB in routing fees (maximum allowed: 1 CKB). In
Quest 1, A paid 1.005 CKB but C received exactly 1 CKB. In Quest 2, C paid
1.005 CKB and A received 1 CKB. The difference went to Bottle B, the node that
forwarded both payments, so Bottle earned 0.01 CKB in total. After both
payments, A and C each ended with 399.995 CKB instead of 400, while Bottle's
side grew to 151.005 CKB in both channels. Each channel still totalled 551 CKB,
which shows that off-chain payments don't create or destroy CKB. They only
change how a channel's fixed amount is split between its two sides.

The actual fee (0.005) was much smaller than the 1 CKB maximum. I think the
1 CKB is a budget the sender allows on top of the invoice amount, not what they
will definitely pay. 0.005 CKB is what this route actually charged, though the
tutorial notes that trampoline routing can use up to the full budget, so a low
fee isn't guaranteed.

### Reserve vs. fee
Each channel was funded with 499 CKB, but only 400 CKB showed as available
because 99 CKB is reserved for on-chain settlement: 98 CKB for the commitment
lock's on-chain storage and 1 CKB for the closing (shutdown) transaction fee.
That reserve can't be used for off-chain payments. It's different from a
routing fee because it isn't paid to another node. Most of it comes back to the
owner when the channel closes, and about 1 CKB pays the closing transaction
fee. A routing fee goes to the node that forwards the payment and doesn't come back.

Bottle's 151 CKB fits the same rule if Bottle put in 250 CKB (250 − 99 = 151),
but I couldn't confirm how much Bottle funded.

## Challenges I faced

### 1. The two channels didn't become ready at the same time
**What I expected:** I expected both channels to become ready at about the same time.

**What happened:** Receiver C's channel reached CHANNEL_READY while Sender A's
channel was still at AWAITING_TX_SIGNATURES. I was tempted to click Open channel
again, but the button was greyed out while the channel was opening.

**How I resolved it:** I didn't reopen the channel. I kept the page open and
waited while A's channel went through NEGOTIATING_FUNDING →
COLLABORATING_FUNDING_TX → SIGNING_COMMITMENT → AWAITING_TX_SIGNATURES →
AWAITING_CHANNEL_READY → CHANNEL_READY. It took about 1–2 minutes. I realised
the two channels are separate on-chain funding transactions opened one after
the other, so each one needs its own confirmation. C's started first, so it
finished first.

### 2. The event log didn't match the channel status
**What I expected:** I expected the event log to add a "channel ready" entry once
each channel opened, matching the status rows above it.

**What happened:** The event log still said "existing or new channel is still
progressing" even after both channels showed CHANNEL_READY.

**How I resolved it:** I trusted the status rows (A–B channel and C–B channel
both CHANNEL_READY), the green "Ready" badges, and the fact that Step 2
(Create invoice) had unlocked. That told me the log only records events when
they happen and doesn't add one when a channel finishes opening. Adding a
"channel ready" event to the log would stop beginners from thinking their
channel is stuck.

### 3. Reading the liquidity bars
**What I expected:** I expected each bar to show only my own node's balance.

**What happened:** At first I wasn't sure which side of each bar belonged to Bottle.

**How I resolved it:** I read the labels under each bar. "Outbound from A" and
"Outbound from C" are my nodes' sides. "Inbound to A" and "Inbound to C" are
Bottle's sides, because inbound capacity is what the peer holds and can move to
me. I checked this after the first payment: "Inbound to C" dropped from 151 to
150 and "Outbound from C" rose from 400 to 401, so the balance had moved from
Bottle's side to C's.

## Reflection

### What I learned about liquidity on the A–B and B–C channels
Before this tutorial, I thought the CKB I held in a channel was also how much I
could receive. In my run, A had 400 CKB available but the maximum invoice was
only 151 CKB, because Bottle had only 151 CKB on its side toward C. This taught
me that sending and receiving use different liquidity. My own balance is what I
can send. What I can receive depends on what the other side of the channel
holds. In a multi-hop payment, the amount is limited by the smallest balance in
the payment's direction along the route. Here that was Bottle's side of the B–C
channel, not A's 400 CKB.

### Why multi-hop payments are useful
A and C never opened a channel with each other, yet A paid C's invoice through
Bottle in a few seconds, with a 0.005 CKB fee and no new on-chain transaction.
This is useful because every channel needs an on-chain transaction and locks up
CKB, including the 99 CKB reserve. You don't need a channel with everyone you
want to pay. One channel to a well-connected node like Bottle is enough to
reach others.

### What I found unexpected and interesting
The most unexpected thing for me was that after C got paid, C could receive
less than before. The maximum invoice dropped from 151 CKB to 150 CKB even
though C now had more CKB. I only understood it once I saw that receiving uses
up the inbound side of the channel, so I had to send 1 CKB back through Bottle
to restore it. I found it interesting that each channel still added up to
exactly 551 CKB after every payment. The payments only moved balance from one
side to the other, and Bottle earned its fees without either channel's total
changing.

## Feedback on the tutorial

### 1. Show clearer progress while a channel opens
When Receiver C's channel was already CHANNEL_READY but Sender A's channel sat
at AWAITING_TX_SIGNATURES for about 1–2 minutes, I expected some sign of how
long it had been waiting, or whether it was stuck. I was tempted to click Open
channel again. The button was greyed out, but nothing told me the wait was
normal. Even after both channels showed CHANNEL_READY, the event log still said
"existing or new channel is still progressing". It would help if the tutorial
added a "channel ready" entry to the event log when each channel finishes
opening. It would also help to show the elapsed time on the current stage,
with a short note like "waiting for on-chain confirmation, this is normal,
don't reopen".

### 2. Make it clearer which side of each liquidity bar belongs to Bottle
When I first saw the liquidity bars, I expected each one to show only my node's
balance. The bars show A, B and C at their ends, but I wasn't sure at first
that B meant Bottle or which side was Bottle's. I worked it out from the
"Outbound from" and "Inbound to" labels and by watching the numbers change
after the payment. It would help if the ends of each bar said "A (you)" and
"Bottle" instead of single letters. A one-line note saying that inbound means
what Bottle holds on its side and can send to you would also help.

### 3. Explain the 1 CKB routing fee budget on the Pay invoice step
When I saw "Max. routing fee: 1 CKB" before paying, I expected the fee to be
around 1 CKB, but the actual fee was only 0.005 CKB. It would help if the
tutorial explained, right where the 1 CKB is shown, that this is the most the
sender allows on top of the amount, not the fee they'll be charged. It could
add that the actual fee appears after payment and that trampoline routing can
use up to the full budget.

## Safety note

All transactions used **Testnet CKB only**. No private keys, seed phrases,
passwords or browser-storage contents are included in this repo.

## References
- [Send a Multi-Hop Invoice Payment](https://www.fiber.world/docs/build/multi-hop-invoice)
- [Multi-Hop Payments](https://www.fiber.world/docs/concept/routing/multi-hop)
- [Understanding Fiber Liquidity](https://www.fiber.world/docs/concept/liquidity/overview)
- [Open a Fiber Channel and Send a Payment](https://www.fiber.world/docs/build/open-channel-payment)
- [Connect to Fiber with a WASM Node](https://www.fiber.world/docs/build/connect-wasm-node)
