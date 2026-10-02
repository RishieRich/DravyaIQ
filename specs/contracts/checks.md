# Contract: checks

Authority for every rule the system applies before a capture can be saved. Implemented in `ui/backend/domain/checks.py` as pure functions:

```python
def run_checks(doc_type: DocType, values: dict[str, str], record: RecordView, policy: Policy) -> list[CheckResult]
```

`RecordView` is a read-only snapshot passed in by the service. Checks never touch the database, the clock or a provider.

```text
CheckResult
  id         str      stable code from the tables below, used by tests and translations
  level      "error" | "warning" | "ok"
  params     dict     the numbers and ids the message needs, already formatted by domain/format.py
  fix        {label_key, field, new_value}|null   one-click correction, only where listed
```

Message text lives in `ui/frontend/src/locales/<lang>.json` under `checks.<id>`, filled from `params`. English text below is the source wording.

## Policy values

`ui/backend/domain/policy.py`, one place, read from fixtures, overridable later per plant.

| Name | Value | Meaning |
| --- | --- | --- |
| `BALANCE_TOLERANCE_KG` | 0.5 | charged minus (output + purge) within this closes |
| `PURGE_NORM_PCT` | 3.0 | purge above this share of charge is flagged |
| `RECIPE_TOLERANCE_PCT` | 5.0 | each component within this deviation from recipe |
| `DELTA_E_LIMIT` | 1.0 | ΔE above this is flagged on lab dip and QC |
| `LOT_SUGGEST_MAX_DISTANCE` | 2 | edit distance for "closest lot" suggestion |
| `TRACE_FLAG_DELTA_E` | 0.9 | sibling batch QC above this is surfaced in trace |

## Parsing helpers (domain/parse.py)

- `number(text) -> Decimal|None`: strip commas, take the first signed decimal number, None if absent. `"396 kg" -> 396`, `"0.08 %" -> 0.08`, `"abc" -> None`.
- `charges(text) -> list[Charge]`: every match of `LOT-ID <sep> n kg` where LOT-ID is alphanumeric groups joined by hyphens (at least one hyphen), separator optional `: = - –`. Upper-cases the lot id. `"PB153-2480 18.2 kg; TIO2-R902-2408 12 kg"` gives two charges.
- `recipe(text) -> list[RecipeLine]`: split on `, ; newline`, each part `NAME n%`. Parts that do not match are dropped and reported by the RECIPE_UNREADABLE check if nothing parses.
- `component_for(material) -> str`: maps material names to canonical component keys. Table in `domain/components.py`, port the prototype's `compFor`. Unknown material returns the material text itself.
- `edit_distance(a, b) -> int`: Levenshtein.
- `closest_lot(lot_id, lots) -> str|None`: smallest distance, ties broken by lot id ascending, None if distance > `LOT_SUGGEST_MAX_DISTANCE`.

## Inward register

| id | level | when | English |
| --- | --- | --- | --- |
| LOT_MISSING | error | lot blank | Lot number is missing. |
| LOT_DUPLICATE | error | lot already on record | Lot {lot} is already in the inward register. |
| QTY_NOT_NUMBER | error | qty does not parse | Net weight is not a number. |
| COA_MISSING | warning | coa blank | No CoA value recorded. Ask the supplier for the test certificate. |
| INWARD_OK | ok | lot new and qty parses | New lot {lot}: {qty} kg of {material}. |

## Lab dip sheet

| id | level | when | English |
| --- | --- | --- | --- |
| SHADE_MISSING | error | shade blank | Shade code is missing. |
| DE_NOT_NUMBER | error | ΔE does not parse | ΔE is not a number. |
| RECIPE_UNREADABLE | error | no recipe line parses | Could not read the recipe components. |
| RECIPE_OVER_100 | error | sum of pct >= 100 | Recipe adds up to 100% or more, leaving no room for carrier. |
| SHADE_REPLACES | warning | shade already on record | This replaces the recipe of record for {shade}. The old recipe is kept as history. |
| APPROVED_ABOVE_LIMIT | warning | result says approved and ΔE > limit | Marked approved with ΔE {de}, above the usual {limit} limit. |
| LABDIP_OK | ok | shade, recipe and ΔE all valid | Recipe of record: {n} components plus carrier, let-down {let_down}. |

## Batch log sheet

