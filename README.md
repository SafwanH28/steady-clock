# Steady Clock

Sealed AI estimates of company results, published before the companies report and graded after.

Each lock covers one fiscal quarter. Before the quarter's results come out, Steady writes an estimate of revenue and diluted EPS for every company in its universe. Each estimate is built only from what the company had filed with the SEC by a fixed cutoff date. Simple statistical baselines are written alongside it.

Every estimate is hashed, and all the hashes go into one Merkle root, which is published here. Once the companies have reported, every row is revealed with its proof. Anyone can then check two things:
- the estimate existed before the result;
- nothing was changed after it was sealed.

## What is published, and when

| When | File | What it proves |
|---|---|---|
| At the lock | `<lock>/manifest.json` | The Merkle root, counts, and hashes of the pre-registration, prompt, universe, void list, code and input packets |
| At the lock | `<lock>/preregistration.md` | The exact test, written before any estimate was sealed |
| After grading | `<lock>/rows.jsonl`, `<lock>/proofs/`, `<lock>/packets/`, results | Every estimate with its Merkle proof, the inputs, the actuals and the score, whatever the score is |

No estimate is published while its quarter is live.

## How to verify a revealed row

1. Compute `sha256(canonical(row))`: JSON with keys sorted at every level and no whitespace.
2. Fold it with the published proof. Each step is `[side, hash]`; `L` means `sha256(hash || current)` and `R` means `sha256(current || hash)`, both on the raw 32-byte values.
3. Compare the result with the root in `manifest.json`.

The root's publication time is shown by this repository's commit history. Where present, a `.ots` file proves it against the Bitcoin blockchain (OpenTimestamps).

## Locks

| Lock | Quarter | Status |
|---|---|---|
| 2026Q3 | Fiscal quarters ending Sep 10 to Oct 10, 2026 | Being sealed |

These are AI estimates, not consensus and not investment advice.
