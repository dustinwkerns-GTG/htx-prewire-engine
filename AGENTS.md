# HTX Prewire Engine — AGENTS.md

**Repository:** `htx-prewire-engine`
**Owner:** Dustin Kerns, HomeTroniX of Colorado
**Version:** 3.0 (v0.3.0 — adds project stages + stage-aware control file gating; lane backfill task)
**Last updated:** 2026-05-15

---

## 1. Purpose

This repository is the durable source of truth for HomeTroniX's prewire proposal engine — the rules, schemas, prompts, mappings, templates, fixtures, and tests that turn project intake into a reliable proposal support package.

The repo is consumed by three different agents, each playing a specific role:

- **Claude Cowork** runs on Dustin's desktop and is the primary runtime. It loads this repo as an installed plugin, executes the skills against real project files in OneDrive, populates the existing HTX workbook, and prepares the Dustin Review Brief.
- **Codex** (the OpenAI cloud coding agent) edits this repo. It takes corrections from Dustin, converts them into versioned rule updates, opens pull requests, and maintains the test suite.
- **Prewire Helper GPT** is the conversational backup. It receives periodic snapshots of the active rules but is not the production engine.

Dustin is final authority for all pricing, all client-facing communication, all final scope decisions, all plan markings, all Review Needed resolutions, and any merge of a behavior-changing pull request.

---

## 2. System Architecture

```
PROJECT INTAKE (OneDrive)
        ↓
Cowork loads this repo as a plugin
        ↓
Cowork applies the rules and skills here
        ↓
Cowork writes outputs into OneDrive 12-folder structure
Cowork populates the existing HTX_Prewire_Calculator_BLANK_MASTER workbook
Cowork drafts the Dustin Review Brief
        ↓
Dustin reviews / approves / corrects
        ↓
Corrections become PRs (Codex) or local rule edits
        ↓
Repo updates → Cowork reloads → improved engine
```

