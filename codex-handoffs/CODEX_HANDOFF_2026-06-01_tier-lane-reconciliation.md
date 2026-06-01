# Codex Handoff — Tier / Lane Reconciliation

---

## Handoff Summary

**Date:** 2026-06-01
**Submitted by:** Cowork
**Confidence:** Medium
**Dustin Approval Required Before PR:** Yes (structural change — adds a dimension + two lanes)

---

## Observed Issue

Not a bug — a deliberate model extension that needs reconciling with the engine. Cowork adopted the **HTX Project Tiering Standard v1.0** (approved 2026-06-01) which introduces a **complexity-tier** dimension (Tier 1 Quick Quote / Tier 2 Standard / Tier 3 Complex-Commercial) plus folder-level **system variants** (Prewire, Lighting/Shade, Theater, Security, Surveillance, Conference Rooms, Network).

The engine's `lanes.yaml` already overlaps most of this (full_prewire, lighting_control_only, service_addon, bid_walk_pre_plan) and has a **superior project-stage model** for when control files become required. But the two models aren't reconciled:
- Engine lanes carry **no explicit tier label**.
- There is **no Tier-3 (complex / commercial / multi-phase + commissioning) lane**.
- There is **no explicit quick_quote lane** distinct from service_addon / bid_walk.

Goal: one agreed model so Cowork's tiers and the engine's lanes are the same source of truth, with the engine's stage model remaining canonical for "when is a file required."

---

## Source Evidence

- `09 Internal Operations & SOPs\SOPs\HTX_Project_Tiering_Standard.md` (v1.0, approved 2026-06-01).
- COWORK.md tier-gating edits 2026-06-01 (creation workflow + 3 Definitions of Done now tier-aware).
- `mappings/lanes.yaml` (current — 4 lanes + project_stages + required_control_files_by_stage).
- Governance scanner false-positive fix 2026-06-01 (control files live in subfolders; 238 → 14).

---

## Affected Project or Signal

Rule category, not a single project. Affects every new-project intake + the governance control-file check across all active projects.

---

## Suspected Rule Files

- `mappings/lanes.yaml` — add `tier` to each lane; add two lanes.
- `rules/01-project-lanes.md` — lane definitions + the tier dimension + `lane_determination_precedence`.
- `rules/18-project-stages.md` — confirm the stage model stays canonical for required-file timing (likely no change, just cross-reference).
- `rules/04-output-structure.md` — output structure for the two new lanes.

---

## Proposed Rule Language

**1. Add a `tier` field to every lane** (complexity spine; system type stays a sub-variant on the Cowork folder side):
- `full_prewire` → `tier: 2`
- `lighting_control_only` → `tier: 2`
- `service_addon` → `tier: 1`
- `bid_walk_pre_plan` → `tier: 1`

**2. New lane `quick_quote` (tier 1):**
```yaml
  - id: quick_quote
    display_name: "Quick Quote"
    description: "No construction plans; single system or single location (TV mount, soundbar, service call, one-off). Minimal structure."
    tier: 1
    source_priority: [conversation_notes, change_request]
    output_structure: "minimal"
    required_control_files:
      - PROJECT_CONTEXT.md        # lite
    # REVIEW_NEEDED.md only if something is genuinely unclear
    definition_of_done_items: 3   # see COWORK.md § Tier 1 (Quick Quote) DoD
    workbook_tabs_primary: ["subset or none"]
```

**3. New lane `complex_commercial` (tier 3):**
```yaml
  - id: complex_commercial
    display_name: "Complex / Commercial / Multi-phase"
    description: "Commercial, multi-building, phased, or multi-system with commissioning (e.g., Trinnov theater). Full prewire base plus commissioning + phase structure."
    tier: 3
    inherits: full_prewire        # same prewire logic/source priority/workbook as full_prewire
    adds_structure: [Commissioning, "Phase folders (Phase 1, Phase 2, …)"]
    required_control_files_by_stage: { use: full_prewire }   # + per-phase / commissioning sign-off
    definition_of_done_items: "full_prewire DoD + commissioning sign-off + per-phase approval"
```

**4. Most system variants are Cowork-side folder structure, NOT engine lanes** — Theater / Security / Surveillance / Network sit *inside* full_prewire's logic and don't need separate lanes.

**4a. EXCEPTION — Conference Rooms get their own lane (Dustin confirmed 2026-06-01):** commercial AV conference rooms are specialized and typically require *separate equipment* (displays, codecs/video-conferencing, DSP, room control, mics/ceiling arrays) — distinct enough from residential prewire to warrant a dedicated lane, not a full_prewire sub-variant.
```yaml
  - id: conference_room
    display_name: "Conference Room / Commercial AV"
    description: "Specialized commercial conference-room AV — separate equipment set (displays, VC codecs, DSP, room control, mic arrays). Not residential prewire."
    tier: 2                      # escalates to tier 3 (complex_commercial) when multi-room / multi-building / phased
    source_priority: [client_av_requirements, room_drawings_or_walkthrough, prior_conference_room_projects]
    output_structure: "[UNKNOWN — Codex to define commercial-AV output set]"
    reference_block: "conference_room"   # new — Codex to author
    required_control_files_by_stage: { use: full_prewire }
    behavior_switches:
      speaker_pair_counting: false
      maestro_ra2_check: "if_lighting_in_scope"
      address_cross_check: "mandatory"
      contamination_check: "mandatory"
    workbook_naming_template: "[ClientName] - Conference Room AV - Rev[N] - [Date].xlsx"
```
Codex to fill the specifics (workbook tabs, scope language, reference block) — Cowork flagged the lane; the commercial-AV detail is Codex's to author.

**5. The engine's `required_control_files_by_stage` stays canonical** for when each control file is required. Cowork's governance scanner and COWORK.md will **defer to this stage model** rather than a parallel tier→file list.

**6. Extend `lane_determination_precedence`** so "no plans + single system/location" resolves to `quick_quote` (tier 1) instead of defaulting up, and "commercial / multi-phase / commissioning signals" resolves to `complex_commercial`.

---

## Test Idea

- Fixture A: intake email, no plans, "mount one TV" → detect `quick_quote` (tier 1); required files = `PROJECT_CONTEXT.md` only; no plans-pipeline folders.
- Fixture B: commercial multi-building + Trinnov commissioning → detect `complex_commercial` (tier 3); inherits full_prewire stage files + commissioning + phase folders.
- Fixture C: existing tagged full_prewire project at `review` stage missing `FINAL_SCOPE_NOTES.md` → NOT flagged (stage model: that file is only required at `final_approved`).

---

## Recommended Next Action

- [x] Review with Dustin before converting (structural change — adds a `tier` dimension + **three** new lanes: `quick_quote`, `complex_commercial`, `conference_room`). Conference Rooms confirmed as its own lane by Dustin 2026-06-01; no open judgment calls remain — Codex authors the commercial-AV specifics (workbook tabs, scope language, reference block).

---

## Advisory Notice

This handoff is an observation artifact. It does not change engine behavior until Codex converts it into a PR and Dustin reviews and merges it. Do not treat this file as an active rule. Cowork-side changes (COWORK.md, governance scanner) will be aligned to whatever lane/stage shape Dustin and Codex finalize here.
