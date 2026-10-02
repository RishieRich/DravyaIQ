# Contract: data model

Authority for every stored shape. Code in `ui/backend/domain/models.py` mirrors this file. If they disagree, this file wins and the code is the bug.

## Conventions

- Weights are `Decimal` kilograms, quantized to 0.1, half-up. Never `float` in the domain layer.
- Identifiers (lot, shade, batch, dispatch note) are stored upper-case and trimmed. Lookups normalize the same way.
- Dates are ISO `YYYY-MM-DD` in storage. Sheets write `DD/MM/YYYY`; the parser converts and keeps the raw text in the capture.
- Every record row carries `source` (see Provenance) and a `version` integer starting at 1.
- Records are append-only. A correction inserts a new row with `version + 1` and `supersedes` pointing to the old row id. Reads return the highest version.

## Entities

```text
Lot                       one supplier lot received at the gate (inward register)
  lot_id        str       "PB153-2408"
  material      str       "Pigment Blue 15:3"
  component     str       canonical component key used by recipes: "PB15:3", "TiO2", "PE wax", "CB N330", "PY83", "PR57:1", "carrier"
  supplier      str
  qty_kg        Decimal
  received_on   date
  coa_value     str|null  free text from the supplier certificate, e.g. "Strength 102.4%"

Shade                     a colour recipe approved in the lab (lab dip sheet)
  shade_code    str       "WS-4471"
  name          str
  customer      str|null  null for catalogue shades
  catalogue     bool
  carrier       str       "LLDPE", "PP homopolymer"
  recipe        list[RecipeLine]   pigments and additives only; carrier is the remainder to 100%
  let_down      str       "1:40 (2.5%)", kept as text
  delta_e       Decimal   approval ΔE against the master standard
  result        str       "Approved" | "Rejected"
  approved_on   date

RecipeLine
  component     str       must match a Lot.component key
  pct           Decimal   percent of total charge, 0 < pct < 100, sum of lines < 100

Batch                     one extruder run (batch log sheet)
  batch_id      str       "B-TS01-0925-A"
  line_shift    str
  shade_code    str       FK Shade
  made_on       date
  charges       list[Charge]
  output_kg     Decimal   good output
  purge_kg      Decimal   purge and start-up loss

Charge
  lot_id        str       FK Lot
  kg            Decimal

QcReading                 one QC register line for a batch
  batch_id      str       FK Batch
  delta_e       Decimal
  mfi           str       kept as text with unit, "21.4 g/10min"
  dispersion    str       "Pass", "Pass, no specks", "Minor specks"
  moisture      str       "0.08%"
  result        str       "Pass" | "Fail"
  tested_on     date

Dispatch                  one dispatch slip
  dispatch_note str       "DN-2609-431"
  customer      str
  shade_code    str
  batch_id      str       FK Batch
  qty_kg        Decimal
  dispatched_on date
  complaint     str|null  set later from the trace screen, never from a capture

Capture                   one photographed sheet moving through read, confirm, check, save
  capture_id    uuid
  doc_type      "inward" | "labdip" | "batchlog" | "qc" | "dispatch"
  image_path    str|null  local file under data/uploads/, null for prepared readings
  read_mode     "live" | "prepared" | "manual"
  reader_id     str|null  provider and model id for live reads
  status        "reading" | "review" | "saved" | "abandoned"
  fields        list[CaptureField]
  checks        list[CheckResult]   last run
  created_at    datetime
  saved_at      datetime|null
  saved_record  {entity, id, version}|null

CaptureField
  key           str
  raw_value     str       exactly what the reader or person entered first
  value         str       current value after edits
  confidence    "high" | "low"
  confirmed     bool      true when high confidence, or when a person pressed "Looks right" or edited it
  edited        bool
```

## Provenance

Every saved record has `source`:

```text
source
  doc_type      the five document types, or "seed" for fixtures
  capture_id    uuid|null    null for seed rows
  mode          "seed" | "live" | "prepared" | "manual"
  captured_at   datetime|null
  confirmed_by  str|null     the stub user name in v1, "Supervisor"
```

The certificate and trace screens render provenance with one function, `domain/provenance.py::describe(source) -> str`. Example outputs:

- seed: `Batch log sheet. Sample record loaded with the demo.`
- prepared: `Batch log sheet, photographed 2 Oct 2026, 11:40. Prepared reading, then confirmed by Supervisor.`
- live: `QC register, photographed 2 Oct 2026, 11:52. Read live by <reader_id>, then confirmed by Supervisor.`

## Derived values (never stored)

| Value | Owner function | Rule |
| --- | --- | --- |
| Lot used kg | `domain/stock.py::used_kg(lot_id)` | sum of `Charge.kg` for that lot across current batch versions |
| Lot remaining kg | `domain/stock.py::remaining_kg(lot_id)` | `qty_kg - used_kg` |
| Batch charged kg | `domain/balance.py::charged_kg(batch)` | sum of its charges |
| Batch gap kg | `domain/balance.py::gap_kg(batch)` | `charged - output - purge`, signed |
| Batch shipped kg | `domain/stock.py::shipped_kg(batch_id)` | sum of dispatch qty for the batch |
| Batch available kg | `domain/stock.py::available_kg(batch_id)` | `output - shipped` |
| Recipe adherence | `domain/recipe.py::adherence(batch, shade, lots)` | see `checks.md` |
| Blast radius | `domain/trace.py::blast_radius(lot_id)` | see `views.md` |
| Plant totals | `domain/balance.py::plant_totals()` | see `views.md` |

A component, route or model prompt that recomputes any of these is a defect.

## Storage

SQLite file at `data/dravya.db` in v1, behind `repositories/`. Schema created by `scripts/seed.py` from SQLAlchemy models, then fixtures loaded. Postgres later is a `DATABASE_URL` change plus a migration tool, decided in a future ADR.
