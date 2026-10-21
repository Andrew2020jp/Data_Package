# Creation-commit AI attribution for the 71 L1/L2 records

Checked on 2026-10-21. The script made the first pass. The first author then manually reviewed all
71 results and agreed with every one. The human review is recorded in the dataset, not in this
folder: `adr_dataset.xlsx` → Non-L3 Creation Evidence → `Human check (creation commit)`. The
`Human review status` column in `creation_ai_evidence.csv` is script output, and it stays
`not reviewed`.

## Question

The 71 records in `data/adr_dataset.xlsx` → **Non-L3 Creation Evidence** (51 L2, 20 L1) each have a traced
creation commit and a selected (reviewed) AI-attributed commit. This check asks only one thing:
**does the creation commit itself carry an explicit AI signature?**

"Creation" means the file was added to the examined repository. It says nothing about who originally
wrote the document. The AI signature on the reviewed commit is never used as evidence for the creation
commit.

## Run

```
pip install openpyxl
GITHUB_TOKEN=<token> python scripts/check_non_l3_creation_ai.py   # run or resume
python scripts/check_non_l3_creation_ai.py --offline              # rebuild outputs from the cache only
python scripts/check_non_l3_creation_ai.py --selftest             # rule tests only
```

Run these commands from `data_package/`. The token is read from `GITHUB_TOKEN` or `GH_TOKEN`. It is not
written to code, logs or outputs. Every API response is cached in `api_cache/`, so a rerun only fetches
what is missing. Failed requests are not cached, so a rerun retries them.

## Rule

This is the paper's rule (Sec. II-B). An AI signature is one of:

1. a `Co-authored-by` trailer that names an AI tool or agent;
2. an author or committer identity that names an AI tool or agent;
3. a "generated with/by" marker that names an AI tool.

The following do not count: a tool name elsewhere in the commit message, a generic `[bot]` identity, and
AI wording in the ADR text.

The paper does not list tool names. The patterns are copied unchanged from
`others/verify_records_20260910.py`, the script that confirmed the current records. Its tool list is
longer than the one in `repro/rerun_screening_from_849.py`: it adds `jules`, `coderabbit` and `amp\b`, and
`amp\b` also matches inside words such as "camp". A hit matched only by those extra names would be marked
`needs human review`. No record hit this case.

As a check on the rule, all 35 distinct reviewed commits were re-detected as AI-signed.

## Checking that the creation commit adds the file

| How the addition was verified | Records |
|---|---|
| The creation commit adds the file at its current path | 48 |
| Added at an earlier path, with the link to the current path traced through renames via the GitHub API | 20 |
| Added at an earlier path. The reviewed commit lists ≥3,000 files, which GitHub truncates, so the link was checked instead: the current path in that commit has the same blob as the earlier path in its parent (pure move) | 3 |

If the addition cannot be verified, the record is set to `unresolved`, whatever its attribution. For 17 of
the 20 rename records, the earlier path from the 2026-09-10 trace table was used as a clue and then
re-traced independently. The rename chain agreed in all 17.

## Results (automatic check, confirmed by human review)

In the dataset, column `AI check (creation commit)` shows these results as `AI trace` and `No AI trace`.

| | L2 | L1 | Total |
|---|---|---|---|
| AI trace at creation | 30 | 7 | 37 |
| No AI trace at creation | 21 | 13 | 34 |
| Unresolved | 0 | 0 | 0 |

- The 37 records come from 32 distinct creation commits.
- Evidence: 35 trailers and 2 author identities. Tools: Claude 30, Copilot 4, Cursor 3.
- For these 37 records, both the creation commit and the reviewed commit carry an AI signature.

"No qualifying AI attribution found" does **not** mean AI was not used, and it does **not** mean
"later only". AI may have been used without a signature. There may also have been other AI commits between
creation and the selected commit, which this check does not look at.

## Files

| File | Content |
|---|---|
| `creation_ai_evidence.csv` | One row per record: SHAs and links, workbook times, API committer date, file status and how it was verified, automatic judgment, evidence type and text, tool, evidence source, re-detection of the reviewed commit, human review status, old lifecycle clues, notes |
| `creation_ai_summary.csv` | Counts by level, the rule check and coverage |
| `suggested_corrections.csv` | Workbook notes that the evidence contradicts (not applied) |
| `old_clues.json` | Fields copied from earlier internal trace and lifecycle tables (earlier file paths, old lifecycle tags), used only as clues. With this file, the package reproduces the results without the internal files |
| `api_cache/` | Raw GitHub API responses (commits, path histories, blobs), with each source URL |

## Notes on individual records

- **Rows 51 and 59.** The workbook says "Creation and reviewed commit are the same commit … the linked
  commit adds this ADR file". The SHAs differ, and the reviewed commits modify an existing file. The zero
  interval comes from identical committer timestamps. The L2 classification is unchanged. See
  `suggested_corrections.csv`.
- **Rows 288 and 289.** The creation commit `3f397d57c3f4` has a `Co-Authored-By: Claude Opus 4.8` trailer.
  The 2026-07-31 lifecycle table tagged it "later only" for the same SHA. Human review kept `AI trace`.
- **Rows 29, 41 and 199.** The creation commit message repeats the AI trailer 10–41 times (squash-style
  commit). The rule still counts it, but a person should check that the trailer covers the ADR file.
- **Row 72.** The message names Codex without a signature. Not counted.

## Limits

- Human review confirmed the script's results. It did not add any further evidence.
- The check covers only the creation commit and the selected commit. It is not a full lifecycle analysis.
- L3 records are out of scope.
- The current state of GitHub may differ from the state when the data was collected. Every response used
  is cached here.
- The script only reads `adr_dataset.xlsx`. The two columns in the workbook were added separately and
  then reviewed by hand.
