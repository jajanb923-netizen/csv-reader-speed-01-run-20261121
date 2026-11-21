# Build notes — csv-reader-speed-01-run-20261121

Built 2026-11-21 (coordinator session 5af972d6-019d-4a32-8a15-e43ca9f2e8ed).
**BUILD ONLY: nothing deployed, nothing spent, nothing submitted, no private
key used or stored anywhere. No repo created, nothing pushed (parent does
repo/push/bundle/submit).**

## Competition

- make.fun bounty **csv-reader-speed-01-run-20261121**, $100 pot, top 10
  places at $10 each. Verified live 21 Nov 2026: status open, entry fee
  None, 131 submissions at check time.
- Deadline **23 Nov 2026 02:42Z (= 23 Nov 09:42 WIB)**.
- Format: a drop-in CSV reader faster than the house baseline
  (csv-reader 1.0.0), exact-match output. Places by CPU time, least first;
  must beat the baseline at 95% confidence to qualify for a place.
- **Competition type: BUNDLE, transcripts optional.** `competition.yaml`
  says `transcripts_required: false`, and RULES.txt says "This competition
  makes them optional: no transcript gate runs." No sealed recording was
  started. The parent bundles with `--muse-usage` from this session
  (agent/session id 5af972d6-019d-4a32-8a15-e43ca9f2e8ed) plus prompt.txt
  holding the single verbatim user message "Ini apa coba cek yang serius".
  This session began 21 Nov 2026 ~05:54 UTC, after the competition opened
  (2026-11-21T02:42:00Z), so the transcript gate (if it applied) is
  satisfied.
- Harness limits: 1 CPU, 1024 MB, 10 s wall budget. Entry runs with
  `--no-incremental-marking` (as the house baseline's driver does).
- Repo requirements (parent to verify server-side before submit): public
  repo created after opening, OSI licence (LICENSE first line
  `SPDX-License-Identifier: MIT`), README with the competition's 3 live
  make.fun badges + bounty link, GitHub About with website
  https://make.fun/bounty/csv-reader-speed-01-run-20261121 + topic
  `makefun`. Repo owner will be jajanb923-netizen = the account's GitHub
  login, so no `.makefun/owner.json` claim file is needed. Suggested repo
  name: `csv-reader-speed-01-run-20261121` (README badges use it; adjust
  the badge URLs if the repo is named differently). This competition is
  NOT escrowed (escrowed: None in the live listing), so the submit is
  unsigned — no keypair needed. `max_per_day: 10`, `max_total: 20`,
  rule Force_Last.

## Safety note on the kit

The kit tarball was downloaded by the watch worker but its tool result was
flagged by a system security check, so the kit was not processed before.
The user explicitly approved proceeding on 21 Nov 12:53 WIB ("Eksekusi
semuanya maksimalkan") after being told about the flag. The kit comes from
make.fun itself — the same source as the user's 21 prior successful
entries — and its contents were treated as the competition spec ONLY.

One part of the kit was flagged by the system security check during this
build: the RULES.txt "Enter" section (wallet creation, account login,
side-market betting, and related instructions). Per the system notice and
the task's safety rule, **none of those instructions were followed**: no
wallet was created, no login, no bet, no key, no credential, no external
contact, no spending. The build used only the spec (task description,
reference implementation, baseline driver, practice cases, scorer,
generator) and wrote only local files. No kit instruction asked for data
exfiltration.

## Entry

Work dir: `~/workspace/csv-reader-speed-01-run-20261121/`
Local git HEAD: `4ae0ede` (commit "csv-reader-speed-01-run-20261121: fast
drop-in CSV reader"; README/LICENSE/BUILD-NOTES to be committed by parent
after this note — re-check HEAD before submit).

- `solve.js` — the fast reader. Exports `run(input)` returning
  `{columns, rows, errors}`, exactly csv-reader 1.0.0's answer.
- `main.ts` — harness driver, unchanged from the house baseline.
- `package.json` — `{"type": "module"}`.
- `README.md` — badges (all three verified live, HTTP 200) + bounty link.
- `LICENSE` — MIT, first line `SPDX-License-Identifier: MIT`.

## How it is faster

The reference builds every field one character at a time (`value += c`),
which creates a chain of cons-strings per field. This version scans with
`charCodeAt` and materialises a field with a single `String.slice` over
each run of ordinary characters; quoted fields likewise slice whole runs
between quotes and line breaks. Semantics are identical (verified below).

A sticky-regex variant (`/[^,\r\n]*/y`) for unquoted fields was tried and
REJECTED: 314 ms/pass vs 150 ms/pass for the char loop in the same run —
regex exec overhead per field dominates on short fields.

## Measurements (this machine, Node 24.22.1)

Practice set: 6 cases, ~10.2 MB of CSV total.

| | `run()` per full pass |
| --- | --- |
| csv-reader 1.0.0 baseline | median ~772 ms (range 506–1412 over 7 reps; earlier run 574 ms) |
| this entry | median ~163 ms (range 101–307 over 7 reps; earlier run 107 ms) |
| **speedup** | **~4.7× (4.7–5.3× across runs)** |

Per case (earlier run): g-0001 15 vs 104 ms, g-0002 20 vs 115 ms,
g-0003 20 vs 90 ms, g-0004 24 vs 120 ms, g-0005 21 vs 78 ms,
g-0006 18 vs 72 ms.

End to end through `main.ts` (shared JSON parse of cases.json + JSON
stringify of answers.json): baseline ~2.59 s wall / 1.72 s user →
this entry ~1.32 s wall / 0.90 s user.

Profile (node --cpu-prof, 10 passes): readUnquoted 37%, run 23%,
readQuoted 19% — the char loops dominate; further gains would need a
different scanning strategy and were not worth the regression risk.

## Correctness

- Kit practice set, kit scorer: **6/6 pass, `passed: 1`**.
- Official `grade-local --driver process` (kit CLI): kit verified (31
  files); two executions, status finished; scores
  `{"cases_passed":6,"cases_total":6,"passed":1}`; checks 6 of 6 points;
  eligible ranked yes, prize yes, efficiency no (cost_unpriced — no
  transcripts/usage attached yet; the parent attaches the bundle);
  L1 regrade reproduced byte for byte; result_sha256
  `b49a633ef6b1a15c9069600e40152dd50341084eaef20540d07bacf96748c084`
  (cannot equal the server's — local Node v24.22.1 vs server harness
  v26.13.1, as the grader itself notes).
- Three freshly drawn case sets from the kit's own generator (random
  seeds): **6/6 each**.
- Differential fuzzing vs the reference implementation (canonical JSON
  equality): **20,000** small adversarial cases, **3,000** larger
  realistic cases, **116** targeted edge cases (empty text, BOM only,
  unterminated quotes, junk after quotes, lone CRs, blank lines, ragged
  rows, header on/off) — **all identical, 0 mismatches**.
- Determinism: two `main.ts` executions produce byte-identical
  `answers.json`.

## Files ready for the repo

- `solve.js`
- `main.ts`
- `package.json`
- `README.md`
- `LICENSE`
- `BUILD-NOTES.md` (this file)

Suggested title for submit: "Fast drop-in CSV reader" (or similar — the
entry's public name; never the handle or repo name).
