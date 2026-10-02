# Dravya IQ constitution

Governing rules for this repository. Every rule below is adopted, which means it has one line of proof that can be checked. A rule that cannot be proven is deleted, not left as a promise.

Created 2026-10-02. Baseline: ARQ engineering constitution. Rule VIII (dependencies proven early) is not adopted: v1 has one optional dependency, the reader, and slice 04 proves it.

## I. The repository is the durable artifact

Lasting behavior, decisions, setup and continuation state live here, not in chat.

Proof: `specs/` holds the brief, slices and contracts; `specs/decisions/` holds ADRs; `HANDOFF.md` holds continuation state and is updated at the end of every slice.

## II. One fact has one authority

Every figure, status, permission and derived value has one canonical source and one computation path. UI and models display facts, they do not recreate them.

Proof: kilograms, percentages, gaps, adherence and blast radius are computed only in `ui/backend/domain/` per the table in `specs/contracts/data-model.md`; the API returns display strings; slice 03 criterion 4 greps the frontend for arithmetic on record numbers.

## III. Evidence travels with consequential output

A value that ends up on a certificate or a trace carries its source.

Proof: every record row has a `source` per `specs/contracts/data-model.md`; `domain/provenance.py::describe` is the only sentence builder; golden case `certificate-after-capture` asserts the ΔE line's source.

## IV. Unsupported knowledge is explicit

Missing data, unsupported questions, disconnected integrations and unimplemented features are labelled honestly. Nothing is estimated silently.

Proof: certificate lines are `null` when unrecorded and render "not recorded" with `issuable: false` (golden case `certificate-no-qc`); `/api/health` reports `reader: "unavailable"` with a reason; the "Sample data" pill shows while fixtures are loaded.

## V. The deterministic core works without optional services

Proof: `NullReader` is the default when `MODEL_PROVIDER` is unset; brief criterion 8 runs the whole journey with no key; `ui/backend/domain/` imports no framework, database or vendor SDK (slice 01 criterion 3).

## VI. Validation happens before commitment

Proof: `domain/gate.py::can_save` blocks on any error or unconfirmed field; `POST /api/captures/{id}/commit` re-runs checks server-side and returns 409 without writing; commit is idempotent on capture id (slice 02 criterion 4).

## VII. Changes preserve history when history matters

Proof: repositories insert a new version with `supersedes` instead of updating rows; `SHADE_REPLACES` and `QC_REPLACES` warnings tell the user the old record is kept; a repository test asserts row count grows on correction.

## IX. Generated or probabilistic output is guarded

The model reads photos into text. It never computes, and nothing it returns is saved without a person and the checks.

Proof: `providers/reader.py::normalize` validates the schema and downgrades anything not explicitly high confidence to low; every saved value passes `run_checks` and the gate; `scripts/eval_reader.py` fails the slice on any wrong value marked high.

## X. Security and privacy are structural

Proof: `.env` is gitignored and `.env.example` holds names only; API errors use `{error: {code, message}}` with no provider bodies; slice 04 criterion 4 greps logs and `data/` for the key; tests run on a temporary SQLite file, never `data/dravya.db`.

## XI. Design serves comprehension

Proof: colours come only from `ui/frontend/src/styles/theme.css` (checked by `verify.py`); status always pairs an icon shape and text with colour; `specs/contracts/screens.md` defines empty, loading and error states for every screen; Capture is checked at 360 px and by keyboard only.

## XII. Completion is demonstrated

A slice is complete only when its acceptance criteria pass, the primary journey works, and known limits are stated.

Proof: each slice spec has a Verification table filled with the command actually run and its result; `HANDOFF.md` lists only checks that were run.

## Conflicts

Where a rule conflicts with a product requirement, the conflict and its resolution are recorded in `specs/decisions/`.
