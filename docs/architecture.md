# Architecture: Dravya IQ

Durable notes. Update when behavior, setup or structure changes, not on every commit.

## Shape

```text
ui/frontend (React)                         phone, tablet, laptop browser
   │  fetch /api/*  (display strings only, no arithmetic)
   ▼
ui/backend/api        FastAPI routes: parse, call one service, serialize
   ▼
ui/backend/services   capture, record, trace, certificate, material, shades
   │                      │
   ▼                      ▼
ui/backend/domain     parse, checks, gate, balance, recipe, stock, trace, provenance, format, policy
   ▲   (pure Python, Decimal, no framework, no I/O)
   │
ui/backend/repositories   SQLAlchemy, append-only versioned rows  ──►  data/dravya.db
ui/backend/providers      Reader: NullReader | FakeReader | AnthropicReader  ──►  data/uploads/, data/reader_log/
```

## Source of truth

| Fact | Owner module | Read by |
| --- | --- | --- |
| Check results and save gate | `domain/checks.py`, `domain/gate.py` | `services/capture.py`, Capture screen |
| Lot used, remaining; batch shipped, available | `domain/stock.py` | checks, Material, Trace |
| Batch charged, gap; plant totals | `domain/balance.py` | checks, Material |
| Recipe adherence | `domain/recipe.py` | checks, Shades |
| Dispatch tree, default suspect, blast radius | `domain/trace.py` | `services/trace.py`, Trace |
| Certificate lines and issuability | `services/certificate.py` over records | Certificates |
| Provenance sentence | `domain/provenance.py` | Certificates, Trace |
| Number display | `domain/format.py` | every API response |
| Tolerances | `domain/policy.py` | checks, trace |
| Field lists and hints per document | `data/samples/*.json` | reader prompt, Capture |
| UI copy and check messages | `ui/frontend/src/locales/*.json` | every screen |

## Optional providers

| Provider | Purpose | Behavior when absent |
| --- | --- | --- |
| `AnthropicReader` | read a photographed sheet into text fields with per-field confidence | `NullReader`: health says unavailable, Capture offers prepared reading and typing |
| `FakeReader` | deterministic reads for tests and offline demos | not used in normal runs |

## History model

Append-only versioned rows. A correction inserts version n+1 with `supersedes` set. Reads take the latest version. Captures are kept with their raw reader values, edits and check results, so any saved value can be followed back to the photo and the person who confirmed it. This matters because a certificate or a recall decision rests on these records.

## Operations

- Seeding: `python scripts/seed.py` creates the schema and loads fixtures; `--reset` drops and reloads; `--report` prints counts.
- Migrations: none in v1. Schema is created from models. Alembic arrives with Postgres in a later ADR.
- Health check: `GET /api/health` returns status, reader availability with reason, and whether data is sample or plant.
