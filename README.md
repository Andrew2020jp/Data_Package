# Replication package: AI participation in Architecture Decision Records

This package contains the data and scripts behind the paper's numbers.

## Contents

| Path | What it is |
|---|---|
| `data/repositories_849.csv` | The 849 public GitHub repositories collected on 29 June 2026 (`full_name`, `created_at`, `stars`, `contributors`). |
| `data/adr_dataset.xlsx` | Full dataset. Main sheet: **Manual Review** (299 ADR records, one row per repository and ADR file path). Other sheets: repository-level screening of all 849 repositories, timestamps and creation-commit AI checks for the 71 L1/L2 records, excluded non-ADR material, and dataset notes. |
| `scripts/reproduce_funnel.py` | Recomputes every count in the selection funnel (849 → 191 → 117 → 107 → 299 records → 253 confirmed). `offline` mode needs no network. `live` mode re-queries GitHub; results will differ because repositories change over time. |
| `scripts/check_non_l3_creation_ai.py` | Checks whether the creation commit of each of the 71 L1/L2 records carries an explicit AI signature, using the same rule as the selected commits. |
| `analysis/non_l3_creation_ai/` | Output of that check: per-record evidence, summary, cached GitHub API responses and its own README. |

All scripts stop with an error if a recomputed number does not match the paper.

## Creation-commit AI check (sheet `Non-L3 Creation Evidence`)

For the 71 L1/L2 records, the selected commit is a later change. Column `AI check (creation commit)` records whether the earlier commit that created the file also carries an explicit AI signature: `AI trace` for 37 records (L2: 30, L1: 7) and `No AI trace` for 34 (L2: 21, L1: 13). The first author manually reviewed all 71 results (column `Human check (creation commit)`). `No AI trace` does not mean that AI was only used later, because AI use may leave no signature and the commits between creation and the selected commit were not examined. Column `Human check (timestamps)` records the manual check of the two timestamps.

## Commit levels (column `Commit Level` in Manual Review)

| Level | Meaning |
|---|---|
| L3 | The linked commit carries an explicit AI signature and adds the ADR file. |
| L2 | The linked commit carries an explicit AI signature and edits the ADR content. |
| L1 | The linked commit carries an explicit AI signature and only renames or moves the ADR. |
| L0 | The linked commit carries no explicit AI signature, and the ADR text contains no explicit AI-authorship statement. Excluded from RQ2 and RQ3. |

An explicit AI signature is an AI co-author trailer, an AI author or committer identity, or a "Generated with" marker. A commit message that only names an AI tool is not a signature.

## How the 849 repositories were collected

The repositories were found by searching GitHub for `0001-record-architecture-decisions.md`, the first ADR file created by adr-tools; 834 of the 849 contain this file. GitHub code search returns at most 1,000 results per query and its ranking changes over time, so re-running the search does not return the same list. The list itself is therefore provided in `data/repositories_849.csv`.

## Running

```
pip install openpyxl
python scripts/reproduce_funnel.py offline
python scripts/check_non_l3_creation_ai.py --offline   # creation-commit AI check from the cache
```

`live` mode of `reproduce_funnel.py` needs the GitHub CLI (`gh`) logged in. `check_non_l3_creation_ai.py` without `--offline` re-queries GitHub and reads a token from `GITHUB_TOKEN` or `GH_TOKEN`.

## License

Data: CC BY 4.0. Scripts: MIT License.
