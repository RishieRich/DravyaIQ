# Slice 01: deterministic core

Part of: `specs/000-product-brief.md`. Status: planned.

## Outcome

Every rule, total and trace in Dravya IQ exists as tested Python with no web framework, no database and no model involved. Running the golden suite proves the core says exactly what the contracts say.

## In scope

- `ui/backend/domain/`: `models.py`, `parse.py`, `components.py`, `format.py`, `policy.py`, `stock.py`, `balance.py`, `recipe.py`, `checks.py`, `trace.py`, `provenance.py`, `gate.py`
- `RecordView`: an in-memory, read-only snapshot type the domain reads, built from fixtures in tests
- `ui/backend/tests/test_golden_checks.py` driven by `data/golden/checks.json`
- `ui/backend/tests/test_golden_views.py` driven by `data/golden/views.json` (material and trace cases; certificate cases land in slice 03)
- Unit tests for `format.kg`, `parse.number`, `parse.charges`, `parse.recipe`, `parse.closest_lot`, including the examples in `specs/contracts/checks.md`
- `ui/backend/requirements.txt` and `pyproject.toml` with pytest, ruff

## Out of scope

- FastAPI, SQLite, any HTTP route, any provider, any frontend

## Contracts

- `specs/contracts/data-model.md`
- `specs/contracts/checks.md`
- `specs/contracts/views.md` (material and trace parts)

## Acceptance criteria

1. `pytest ui/backend/tests -q` passes, and every case in `data/golden/checks.json` runs as its own parametrized test with the case id in the test name.
2. Each golden case asserts the exact ordered list of `(level, id, params)`, not just ids.
3. `grep -rE "import (fastapi|sqlalchemy|anthropic|requests|httpx)" ui/backend/domain` returns nothing.
4. `grep -rn "float(" ui/backend/domain` returns nothing. Weights are `Decimal`.
5. The material golden case gives received 10,850, charged 3,306, output 3,258, purge 42 (1.3%), gap 6 on `B-TS01-0910-A`.
6. The trace golden case for `DN-2609-407` gives 2 batches, 2 customers, 1,782 kg and a sibling flag on `B-TS01-0912-B`.

## Verification

| Criterion | How it was checked | Result |
| --- | --- | --- |
| 1, 2, 5, 6 | `pytest ui/backend/tests -q` | not run yet |
| 3 | `grep -rE "import (fastapi\|sqlalchemy\|anthropic\|requests\|httpx)" ui/backend/domain` | not run yet |
| 4 | `grep -rn "float(" ui/backend/domain` | not run yet |

## Notes

The golden files were generated from a scratch reference implementation of the contracts and reviewed by hand. If a golden case and the contract disagree, stop and flag it in `HANDOFF.md`. Do not edit a golden case to make a test pass.
