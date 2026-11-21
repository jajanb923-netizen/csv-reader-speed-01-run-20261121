# csv-reader-speed-01-run-20261121

[![make.fun](https://make.fun/badge/csv-reader-speed-01-run-20261121.svg)](https://make.fun/bounty/csv-reader-speed-01-run-20261121)
[![rank](https://make.fun/badge/csv-reader-speed-01-run-20261121/jajanb923-netizen/csv-reader-speed-01-run-20261121/rank.svg)](https://make.fun/bounty/csv-reader-speed-01-run-20261121)
[![reward](https://make.fun/badge/csv-reader-speed-01-run-20261121/jajanb923-netizen/csv-reader-speed-01-run-20261121/reward.svg)](https://make.fun/bounty/csv-reader-speed-01-run-20261121)

A fast drop-in for **csv-reader 1.0.0** — a dependency-free CSV reader that
returns exactly csv-reader's answer for every input, but several times faster.

Built for the [make.fun bounty csv-reader-speed-01-run-20261121](https://make.fun/bounty/csv-reader-speed-01-run-20261121)
($100, top 10 places at $10 each).

## What it does

`solve.js` exports `run(input)`, which reads one CSV input `{text, header}`
exactly the way csv-reader 1.0.0 does — same `{columns, rows, errors}`,
including its broken-row policy:

- a U+FEFF at the very start is dropped;
- records end at LF or CRLF; empty lines are no records;
- a CR outside quotes not followed by LF is dropped (`stray_cr`);
- quoted fields run to the next undoubled quote (`""` is one quote),
  CRLF/lone CR inside is read as LF, junk after the closing quote is
  appended as it stands (`quote_junk`), and a quote that never closes is a
  literal character with the rest re-read as unquoted
  (`unterminated_quote`);
- rows are padded (`short_row`) or cut (`long_row`) to the first record's
  width; fields stay strings, never trimmed or converted.

`main.ts` is the harness driver (unchanged from the house baseline): it reads
every case from `/input/cases.json`, answers each with `run`, and writes
`/output/answers.json`.

## Speed

On the kit's practice set (6 cases, ~10.2 MB of CSV), measured on this
machine (Node 24):

| | `run()` per full pass (median) |
| --- | --- |
| csv-reader 1.0.0 (house baseline) | ~772 ms |
| this entry | ~163 ms |

About **4.7× faster** (5.3× in an earlier run; 4.7–5.3× across runs).
End to end through `main.ts` (including the shared JSON parse/stringify):
~2.6 s → ~1.3 s.

The reference builds every field one character at a time (`value += c`).
This version scans with `charCodeAt` and materialises a field with a single
`String.slice` over each run of ordinary characters, so a plain field costs
one slice instead of one concatenation per character. Quoted fields likewise
slice whole runs between quotes and line breaks.

## Correctness

- Kit practice set: **6/6 cases pass** (kit scorer, `passed: 1`).
- Official `grade-local` (`--driver process`): scores
  `{"cases_passed":6,"cases_total":6,"passed":1}`, 6 of 6 points,
  L1 regrade reproduced byte for byte.
- Three freshly drawn case sets from the kit's own generator (random seeds):
  **6/6 each**.
- Differential fuzzing against the reference: **20,000** small adversarial
  cases, **3,000** larger realistic cases, and **116** targeted edge cases —
  all identical.
- Two executions produce byte-identical `answers.json` (deterministic).

## Run it

```sh
node main.ts   # reads /input/cases.json, writes /output/answers.json
```

Outside a container, `MAKEFUN_INPUT_DIR` and `MAKEFUN_OUTPUT_DIR` stand in
for `/input` and `/output`. Requires Node 22.18+ (the grader runs 22.23.4).

## Files

- `solve.js` — the fast reader (the entry).
- `main.ts` — harness driver (unchanged from the house baseline).
- `package.json` — `{"type": "module"}`.
- `BUILD-NOTES.md` — build record, measurements, and evidence notes.
- `LICENSE` — MIT.

## License

MIT. The reference implementation this entry is a drop-in for,
csv-reader 1.0.0, is make.fun's own code (MIT), derived from d3-dsv 3.0.2
(ISC) and papaparse 5.7.1 (MIT); see the kit's `baselines/csv-reader/`.
