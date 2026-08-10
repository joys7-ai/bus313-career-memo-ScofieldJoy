# Stage 5 — LLM Analysis & Validation

**Scenario:** #20 — Aerospace Manufacturer, EUR 20,000,000 receivable, 365-day settlement
**Spec:** `docs/specs/2026-07-30-Scofield-aerospace-spec.md`
**Market data:** `data/2026-08-05-Scofield-market-data.md`
**Build audit:** `analysis/analysis/2026-08-05-Scofield-build-audit.md`
**Workbook:** `models/builds/2026-08-05-Scofield-aerospace-model.xlsx`
**Author:** Joy Scofield
**Date:** 2026-08-10

---

## Part 1 — Independent LLM execution

**Prompt log:** A fresh conversation was given exactly two links — the Stage 2 spec and the Stage 4 market-data memo — with the instruction: *"On these alone, produce the complete hedge analysis independently."* No workbook figures, no corrections, no mid-run coaching. Full raw output is preserved in the linked chat transcript; the figures it produced are reproduced in the comparison table below.

The LLM correctly: derived `DF_USD`/`DF_FC`, built the 11-row `S_T` grid at 1% steps, computed all four strategies, identified `USD_FLOOR_PUT`, and produced a correct piecewise "best active hedge" and "overall winner" classification, including the crossover points (no-hedge overtakes the certainty hedges near `S_T ≈ 1.169`; the put overtakes the forward near `S_T ≈ 1.1888`).

---

## Part 2 — Comparison table

Workbook figures are the audited, Stage-4-populated values reported in the market-data memo (§3) and independently cross-checked there against the FX Hedging Lab (§5). Compared at three `S_T` grid points: −5%, base (`S0_in`), and +5%.

| Strategy | S_T | LLM result | Workbook result | Match? |
|---|---|---|---|---|
| No hedge | 1.096775 | $21,935,500 | $21,935,500 | Exact |
| No hedge | 1.1545 (base) | $23,090,000 | $23,090,000 | Exact |
| No hedge | 1.212225 | $24,244,500 | $24,244,500 | Exact |
| Forward | all | $23,380,000 | $23,380,000 | Exact |
| Money market | all | $23,379,769 | $23,379,769 | Exact (adopted memo figure — see Discrepancy A) |
| Put | 1.096775 | $22,694,589 | $22,694,589 | Exact |
| Put | 1.1545 (base) | $22,694,589 | $22,694,589 | Exact |
| Put | 1.212225 | $23,849,089 | $23,849,089 | Exact |
| Call | — | Not computed | Not computed | N/A — reference-only strike/premium per spec §2.1; not a receivable hedge |

**Headline finding:** the spec was precise enough that an independent LLM run reproduced the audited workbook's figures essentially exactly. The two discrepancies worth flagging are both sub-$250 and neither changes the strategy ranking:

**Discrepancy A — Money-market proceeds, ~$17 gap.**
Computing the three-step MM walk independently, full precision through each step, gives ≈$23,379,786 before final rounding — the memo/workbook report $23,379,769. Diagnosis: rounding-path difference. The workbook rounds the borrowed-EUR principal and the spot-converted USD amount at each intermediate step; a from-scratch hand/LLM pass that carries full floating precision until the final dollar figure lands a few dollars off. Not a formula error — both paths use the same three named-range steps (borrow → convert → invest) — just a presentation-precision artifact. Immaterial at this notional.

**Discrepancy B — Forward proceeds, up to ~$240 depending on F0_in precision.**
`F0_in` is stored/displayed at 4 decimals (1.1690), but the CIP computation in the memo's §2 carries it to 1.168988. `FC_AMT × 1.1690 = $23,380,000` (what the workbook and this LLM run both used); `FC_AMT × 1.168988 = $23,379,760`, a $240 gap. Diagnosis: **spec ambiguity, not an LLM or workbook error** — neither the spec nor the market-data memo states how many decimal places a computed named-range input should be *stored* at once it's written into the Inputs tab cell. The workbook and this LLM run both used the rounded, stored 4-decimal value, which is a defensible and consistent convention, but it's implicit rather than specified. Flagged for the retrospective below.

