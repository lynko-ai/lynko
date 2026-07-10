# Benchmark Methodology — codebase deep dive

Six codebase-analysis tasks were run against the same private Go/TypeScript monorepo in two arms: a strong native baseline (filesystem + bash + grep in the same harness) and Lynko. Lynko used fewer harness tool calls on every task. Estimated billed-input savings ranged roughly 12–54% per task versus the native baseline. Task scores were tied or higher on five tasks and one check lower on one (T1: native 21/22 vs Lynko 20/22).

## Per-task results

| Task | Run shape | Score (native / lynko) | Est. billed-input savings |
|---|---|---|---:|
| T1 — auth architecture | 1 run each | 21/22 vs 20/22 | ~54% |
| T2 — operation registry audit | 1 run each | 20/23 vs 23/23 | ~25%¹ |
| T3 — refactor blast radius | 3 runs each | 35/42 vs 39/42 | ~37% |
| T4 — proto field rename | 3 runs each | 14.3/17 vs 15.0/17 | ~12%² |
| T5 — listing contract drift | 3 runs each (paired, 2026-05-18) | 18/18 vs 18/18 | ~22%³ |
| T6 — error surface contract audit | 3 runs each | 16/16 vs 16/16 (floor scoring)⁴ | ~47% |

Run shapes differ by task: T1 and T2 use one selected run per arm; T3–T6 use three-run means. Do not read the rows as uniform sample sizes.

## How the estimate works

Savings are computed as `1 − (N_lynko / N_native) × (FC_lynko / FC_native)`, where N is the harness-measured tool-call count and FC the final context size. Cumulative billed input scales as roughly `N·FC/2` because each turn re-reads accumulated context. Numbers carry ±5 percentage points of directional uncertainty and are **estimated billed input, not invoice-measured cost**. No cross-task aggregate winner is computed. Wall-clock latency does not track billed-input savings.

## Caveats

1. **T2** under a prompt-caching correction (cache reads at 10%) computes to ~17% savings; the ~25% shown uses the same formula as every other task for consistency. T2 is also the only task where Lynko's final context exceeds native's — the call-count advantage dominates.
2. **T4** is fragile at n=3: the arm-mean saving depends heavily on one aggressive-batching run; excluding it, the formula flips to roughly −18% (Lynko more expensive than native). Do not over-interpret the T4 row.
3. **T5** shows the paired 2026-05-18 comparison as canonical. A 2026-05-28 Lynko-only rerun (after `outline()` gained TypeScript support) reached ~49% versus the prior native run — supplemental evidence, not a paired result.
4. **T6** uses three-layer scoring; the 16/16 shown is the floor layer only. Objective bands and comparative dimensions differentiate further in the full run summary.

Full per-run summaries live in our internal benchmark repository. Early-access users can ask for the complete scorecard and any run summary — we will share them.