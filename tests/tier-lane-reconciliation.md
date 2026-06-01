# Test — Tier / Lane Reconciliation

**Date:** 2026-06-01
**Origin:** Codex handoff `codex-handoffs/CODEX_HANDOFF_2026-06-01_tier-lane-reconciliation.md`
**Rules:** `rules/01-project-lanes.md`, `mappings/lanes.yaml`

## Fixture A — Quick Quote

### Input

Intake note says: "Can you quote mounting one TV and a soundbar in the basement? No plans yet."

### Expected output

- Lane: `quick_quote`
- Tier: `1`
- Required control files at intake: `PROJECT_CONTEXT.md`
- No plan-heavy Full Prewire pipeline is required.
- `REVIEW_NEEDED.md` is generated only if the request is genuinely unclear.

### Anti-output

- Do not default to `full_prewire` because the client is a normal builder/client.
- Do not require all seven control files at intake.

## Fixture B — Complex Commercial

### Input

Intake note says: "Commercial multi-building AV/prewire project, split into Phase 1 and Phase 2, with Trinnov theater commissioning and final commissioning sign-off."

### Expected output

- Lane: `complex_commercial`
- Tier: `3`
- Inherits: `full_prewire`
- Required control-file timing uses `required_control_files_by_stage: { use: full_prewire }`.
- Adds phase structure and commissioning sign-off.

### Anti-output

- Do not model this as standard Tier 2 `full_prewire` with no phase/commissioning behavior.
- Do not replace `required_control_files_by_stage` with a flat tier-to-file list.

## Fixture C — Stage-Aware Final Scope Notes

### Input

Existing tagged `full_prewire` project at `review` stage is missing `FINAL_SCOPE_NOTES.md`.

### Expected output

- Lane: `full_prewire`
- Tier: `2`
- `FINAL_SCOPE_NOTES.md` is **not** flagged missing at `review` stage.
- `FINAL_SCOPE_NOTES.md` becomes required only at `final_approved` per `required_control_files_by_stage`.

### Anti-output

- Do not flag `FINAL_SCOPE_NOTES.md` missing before `final_approved`.
- Do not treat Tier 2 as "all seven files always required."

## Fixture D — Conference Room / Commercial AV

### Input

Intake note says: "Conference room AV: displays, Teams codec, DSP, room control, ceiling mic array. One room only; no multi-building phase structure."

### Expected output

- Lane: `conference_room`
- Tier: `2`
- Uses commercial-AV equipment categories.
- Does not use residential speaker-pair counting by default.
- Escalates to `complex_commercial` only if multi-room, multi-building, phased, or commissioning-heavy signals are present.

### Anti-output

- Do not classify as residential `full_prewire` solely because AV equipment is in scope.
- Do not escalate to Tier 3 without complexity signals.
