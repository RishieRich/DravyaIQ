# Kickoff prompt for Claude Code

Paste this as the first message in Claude Code, opened at the repository root.

```text
You are building Dravya IQ v1 from the specs in this repo. Start by reading agents/AGENTS.md and follow it as your standing instructions (it is in agents/, not the root, so read it explicitly). Then read constitution.md, specs/000-product-brief.md, every file in specs/contracts/, and HANDOFF.md.

Build slice 01 only: specs/001-deterministic-core.md. Pure Python in ui/backend/domain/, Decimal for every weight, no framework or vendor imports, tests driven by data/golden/checks.json and the material and trace cases in data/golden/views.json. Do not edit any golden case; if one disagrees with a contract, stop and tell me which and why.

When every slice 01 acceptance criterion passes, fill the Verification table in the slice spec with the commands you ran and their real output, update HANDOFF.md, and stop. Report as Delivered, Verified, Not built, Next.
```

For later slices, replace the slice paragraph with the next spec path, for example `specs/002-capture-journey.md`, and keep the rest.