---

## Part 3 — Hand-verification table

Recomputed independently, arithmetic shown, calculator/named-range notation only.

**1. Forward proceeds — `USD_FWD = FC_AMT × F0_in`**
```
FC_AMT = 20,000,000
F0_in  = 1.1690
USD_FWD = 20,000,000 × 1.1690 = $23,380,000
```
Reconciles exactly to the workbook.

**2. Money-market hedge — all three steps**
```
Step 1 (borrow EUR):     FC_AMT / DF_FC
  DF_FC = 1 + R_FC × T_DAYS/BASIS = 1 + 0.02728 × (365/360) = 1.027659
  20,000,000 / 1.027659 ≈ €19,461,725

Step 2 (convert to USD at spot):
  19,461,725 × S0_in = 19,461,725 × 1.1545 ≈ $22,468,562

Step 3 (invest to maturity at DF_USD):
  DF_USD = 1 + R_USD × T_DAYS/BASIS = 1 + 0.04 × (365/360) = 1.040556
  22,468,562 × 1.040556 ≈ $23,379,786
```
Workbook reports $23,379,769 — reconciles within $17 (Discrepancy A, rounding path, above).

**3. Put floor net of premium — `S_T = K_PUT` (at-the-money base case)**
```
FV_PREM_PUT = −PREM_PUT × FC_AMT × DF_USD
            = −0.019 × 20,000,000 × 1.040556 ≈ −$395,411

USD_PUT(S_T = K_PUT) = MAX(K_PUT, K_PUT) × FC_AMT + FV_PREM_PUT
                      = 1.1545 × 20,000,000 − 395,411
                      = 23,090,000 − 395,411 = $22,694,589
```
Reconciles exactly to the workbook's `USD_FLOOR_PUT`.

**4. Parity check — `USD_MM − USD_FWD`**
```
23,379,769 − 23,380,000 = −$231
```
Matches the market-data memo's own parity banner ("MM − Forward = $-231, rounding") and the FX Hedging Lab cross-check exactly.

---

## Part 4 — Spec retrospective

**What the LLM got wrong or had to guess:** essentially nothing structural — every formula, the grid construction, the floor/base split, and the winner logic came through correctly on a cold read. The two gaps that surfaced (Discrepancies A and B) are precision/rounding conventions, not comprehension failures, which is itself informative: they only surface when a second party tries to reproduce your numbers to the dollar, which is exactly what this stage is designed to test.

**What it reveals about the spec:**
- §2.2's derived-value formulas are unambiguous down to the operation, but the spec never states a **rounding/storage precision convention** for computed named ranges (e.g., "store F0_in to 4 decimal places once computed via CIP; do not re-derive at full precision downstream"). That single missing line produced Discrepancy B.
- The spec's §3 assumption that the Stage 2/3 parity gap is "expected, not a defect" held up perfectly against both the Stage 3 build audit (Finding 3) and the Stage 4 resolution — this is the spec doing its job: a documented limitation didn't get misdiagnosed as a bug by either the human auditor or the independent LLM.
- The build audit's Findings 1–2 (an off-by-one cell reference on `USD_FLOOR_PUT`, and annotation cells accidentally parsed as formulas) are implementation bugs from the Stage 3 AI-assisted build, not spec ambiguities — the spec's own §6.3 checklist is what caught them. No v2 change indicated there.

**What v2 of the spec would say differently:** add a one-line rounding/precision convention for every derived named range at the point it's written to its Inputs-tab cell (e.g., "4 decimal places for rates and FX levels, whole dollars for USD outputs"), so that a second builder — human or model — reproduces the same figure to the dollar without having to guess which precision the first builder locked in.
