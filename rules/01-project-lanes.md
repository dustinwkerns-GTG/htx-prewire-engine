# Rule 01 — Project Lanes

Every project belongs to exactly one lane. The lane is set in `AI_RUN_PACKAGE_PreWire_v1.md` at intake (Phase 1 Step 4 in Cowork's intake workflow). The lane determines source priority, workbook tabs, reference blocks, QA/checklist behavior, and Definition of Done.

`mappings/lanes.yaml` is the machine-readable source of truth for lane IDs, tier labels, stage-aware control-file timing, and lane-determination precedence. This rule explains the engine behavior in human-readable form.

---

## Complexity Tiers

The lane model now carries an explicit `tier` field so Cowork's HTX Project Tiering Standard and the engine's lane rules stay reconciled:

| Tier | Name | Meaning | Typical lanes |
|---|---|---|---|
| 1 | Quick / light | Minimal scope, no plan-heavy workflow, single system/location, or small service/add-on. | `quick_quote`, `service_addon`, `bid_walk_pre_plan` |
| 2 | Standard | Normal residential prewire, lighting-control-only, or standard conference-room AV. | `full_prewire`, `lighting_control_only`, `conference_room` |
| 3 | Complex commercial | Commercial, multi-building, multi-phase, or commissioning-heavy work. | `complex_commercial` |

The tier is a complexity spine, not a replacement for project stages. The engine's `required_control_files_by_stage` model remains canonical for **when** each control file becomes required.

---

## The Seven Lanes

### Full Prewire (`tier: 2`)

Standard HTX prewire + AV with marked plans.

| Property | Value |
|---|---|
| Primary source | Marked HTX prewire drawings |
| Secondary sources | Builder/client notes → current calculator → full construction plan set → completed examples (reference only) |
| Workbook tabs | Pricing Calculator + Room Takeoff (primary), Takeoff Summary, Options-Alternates, Builder Scope, Review Needed, iPoint Entry, Scope Notes, Tech Handoff, Revision Tracker, Project Info, File Inventory, Relevant Sheet Review |
| Output structure | 12-section standard (see `rules/04-output-structure.md`) |
| Reference block | `full_prewire_standard` |
| Required control-file timing | `required_control_files_by_stage` in `mappings/lanes.yaml` |
| Definition of Done | 13 items (see COWORK.md § Definition of Done) |
| Typical project | Murphys Custom Homes / Welsby / 120 Woodmen Ct |

### Lighting Control Only (`tier: 2`)

Lutron (or similar) package only, no standard prewire. **Source priority is INVERTED.**

| Property | Value |
|---|---|
| Primary source | Client / architect device spreadsheet (typically Lutron) |
| Secondary sources | Builder/client notes → RCPs (spatial reference only) → prior HTX lighting projects (reference only) |
| Workbook tabs | Lighting Control Takeoff, Lighting Control Summary, Room Lighting Takeoff, Room Lighting Summary (primary). NOT Pricing Calculator. NOT Room Takeoff. |
| Output structure | 10-section variant (see COWORK.md § AI_RUN_PACKAGE Reference Blocks — Lighting Control Only variant) |
| Reference block | `lighting_control_only_variant` |
| Required control-file timing | `required_control_files_by_stage` in `mappings/lanes.yaml` |
| Definition of Done | 11 items (see COWORK.md) |
| Mandatory skill | `htx-spreadsheet-quirks-scan` runs on every intake |
| Typical project | Basich / 5210 Vickery |

### Quick Quote (`tier: 1`)

No construction plans; single system or single location, such as a TV mount, soundbar, service call, or one-off request. Minimal structure.

| Property | Value |
|---|---|
| Primary source | Conversation notes |
| Secondary sources | Change request or small-scope supporting details |
| Workbook tabs | Subset or none; Project Info / Scope Notes / Review Needed only when useful |
| Output structure | Minimal |
| Reference block | `quick_quote` |
| Required control-file timing | Stage model in `mappings/lanes.yaml`; `PROJECT_CONTEXT.md` only by default |
| Definition of Done | 3 items (see COWORK.md § Tier 1 Quick Quote DoD) |

`REVIEW_NEEDED.md` is generated only when something is genuinely unclear; do not expand a Quick Quote into the full plans pipeline by default.

### Service / Add-On (`tier: 1`)

Existing client, scoped change.

| Property | Value |
|---|---|
| Primary source | Prior project records + change request |
| Secondary sources | Updated drawings if provided |
| Workbook tabs | Subset of standard, starting from copy of prior project workbook when relevant |
| Output structure | Scoped to the change |
| Required control-file timing | Stage model in `mappings/lanes.yaml`; lighter minimum footprint than Full Prewire |
| Definition of Done | Project-specific |

### Bid Walk / Pre-Plan (`tier: 1`)

Relationship building before plans exist. Minimal output.

| Property | Value |
|---|---|
| Primary source | Conversation notes from Dustin |
| Secondary sources | Any reference materials (specs, photos, addresses) |
| Workbook tabs | Project Info + Scope Notes only |
| Output structure | Project Info + scope direction; no takeoff |
| Required control-file timing | Stage model in `mappings/lanes.yaml`; `PROJECT_CONTEXT.md` only unless promoted |
| Definition of Done | PROJECT_CONTEXT created, scope direction captured, follow-up actions logged |

### Conference Room / Commercial AV (`tier: 2`)

Specialized commercial conference-room AV. This lane is distinct from residential Full Prewire because it usually requires a separate equipment set: displays, video-conferencing codecs, DSP, room control, microphones, and ceiling arrays.

| Property | Value |
|---|---|
| Primary source | Client AV requirements |
| Secondary sources | Room drawings or walkthrough → prior conference-room projects |
| Workbook tabs | Project Info, File Inventory, Relevant Sheet Review, Scope Notes, Review Needed, Builder Scope, iPoint Entry, Tech Handoff, Revision Tracker |
| Output structure | Commercial AV |
| Reference block | `conference_room` |
| Required control-file timing | Uses the Full Prewire stage model until a commercial-AV-specific fixture justifies a narrower model |
| Escalation | Escalate to `complex_commercial` when multi-room, multi-building, phased, or commissioning-heavy |

### Complex / Commercial / Multi-phase (`tier: 3`)

Commercial, multi-building, phased, or multi-system work with commissioning. This lane inherits Full Prewire's source priority, workbook approach, and stage-aware control-file model, then adds phase structure and commissioning sign-off.

| Property | Value |
|---|---|
| Inherits | `full_prewire` |
| Adds | Commissioning and phase folders (Phase 1, Phase 2, …) |
| Output structure | 12-section standard plus commissioning and phases |
| Required control-file timing | `required_control_files_by_stage: { use: full_prewire }` |
| Definition of Done | Full Prewire DoD + commissioning sign-off + per-phase approval |

---

## Lane Determination Rules

The lane is set at intake based on these signals, in order of precedence:

1. **Explicit Dustin instruction** in the `[HTX PROJECT]` email body or dispatch text — explicit lane/tier always wins.
2. **Complex commercial signals** — commercial plus multi-building, multi-phase, commissioning-heavy, or comparable complexity resolves to `complex_commercial`.
3. **Conference-room signals** — conference-room AV requirements, displays, VC codecs, DSP, room control, or microphone arrays resolve to `conference_room` unless the complexity rule above escalates it.
4. **Quick Quote signals** — no construction plans plus single system/location resolves to `quick_quote` instead of defaulting upward.
5. **Email subject keywords** — "lighting only", "Lutron only", "service", "add-on", "bid walk", "pre-plan", "quick quote", "conference room", etc.
6. **Attachment types** — only a Lutron device spreadsheet + RCPs and no marked prewire plans suggests Lighting Control Only.
7. **Builder/client context** — established client with prior project records suggests Service / Add-On.
8. **Default** — if none of the above are clear, ask Dustin before proceeding. Do not default to Full Prewire just because it is most common.

When the lane is ambiguous, Cowork should:
- Create the project folder with the standard 12-subfolder structure (lane-independent)
- Create `PROJECT_CONTEXT.md` with **Lane: [PENDING — ask Dustin]** and **Tier: [PENDING — ask Dustin]**
- Flag in the next briefing: "New intake — lane/tier not auto-detected. Need confirmation."
- Wait for Dustin to set the lane before generating `AI_RUN_PACKAGE_PreWire_v1.md`

---

## Stage-Aware Control Files

Do not replace the engine's stage model with a parallel tier-to-file list. `required_control_files_by_stage` in `mappings/lanes.yaml` is canonical for when files become required:

- `intake`
- `engine_run`
- `review`
- `final_approved`

Tier 1 lanes may intentionally require fewer files at intake, but that is still represented through the stage model. Tier 2/3 lanes may inherit the Full Prewire stage model. `FINAL_SCOPE_NOTES.md` is not required until `final_approved` unless a lane-specific rule explicitly says otherwise.

---

## Lane-Specific Behavior Switches

The lane controls these behaviors. When in doubt about which behavior applies, consult `mappings/lanes.yaml`.

| Behavior | Full Prewire | Lighting Control Only | Quick Quote | Service / Add-On | Bid Walk | Conference Room | Complex Commercial |
|---|---|---|---|---|---|---|---|
| Spreadsheet quirks scan | Run if client device list present | Always run | Skip | Run if relevant to change | Skip | If client device list present | If client device list present |
| RCPs as spatial only | No | Yes | Skip | Per change | Skip | Yes | No |
| Wait for marked plans | Yes | No | No | Per change | Skip | No | Yes |
| Speaker pair counting | Yes | No | If relevant | If relevant | Skip | No | Yes |
| Workbook lighting tabs | Skip | Populate | Skip | Per change | Skip | Skip unless lighting in scope | Skip unless lighting in scope |
| Workbook prewire tabs | Populate | Skip | Skip | Per change | Skip | Commercial-AV output set | Populate / inherit Full Prewire |
| Maestro vs RA2 generation check | Skip unless lighting in scope | Mandatory | If lighting in scope | If relevant | Skip | If lighting in scope | If lighting in scope |
| Hub/repeater sizing fallback | Skip | Mandatory when system hardware unsupported | Skip | If relevant | Skip | Skip | Skip |
| Exterior entertainment as Optional | Yes | N/A | N/A | Per change | Skip | N/A | Yes |
| Address cross-check | Mandatory | Mandatory | Recommended | Recommended | Recommended | Mandatory | Mandatory |
| Contamination check | Mandatory | Mandatory | Recommended | Mandatory | Recommended | Mandatory | Mandatory |
| Commissioning sign-off | No | No | No | No | No | If required by scope | Mandatory |
| Phase scope tracking | No | No | No | No | No | If phased | Mandatory |

---

## Lane Switches Mid-Project

A lane can change mid-project (rare). When this happens:

1. Update `PROJECT_CONTEXT.md` to reflect the new lane, tier, and date of change.
2. Log the change in `AI_HANDOFF_LOG.md` with reason.
3. Add a Review Needed entry: "Lane changed from [X] to [Y] on [date]. Confirm any prior takeoff aligns with new lane rules."
4. Re-evaluate the workbook tabs in use — copy data to the correct tabs for the new lane if needed.
5. Notify Dustin in the next briefing.

Do not silently re-classify a project. Lane changes are a Dustin decision.

---

## Cross-references

- `mappings/lanes.yaml` — machine-readable lane definitions, tiers, stage-aware control files, and precedence
- `rules/02-lighting-control-only-lane.md` — full operating block for the Lighting Control Only lane
- `rules/09-lighting-control-full-prewire.md` — lighting control handling within the Full Prewire lane
- `rules/18-project-stages.md` — stage model for control-file timing
- `COWORK.md` (Dustin's machine) — Cowork-side operating model and Definition of Done
- `AGENTS.md` § 3 — repo-side lane summary
