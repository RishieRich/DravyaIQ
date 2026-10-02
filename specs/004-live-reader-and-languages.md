# Slice 04: live photo reading and shop-floor languages

Part of: `specs/000-product-brief.md`. Status: planned. Needs: slice 03 verified.

## Outcome

A supervisor photographs a real sheet with a phone browser, the reader fills the fields with honest confidence, and the capture screen speaks Hindi, Gujarati or Marathi.

## In scope

- `AnthropicReader` behind `providers/reader.py`, configured by `MODEL_PROVIDER`, `MODEL_ID`, `MODEL_API_KEY`
- Prompt from `agents/prompts/reader-prompt.md`, output schema validation and `normalize`
- `scripts/eval_reader.py` over the five sample sheets rendered to PNG, reporting exact match and "wrong but marked high"
- Locale files `en.json`, `hi.json`, `gu.json`, `mr.json` under `ui/frontend/src/locales/`, covering Capture screen labels, document names, role names, field labels and every check message id. Hindi ported from the prototype. Gujarati and Marathi drafted, each file carrying `"_status": "needs_native_review"`
- Language switch on Capture, remembered per device
- Mobile layout for Capture at 360 px width, camera input via `accept="image/*" capture="environment"`

## Out of scope

- Office screens in regional languages
- Offline queue, handwriting training, any second vendor

## Acceptance criteria

1. Brief criteria 8 and 9 pass.
2. With a valid key, reading the sample batch log PNG returns the misread lot `PB153-2480` or flags it low. A wrong value marked high on any of the five sample sheets fails the slice.
3. Pulling the network mid-read shows a plain message within 60 s and leaves the capture editable.
4. The key appears in no log line, no response body and no file under `data/`. Checked by grepping for the key's first 8 characters after a full run.

## Verification

| Criterion | How it was checked | Result |
| --- | --- | --- |
| 2 | `python scripts/eval_reader.py` | not run yet |
| 3 | by hand, network off | not run yet |
| 4 | `grep -rF "$KEY_PREFIX" data/ logs/` | not run yet |

## Notes

Check the current vision-capable model id and SDK call shape in the provider's documentation at build time. Do not write either from memory.
