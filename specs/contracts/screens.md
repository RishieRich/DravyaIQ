# Contract: screens

How the app looks and reads. Theme: Astra ink and orange, tokens only, from `ui/frontend/src/styles/theme.css`. Light and dark. Reference look and copy: `docs/reference/shade-ledger-prototype.htm`.

## Who reads which screen

| Screen | Main reader | Device | Language |
| --- | --- | --- | --- |
| Capture | supervisor and floor staff | phone or shop tablet | English, Hindi, Gujarati, Marathi |
| Trace | owner, QC head | laptop | English |
| Certificates | QC head, dispatch | laptop, print | English |
| Shades | lab chemist, owner | laptop | English |
| Material | owner, stores | laptop | English |

## Shell

- Sticky ink header: "Dravya IQ" lockup with a small "ARQ ONE" mark, five tabs in the order above, a "Sample data" pill while fixtures are loaded, theme toggle, and in development a "Reset demo" action that asks for confirmation in-page (no browser dialogs).
- Below 1080 px tabs become a bottom bar with icons and labels. Never icons alone.
- Every screen opens with a one-line plain-English explanation of what it answers, in the muted text colour.

## Capture

Left: the sheet. Right: what was read and what the checks say.

1. Document picker as five large cards, each with the document name, who fills it ("Filled by: Extruder operator") and a small icon.
2. Sheet panel: the sample sheet drawn on canvas, or the uploaded photo. Buttons: "Read this sheet" (live, only when the reader is available), "Use prepared reading" (samples only), "Type it in".
3. Reading state: progress line, "This can take up to a minute", Stop button.
4. Review panel: one row per field. Low-confidence fields have an amber left border, an amber "Check this" tag with an icon and text, and a "Looks right" button. Editing a field confirms it.
5. Checks list under the fields, grouped error, warning, ok. Each item has an icon shape (cross, triangle, tick) and text, so colour is never the only signal. A fix button sits inline on the check it fixes.
6. Save bar, sticky at the bottom: disabled state says in words what is blocking ("Confirm 1 highlighted field", "Fix 1 item marked in red"). Enabled state is the orange primary button "Save to record".
7. Saved state: a green confirmation with the record id and "Open in Shades / Trace / Certificates / Material".

Pointers for first-time users: a three-step strip above the sheet ("1 Photograph the sheet, 2 Check the highlighted boxes, 3 Save") that highlights the current step. No tours, no modals.

## Trace

Left: dispatch list with search, complaint tag in red with text. Right, top to bottom:

1. "What went into DN-2609-407": dispatched node, batch node with QC summary, lots as large buttons, the default suspect pre-selected with text "Suspect".
2. Blast radius panel with a red-soft wash: three big numbers (batches, customers, kg shipped) with labels, the supplier and CoA line for the lot, the table of batch, shade, dispatch, customer, kg, and amber notes for sibling flags in a sentence ("The record already held this signal on 12 Sep 2026").
3. "Record a complaint" text field on the dispatch.

## Certificates

Left: batch list with a "QC" or "No QC" tag. Right: the certificate as a paper-like card with plant name, title, line items, and a diagonal "SAMPLE DATA" watermark while fixtures are loaded. Values are dotted-underline buttons; clicking shows the provenance sentence below the card. "not recorded" is an amber tag. When not issuable, a banner above the card explains why and the Print button is disabled with a reason.

## Shades

List of shades with customer or "Catalogue". Detail: recipe of record as a component table with percent and the carrier remainder, approval ΔE and date, then batches made against it with an adherence mini-table per batch (expected kg, actual kg, deviation with sign) and QC ΔE. A small ΔE-by-batch dot plot uses the blue series token and labels each point.

## Material

Five key figures in a row: received, charged, good output, purge (kg and %), unaccounted. Unaccounted is green with a tick when within tolerance, red with a warning icon and the batch list when not. Then the lot table: lot, material, supplier, received date, qty, a used-share meter with the kg written next to it, remaining, batch count. A row expands to show which batches drew how much.

## Rules for every screen

- Numbers in `--arq-font-num`, right-aligned in tables, display strings from the API.
- Empty state says what to do next ("No dispatches yet. Capture a dispatch slip to see it here.").
- Loading uses skeleton rows, not spinners over content.
- Errors say what happened and what to do, never a code alone.
- Focus ring is the accent colour, 2 px, visible on every control.
- `prefers-reduced-motion` turns off transitions.
- Minimum touch target 44 px on Capture.
