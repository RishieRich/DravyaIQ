# Contract: photo reader

The only place a model is used in v1. Lives behind `ui/backend/providers/reader.py`.

```python
class Reader(Protocol):
    id: str                                   # "<provider>:<model>", shown in provenance
    def available(self) -> tuple[bool, str]:  # (ok, reason) and never raises
    def read(self, image: bytes, mime: str, doc: DocSpec) -> ReaderResult
```

```text
ReaderResult
  fields   list[{key, value, confidence: "high"|"low"}]
  note     str
  error    null | "timeout" | "unavailable" | "bad_output" | "refused"
```

## Rules

1. The reader copies text. It never computes, totals, rounds, corrects spelling or fills a field it cannot see.
2. Output is validated against a schema. Unknown keys are dropped, missing keys become empty with confidence `low`, any value that is non-empty but came back without `"high"` becomes `low`. This is `providers/reader.py::normalize`, a port of the prototype's `normalize`.
3. A field is pre-confirmed only when its confidence is `high` and its value is non-empty. Everything else waits for a person.
4. A reader failure never blocks the journey. The capture opens in review with empty fields and a plain message.
5. The image and the raw model response are written to `data/uploads/` and `data/reader_log/` locally for audit. Neither is logged to stdout. Keys never appear in either.

## Prompt

Template in `agents/prompts/reader-prompt.md`, filled per document type from the field list and hints. Do not edit the rules section without adding a golden image case that shows why.

## Providers

| Setting | Behaviour |
| --- | --- |
| `MODEL_PROVIDER` unset | `NullReader`: `available()` returns `(False, "No reader configured")`. Live option hidden, prepared and manual work. |
| `MODEL_PROVIDER=anthropic` | `AnthropicReader` using the official SDK and a vision-capable model from `MODEL_ID`. Check current model ids in the provider's docs at build time. |
| `MODEL_PROVIDER=fake` | `FakeReader` returns the prepared reading for the doc type. Used by tests and offline demos. |

A second vendor is one more class implementing `Reader`. Nothing outside `providers/` imports a vendor SDK.

## Reading quality

`data/golden/reader/` holds photographed sheets with their true values when real ones exist. Until then, the five synthetic sample sheets are the set. `scripts/eval_reader.py` reports per-field exact match and the share of wrong values that were flagged low confidence. The number that matters is **wrong and marked high**, which must be zero on the set before any pilot.
