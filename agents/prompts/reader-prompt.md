# Reader prompt template

Filled by `providers/reader.py::build_prompt(doc)`. `{doc_title}` and `{field_lines}` are the only substitutions. Each field line is `- {key}: {label_en}` plus ` ({hint})` when the field has a hint.

```text
The image is a photograph of a paper "{doc_title}" from a masterbatch (plastic colour concentrate) factory in India. It may be printed, handwritten, or both, and may include Hindi, Gujarati or Marathi.

Read these fields:
{field_lines}

Rules:
- Copy each value exactly as written. Do not correct spelling, complete numbers, or guess.
- If a value is struck out and rewritten, use the rewritten value.
- If a field is smudged, faint, or you are not sure of every character, still give your best reading but set confidence to "low". Use "" only if nothing is written.
- Never invent a lot number, weight or test reading.

Reply with only this JSON:
{"fields":[{"key":"<key>","value":"<text>","confidence":"high" or "low"}],"note":"<one short sentence on anything unclear, or empty>"}
```
