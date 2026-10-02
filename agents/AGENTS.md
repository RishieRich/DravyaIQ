# Agent instructions: Dravya IQ

Authoritative instructions for any coding agent working in this repository. Other agent files point here rather than restating this.

## Read first

1. `constitution.md` at the root. Every rule listed there is adopted and provable.
2. `specs/000-product-brief.md` for the outcome, the primary journey, invariants and non-goals.
3. `HANDOFF.md` for what is verified and what is next.
4. `specs/contracts/` before writing any rule, route or screen. These are the build authority.
5. `specs/decisions/` before changing anything structural.
6. `docs/glossary.md` if a shop-floor word is unclear. `docs/reference/shade-ledger-prototype.htm` shows the intended behavior and look; contracts win where they differ.

## Non-negotiables

- One fact, one authority. No number recomputed in a component.
- Deterministic code owns facts, totals, validation, permissions and state transitions. Models narrate or classify within supplied evidence.
- Missing data, disconnected integrations and unimplemented features are labelled, never estimated.
- The primary journey works with optional providers switched off.
- Secrets never enter source control, logs, traces, client URLs or error responses.
- Colors come from the token layer in `ui/frontend/src/styles/theme.css`. A hex value in a component is a bug.
- Accessibility is structural: visible focus, semantic labels, reduced motion, readable contrast, no meaning by color alone.
- Completion is demonstrated. Never report a check that was not run.
- The model reads photos into text. It never computes, rounds or fills a value. Every weight is `Decimal` in `ui/backend/domain/`.
- Golden files in `data/golden/` are fixed. If code and a golden case disagree, report it; do not edit the case.
- Records are append-only. Corrections are new versions.

## Boundaries

```text
ui / routes / cli  →  services  →  domain  →  repositories and providers
```

Entrypoints parse, authorize, call one use case, serialize. Domain code imports no framework and no vendor SDK. Repositories are the only ordinary path to durable data of their type.

## Working rules

- Make the smallest coherent change that satisfies the outcome.
- Inspect the working tree before editing and preserve unrelated changes.
- Reproduce a defect or identify the failing invariant before changing code.
- Run focused tests during work, then the repository gates before handoff.
- Update `HANDOFF.md` when continuation state changed. Do not write a second description of the codebase.

## Theme

This project uses the Astra ink and orange ARQ visual system. Tokens live in `ui/frontend/src/styles/theme.css`. Requested color changes are made in the token layer, then every derived state is checked: hover, focus, selected, disabled, light, dark, chart series, status colors.

## Commands

```bash
cd ui/frontend && npm install && cd ../backend && pip install -r requirements.txt
npm --prefix ui/frontend run dev  # and: uvicorn api.main:app --reload --app-dir ui/backend
pytest ui/backend/tests && npm --prefix ui/frontend run build
python scripts/seed.py
```

## Slice discipline

- Build slices in order: 01, 02, 03, 04. Do not start a slice while the previous one has a failing acceptance criterion.
- At the end of each slice: fill its Verification table with the commands actually run and their results, update `HANDOFF.md`, and stop for review.
- Anything a slice needs that its spec does not say: choose the smallest option, record it under the slice's Notes, and carry on.

## Report format

Delivered, then Verified, then Not built, then Next. Lead with the outcome. State deviations once.
