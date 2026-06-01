# Prompt Update Log

Every prompt or rule change that affects behavior is logged here. See `AGENTS.md` § 12 (Correction Workflow) and § 13 (Version Control Discipline) for the full process.

Format per entry:
- Date
- Version
- Summary
- Reason
- Files changed
- Affected workflow / lane
- Tests added or updated
- Dustin approval status

---

## 2026-06-01 — v0.3.5 — Tier/lane reconciliation

**Summary:** Reconciles the HTX Project Tiering Standard v1.0 with the engine lane model by adding an explicit `tier` field to every lane, introducing `quick_quote`, `conference_room`, and `complex_commercial`, and extending lane-determination precedence for quick-quote and commercial complexity signals.

**Reason:** Cowork adopted the approved Tier 1 / Tier 2 / Tier 3 complexity model, while the engine's `mappings/lanes.yaml` still had four lanes with no explicit tier label. The models now share one source of truth while preserving the engine's stage-aware control-file timing model.

**Files changed:**
- `mappings/lanes.yaml` — bumped schema to 1.2, added tiers to existing lanes, added `quick_quote`, `conference_room`, and `complex_commercial`, and updated `lane_determination_precedence`.
- `rules/01-project-lanes.md` — documented the tier spine, seven lanes, quick-quote/commercial precedence, and the requirement that `required_control_files_by_stage` remains canonical.
- `rules/99-corrections-workflow.md` — added to the canonical GitHub branch from the engine working tree so this PR can follow the documented workflow.
- `AGENTS.md` — updated the top-level lane summary to show seven lanes and tier labels.
- `tests/tier-lane-reconciliation.md` — manual regression fixture covering Quick Quote, Complex Commercial inheritance, Conference Room AV, and final-scope stage gating.
- `prompt_versions/Prompt_Update_Log.md` — this entry.

**Affected workflow / lane:** All lanes; especially Tier 1 quick quote/service/bid walk, Tier 2 conference room / standard lanes, and Tier 3 complex commercial.

**Cowork-side update needed:** Yes. Cowork should reload/use `mappings/lanes.yaml` as the canonical lane/tier bridge and defer control-file timing to `required_control_files_by_stage` instead of maintaining a parallel tier-to-file list.

**Tests added or updated:**
- `tests/tier-lane-reconciliation.md` — manual regression test fixture.

**Dustin approval status:** Draft pending PR review. Dustin approval required before merge. Implemented by Hermes because Codex connection was down; PR must not be merged by Hermes.

---

## 2026-05-29 — v0.3.4 — Docs correction: GitHub account and local path

