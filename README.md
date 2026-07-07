# Token_Optimizer — Savings Stress Test (v1.1, ten moves)

A **measured A/B** of what the Token_Optimizer discipline actually saves. For each of the
ten moves it builds a realistic artifact, then counts the **real tokens** of the naive
path versus the optimized path with a proper tokenizer. No hand-waving, and re-runnable.

## Headline

On a heavy tool-and-file session (assumptions below), the discipline saves about
**464,000 tokens** — roughly **$1.08 to $2.70**, depending on the model and rate. Most of
it is input-side (avoided re-reads, full-log pulls, raw tool output). The three biggest
mechanical levers are trimming output at the source, pulling deltas instead of the whole
world, and one-line sentinels instead of re-reading files.

## Method

- **A/B, real tokens.** Every move has a naive artifact and an optimized one; both are
  tokenized and compared. `Saved per use = naive − optimized`.
- **Tokenizer:** tiktoken `cl100k_base`, used as a proxy for Claude's tokenizer. Absolute
  counts differ by ~10-20%, but the **reduction ratio is tokenizer-robust** — that is the
  part you can trust.
- **Session model:** `Session = (Saved per use) × (invocations per session)`. The per-use
  numbers are **measured**; the invocation counts are **stated assumptions** for a heavy
  session (edit `FREQ` in `stress_test.py`). Change them and the total moves.

## Results (measured)

| Move | Naive | Optimized | Saved/use | x/sess | Session | Cut |
|---|--:|--:|--:|--:|--:|--:|
| Trim tool output at the source | 9,599 | 8 | 9,591 | 20 | **191,820** | 99.9% |
| Pull deltas, not the world | 21,976 | 354 | 21,622 | 6 | **129,732** | 98.4% |
| Verify with sentinels, not re-reads | 8,118 | 2 | 8,116 | 12 | **97,392** | 100% |
| Reference by hash, don't re-quote | 6,254 | 22 | 6,232 | 3 | **18,696** | 99.6% |
| Consult the oracle *(new, est.)* | 6,619 | 86 | 6,533 | 2 | **13,066** | 98.7% |
| Spend 30s on the spec *(est.)* | 5,649 | 20 | 5,629 | 1 | **5,629** | 99.6% |
| Batch the round trips | 600 | 75 | 525 | 6 | **3,150** | 87.5% |
| Keep state files (warm starts) | 3,336 | 416 | 2,920 | 1 | **2,920** | 87.5% |
| Encode procedures as skills | 366 | 13 | 353 | 5 | **1,765** | 96.4% |
| Don't ignore low-hanging fruit *(new, est.)* | 139 | 17 | 122 | 2 | **244** | 87.8% |

**Heavy-session total: 464,414 tokens** (input 445,475 / output 18,939).

## Dollars per heavy session

| Rate | Per heavy session |
|---|--:|
| Introductory ($2 / $10 per M, through 2026-08-31) | ~$1.08 |
| Standard ($3 / $15 per M) | ~$1.62 |
| Opus 4.8 ($5 / $25 per M) | ~$2.70 |

Output tokens cost 5x input. Moves 9 and 10 book their savings on the output side (a wrong
build you don't generate), which is why the dollar figure rose more than the token total
did when they were added.

## Reading it honestly

- Per-use savings are **measured**; the session total leans on the frequency assumptions.
- The tokenizer is a **proxy** — trust the percentages more than the absolute counts.
- **Moves 7, 9, and 10 are conservative estimates, not measurements.** "Spend 30s on the
  spec," "consult the oracle," and "don't ignore low-hanging fruit" all save by preventing
  a wrong build or a user error — a thing you never build, or a mistake a picker prevents,
  is a cost you can't fully see. They're counted at a low frequency and modest size on
  purpose; in practice moves 7 and 9 can dwarf the mechanical saves, while move 10's real
  payoff is user experience, only partly visible in tokens.
- On a **subscription plan** you don't pay per token, so the dollars are really
  **headroom against your usage limits**, not cash.

## Files
- `token-savings-report.html` — a self-contained visual report; open it in any browser.

## Verdict

Clearly net-positive on tool-heavy sessions — real dollars on the big ones, or genuine
headroom on a subscription — and negligible on light chat. Adding moves 9 and 10 (v1.1)
lifted the heavy-session total from ~451k to ~464k and raised the dollar value more sharply
(they save output tokens). It earns its place without any ongoing tracking; re-run this
test whenever you want a fresh, honest number.

*Rates from Anthropic's published pricing (July 2026).*
