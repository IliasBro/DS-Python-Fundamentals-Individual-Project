# Limitations of AI: Worked Example

This note documents a concrete case where the AI assistant produced a plausible but incorrect result during this project. It is provided as supporting evidence for the "Limitations of AI" subsection of the AI Usage Documentation.

## What happened

While drafting the results narrative, the AI assistant wrote that the hospital ranking fully reverses after adjusting for severity, with Hospital_B moving from worst to best. The story was clean and intuitive: Hospital_B treats the hardest cases, so once you compare like with like, it should come out on top.

## Why it was wrong

The claim was never checked against the actual adjusted numbers. When the figures were recomputed, the picture was different:

Naive recovery rate: Hospital_A 81.3%, Hospital_C 74.6%, Hospital_B 68.1%

Recovery rate within each severity group:

| Severity | Hospital_A | Hospital_B | Hospital_C |
|---|---|---|---|
| low | 92.1% | 92.9% | 92.2% |
| medium | 72.2% | 76.5% | 71.8% |
| high | 55.9% | 39.3% | 43.1% |

Adjusted ranking (mean across severity groups): Hospital_A 73.4%, Hospital_B 69.5%, Hospital_C 69.0%

So the adjustment shrinks the gaps sharply, and Hospital_B is best in the low and medium groups, but Hospital_A still leads overall and even looks best in the high-severity group. There is no full reversal.

## Why the simple method cannot recover the truth

The data generation code assigns true quality effects of A -0.01, B +0.03, C +0.01, so the true order is B > C > A. Severity-only adjustment does not recover this because:

- Comorbidity is a second confounder that still varies within each severity band. In the high-severity group, Hospital_B's patients average a comorbidity score of 4.51 versus 3.78 for Hospital_A, which further depresses B's recovery rate.
- Hospital_A's high-severity group has only 59 patients, so its 55.9% rate is noisy and probably overstated.
- Averaging the three group rates with equal weight ignores the very different group sizes.

## Lesson

The error was caught only by manual recomputation, not by reading the AI output. This is a direct illustration that AI output is a draft to be validated, not a source of truth. The corrected, more honest conclusion (adjustment removes most of the naive gap but does not produce a reliable ranking) is also a stronger answer to the assignment than the original overclaim.
