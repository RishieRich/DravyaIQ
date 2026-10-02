# Handoff: Dravya IQ

Last updated 2026-10-02.

## Delivered

The specification for v1: brief, four build slices, six contracts, fixtures, sample sheets and golden test cases. No application code yet.

## Verified

| Check | Command | Result |
| --- | --- | --- |
| Golden check cases agree with the checks contract | scratch reference implementation of `specs/contracts/checks.md` run over all 24 cases in `data/golden/checks.json` | generated and reviewed by hand, outside this repo |
| Fixture totals match the brief | sum of `data/fixtures/*.json` | received 10,850, charged 3,306, output 3,258, purge 42, gap 6 |
| Trace numbers match the brief | blast radius of `CB-N330-2409` and `LL-MB-2409` over fixtures | 2 batches, 2 customers, 1,782 kg; 4 batches, 3 customers, 2,468 kg |
| Repository gate | `verify.py` from the ARQ zero-to-one skill | 23 pass, 1 fail: "no tests", expected until slice 01 lands |

Journeys exercised by hand: none in this repo yet. The same journey works in `docs/reference/shade-ledger-prototype.htm`.

## Not built

| Excluded | Why |
| --- | --- |
| All application code | v1 is built slice by slice from these specs |
| KPI board, chat over the record, learning, alerts | roadmap in `specs/005-roadmap-intelligence.md`, after a pilot |
| Auth, multi-plant, hosting | one plant, one laptop in v1 (ADR 0001) |
| Tally, ERP and machine data joins | separate ARQ modules; v1 keeps batch and dispatch ids stable for later joins |

## Known limits

- All data is invented sample data and the UI must say so while it is loaded.
- Gujarati and Marathi strings will be machine-drafted and need a native speaker's review before any plant sees them.
- The five sample sheets are synthetic. Real reading quality is unknown until real sheets from a plant are photographed.

## Operational notes

- `docs/reference/shade-ledger-prototype.htm` is the behavior and look reference. The contracts in `specs/contracts/` win where they differ.
- Do not edit a golden case to make a test pass. A disagreement between a golden case and a contract is a finding to report.

## Next

Next acceptance criterion: slice 01 criterion 1, every case in `data/golden/checks.json` passes as a parametrized test.
Spec: `specs/001-deterministic-core.md`