**Summary:** Corrected two stale references in AGENTS.md and CODEX.md.
**Reason:** Repo owner account was referenced as `dustinwkerns` throughout; confirmed by Dustin 2026-05-29 that the correct owner is `dustinwkerns-GTG`. Local clone path in CODEX.md was stale (`C:\dev\htx-prewire-engine\`); corrected to actual OneDrive path.
**Files changed:**
- `AGENTS.md` — replaced all instances of `dustinwkerns/htx-prewire-engine` with `dustinwkerns-GTG/htx-prewire-engine`
- `CODEX.md` — same URL fix + corrected local path reference
**Affected workflow / lane:** All lanes (governance docs only — no rule behavior changed)
**Tests added:** None (docs-only change)
**Dustin approval status:** Approved verbally 2026-05-29

---

## 2026-05-15 — v0.3.3 — Skills Layer Governance implementation

**Summary:** Implemented a complete skills governance layer parallel to the engine's rule governance. This adds daily feedback collection, persistent logging, escalation logic, and formal documentation standards for all Cowork scheduled skills.

**Reason:** Skills (SKILL.md automation tasks) had no equivalent of the engine's correction-to-rule workflow. Behavioral drift in skills was invisible until something broke. The new layer gives skills the same discipline as rules: surface failures → log them → act on patterns → close the loop.

**Files changed:**

- `GOVERNANCE.md` — new § 11 (Skills Layer Governance) added; former § 11 Cross-references renumbered to § 12. Covers: skills vs rules distinction, daily feedback loop, acting on candidates, skill retirement criteria, STEP STRUCTURE documentation standard, audit trail entries.
- `09 Internal Operations & SOPs\SKILL_ENHANCEMENT_LOG.md` — new persistent log file created. Append-only table: Date / Skill / Issue / Ideal Behavior / Status / Resolution. Written by EOD STEP 7B; read by governance CHECK 4B.
- `Claude\Scheduled\htx-eod-wrapup\SKILL.md` — STEP 7B updated. Now appends SKILL ENHANCEMENT CANDIDATEs to the persistent log (atomic Write per Rule 13) in addition to the daily EOD document.
- `Claude\Scheduled\htx-daily-governance-check\SKILL.md` — CHECK 4B added (between CHECK 4 and CHECK 5). Reads SKILL_ENHANCEMENT_LOG.md, surfaces escalations (3+ consecutive days) and patterns (2+ in 14 days). Governance feed (CHECK 5) updated with SKILL HEALTH section. Daily report (CHECK 6) updated with SKILL HEALTH line.
- `Claude\Scheduled\htx-morning-briefing\SKILL.md` — STEP STRUCTURE documentation block added (Tier 1).
- `Claude\Scheduled\htx-eod-wrapup\SKILL.md` — STEP STRUCTURE documentation block added (Tier 1, earlier in same session).
- `Claude\Scheduled\htx-email-triage-9am\SKILL.md` — STEP STRUCTURE documentation block added (Tier 1).
- `Claude\Scheduled\htx-email-triage-1pm\SKILL.md` — STEP STRUCTURE documentation block added (Tier 1).
- `Claude\Scheduled\htx-project-trigger\SKILL.md` — STEP STRUCTURE documentation block added (Tier 1, strict sequence + deduplication emphasis).
- `Claude\Scheduled\htx-realtime-email-triage\SKILL.md` — STEP STRUCTURE ROUTING OVERVIEW added (Tier 1, full exception routing map).
- `prompt_versions/Prompt_Update_Log.md` — this entry.

**Affected workflow / lane:** Governance (all cadences — daily feedback loop feeds weekly health check and monthly audit). All active htx-* skills.

**Cowork-side update needed:** COWORK.md § Update Log — add entries for Tier 1 STEP STRUCTURE additions, Tier 2 persistent feedback loop, and Tier 3 governance documentation. (Pending — CLAUDE.md § WORKFLOW 4 could not be updated in this session due to file access constraints.)

**Tests added or updated:** None formal. The daily EOD run is the in-flight test — first SKILL HEALTH line in governance feed will confirm CHECK 4B is operating.

**Dustin approval status:** Approved. Dustin confirmed "Yes that sounds perfect, thanks!" after reviewing the three-tier plan.

**Notes:**

- Rule 3 (Composable Skills from Austin Marchese video) was applied at the STEP STRUCTURE documentation level rather than file decomposition. File decomposition was ruled out due to scheduled task architecture risk — STEP STRUCTURE blocks give the structural clarity without creating silent failure points from missing file references.
- Rule 4 (Skills Get Smarter Every Session) is implemented as the EOD STEP 7B → SKILL_ENHANCEMENT_LOG.md → CHECK 4B feedback cycle.
- STEP STRUCTURE standard applies to all new skills going forward. Legacy skills updated in this batch; any skills created before 2026-05-15 that were not updated should be considered for backfill in the next monthly audit.

---

## 2026-05-15 — v0.3.2 — Monthly audit task workstation-binding + Codex hand-off

**Summary:** Resolves audit recommendation #2 by making the monthly audit task explicitly **workstation-bound** with a best-effort `git pull`. Closes the audit feedback loop by handing the report to Codex for PR + merge.

**Reason:** The autonomous monthly run on 2026-05-15 found that `git pull origin main` fails in the sandbox because the repo is private and the autonomous shell has no credentials. Choices were (a) bind the task to the workstation, where credentials are already in place, or (b) provision a deploy key the sandbox can use. Workstation-binding wins on efficiency: zero new secrets, no new attack surface, no key rotation burden. The minor cost — the task can't run autonomously off the workstation — is acceptable because the workstation is on weekdays anyway, and the audit fires at 9 AM on the 1st of each month.

**Files changed:**

- `scripts/scheduled-task-prompts.md` — monthly audit § Procedure step 1 rewritten as "Pull the latest state (best-effort, workstation-bound)" with explicit guidance that the pull is non-blocking and that sandbox-context runs should note the constraint in § 9.
- On-disk `C:\Users\dusti\OneDrive\Documentos\Claude\Scheduled\htx-engine-monthly-audit\SKILL.md` — updated via `mcp__scheduled-tasks__update_scheduled_task` to match the reference doc.
- `prompt_versions/Prompt_Update_Log.md` — this entry.

**Affected workflow / lane:** Governance (monthly audit cadence).

**Cowork-side update needed:** None. The on-disk SKILL.md was updated in the same session via the scheduled-tasks MCP.

**Tests added or updated:** None. The next monthly run (2026-06-01 at 9:00 AM MST) is the in-flight test — it should produce a clean audit report without the "git pull failed" note in § 9 once it runs on the workstation.

**Dustin approval status:** Workstation-binding decision approved by Dustin. Codex hand-off in progress; PR not yet opened at time of log entry.

**Notes:**

- The audit hand-off to Codex is a single paste-able prompt covering three coordinated file changes: the audit report itself (new), `scripts/scheduled-task-prompts.md` (modified), and `prompt_versions/Prompt_Update_Log.md` (v0.3.1 and v0.3.2 entries added). Codex commits on branch `audit/2026-05-mid` and opens PR titled "Monthly engine audit — May 2026 (mid-month)". No auto-merge per `GOVERNANCE.md` § 2.
- The Lane-Stage-Backfill row-by-row review of the 31 active projects is intentionally deferred to a separate working session with Dustin.

---

## 2026-05-15 — v0.3.1 — Monthly engine audit run + follow-up reconciliation

**Summary:** Logs the first autonomous execution of `htx-engine-monthly-audit` (the scheduled task registered in v0.2.1) and reconciles its three § 8 recommendations against the state of the repo as of EOD 2026-05-15.

**Reason:** The monthly audit fired mid-cycle on 2026-05-15 against repo HEAD `7283007` (the v0.2.1 ship commit). It produced `reports/engine-audit-2026-05-15.md` with an overall **green** health status, plus three follow-ups. Two of the three are already resolved by subsequent commits in this same session — that resolution needs to be captured in the log so future audits don't keep flagging stale findings.

**Audit headline numbers (for the record):**

- Rule library: 19 files; cross-reference graph healthy (avg 4.7 in / 4.5 out).
- Most-cited rule: `02-lighting-control-only-lane` (10 inbound including the spreadsheet-quirks skill).
- Highest fan-out: `05-cowork-integration` (11 outbound — appropriate for a boundary file).
- Watch item: `15-proposal-benchmarks` has zero inbound refs. Re-evaluate at the 2026-08-01 audit; consolidate into `14-scope-writing` if still uncited.
- Stale rules: 0. Retirement candidates: 0. Overdue planned items: 0.
- Next regular run: 2026-06-01 at 9:00 AM MST.

**Reconciliation of § 8 recommendations:**

1. **Backfill v0.1.0 and v0.2.0 entries in `Prompt_Update_Log.md`** — **Already resolved.** The audit ran against HEAD `7283007` where the log only contained the v0.2.1 entry. The v0.2.2 and v0.3.0 commits later that same day fully populated the log with v0.1.0 + v0.2.0 entries. The log now contains all five entries through v0.3.0.
2. **Provision repo access for the autonomous run (`git pull` failed in the sandbox)** — **Open; decision needed from Dustin.** Recommended option: bind the monthly audit to the workstation environment where the repo is already authenticated, rather than provisioning a deploy key the sandbox can use. Keeps credentials off the sandbox.
3. **Catch up on the v0.3.0 backlog before 2026-07-15** — **Partially resolved.** v0.3.0 shipped `rules/18-project-stages.md` and `skills/htx-lane-stage-backfill/`. The remaining items from the original v0.3.0 plan (`schemas/`, `templates/`, `tests/`, `sample_projects/`, `exports/`) carry forward to v0.4.0+. The deadline (2026-07-15) is still 60 days out.

**Files changed:**

- `reports/engine-audit-2026-05-15.md` — produced by the autonomous run (filed in OneDrive working tree; pending PR to repo).
- `reports/engine-audit-2026-05-15.PR-handoff.md` — updated to mark recommendation #1 as resolved and to document the manual cleanup of the stray `.write-test` file.
- `prompt_versions/Prompt_Update_Log.md` — this entry.

**Affected workflow / lane:** Governance (all lanes).

**Cowork-side update needed:** None. The audit itself is the deliverable.

**Tests added or updated:** None. Future test infrastructure (v0.4.0+) will add a fixture verifying the audit can read the rule graph end-to-end without sandbox credentials.

**Dustin approval status:** Pending decision on item #2 (sandbox repo access). Items #1 and #3 noted as already-addressed.

**Notes:**

- This entry doubles as the audit trail Codex consumes when it picks up the audit PR. The PR hand-off doc points at `reports/engine-audit-2026-05-15.md`; Codex's commit message should still reference this version bump for log continuity.
- The audit's "log gap" finding is a real teaching moment: scheduled tasks operating against a remote HEAD can see a snapshot that has already been overtaken by work-in-flight. This is the second sandbox-vs-OneDrive write asymmetry caught in this session (the first being Rule 13). Worth keeping in mind when writing new scheduled-task prompts.
- The stray `reports/.write-test` file is documented for manual cleanup. The sandbox left it during write-permission probing and cannot remove it on the OneDrive mount.

---

## 2026-05-15 — v0.3.0 — Project stages + stage-aware gating + lane backfill task

**Summary:** Adds a four-stage project lifecycle model (`intake` → `engine_run` → `review` → `final_approved`) with per-stage required control files. The weekly health check now evaluates control-file gates relative to a project's current stage instead of demanding all seven files unconditionally. Adds a new `htx-lane-stage-backfill` skill for batch lane+stage inference across active projects. Produces the first lane-stage backfill report covering all 31 active projects.

**Reason:** v0.2.2's first scheduled health-check run flagged three projects (Basich, Murphys Aldrich, Stanton Home Theater) for missing required control files — but two of those projects are pre-review and shouldn't have FINAL_SCOPE_NOTES.md yet by design. The gate was producing false positives. Stage-aware gating fixes the rule (not the data); the backfill task fixes the data debt that accumulated when Project Lanes were formalized on 2026-05-14 without backfilling existing projects.

**Files changed:**

Schema and rules:
- `mappings/lanes.yaml` — schema bumped to 1.1; added top-level `project_stages:` block defining four stages; replaced `required_control_files:` flat lists with `required_control_files_by_stage:` per-stage lists on both `full_prewire` and `lighting_control_only` lanes
- `rules/18-project-stages.md` — new rule defining the stage model, stage transitions, stage-aware gate check procedure, and inference fallback
- `AGENTS.md` — version bumped to v3.0; added rule 18 to repo structure tree

Skills:
- `skills/htx-lane-stage-backfill/SKILL.md` — new skill for batch lane+stage inference across active projects, producing a markdown report for Dustin's review

Scheduled task:
- `htx-engine-weekly-health` (on-disk SKILL.md) — step 6 rewritten to use stage-aware gating per rules/18-project-stages.md; step 7 briefing format extended to include Lane field missing count, Stage field missing count, and stage-aware gate failures
- `scripts/scheduled-task-prompts.md` — same updates to the reference doc

First baseline produced:
- `09 Internal Operations & SOPs\SOPs\Lane-Stage-Backfill-2026-05-15.md` — initial lane+stage proposals for all 31 active projects, awaiting Dustin's review

**Affected workflow / lane:** All lanes (stage model is universal).

**Cowork-side update needed:** Yes (recommended but not blocking).
- COWORK.md `Required Control Files` section currently lists seven files unconditionally. Should add a reference to `rules/18-project-stages.md` noting that the gate is stage-aware in the engine layer.
- COWORK.md `Definition of Done` per lane remains correct — DoD is a different concept from per-stage gating (DoD = what's needed to consider a project complete; gating = what's required at each stage along the way).

**Tests added or updated:** Stage inference logic is verified by the lane-stage-backfill skill's first run (31 projects analyzed; no false positives in the report). Future test infrastructure (v0.4.0+) will add a synthetic fixture project per stage.

**Dustin approval status:** Pending review of the backfill report; pending merge of this PR.

**Notes:**
- This is the third correction-to-rule in this session (after v0.2.1 governance and v0.2.2 Dashboard fix). The cadence — a finding from the scheduled run produces a versioned rule update within hours — is the system working as designed.
- The 31-project backfill report shows: 18 likely `full_prewire`, 10 `bid_walk_pre_plan`, 3 `lighting_control_only`. Stage inference (content-aware) shows 28 at `final_approved`, 2 at `intake`, 1 at `engine_run` — but that 28-at-final number warrants Dustin's scrutiny: many of those projects are sitting in `Active/` but may genuinely be complete. Per COWORK.md monthly archive review, they should likely move to `Completed/`.
- Definition of Done remains a per-lane count (13 for Full Prewire, 11 for Lighting Control Only) — these are separate from per-stage gating and not affected by this change.
- v0.3.0 was previously a placeholder for "test infrastructure, schemas, sample fixtures, etc." — those items now shift to v0.4.0+.

---

## 2026-05-15 — v0.2.2 — First real correction-to-rule (Dashboard tab drift + PR-listing fix)

**Summary:** First correction-to-rule event captured. The htx-engine-weekly-health task — running for the first time as a real scheduled run — found two issues. Both fixed in this release.

**Reason:** The weekly health check is doing its job. On its first real Friday run (executed via "Run now" by Dustin), it found:
1. A real drift between `COWORK.md § Project Lanes` (listed "Dashboard" as a Full Prewire primary tab) and `mappings/lanes.yaml` (didn't include "Dashboard" at all).
2. A reproducible procedural issue: the PR-listing step using `curl` + the fine-grained PAT returns 403 because that PAT is scoped to repo Contents only, not Pull Requests.

Both issues were already noted as risks during the v0.2.1 dry-run; the real run made them blocking.

**Correction (Dustin's effective input, captured from the briefing):**
- "COWORK.md § Project Lanes lists 'Dashboard' as a Full Prewire primary tab; mappings/lanes.yaml has no Dashboard tab. One side is wrong."
- "PR-listing step needs to move off bash-against-GitHub-API."

**Investigation:**
- Workbook inspection (HTX_Prewire_Calculator_BLANK_MASTER_REV6_Optimized_Benchmark_5.14.26.xlsx) confirmed: Dashboard is tab #1 of the 23-tab workbook, present in BOTH the blank master AND the Welsby benchmark. COWORK.md is correct; lanes.yaml is missing it.
- Dashboard is not a primary takeoff tab — it's a summary/overview tab. Belongs in `workbook_tabs_support`, not `workbook_tabs_primary`. Applies to both Full Prewire and Lighting Control Only lanes.
- PR-listing: best fix is Claude in Chrome navigation since Cowork's Chrome session is already logged into GitHub. No new credentials needed.

**Reusable rules:**
1. "When the workbook's tab structure is referenced by both COWORK.md and lanes.yaml, both must include every tab present in the blank master workbook. Dashboard, in particular, applies to all lanes (it's a universal summary tab)."
2. "Scheduled tasks that query GitHub must use Cowork's authenticated Chrome session via Claude in Chrome MCP, not bash + curl + PAT. Fine-grained PATs with Contents-only scope return 403 on pulls API."

**Before / after example (lanes.yaml § full_prewire.workbook_tabs_support):**
- Before: `[Project Info, File Inventory, Relevant Sheet Review, Takeoff Summary, ...]`
- After: `[Dashboard, Project Info, File Inventory, Relevant Sheet Review, Takeoff Summary, ...]`

Same addition applied to `lighting_control_only.workbook_tabs_support`.

**Files changed:**
- `mappings/lanes.yaml` — added Dashboard to support tabs for full_prewire and lighting_control_only
- `scripts/scheduled-task-prompts.md` — rewrote step 2 of the weekly health prompt to use Claude in Chrome instead of curl + PAT
- Scheduled task `htx-engine-weekly-health` (on disk at C:\Users\dusti\OneDrive\Documentos\Claude\Scheduled\htx-engine-weekly-health\SKILL.md) — updated via `mcp__scheduled-tasks__update_scheduled_task`
- `prompt_versions/Prompt_Update_Log.md` — this entry

**Affected workflow / lane:** Full Prewire (workbook tabs), Lighting Control Only (workbook tabs), All (PR-listing step in weekly health check).

**Cowork-side update needed:** No — the same scheduled task now has the corrected prompt. Dustin's next "Run now" or the next Friday scheduled run will use the new logic.

**Tests added or updated:** None directly. The drift-detection procedure itself is the test — it caught this problem the first time it ran. Future tests will be added in v0.3.0+.

**Dustin approval status:** Implicit approval (Dustin reported the findings; fix is the obvious resolution). Final approval on repo merge.

**Notes:**
- This is the FIRST real correction-to-rule event in the system. The corrections workflow is being exercised live for the first time. Format follows `rules/99-corrections-workflow.md` § Seven Steps.
- Outstanding operational debt (not addressed in this release; needs Dustin decisions):
  - **30 of 31 active projects missing the Project Lane field** in PROJECT_CONTEXT.md. Backfill needed.
  - **3 active projects missing required control files** (Basich, Murphys Aldrich, Stanton Home Theater). Notably, Basich — the LC Only working example — has 5 of 7 control files missing.
- Codex was not used for this fix (manual execution from this session). When Codex is invoked for the next correction, the workflow will run autonomously.

---

## 2026-05-15 — v0.2.1 — Governance, continuity, scheduled tasks

**Summary:** Adds `GOVERNANCE.md` as the canonical doc for engine health, drift prevention, rule retirement, and continuity planning. Updates AGENTS.md, CODEX.md, README.md, and COWORK.md to cross-reference governance. Prepares three new scheduled tasks (weekly health check, monthly audit, quarterly architecture review) as a ready-to-register prompt file at `scripts/scheduled-task-prompts.md`. Also logs the Codex activation that completed today.

**Reason:** v0.2.0 shipped the full rule library and Codex preparation, but the system needed an explicit governance layer to prevent drift over time and define what "healthy" looks like. Without governance, the rule library would slowly diverge from real Cowork behavior and from the GPT, the correction-to-rule workflow would become checkbox-compliance, and continuity (handoff, recovery) would be undocumented.

**Files changed:**

Created:
- `GOVERNANCE.md` — engine health, three review cadences (weekly/monthly/quarterly), drift detection between repo / Cowork / GPT / workbook, rule retirement criteria, audit trail, continuity planning (short / medium / long-term + recovery scenarios), per-project quality gates, volunteer onboarding, anti-patterns to avoid
- `scripts/scheduled-task-prompts.md` — three ready-to-register Cowork scheduled-task prompts

Updated:
- `AGENTS.md` — bumped version to v0.2.1, added § 17 Governance and Continuity, renumbered Reading Order to § 18
- `CODEX.md` — added Codex's role in monthly audit, drift detection, rule retirement PRs; updated cross-references
- `README.md` — added GOVERNANCE.md to repo structure and shipped list, added scheduled tasks block
- `prompt_versions/Prompt_Update_Log.md` — this entry

**Affected workflow / lane:** All lanes (governance applies to the whole system).

**Cowork-side update needed:** Yes.
- `COWORK.md` Reference Files section updated with engine-side governance pointer (done in this same session).
- `COWORK.md` Update Log entry added (done).
- Three new scheduled tasks need to be registered by Dustin via `/schedule` in Cowork using the prompts in `scripts/scheduled-task-prompts.md`. (The `create_scheduled_task` tool requires interactive user confirmation, which couldn't be invoked autonomously from this session.)

**Tests added or updated:** None. Test infrastructure planned for v0.3.0+.

**Dustin approval status:** Pending review, repo merge, and scheduled-task registration.

**Notes:**
- Codex was activated earlier in this same session (ChatGPT Codex Connector installed, scoped to dustinwkerns/htx-prewire-engine only; first smoke test passed with 58-second execution time). Activation logged here for the audit trail since v0.2.0 said Codex was "prepared, not activated."
- The three scheduled tasks deliberately complement, not replace, Cowork's existing eight daily/weekly tasks (morning briefing, email triage AM/PM, EOD wrap-up, weekly review, Friday memory check, etc.). Total scheduled task count after registration: 11.
- The monthly audit produces a PR (a report file) rather than just a chat message — keeps the audit trail in the repo where it can be tracked over time.
- The quarterly review creates a calendar event rather than a chat prompt — explicitly carves out 30 minutes of Dustin's time for strategic review.
- `GOVERNANCE.md` § 9 explicitly anticipates that the governance discipline itself will need to evolve. Quarterly architecture reviews include a check on whether the cadences are still right.

---

## 2026-05-15 — v0.2.0 — Full rule set, Cowork integration, Codex preparation

**Summary:** Built out the remaining 18 rule files of the Phase 1 rule set, formalized the integration contract between this repo and Claude Cowork, created machine-readable lane definitions, and prepared Codex integration (`CODEX.md` + PR/issue templates). This brings the engine to a state where Cowork can consume rules from this repo end-to-end and Codex can take over rule maintenance.

**Reason:** v0.1.0 proved one rule + one skill installs and invokes correctly. v0.2.0 fills out the rule library so the engine has full coverage of the prewire workflow, formally documents how the engine interacts with Cowork's existing trigger system (`[HTX PROJECT]` email, watch terms, 12-folder structure, seven control files), and sets up Codex so the correction-to-rule workflow can run autonomously after Dustin sets up the Codex connection.

**Files changed (created):**

Rules (all in `rules/`):
- `00-non-negotiables.md` — absolute rules across all lanes
- `01-project-lanes.md` — lane definitions and determination logic
- `02-lighting-control-only-lane.md` — full operating block for the LC Only lane
- `03-file-classification.md` — classification categories and procedure
- `04-output-structure.md` — 12-section standard + lane variants (formalized Address Cross-Check as section #4)
- `05-cowork-integration.md` — division of labor between repo and Cowork (new)
- `06-prewire-standards.md` — condensed reference (categories, counting rules, room types)
- `07-prewire-logic.md` — marked-plan takeoff procedure
- `08-unmarked-assumptions.md` — conservative assumptions when plans are unmarked
- `09-lighting-control-full-prewire.md` — lighting control on Full Prewire projects
- `10-lutron-generation-check.md` — Maestro vs RA2 Select detection
- `12-calculator-mapping.md` — full symbol → item code → calculator row mapping
- `13-review-needed.md` — 26 category-specific Review Needed triggers
- `14-scope-writing.md` — language standards across audiences
- `15-proposal-benchmarks.md` — reusable phrasing library
- `16-address-cross-check.md` — address verification across sources
- `17-contamination-check.md` — wrong-project file detection
- `99-corrections-workflow.md` — how Dustin corrections become rule updates

Mappings:
- `mappings/lanes.yaml` — machine-readable lane definitions, shared between repo and Cowork (new)

Codex integration:
- `CODEX.md` — Codex's operating instructions on this repo (new)
- `.github/PULL_REQUEST_TEMPLATE.md` (new)
- `.github/ISSUE_TEMPLATE/correction.md` (new)
- `.github/ISSUE_TEMPLATE/feature-request.md` (new)

Top-level docs:
- `README.md` — updated to reflect v0.2.0 shipped content
- `prompt_versions/Prompt_Update_Log.md` — this entry

**Affected workflow / lane:**
- All lanes (rule library expansion)
- Lighting Control Only lane (new rules 02, 10 directly applicable; mandatory skills now documented)
- Full Prewire lane (rules 06–09, 12 cover the standard takeoff)

**Cowork-side update needed:**
- COWORK.md should add a reference to `mappings/lanes.yaml` as the canonical lane source.
- COWORK.md should add a note in the Reference Files section pointing to this repo as the engine source of truth for rules.
- No behavior changes required in Cowork — the integration contract in `rules/05-cowork-integration.md` documents the existing division of labor.

**Tests added or updated:**
- None yet. Test infrastructure planned for v0.3.0+. The new rule files describe expected behaviors clearly enough that tests can be generated retroactively.

**Dustin approval status:** Draft pending review and repo merge.

**Notes:**
- Read of `COWORK.md` shaped the integration approach. The repo is the *intelligence layer*; Cowork remains the *process layer*. They share `mappings/lanes.yaml` as the canonical bridge.
- All 26 Review Needed categories from HTX Review Needed Checklist v2 are now in `rules/13-review-needed.md` as a single consolidated file (vs the originally-planned 26 files).
- The Lighting Control Only lane has the most complete rule coverage in v0.2.0 because that's where the spreadsheet quirks foundation was laid in v0.1.0.
- Codex is prepared but not yet activated. `CODEX.md` is the starting context when Dustin connects Codex to this repo.

---

## 2026-05-15 — v0.1.0 — Initial repo scaffold

**Summary:** Created the HTX Prewire Engine repository as the durable source of truth for the prewire proposal engine. Established Project Lane architecture, modular rule structure, Cowork plugin manifest, and the first shipped rule (Spreadsheet Quirks).

**Reason:** Migration from a single GPT-hosted system prompt to a versioned, testable, multi-runtime architecture. The GPT will remain as conversational backup while Claude Cowork becomes the primary runtime executing skills from this repo. Codex edits the repo.

**Files changed (created):**
- `AGENTS.md` — operating agreement; 15 sections; lane-aware
- `README.md` — repo overview and current shipped state
- `.claude-plugin/plugin.json` — Cowork plugin manifest (v0.1.0)
- `rules/11-spreadsheet-quirks.md` — canonical rule for the ten patterns
- `skills/htx-spreadsheet-quirks-scan/SKILL.md` — skill that executes the rule
- `skills/htx-spreadsheet-quirks-scan/references/ten-patterns.md` — full pattern definitions
- `prompt_versions/Prompt_Update_Log.md` — this file

**Affected workflow / lane:**
- All lanes (architecture-level change)
- Lighting Control Only lane (first concrete rule applies here)

**Tests added or updated:**
- None yet. First test fixture (Basich-derived) planned for v0.2.0 — `tests/test_spreadsheet_quirks_basich.py`

**Dustin approval status:** Draft pending review and repo merge.

**Notes:**
- AGENTS.md formalizes the 12-section output structure (adding Address Sources Cross-Check as section #4) — an evolution of the 11-section standard documented in HTX Prewire Workflow Automation Guide v2 and HTX Prewire Standards v2.
- The Spreadsheet Quirks rule was selected as the first shipped rule because (a) the standards document is unusually crisp; (b) the Basich / 5210 Vickery project provides a real fixture; (c) the ten patterns are individually testable; (d) Lighting Control Only is where the current GPT needed the most hand-holding.

---

*Add new entries above this line, newest first.*