The OneDrive project structure remains unchanged. Production project files live at:
`C:\Users\dusti\OneDrive\Documentos\HomeTroniX of Colorado\01 Clients & Projects\Active\[Project Name]\`

The 12-subfolder structure (`00 Intake` through `11 Final Approved`) is the canonical filing system. Do not change it.

---

## 3. Project Lanes (READ FIRST)

Every project belongs to exactly one lane and every lane carries an explicit complexity `tier`. The lane is set in `AI_RUN_PACKAGE_PreWire_v1.md` at intake. The lane determines source priority, workbook tabs used, and which rules apply. If the lane/tier is missing, do not process — ask Dustin to set it.

| Lane | Tier | Primary Source | Workbook Tabs | Typical Output |
|---|---:|---|---|---|
| **Full Prewire** | 2 | Marked HTX prewire drawings | Pricing Calculator + Room Takeoff (+ standard support tabs) | Standard 12-section proposal support package |
| **Lighting Control Only** | 2 | Client/architect device spreadsheet (Lutron) | Lighting Control Takeoff, Lighting Control Summary, Room Lighting Takeoff, Room Lighting Summary | Lutron-focused scope; no standard prewire/AV |
| **Quick Quote** | 1 | Conversation notes / change request | Subset or none | Minimal one-off quote package |
| **Service / Add-On** | 1 | Prior project records + change request | Subset of standard tabs as relevant | Scoped change package |
| **Bid Walk / Pre-Plan** | 1 | Pre-plan notes only | Project Info + Scope Notes only | Minimal output for pre-plan estimation |
| **Conference Room / Commercial AV** | 2 | Client AV requirements + room drawings/walkthrough | Commercial-AV output set | Specialized conference-room AV package |
| **Complex / Commercial / Multi-phase** | 3 | Inherits Full Prewire; commercial/phase/commissioning signals | Full Prewire base + phase/commissioning structure | Complex commercial package with sign-off tracking |

**Critical inversion in Lighting Control Only:** the client/architect device spreadsheet is the primary source. RCPs are spatial reference only. HTX-marked plans are NOT required. This is the opposite of Full Prewire's source priority and must not be confused.

Detailed lane rules live in `rules/01-project-lanes.md` and `rules/02-lighting-control-only-lane.md`.

---

## 4. Source Priority by Lane

### Full Prewire

1. Marked prewire drawings — primary for device locations and quantities
2. Builder/client notes — primary for requested intent when drawings are incomplete
3. Current HTX calculator — primary structure for line items
4. Full construction plan set — context only (room names, elevations, coordination)
5. Completed calculators / prior proposals — reference only unless Dustin confirms current

### Lighting Control Only

1. Client/architect device spreadsheet (typically Lutron) — primary for devices, quantities, room placement
2. Builder/client notes — primary for requested intent and lane confirmation
3. RCPs and floorplans — spatial reference only; do not use as device source
4. Prior HTX lighting projects — reference only

### Service / Add-On

1. Prior project records — primary for existing scope
2. Change request — primary for new scope
3. Updated drawings if provided — supporting

### Bid Walk / Pre-Plan

1. Pre-plan notes from Dustin or sales — primary
2. Whatever plan materials are available — context only

**Conflict rule (all lanes):** if sources disagree, never silently choose. Flag in Review Needed with the specific conflict and ask Dustin.

---

## 5. Non-Negotiables

These rules apply to all lanes and override anything else:

1. Never invent quantities, pricing, scope, item codes, builder names, client names, or addresses.
2. Classify every file before doing any takeoff work. Use the file classification rule in `rules/03-file-classification.md`.
3. Run a contamination check on every intake. Flag any file that appears to belong to a different project as Critical in Review Needed.
4. Cross-check the project address across every available source — intake email, device spreadsheet header, plan title block, builder records. Any disagreement is Critical. (Address Sources Cross-Check is output section #4 — never skip it.)
5. Apply the source priority for the project's lane.
6. Use the project legend to interpret symbols. Never guess unknown symbols.
7. Every takeoff line includes a confidence label (High / Medium / Low) and a source reference.
8. Low-confidence items must appear in Review Needed.
9. Always separate confirmed base / assumptions / optional / alternate / future upgrade / Review Needed. Never merge categories.
10. Never include lighting control in Full Prewire base scope unless directly supported (marked, noted, in calculator, or Dustin-confirmed).
11. Never include equipment packages, final equipment, programming, commissioning, or training unless specifically requested.
12. Keep pricing, discount, and margin notes internal unless Dustin specifically requests client-facing pricing language.
13. Never send anything client-facing. Drafts only. Dustin sends.
14. Never move files to `11 Final Approved` without Dustin approval.
15. Never overwrite source files in `01 Plans - Raw`.
16. Every behavior-changing rule update is logged in `prompt_versions/Prompt_Update_Log.md` and requires Dustin approval before merge.
17. When in doubt, do less and flag the issue for Dustin.

---

## 6. Standard Output Structure

Every Full Prewire major output uses this 12-section structure (an evolution of the 11-section standard, adding Address Cross-Check as section #4):

1. **File Inventory** — classification of every uploaded file
2. **Relevant Sheet Review** — which sheets from the plan set matter, and which are missing
3. **Project Summary** — Project Lane, builder/client, address, scope intent
4. **Address Sources Cross-Check** — intake vs. spreadsheet vs. plan title block vs. builder records; any disagreement is Critical
5. **Confirmed Scope** — only directly supported items
6. **Assumptions** — clearly labeled, not written like confirmed scope
7. **Room-by-Room Takeoff** — every row includes level, room, system category, device/wire, qty, item code, base/option, notes, confidence, source
8. **Calculator-Ready Table** — aligned to the correct workbook tab structure for the lane
9. **Optional / Alternate Items** — never mixed with base
10. **iPoint Entry Notes** — organized by proposal section, base/option flagged
11. **Builder-Friendly Scope** — clean, coordination-focused, free of sales language
12. **Review Needed** — exceptions only, priority-labeled

For Lighting Control Only projects, sections 7–11 shift to use the Lutron-specific tabs and the Spreadsheet Quirks Checklist outputs.

Detailed section requirements live in `rules/04-output-structure.md`.

---

## 7. Confidence Labels

| Label | Meaning | Calculator Treatment |
|---|---|---|
| **High** | Directly shown on marked plans, notes, current calculator, or legend | Can be mapped as confirmed Base |
| **Medium** | Reasonably supported by plans or repeated project logic, not directly marked | Map as Assumption or Base-Review; note clearly |
| **Low** | Unclear, estimated, conflicting, or dependent on confirmation | Never mapped as confirmed Base; goes to Option, Assumption, or Review Needed |

Every Low-confidence item must appear in Review Needed. Every Estimated quantity must be labeled "Estimated" in notes.

---

## 8. Workbook Tabs by Lane

The canonical workbook is `HTX_Prewire_Calculator_BLANK_MASTER_REV6_Optimized_Benchmark_5.14.26.xlsx` (23 tabs). Cowork populates a copy of this workbook into the project folder — never modifies the master.

The workbook is formula-driven. Pricing Calculator pulls unit prices and descriptions from Legend Database via VLOOKUP, and pulls quantities from Room Takeoff via SUMIFS. Do not generate a parallel calculator sheet; populate the existing tabs and let the formulas do the rollup.

### Full Prewire writes to:
- Project Info, File Inventory, Relevant Sheet Review (always)
- **Room Takeoff** (primary takeoff destination — feeds Pricing Calculator via formulas)
- Takeoff Summary, Options - Alternates, Builder Scope, Review Needed, iPoint Entry, Scope Notes, Tech Handoff, Revision Tracker (as applicable)

### Lighting Control Only writes to:
- Project Info, File Inventory, Relevant Sheet Review (always)
- **Lighting Control Takeoff** (primary — device-level rows)
- **Lighting Control Summary** (rollup)
- **Room Lighting Takeoff** (room-by-room)
- **Room Lighting Summary** (room rollup)
- Options - Alternates, Builder Scope, Review Needed, iPoint Entry, Scope Notes (as applicable)
- Do NOT populate Pricing Calculator or Room Takeoff for this lane.

### Never populate:
- Discount Strategy (Dustin-only)
- Legend Database (reference, not project-specific)
- Lists, Benchmark Reference, Instruction Update (system tabs)

Detailed mapping lives in `mappings/workbook_tabs_by_lane.yaml`.

---

## 9. Item Code Reference

Item codes are defined in the workbook's Legend Database tab. The repo maintains a mirror at `mappings/item_codes.yaml` for offline reference and validation. Use only codes from the Legend Database — do not invent codes.

### Full Prewire codes (from Legend Database):

Network: `DATA_CAT6`, `DATA_CAT6A`, `WAP_CAT6`
Video: `TV_STD`, `TV_4K`
Audio: `SPK_PAIR_DA`, `SPK_BRACKET_PAIR`, `SURR_REMOTE`, `SURR_LOCAL`, `SUB_RG6`
Surveillance: `CAM_CAT6`, `VIDEO_DB`
Security: `SEC_SENSOR_22_4`, `SEC_CONTACT_22_2`, `SEC_KP_SIREN`
Shades: `SHADE_LUTRON`
Lighting: `LIGHT_KP_LOOP`
Conduit: `CONDUIT_SHORT`, `CONDUIT_LONG`, `CONDUIT_XLONG`

### Lighting Control Only codes (LC_* family):

`LC_KEYPAD_LOCATION`, `LC_DIMMER_LOCATION`, `LC_SWITCH_LOCATION`, `LC_3WAY_LOCATION`, `LC_4WAY_LOCATION`, `LC_3WAY_DIMMER_LOCATION`, `LC_DOOR_SWITCH`, `LC_MOTION_SENSOR`, `LC_LOAD_COUNT_PRIMARY`, `LC_SPECIALTY_LOAD`, `LC_EXT_SPECIALTY_LOAD_REVIEW`, `LC_EXT_HOME_LOAD_PRICED`, `LC_LANDSCAPE_SPECIALTY_HOLD`, `LC_PROGRAMMING_ALLOWANCE`, `LC_ENGRAVING_ALLOWANCE`

### Lutron device codes (from client spreadsheets):

Maestro family: `MA-S8AM-WH`, `MA-T51MN-WH`, `MA-T530G-WH`, `MACL-153-WH`, `MSCL-OP153M-WH`
RA2 Select family: `RRD-PRO-WH`, `RRD-2ANF-WH`, `RRD-8ANS-WH`, `RD-RD-WH`
System hardware: `L-REPPRO-BL`, `RR-SEL-REP2-BL`
Picos and faceplates: `CW-1-WH`, `CW-2-WH`, `CW-3-WH`, `CW-4-WH`, Pico P01/P02/etc.
OCC sensor default: `LRF2-OCR2B-P-WH` (confirm before order)

**Maestro / RA2 Select rule:** Maestro devices do not integrate with RA2 Select systems. If active rows mix generations, flag the generation question explicitly. Detail in `rules/10-lutron-generation-check.md`.

---

## 10. Standards Documents

The repo references the following HTX standards documents (canonical PDFs live in `references/standards/`):

| Document | Purpose | Rule File |
|---|---|---|
| HTX Prewire Workflow Automation Guide v2 | End-to-end workflow | `rules/07-prewire-logic.md` |
| HTX Prewire Standards v2 | System categories, room-type standards, category-specific includes/excludes | `rules/06-prewire-standards.md` |
| HTX Prewire Logic Document | Source priority, takeoff logic | `rules/07-prewire-logic.md` |
| HTX Standard Assumptions for Unmarked Projects v2 | Conservative defaults | `rules/08-unmarked-assumptions.md` |
| HTX Lighting Control System Info & Proposal Logic v1 | Full Prewire lighting control rules | `rules/09-lighting-control-full-prewire.md` |
| HTX Client Spreadsheet Quirks Checklist v1 | Ten patterns for Lighting Control Only intake | `rules/11-spreadsheet-quirks.md` |
| HTX Calculator Mapping Guide v2 | Plan → calculator mapping | `rules/12-calculator-mapping.md` |
| HTX Review Needed Checklist v2 | 26 category-specific Review Needed checklists | `rules/13-review-needed.md` (consolidated) |
| HTX Scope Writing Style Guide v2 | Builder-facing and client-facing language | `rules/14-scope-writing.md` |
| HTX Proposal Benchmark Library v1.2 | Proposal phrasing reference | `rules/15-proposal-benchmarks.md` |

If a project's `AI_RUN_PACKAGE` references a standard not in this index, ask Dustin to add it before proceeding.

---

## 11. Repo Structure

```
htx-prewire-engine/                       # v0.2.0 state
├── AGENTS.md                              # This file
├── CODEX.md                               # Codex integration guide
├── README.md                              # Repo overview
├── .gitignore
├── .claude-plugin/                        # Cowork plugin manifest
│   └── plugin.json
├── .github/                               # GitHub workflow templates
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── ISSUE_TEMPLATE/
│       ├── correction.md
│       └── feature-request.md
├── prompt_versions/
│   └── Prompt_Update_Log.md               # Every behavior change logged here
├── rules/                                 # 19 modular rule files
│   ├── 00-non-negotiables.md
│   ├── 01-project-lanes.md
│   ├── 02-lighting-control-only-lane.md
│   ├── 03-file-classification.md
│   ├── 04-output-structure.md
│   ├── 05-cowork-integration.md
│   ├── 06-prewire-standards.md
│   ├── 07-prewire-logic.md
│   ├── 08-unmarked-assumptions.md
│   ├── 09-lighting-control-full-prewire.md
│   ├── 10-lutron-generation-check.md
│   ├── 11-spreadsheet-quirks.md
│   ├── 12-calculator-mapping.md
│   ├── 13-review-needed.md               # 26 category-specific Review triggers (consolidated)
│   ├── 14-scope-writing.md
│   ├── 15-proposal-benchmarks.md
│   ├── 16-address-cross-check.md
│   ├── 17-contamination-check.md
│   ├── 18-project-stages.md
│   └── 99-corrections-workflow.md
├── skills/                                # Cowork-callable workflows
│   └── htx-spreadsheet-quirks-scan/       # v0.1.0 — smoke-tested
│       ├── SKILL.md
│       └── references/
│           └── ten-patterns.md
├── mappings/
│   └── lanes.yaml                         # Canonical lane definitions (shared with Cowork)
├── schemas/                               # (planned v0.3.0+)
├── templates/                             # (planned v0.3.0+)
├── scripts/                               # (planned v0.3.0+)
├── tests/                                 # (planned v0.3.0+)
├── references/                            # (PDFs uploaded as needed)
└── sample_projects/                       # (planned v0.3.0+)
    ├── welsby/                            # Full Prewire fixture
    └── basich/                            # Lighting Control Only fixture
