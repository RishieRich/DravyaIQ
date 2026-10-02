# Slice 02: capture, confirm, check, save

Part of: `specs/000-product-brief.md`. Status: planned. Needs: slice 01 verified.

## Outcome

A supervisor opens Capture, picks one of the five document types, sees the sample sheet, loads its prepared reading (or uploads a photo and types values), confirms the highlighted fields, applies the suggested lot fix, watches the checks turn green, and saves. The saved record appears in the database with its provenance, and the app offers to open where it landed.

## In scope

- SQLite storage via SQLAlchemy 2 in `ui/backend/repositories/`, append-only versioned rows per `specs/contracts/data-model.md`
- `scripts/seed.py`: creates `data/dravya.db`, loads `data/fixtures/`, idempotent, `--reset` flag
- Services: `services/capture.py` (start, edit fields, apply fix, commit), `services/record.py` (builds `RecordView` from repositories)
- API routes for health, doc types, samples and captures from `specs/contracts/api.md`
- Frontend: app shell, Capture screen, sample sheet rendered on canvas from `data/samples/<type>.json` paper rows, review panel with fields and checks, save gate, saved confirmation with "Open in ..." link
- `FakeReader` and `NullReader` from `specs/contracts/reader.md`. No vendor reader in this slice.
- Image upload stored to `data/uploads/`, manual entry path when no reader

## Out of scope

- Trace, certificates, shade memory, material screens (slice 03)
- Live vendor reader, Gujarati and Marathi (slice 04)

## Contracts

- `specs/contracts/api.md` (health, doc-types, samples, captures)
- `specs/contracts/screens.md` (shell and Capture)
- `specs/contracts/checks.md` (gate)

## Acceptance criteria

1. `python scripts/seed.py --reset` twice in a row leaves exactly 7 lots, 4 shades, 6 batches, 6 QC readings, 6 dispatches.
2. Batch log sample, prepared reading: the lot error shows with a "Use PB153-2408" button; the purge field is highlighted for confirmation; Save is disabled and says why in words.
3. After the fix and the confirmation, the checks show the balance and recipe passes and Save is enabled. Saving creates batch `B-TS01-0925-A` version 1 with `source.mode = "prepared"` and the capture id.
4. `POST /api/captures/{id}/commit` on a capture with an open error returns 409 with the fresh checks, and the batch count is unchanged. A second commit of an already saved capture returns the same record and creates nothing.
5. With `MODEL_PROVIDER` unset, the capture screen shows "Live reading is not set up on this machine" and offers prepared reading and typing. Nothing errors.
6. Upload of a 9 MB image returns a plain "Photo is too large" message.
7. API tests in `ui/backend/tests/api/` cover criteria 2 to 4 through HTTP.
8. Keyboard only: the whole capture flow is completable with Tab, Enter and Space, with visible focus.

## Verification

| Criterion | How it was checked | Result |
| --- | --- | --- |
| 1 | `python scripts/seed.py --reset && python scripts/seed.py --reset --report` (prints record counts) | not run yet |
| 2, 3, 5, 8 | by hand in the browser, then `npm --prefix ui/frontend run test` | not run yet |
| 4, 7 | `pytest ui/backend/tests/api -q` | not run yet |
| 6 | `pytest ui/backend/tests/api -q -k too_large` | not run yet |

## Notes

The sample sheet canvas is a port of `drawDoc` in `docs/reference/shade-ledger-prototype.htm`. Paper colours used by the canvas are tokens too (`--arq-paper`, `--arq-paper-ink`), added to `theme.css`, not hex in the component.