| id | level | when | English |
| --- | --- | --- | --- |
| BATCH_MISSING | error | batch blank | Batch number is missing. |
| BATCH_DUPLICATE | error | batch on record | Batch {batch} is already in the record. |
| SHADE_NO_RECIPE | error | shade not on record | Shade {shade} has no recipe of record. Capture its lab dip sheet first. |
| LOTS_UNREADABLE | error | no charge parses | Could not read the lots charged. |
| LOT_NOT_FOUND | error | a charged lot is not on record | Lot {lot} is not in the inward register. The closest lot on record is {suggestion}. (second sentence only when a suggestion exists) |
| LOT_INSUFFICIENT | error | charge kg > remaining + 0.01 | Lot {lot} has only {remaining} kg left. |
| NUMBERS_UNREADABLE | error | output or purge does not parse | Output or purge is not a number. |
| BALANCE_CLOSES | ok | abs(gap) <= tolerance | Balance closes: {charged} kg charged, {accounted} kg accounted for. |
| BALANCE_OFF | warning | abs(gap) > tolerance | Balance is off by {gap} kg: {charged} kg charged, {accounted} kg accounted for. |
| PURGE_ABOVE_NORM | warning | purge / charged * 100 > norm | Purge loss {pct}% is above the {norm}% norm. |
| RECIPE_WAITING | warning | shade known but a lot is not found | Recipe check waits until every lot is matched. |
| RECIPE_MATCH | ok | every row within tolerance and no extras | Charge matches the recipe of record for {shade}, every component within {tol}%. |
| RECIPE_DEVIATION | warning | a row outside tolerance, one result per row | {component}: {actual} kg charged against {expected} kg in the recipe ({dev}%). |
| RECIPE_EXTRA | warning | a charged component not in recipe and not carrier | {component} is not in the recipe for {shade}. |

`LOT_NOT_FOUND` with a suggestion carries a fix: label `Use {suggestion}`, field `lots`, new value is the lots text with that lot id replaced (case-insensitive, first occurrence).

### Recipe adherence (domain/recipe.py)

```text
T = charged kg of the batch
for each recipe line:      expected = T * pct / 100, actual = sum of charges whose lot.component == line.component
carrier row:               expected = T * (100 - sum(pct)) / 100, actual = sum of charges whose lot.component == "carrier"
any other charged component: extra row
deviation % = (actual - expected) / expected * 100, one decimal, sign shown
```

Component matching normalizes both sides: lower-case, remove spaces, dots, underscores, hyphens.

## QC register

| id | level | when | English |
| --- | --- | --- | --- |
| BATCH_MISSING | error | batch blank | Batch number is missing. |
| BATCH_NOT_FOUND | error | batch not on record | Batch {batch} is not in the record yet. Capture its batch log first. |
| DE_NOT_NUMBER | error | ΔE does not parse | ΔE is not a number. |
| QC_REPLACES | warning | batch already has QC | This replaces the QC reading already on record. The old reading is kept as history. |
| DE_ABOVE_LIMIT_PASS | warning | ΔE > limit and result says pass | ΔE {de} is above {limit} but marked pass. |
| QC_OK | ok | batch found and ΔE parses | ΔE {de} for this batch; {shade} was approved at ΔE {approved_de}. |

## Dispatch slip

| id | level | when | English |
| --- | --- | --- | --- |
| DN_MISSING | error | dispatch note blank | Dispatch note number is missing. |
| DN_DUPLICATE | error | dispatch note on record | {dn} is already in the record. |
| BATCH_NOT_FOUND | error | batch not on record | Batch {batch} is not in the record yet. Capture its batch log first. |
| QTY_NOT_NUMBER | error | qty does not parse | Quantity is not a number. |
| SHADE_MISMATCH | error | batch shade differs from slip shade | Batch {batch} is shade {batch_shade}, not {shade}. |
| OVER_DISPATCH | error | qty > available + 0.01 | Only {available} kg of batch {batch} is left to dispatch. |
| NO_QC | warning | batch has no QC reading | This batch has no QC reading on record, so no certificate can be issued for it yet. |
| CUSTOMER_MISMATCH | warning | shade is customer-specific and slip customer differs (normalized) | Shade {shade} was matched for {shade_customer}. |
| DISPATCH_OK | ok | shade matches and qty within available | {qty} kg of {shade} to {customer}. |

## Gate

```python
def can_save(fields: list[CaptureField], checks: list[CheckResult]) -> SaveGate
```

Save is allowed only when no check has level `error` and every field is `confirmed`. The service re-runs checks on the server at commit time against the current record and refuses with HTTP 409 and the fresh check list if the gate fails. The client's view of the gate is advisory only.

## Number formatting (domain/format.py)

`kg(Decimal) -> str`: round half-up to one decimal, drop a trailing `.0`, Indian digit grouping. `400 -> "400"`, `1782 -> "1,782"`, `10850 -> "10,850"`, `18.25 -> "18.3"`, `123456.7 -> "1,23,456.7"`. Percentages one decimal. Formatting happens once, in the domain, and the UI prints the string.
