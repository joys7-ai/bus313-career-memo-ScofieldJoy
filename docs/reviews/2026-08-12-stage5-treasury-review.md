# Stage 5 review — aerospace LLM analysis & validation · Treasury sign-off

Joy — the best paragraph in this document is the one where you decline to blame the spec:

> "The build audit's Findings 1–2 (an off-by-one cell reference on `USD_FLOOR_PUT`, and annotation cells accidentally parsed as formulas) are implementation bugs from the Stage 3 AI-assisted build, not spec ambiguities — the spec's own §6.3 checklist is what caught them. No v2 change indicated there."

A retrospective creates pressure to convert every defect into a lesson, and the easy version of this stage turns three bugs into three spec findings. You sorted them: two were build errors that the spec successfully caught (evidence the spec *worked*), one was a genuine gap. Knowing which failures are yours to fix upstream and which were already handled is the difference between a retrospective and a confession.

| Criterion | Score |
|---|---|
| LLM execution & comparison | 25 / 25 |
| Hand verification | 25 / 25 |
| Recommendation & executive voice | 25 / 25 |
| Spec retrospective | 17 / 17 |
| Repo polish | 6.4 / 8 |
| **Total** | **99 / 100** |

**What you did well — and why it matters**

- **Both crossover points are correct, and one of them is correct for a reason worth naming.** Unhedged overtakes the certainty hedges at 1.169 — which is just `F0_in`. That is not a coincidence: a forward's break-even is the forward rate by construction, always. And the put overtakes the forward at 1.18877, which is `F0_in` plus the *carried* premium (395,411 / 20,000,000 = 0.01977). Using the future-valued premium rather than the raw USD 0.019 is the subtle half, and you got it because you set that convention in Stage 2 and never drifted.
- **Your parity residual ties to the market-data memo exactly.** 23,379,769 − 23,380,000 = −USD 231, matching the memo's own banner and the Lab cross-check. Three independent artifacts agreeing on a number is what a reviewer actually wants to see.
- **Discrepancy B is diagnosed correctly and attributed correctly.** `F0_in` stored at four decimals against 1.168988 carried at full precision, a USD 240 spread on this notional, and — crucially — you called it spec ambiguity rather than an LLM error or a workbook error. Neither party did anything wrong; the document never said which precision to use. That is a real spec finding and your proposed v2 line fixes it.
- **The put's cost is stated as a decision, not as a number.** "The put is only the better choice if we're prepared to pay $395,411 today for a specific, deliberate view that EUR will rally more than 3% before settlement — nothing in our current market read supports paying for that view." That sentence tells a CFO exactly what they would be buying and what belief they would have to hold to buy it.
- **You scoped out hedge accounting explicitly rather than silently.** "Not assessed here (ASC 815 out of scope per the spec's stated limitations) — flag for a follow-up with Controllership." Naming what you did not do, and who should, is how a memo survives being forwarded.

**The one thing to fix — Discrepancy A points the other way**

Your §2 reports the money-market gap like this: full-precision recomputation gives ≈USD 23,379,786, the workbook reports USD 23,379,769, and the cause is that *"the workbook rounds the borrowed-EUR principal and the spot-converted USD amount at each intermediate step,"* while your from-scratch pass carried full precision.

I ran it. The mechanism does not hold, and the direction is reversed.

Carrying full precision the whole way through your own stated inputs:

```
DF_FC  = 1 + 0.02728 × 365/360 = 1.0276589
borrow = 20,000,000 / 1.0276589 = 19,461,710.71 EUR
USD    = 19,461,710.71 × 1.1545 = 22,468,544.21
DF_USD = 1 + 0.04 × 365/360 = 1.0405556
USD_MM = 22,468,544.21 × 1.0405556 = 23,379,769.34
```

That is the **workbook's** figure, to the dollar. The workbook is carrying full precision; your hand-walk is the one that rounded. The USD 17 enters at your borrowed-EUR line: you wrote 19,461,725, which is 14.29 EUR above the value your own stated `DF_FC` produces, and 19,461,725 × 1.1545 × 1.0405556 = 23,379,786.51 — your hand figure, exactly.

Two things follow, and the second is the one that matters.

The small one: the workbook needs no defence here, and Discrepancy A should be rewritten as a slip in the hand-check rather than a rounding-path property of the model. Nothing downstream moves — USD 17 on USD 23.4m changes no ranking, no recommendation, no crossover.

The larger one: you proposed a mechanism and did not test it. The claim "the workbook rounds at each step" is checkable in about a minute — recompute *with* rounded intermediates and see whether you land on 23,379,769. (You do, near enough: 23,379,769.32. Which is the tell — if rounding produced the workbook's number, the workbook is the rounded one, not yours.) The hypothesis was available for falsification and went untested.

Notice that this is the same failure you correctly caught the model in on Discrepancy A's sibling: asserting a specific reconciliation without the arithmetic behind it. Your validation doc holds the LLM to a standard it then does not quite apply to itself. That symmetry is worth sitting with — it is the most useful thing in this submission, and it costs you nothing on the grade, because the hand verification you *did* show is complete and three of four figures reconcile exactly. The habit to take forward is simple: when you name a cause, recompute under it and confirm it lands on the number you are explaining.

**Repo polish — 1.6 points**

Only gap is a missing `LICENSE`. Add one (MIT is fine for coursework) and you are at 100.

One filing note: `analysis/analysis/2026-08-05-Scofield-build-audit.md` has a doubled directory. Worth flattening before this repo is something you hand to someone.

— Treasury

---

### How to work this review — professional workflow

Treat this PR the way an analyst treats feedback from Treasury — a review is a proposal to engage with, not a checklist to rubber-stamp:

1. **Read it yourself first.** Understand each point and form your own view before changing anything. Disagreeing *with a documented reason* is a legitimate, senior response.
2. **Stress-test it with an LLM (pushback pass).** Paste this review and your spec into your AI assistant and ask it to (a) explain anything you're unsure of more deeply, and (b) argue the *other side* — where might the reviewer be wrong, and what would you give up by making each change. You're building judgment, not just executing edits.
3. **Decide, then draft the changes with the LLM.** For the points you accept, have the AI help implement them — you specify exactly what and why. Your spec is the prompt; precise in, correct out.
4. **Verify — non-negotiable.** Re-run your own checks (`scripts/recalc.py`, the parity tie-out, sensitivity continuity, no error cells) and confirm the numbers before you commit. An AI will hand you a confident wrong edit; verification is what makes the result *yours*.
5. **Close the loop on the PR.** Reply in the thread with what you changed, what you pushed back on and why, then commit and push. Writing down the reasoning is exactly how this works on a real team.

*This is the same human-in-the-loop discipline the whole project is built on: the LLM drafts, you edit and verify, and you own the result.*
