# Product brief: Dravya IQ

Created 2026-10-02. Owner: Rishi, ARQ ONE AI Labs.

Dravya IQ is the material record for factories that make things to a recipe from lot-controlled inputs. It is one module of the ARQ ONE MSME operating system (Astra for money, Kar IQ for tax, Yantra IQ for machines, Dravya IQ for material). The first edition is for masterbatch makers, demoed to prospects as "Shade Ledger".

## User and change

- User: the plant supervisor and the people who fill shop-floor paper at a masterbatch plant (storekeeper, lab chemist, extruder operator, QC assistant, dispatch clerk). Buyer is the owner.
- Change: the five paper sheets they already fill become one connected, checked record. A complaint that took two days to trace now takes seconds, and a certificate builds itself from readings already taken.

## Primary journey

A supervisor photographs a filled batch log sheet, confirms the fields the reader was unsure of, sees every check against the record pass or fail, saves it, then opens a dispatch from that batch and sees the lots that went into it, which other customers received the same lots, and a certificate of analysis whose every value links back to the sheet it came from.

Everything in v1 exists to make that sentence true.

## Invariants

- Every kilogram, percentage and total is computed in `ui/backend/domain/`. The model reads photos into text fields. It never computes, estimates or rounds a figure.
- Nothing is saved while any check returns an error, or while any low-confidence field is unconfirmed by a person.
- Every saved value carries its source: document type, capture id, capture time, read mode (live, prepared or manual) and confirming person.
- A certificate never prints a value that was never recorded. It shows "not recorded" and refuses to issue.
- Records are never deleted. A correction creates a new version that supersedes the old one.
- The whole journey works with no model key configured, using prepared readings or manual entry.

## Assumptions

| Assumption | Default taken | Cost if wrong |
| --- | --- | --- |
| One plant, one company | Single workspace, no tenant scoping in v1 | Add tenant column and scoping before any second customer |
| Local only | Runs on the developer laptop, SQLite file | Swap repository implementation to Postgres for hosting |
| Masterbatch is the first industry | Five document types: inward, lab dip, batch log, QC, dispatch | Paints and compounds reuse the same five with relabelled fields |
| Weights are kilograms to one decimal | Decimal arithmetic, kg, 0.1 precision | Unit conversion layer if a plant logs in tonnes or grams |
| Tolerances | Recipe component deviation 5%, mass balance 0.5 kg, purge norm 3%, delta E limit 1.0 | Each is one config value in `ui/backend/domain/policy.py` |
| Photo reader | One vision model behind `providers/reader.py`, configured by env | Swap provider in one file |
| Languages | Capture screen in English, Hindi, Gujarati, Marathi. Office screens in English | Add a locale file |
| Auth | A single stub user, "Supervisor" | Real auth before any hosted pilot |

## Non-goals for v1

- KPI dashboards, learning and suggestions, the ask-anything chatbot (planned in `specs/005-roadmap-intelligence.md`, the data model must not block them)
- Authentication beyond a stub user, roles, multi-plant, multi-tenant
- Hosting, containers, CI, billing
- WhatsApp or email alerts
- Reading from or writing to Tally or any ERP
- Bulk import of old registers, offline photo queue, native mobile app (responsive web only)
- Machine or PLC data (that is Yantra IQ)

## Data

- Source: deterministic fixtures in `data/fixtures/` (7 lots, 4 shades, 6 batches with QC readings, 6 dispatches, one carrying a customer complaint) and five synthetic sample sheets with prepared readings in `data/samples/`.
- Behaviour reference: `docs/reference/shade-ledger-prototype.htm`, the single-file demo this v1 grows from. Its rules are now written as contracts in `specs/contracts/`. Where the two disagree, the contracts win.
- Provenance and date: invented for demonstration, 2026-10-02. No real company's data.
- Labelled in the UI as sample data: yes, a persistent "Sample data" pill in the header while fixtures are loaded.

## Acceptance criteria

1. `python scripts/seed.py` loads the fixtures, and the Material screen shows 7 lots, received 10,850 kg, charged 3,306 kg, good output 3,258 kg, purge 42 kg (1.3%), unaccounted 6 kg, and the 6 kg is traced to batch `B-TS01-0910-A`.
2. Capturing the sample batch log with its prepared reading shows lot `PB153-2480` as an error with the suggestion `PB153-2408`. Applying the suggestion turns that check to pass.
3. With the lot fixed and the purge field confirmed, the checks show "Balance closes: 400 kg charged, 400 kg accounted for" and "Charge matches the recipe of record for WS-4471, every component within 5%", and Save becomes enabled.
4. Save is disabled while any error or any unconfirmed low-confidence field remains. Calling the commit endpoint directly in that state returns 409 and writes nothing.
5. After saving the batch log, then the sample QC and dispatch sheets, the trace for `DN-2609-431` shows batch `B-TS01-0925-A`, its four lots, and the certificate for that batch shows delta E 0.66 with a source line naming the QC register capture.
6. The trace for `DN-2609-407` with suspect lot `CB-N330-2409` shows 2 batches, 2 customers, 1,782 kg, and a warning that `B-TS01-0912-B` was logged with "Minor specks".
7. Immediately after saving the batch log for `B-TS01-0925-A`, and before its QC sheet is captured, its certificate shows "not recorded" for every QC field and a "Cannot issue yet" banner, and the issue action is disabled.
8. With no model key set, the capture screen says live reading is unavailable, and the whole journey still completes on prepared readings.
9. Switching the capture screen to Hindi, Gujarati or Marathi changes every label on it and every check message. Gujarati and Marathi strings ship marked `needs_native_review` until a native speaker signs them off.
10. `pytest ui/backend/tests` passes, including every case in `data/golden/checks.json` and `data/golden/views.json`.

## Verification commands

```bash
python scripts/seed.py
pytest ui/backend/tests -q
npm --prefix ui/frontend run build
npm --prefix ui/frontend run test
```

## Open questions

- Which vision model identifier to use. Verify against current provider docs at build time, do not take it from memory.
- Whether Welset's real sheets carry fields the five document types do not cover. Answer after the first site visit, then version the document schemas.
