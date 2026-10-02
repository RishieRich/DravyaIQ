# Contract: views

Authority for what each read screen computes. All of it lives in `ui/backend/domain/` and `ui/backend/services/`. The frontend renders what the API returns and computes nothing.

Every number the API returns is sent twice: the raw `Decimal` as a string under `value`, and the display string under `display` from `domain/format.py`. Components print `display`.

## Material in and out

`domain/balance.py::plant_totals(record) -> PlantTotals`

```text
received_kg   sum of Lot.qty_kg
charged_kg    sum of every Charge.kg across current batches
output_kg     sum of Batch.output_kg
purge_kg      sum of Batch.purge_kg
purge_pct     purge / charged * 100, one decimal
gap_kg        charged - output - purge
gap_batches   list of {batch_id, gap_kg} where abs(gap) > BALANCE_TOLERANCE_KG, largest first
```

Per lot row: lot id, material, supplier, received date, qty, used, remaining, percent used, list of batches with kg drawn.

Fixture expectation: received 10,850, charged 3,306, output 3,258, purge 42 (1.3%), gap 6, gap_batches `[B-TS01-0910-A: 6]`.

## Shade memory

Per shade: recipe of record with version history, approval ΔE and date, customer or "Catalogue", every batch made with its adherence rows and QC ΔE, sorted newest first. A ΔE trend is a list of points, not a chart computed in the browser.

## Trace

`domain/trace.py`

```text
dispatch_tree(dn) -> {dispatch, batch|null, qc|null, charges: [{lot_id, kg, lot|null}]}
default_suspect(batch) -> lot_id   first charge whose lot component is neither "carrier" nor "PE wax", else first charge
blast_radius(lot_id, exclude_batch=None) -> {
    lot, batches: [...], rows: [{batch_id, shade, dispatch_note|null, customer|null, kg}],
    batch_count, customer_count, shipped_kg,
    sibling_flags: [{batch_id, dispersion, delta_e, tested_on}]
}
```

- Rows: for each batch that charged the lot, one row per dispatch of that batch, or one "in stock" row with the undispatched kg if it has none.
- `customer_count` counts distinct customers across dispatched rows. `shipped_kg` sums dispatched rows only.
- `sibling_flags`: batches using the lot, other than `exclude_batch`, whose QC dispersion does not start with "pass" (case-insensitive) or whose ΔE > `TRACE_FLAG_DELTA_E`.

Fixture expectation for `DN-2609-407`: batch `B-TS01-0910-A`, default suspect `CB-N330-2409`, 2 batches, 2 customers (Krishna Pipes, Vardhman Moulders), 1,782 kg shipped, one sibling flag for `B-TS01-0912-B` with "Minor specks".

## Certificate of analysis

`services/certificate.py::build(batch_id) -> Certificate`

Lines in this order, each `{label_key, value|null, source|null}`:

1. Shade (code and name) from Shade
2. Batch from Batch
3. Date of manufacture from Batch
4. Customer, from dispatches of the batch, comma joined, or the literal state `not_dispatched`
5. Carrier from Shade
6. Recommended let-down from Shade
7. ΔE vs master standard from QcReading
8. MFI from QcReading
9. Dispersion from QcReading
10. Moisture from QcReading
11. Result from QcReading
12. Raw material lots from Batch charges

`issuable` is true only when every line has a value. When a QC reading is absent, lines 7 to 11 are `null`, the UI shows "not recorded", and `blocking_reason` is `no_qc_reading`. There is no code path that fills a null line with a default.

Header shows the plant name from config and a "Sample data" watermark while fixtures are loaded. Print view is the browser's print of the certificate page with a print stylesheet. PDF generation is out of scope for v1.