```

---

## 12. Correction Workflow

When Dustin corrects an output, follow `rules/99-corrections-workflow.md`. The short version:

1. Capture the exact correction (Cowork drafts; Dustin confirms wording).
2. Identify the rule(s) it would affect.
3. Draft the proposed rule update in the correct file under `rules/`.
4. Add or update a regression test in `tests/` that would have caught the original mistake.
5. Update `prompt_versions/Prompt_Update_Log.md` with date, summary, files changed, tests added, Dustin approval status.
6. Open a pull request. Dustin reviews and merges.
7. After merge, Cowork reloads the plugin and the new rule is active.
8. The next periodic GPT knowledge snapshot will carry the change to the GPT.

**Do not apply behavior-changing rules locally without going through the PR process.** Drift between local and repo is the failure mode this workflow exists to prevent.

---

## 13. Version Control Discipline

- Every prompt or rule change that affects behavior is logged in `prompt_versions/Prompt_Update_Log.md`.
- Every rule change ships with at least one test.
- Pull requests describe what changed, why, and which projects motivated the change.
- The `active/Master_Prompt_Current.md` file is the canonical GPT system prompt; archive prior versions before overwriting.
- Tag releases when meaningful improvements ship (e.g., `v1.1-lighting-only-stable`).

---

## 14. Safety and Approval Boundaries

Cowork and Codex may:

- Read all project files.
- Create drafts in the OneDrive project folder.
- Populate workbook tabs in a project copy of the blank master.
- Draft the Dustin Review Brief.
- Open pull requests against this repo.
- Draft emails (never send).
- Capture corrections and propose rule updates.

Cowork and Codex must not:

- Send any client-facing communication.
- Move files to `11 Final Approved`.
- Modify files in `01 Plans - Raw`.
- Approve final pricing.
- Approve final scope.
- Merge a behavior-changing PR without Dustin's review.
- Apply a rule change that hasn't been logged in `Prompt_Update_Log.md`.
- Make assumptions about Lutron generation, hub/repeater counts, or controlled loads without flagging in Review Needed.

When in doubt, do less and flag the issue for Dustin.

---

## 15. Cowork Integration (v0.2.0)

The canonical Cowork operating doc lives at `C:\Users\dusti\OneDrive\Documentos\HomeTroniX of Colorado\COWORK.md` on Dustin's machine. The full integration contract — what Cowork does vs what the engine does — lives in `rules/05-cowork-integration.md`. The short version:

**Cowork owns the process layer:**
- `[HTX PROJECT]` email trigger detection
- Active project Watch Terms monitoring (Trigger 2)
- Intake queue fallback (Trigger 3)
- Dispatch text trigger (Trigger 4, future)
- Project folder creation (12 subfolders)
- The seven control file generation
- File intake to `01 Plans - Raw\`
- Filing to `07 Correspondence\`, `11 Final Approved\`, etc.
- Granola, Notion, Asana, iPoint, Visio integrations
- Notification to Dustin / morning briefings
- Weekly OneDrive review

**The engine (this repo) owns the intelligence layer:**
- Rule files in `rules/`
- Skills in `skills/`
- Machine-readable mappings in `mappings/`
- Standards interpretation, item codes, scope language standards
- Validation logic (classification, contamination, address cross-check, Lutron generation, quirks scan)

**Shared source of truth:**
- `mappings/lanes.yaml` — canonical lane definitions, read by both systems

**Skill invocation by Cowork:**
- When intake = Lighting Control Only → Cowork invokes `htx-spreadsheet-quirks-scan`
- (Planned) When file intake completes → Cowork invokes `htx-file-classification`, `htx-contamination-check`, `htx-address-cross-check`
- (Planned) When lane = Lighting Control Only → Cowork invokes `htx-lutron-generation-check`
- (Planned) When GPT output lands → Cowork invokes `htx-output-qa`

The trigger system (`[HTX PROJECT]`, watch terms, etc.) is fully Cowork-side. The engine layers in at the points Cowork's intake workflow calls for specialist intelligence.

---

## 16. Codex Integration (v0.2.1 — active as of 2026-05-15)

Codex is the repo editor for htx-prewire-engine. See `CODEX.md` for full setup, conventions, and workflow.

**Current state (v0.2.1+):**
- Codex is connected to `dustinwkerns-GTG/htx-prewire-engine` via the ChatGPT Codex Connector.
- Smoke test passed 2026-05-15 (Codex read AGENTS.md + CODEX.md + rules, summarized lanes and guardrails correctly; 58-second execution time).
- Codex handles corrections per `rules/99-corrections-workflow.md` — opens PRs, logs to `prompt_versions/Prompt_Update_Log.md`, runs the monthly audit PR.
- Dustin reviews and merges all PRs. No auto-merge.

**Repo location:** `C:\Users\dusti\OneDrive\Documentos\HomeTroniX of Colorado\HTX Prewire Engine\`

**GitHub:** `https://github.com/dustinwkerns-GTG/htx-prewire-engine` (private)

