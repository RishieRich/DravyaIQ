# ADR 0001: stack and hosting

Date 2026-10-02. Status: accepted.

## Context

v1 runs on one laptop for demos and a first pilot at one plant. It needs exact decimal arithmetic, a small relational record with history, image upload, one optional vision model, and a UI that works on a shop tablet and a phone browser.

## Decision

- Frontend: React 18, Vite, TypeScript, plain CSS with the ARQ token layer. React Router for the five screens. Vitest and Testing Library for component tests. No UI kit.
- Backend: Python 3.12, FastAPI, Pydantic v2, SQLAlchemy 2. `decimal.Decimal` for every weight.
- Storage: SQLite file `data/dravya.db` behind repositories. Fixtures in `data/fixtures/` loaded by `scripts/seed.py`.
- Model provider: one `Reader` interface. Primary `AnthropicReader` (vision-capable model, id from env, verified against provider docs at build time). Test and offline `FakeReader`. With neither configured, `NullReader`: live reading hidden, prepared reading and typing work.
- Hosting: none in v1. Local only, two processes: Vite dev server on 5173 proxying `/api` to Uvicorn on 8000.

## Constraints this creates

- Request payload ceiling: image upload 8 MB, server downscales to 2000 px longest side before the reader.
- Request timeout ceiling: reader call 60 s, then the capture opens for manual entry.
- Region and latency: reader calls leave the laptop for the provider's API; with no network the journey still works on prepared readings and typing.
- Other: SQLite means one writer at a time, which is fine for one plant and one supervisor. Images are stored on local disk under `data/uploads/`, gitignored.

## Alternatives considered

| Option | Why not |
| --- | --- |
| PostgreSQL from day one | Adds a service to install for a laptop demo. The repository layer makes the switch a later, contained change. |
| Single-file HTML like the prototype | No durable record, no server-side gate, no history. Fine for the pitch, not for a pilot. |
| Next.js full stack | Splits rules between server components and API routes and pulls the domain into TypeScript. Python keeps the domain testable and shared with future Tally and Yantra IQ work. |
| OCR engine plus rules instead of a vision model | Handwritten mixed-script sheets with strike-throughs read poorly on classic OCR. The model is only a reader behind a guard, so it can be swapped. |

## Revisit when

A second plant or a second concurrent user goes live, images need to sync from phones without the laptop, or the plant asks for hosted access. Then: Postgres, object storage, real auth, in a new ADR.
