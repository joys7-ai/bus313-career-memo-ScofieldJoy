| | |
|---|---|
| **To** | Chief Financial Officer |
| **From** | Joy Scofield, Treasury Analyst |
| **Date** | 2026-08-10 |
| **Re** | EUR 20,000,000 receivable — hedge recommendation (live data, Stage 4/5) |

---

## A. Exposure summary

As flagged in the July framing memo, the EUR 20,000,000 receivable settling in 365 days leaves our margin fully exposed to EURUSD — our costs are dollar-denominated, so every cent of movement between now and settlement flows straight to the bottom line. That analysis has now been run against live, sourced market data (spot, Treasury and Euribor rates, and a CIP-implied forward) and independently reproduced by a second, unrelated model with no access to our workbook. This memo is the decision point the framing memo asked to come back to.

## B. Hedge outcomes

At today's live inputs, the picture is more favorable than the placeholder analysis suggested: the forward locks in **$23,380,000**, about $290,000 above today's spot-equivalent value of $23,090,000. The money-market hedge replicates this almost exactly ($23,379,769) — the two are the same trade, priced two different ways, and now agree to within $231 once one consistent dataset feeds both.

The put floor sits at **$22,694,589** — $395,411 below the forward, which is the premium we'd carry for the right, not the obligation, to sell at the strike. Doing nothing is not a neutral option: at today's spot it delivers $23,090,000, already $290,000 below what the forward guarantees, with no floor beneath it if the euro weakens further.

## C. Sensitivity interpretation

The two certainty hedges (forward, money market) are flat lines — they don't care what EURUSD does between now and settlement, which is the entire point. The unhedged position and the put both move with the market, but differently: unhedged is a straight line with no protection in either direction; the put is flat below its strike (the floor) and rises linearly above it, net of the premium already paid.

Practically, that means: if the euro weakens or stays roughly flat, the forward and money-market hedge are unambiguously better than no hedge, and better than the put. If the euro **appreciates more than about 3% above spot** before settlement, the unhedged and put positions start to overtake the forward — the put in particular crosses above the forward's guaranteed proceeds once EURUSD moves past roughly 1.189, because its upside is uncapped while the forward's is fixed. That crossover is the entire cost/certainty tradeoff in one number.

## D. Recommendation

**Execute the forward hedge** for the full EUR 20,000,000 notional, locking in $23,380,000.

The money-market hedge is not worth pursuing as an alternative here — it delivers the same $23,380K (within $231) but adds a borrow/convert/invest chain with more operational moving parts and no offsetting benefit once forward pricing and the synthetic replication line up this closely. The put is only the better choice if we're prepared to pay $395,411 today for a specific, deliberate view that EUR will rally more than 3% before settlement — nothing in our current market read supports paying for that view on this contract.

## E. Executive justification

- **Cash-flow certainty:** the forward converts a $23.09M–$24.24M range (the ±5% no-hedge band) into a single known number for budgeting and covenant purposes.
- **Cost:** zero premium outlay, versus $395,411 for the put — a real cash cost today for optionality we don't currently have a thesis to pay for.
- **Liquidity/operational load:** the forward is a single trade at t₀ and t₁; the money-market hedge ties up EUR borrowing capacity for the full year for an economically identical outcome.
- **Optionality forgone:** if EUR rallies hard, we give up upside above $23.38M. Given the contract's size relative to the premium cost, that's a trade we recommend accepting rather than paying for.
- **Accounting:** not assessed here (ASC 815/hedge accounting treatment out of scope per the spec's stated limitations) — flag for a follow-up conversation with Controllership before execution if cash-flow hedge designation is desired.

I'd like sign-off to execute the forward with our existing FX counterparty at the earliest practical date; happy to walk through the sensitivity grid in person if useful before we lock it in.
