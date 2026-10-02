# 005: roadmap, not v1

Status: not planned for v1. Kept here so the v1 data model does not block it.

These were discussed for Dravya IQ and are deliberately out of v1. Each one reads the record built by slices 01 to 04. None of them computes a number itself.

| Capability | What it needs from v1 | Guard |
| --- | --- | --- |
| Owner KPI board: unaccounted kg per month, purge % by line, first-time-right shades, complaint-to-cause time, certificates issued without retyping | versioned records with dates, `plant_totals` by period | numbers from `domain/`, never from a model |
| Ask the record (chat): "which batches used PB153-2408", "show Sagar's last five ΔE readings" | a read-only query service over repositories | the model picks a query and narrates the result, every figure cited to a record id |
| Learning: recipes that drift, suppliers whose lots show up in complaints, lines with rising purge | history and complaint links | suggestions labelled as patterns, with the batches behind them |
| Alerts on WhatsApp | check results and gate events | outbound only, opt-in |
| Tally and ERP join | batch and dispatch ids stable and exportable | read-only import first |
| Yantra IQ join | batch id and line shift on every batch | machine data shown beside material data, not merged into it |

Revisit after the first plant has used v1 for four weeks.
