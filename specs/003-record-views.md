# Slice 03: the record answers questions

Part of: `specs/000-product-brief.md`. Status: planned. Needs: slice 02 verified.

## Outcome

The four office screens turn saved sheets into answers: Trace (a complaint to its blast radius in one click), Certificates (built from the record, every value clickable to its source, refusing to print what was never measured), Shades (recipe of record and every batch made against it), Material (kilograms in, out, and what does not add up).

## In scope

- `services/certificate.py`, `services/trace.py`, `services/material.py`, `services/shades.py` over the domain functions from slice 01
- API routes for material, shades, dispatches, trace, complaint and certificates from `specs/contracts/api.md`
- Four screens per `specs/contracts/screens.md`
- After a capture is saved, "Open in ..." takes the user to the right screen focused on the new record: inward to Material, lab dip and batch log to Shades, QC to Certificates, dispatch to Trace
- Print stylesheet for the certificate
- Certificate golden cases from `data/golden/views.json`

## Out of scope

- PDF export, emailing certificates, editing records from these screens other than the complaint text

## Contracts

- `specs/contracts/views.md`
- `specs/contracts/screens.md`

## Acceptance criteria

1. Brief criteria 1, 5, 6 and 7 pass by hand and as API tests.
2. Every value on a certificate opens its provenance line from `domain/provenance.py::describe`, and the UI never builds that sentence itself.
3. Picking a different suspect lot on Trace updates the blast radius from the API, not from client arithmetic. With `LL-MB-2409` selected for `DN-2609-407`, it shows 4 batches, 3 customers, 2,468 kg.
4. No React component contains arithmetic on kilograms or percentages. Reviewed by `grep -rnE "\\.reduce\\(|\\* ?100|/ ?100" ui/frontend/src/screens` returning nothing that touches record numbers.
5. Empty, loading and error states exist on all four screens.

## Verification

| Criterion | How it was checked | Result |
| --- | --- | --- |
| 1, 3 | `pytest ui/backend/tests -q -k "views or trace or certificate"` | not run yet |
| 2, 5 | by hand in the browser | not run yet |
| 4 | the grep in criterion 4 | not run yet |

## Notes

Trace is the screen that sells the product. Get its first paint right before polishing anything else: the dispatch, the batch, the lots as large buttons, then the three big numbers.