---

## 17. Governance and Continuity

The engine stays healthy through three governance cadences and explicit continuity plans. See [`GOVERNANCE.md`](./GOVERNANCE.md) for the full discipline.

**Quick summary:**

- **Weekly** — Cowork runs an engine health check Friday 4:30 PM (open PRs, recent rule changes, drift between `COWORK.md` and the repo, control-file completeness on active projects). Surfaces findings in Monday morning briefing.
- **Monthly** — Cowork runs a rule library audit on the 1st of each month (stale rules, retirement candidates, corrections-to-rules summary). Produces a report as a PR.
- **Quarterly** — Cowork schedules a 30-min architecture review for Dustin (lane health, structural changes, integration quality with Cowork and the GPT).

**Drift detection** focuses on three boundaries:
1. Repo ↔ `COWORK.md` — canonical bridge is `mappings/lanes.yaml`.
2. Repo ↔ Prewire Helper GPT — flattened knowledge snapshot at intervals.
3. Repo ↔ workbook Legend Database — `mappings/item_codes.yaml` mirror (planned v0.3.0).

**Continuity plans** cover short-term absence (Cowork keeps running on schedule), medium-term (delegate handles routine intake), long-term (documented handoff), and recovery (GitHub + local clone + OneDrive copy = three copies of the rule library).

**Rule retirement** is allowed but conservative — see GOVERNANCE.md § 4. Stable rules that just work are the most valuable rules; do not retire them merely because they're old.

---

## 18. Reading Order for New Contributors

If you (a future contributor, including a future Codex session) are picking up this repo:

1. Read this file (AGENTS.md) end-to-end.
2. Read `CODEX.md` if you are Codex or representing Codex's workflow.
3. Read `GOVERNANCE.md` to understand the health-of-the-system discipline.
4. Read `rules/01-project-lanes.md` next.
5. Read `rules/00-non-negotiables.md`.
6. Read `rules/05-cowork-integration.md` to understand the runtime / repo split.
7. Read the `references/standards/` PDFs as needed by your specific task.
8. Look at sample projects in `sample_projects/` when they exist (planned v0.3.0+).
9. Check `prompt_versions/Prompt_Update_Log.md` to understand what has changed recently.
10. Run the test suite (`pytest tests/`) when it exists (planned v0.3.0+).

---

*This file is the operating agreement. Detailed implementation lives in `rules/`. Edit AGENTS.md when the operating model changes; edit `rules/*.md` when specific behaviors change.*
