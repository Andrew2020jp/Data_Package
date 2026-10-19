# Replication package: AI participation in Architecture Decision Records

This package contains the data and scripts behind the paper's numbers and figures.

## Contents

| Path | What it is |
|---|---|
| `data/repositories_849.csv` | The 849 public GitHub repositories collected on 29 June 2026 (`full_name`, `created_at`, `stars`, `contributors`). |
| `data/adr_dataset.xlsx` | Full dataset. Main sheet: **Manual Review** (299 ADR records, one row per repository and ADR file path). Other sheets: repository-level screening of all 849 repositories, timestamps for the 71 L1/L2 records, excluded non-ADR material, and dataset notes. |
| `scripts/reproduce_funnel.py` | Recomputes every count in the selection funnel (849 → 191 → 117 → 107 → 299 records → 253 confirmed). `offline` mode needs no network. `live` mode re-queries GitHub; results will differ because repositories change over time. |
| `scripts/make_fig1_overview.py` | Draws Fig. 1 from the dataset. |
| `scripts/make_fig2_4_results.py` | Draws Figs. 2–4 from the dataset. |

All scripts stop with an error if a recomputed number does not match the paper.

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
pip install openpyxl matplotlib cairosvg
python scripts/reproduce_funnel.py offline
python scripts/make_fig1_overview.py       # -> figures/fig1_overview.{svg,pdf,png}
python scripts/make_fig2_4_results.py      # -> figures/results_rq{1,2,3}.pdf
```

`live` mode of `reproduce_funnel.py` needs the GitHub CLI (`gh`) logged in.

## License

Data: CC BY 4.0. Scripts: MIT License.
