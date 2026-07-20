# Estonian II-pillar pension funds — look-through holdings dataset (31.05.2026)

A security-level **look-through dataset** of all actively managed Estonian second-pillar
pension funds (SEB, Swedbank, Luminor, LHV — 18 funds) and their passive benchmarks
(each manager's own index fund + Tuleva's bond fund TUK00), built for the master's thesis
*closet indexing in Estonian II-pillar pension funds* (Sten Ehrlich, TalTech, 2026).

Every fund's portfolio is expanded through its underlying funds/ETFs to individual
**securities (equities)** and **issuers (bonds)**, so that Active Share and any other
holdings-based measure can be computed with a few lines of pandas — see
[`active_share.ipynb`](active_share.ipynb), which reproduces the thesis numbers from
these CSVs alone.

## Files

| File | Contents |
|---|---|
| `funds.csv` | one row per fund: code, name, manager, role (`active`/`benchmark`), benchmark code (documents the thesis pairing — the notebook assigns benchmarks itself, so alternative pairings need no dataset change), equity-coverage %, matching rule, TUK00 bond-vector variant (a data property: which key regime the fund's bond vector uses) |
| `fund_sleeves.csv` | asset-class weights per fund in % of NAV (`eq`, `bond`, `re`, `pe`, `gold`, `cash`, `total`) |
| `holdings.csv` | every portfolio's holdings vectors: `code, vector, key, weight_pct` — active funds, the four index funds (looked through to securities) and TUK00 (~1 000 bond issuers); each (fund, vector) sums to 100 (within-sleeve renormalised) |
| `sources.csv` | full citation registry: every raw source file with provider, URL, download date, as-of date, and which underlying funds it serves (`EXACT` vs `SAME`-index match) |
| `reference_results.json` | the published equity/bond/composite Active Share per fund — the golden values any recomputation must reproduce |
| `active_share.ipynb` | the complete calculation: loads the CSVs (locally or from raw.githubusercontent.com), computes all three Active Share measures, asserts every fund against the reference results |

**Vectors** (`vector` column): `equity` (headline security-level look-through),
`equity_strict` (no cross-identifier fallback — sensitivity upper bound), `unit`
(wrapper level, no look-through — lower bound), `bond` (issuer level),
`bond_floor`/`bond_ceiling` (SEB bond-sleeve uncertainty band), and for TUK00 both an
issuer-keyed and a name-keyed variant (`funds.csv` says which one each fund is
compared against).

## Provenance and construction

- **Fund portfolios:** monthly reports of each pension fund as of **31.05.2026**
  (pensionikeskus.ee). Parsing was fail-closed: parsed section sums had to match the
  PDFs' own printed subtotals.
- **Underlying funds/ETFs:** the providers' published constituent files (iShares, SPDR,
  Amundi, Xtrackers, provider factsheets/holdings JSONs). Each source, its URL, download
  date, and as-of date is listed in `sources.csv`. Raw provider files are **not**
  redistributed here (their terms of use generally do not allow it); the published
  weights are derived data — fund weight × constituent weight, aggregated. Anyone can
  re-download the raw inputs from the URLs in `sources.csv`.
- **Matching:** equities by ISIN where both sides carry ISINs, otherwise by normalised
  name (the rule actually used per manager is recorded in `funds.csv`); bonds are
  aggregated to **issuer** level (government by country, corporates by normalised name),
  because a fund and an index almost never hold identical bond issues. Where a fund holds
  an index fund whose exact constituents are unpublished, the current same-index physical
  ETF was substituted — every such substitution is flagged `SAME` in `sources.csv`.
- **Composite-benchmark rule (uniform across managers, written method revision
  2026-07-19):** each fund's composite benchmark = its equity share in the manager's own
  index fund + (bonds+PE+RE+gold) share in TUK00, cash-neutral. Positions no passive
  benchmark can hold (PE, real estate, gold) count as fully active.
- **Construction pipeline:** built with LLM assistance (Claude), with all methodology
  decisions pre-registered and signed off by the author before computation, and verified by
  a four-layer check suite (golden toy tests + benchmark zero-assertions; parsed data vs
  PDF subtotals; raw-file quality + sha256 provenance; consistency against independently
  adversarially-reviewed archived results). The full pipeline lives in the author's
  analysis repository; this repo publishes its finished data product.

## Quality checks you can run yourself

Run `active_share.ipynb` top to bottom (only `pandas` needed). It contains a hand-worked
formula check and a **golden check**: the equity-sleeve, bond-sleeve, and composite
Active Share of all 18 funds, recomputed from these CSVs, must match
`reference_results.json` to 0.01 pp — the notebook raises if any value differs.
Structural invariants: every (fund, vector) group in the holdings files sums to 100;
every active fund in `funds.csv` has a benchmark whose portfolio is present.

## Known limitations

- **Snapshot heterogeneity:** fund reports are as-of 31.05.2026, but constituent files
  span 01.05–24.06.2026 (each file's as-of date is in `sources.csv`). Index-fund
  constituents drift slowly, so the effect on Active Share is small, but it is not zero.
- **Name-matched managers:** where matching is by normalised name rather than ISIN,
  identifier mismatches are possible in both directions; sensitivity vectors
  (`equity_strict`, `unit`) bound the effect.
- **Private/unlisted assets** (private equity, real estate, forest, shareholder loans)
  have no look-through — they enter as asset-class buckets in `fund_sleeves.csv`, which is
  exactly how the composite Active Share treats them (no passive benchmark can hold them).
- The dataset reflects **one month-end**. It is not a time series.

## Disclaimer

This is a research dataset compiled by a private individual from publicly available
documents, provided *as is*, without warranty of any kind. It is **not investment advice**,
not an official publication of any fund manager, and may contain errors despite the checks
described above. Fund weights are derived estimates, not official records. Verify against
primary sources (`sources.csv`) before relying on any figure.

## License and citation

Data (CSV/JSON files): **CC BY 4.0** — see `LICENSE-DATA.md`.
Code (`active_share.ipynb`): **MIT** — see `LICENSE`.

Please cite as: Ehrlich, S. (2026). *Estonian II-pillar pension funds — look-through
holdings dataset, 31.05.2026*. https://github.com/StenEhrlich/estonian-pension-lookthrough
(see `CITATION.cff`).
