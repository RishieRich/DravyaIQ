# Dravya IQ

The material record for masterbatch plants: photograph the five paper sheets the floor already fills, confirm what the reader was unsure of, and get a checked, traceable record that answers complaints, builds certificates and shows where every kilogram went. Part of the ARQ ONE MSME operating system.

Status: v1 spec ready, code not started. Theme: Astra ink and orange. Stack: React + Vite + TypeScript, FastAPI, SQLite. Started 2026-10-02.

## What is real

- Working: nothing yet. The target is the primary journey in `specs/000-product-brief.md`: photograph a batch log, confirm, save, and see it in shade history, trace and certificate.
- Sample data: invented fixtures in `data/fixtures/` (7 lots, 4 shades, 6 batches, 6 dispatches) and five sample sheets in `data/samples/`. No real company's data.
- Not built: see the non-goals in `specs/000-product-brief.md` and the roadmap in `specs/005-roadmap-intelligence.md`.

## Build order

| Slice | Spec | Outcome |
| --- | --- | --- |
| 01 | `specs/001-deterministic-core.md` | every rule and total tested in pure Python against golden cases |
| 02 | `specs/002-capture-journey.md` | capture, confirm, check and save works end to end on prepared readings |
| 03 | `specs/003-record-views.md` | trace, certificates, shades and material answer questions from the record |
| 04 | `specs/004-live-reader-and-languages.md` | live photo reading and Hindi, Gujarati, Marathi on the capture screen |

## Run it

```bash
# setup
cd ui/frontend && npm install && cd ../backend && pip install -r requirements.txt && cd ../..

# seed deterministic fixture data
python scripts/seed.py --reset

# dev, two terminals
uvicorn api.main:app --reload --app-dir ui/backend
npm --prefix ui/frontend run dev

# test
pytest ui/backend/tests -q && npm --prefix ui/frontend run test && npm --prefix ui/frontend run build
```

Configuration: copy `.env.example` to `.env`. Startup fails loudly and names any missing required key. The reader keys are optional.

## Layout

| Path | Holds |
| --- | --- |
| `constitution.md` | governing rules, each with one proof line |
| `specs/` | product brief, slice specs, contracts, decision records |
| `specs/contracts/` | data model, checks, views, API, reader, screens: the build authority |
| `agents/` | instructions for coding agents, `AGENTS.md` is authoritative; `prompts/` holds the reader prompt |
| `ui/frontend/` | app, screens, components, locales, theme tokens |
| `ui/backend/` | api, services, domain, repositories, providers, tests |
| `data/` | fixtures, sample sheets, golden test cases |
| `scripts/` | seed, reader evaluation |
| `docs/` | architecture, glossary, the original prototype for reference |

## Continuing this work

Read `HANDOFF.md` first. It states what is verified, what is not built, and the next acceptance criterion.
