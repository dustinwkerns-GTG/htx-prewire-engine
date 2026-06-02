# GOVERNANCE.md — Engine Health, Drift Prevention, and Continuity

This file defines how the htx-prewire-engine stays healthy over time. The rule library, the skills, the integration with Cowork, and the relationship with the Prewire Helper GPT all decay if no one watches them. This document is the watching.

Owner: Dustin Kerns. Reviewer of record. Final approver of all rule and governance changes.

---


## 0. Current Status Note — 2026-06-01 System Sync

- **Tier/lane reconciliation merged:** Project Tiering Standard v1.0 is now reflected in the canonical engine lane model (`tier` on lanes plus `quick_quote`, `complex_commercial`, and `conference_room`).
- **Codex reconnect pending:** Codex was found pointed at non-existent `dustinwkerns/htx-prewire-engine`; do not treat Codex as a reliable editor until the GitHub app/environment is reconnected to `dustinwkerns-GTG/htx-prewire-engine` and a throwaway canonical-write test passes.
- **Canonical owner rule:** governance checks and editor handoffs must use `dustinwkerns-GTG/htx-prewire-engine`; any PR/branch result must be verified on GitHub before it is trusted.

---

## 1. Why This Document Exists

A system like this fails in two ways:

**Drift** — the repo and Cowork (and the GPT) stop agreeing. A lane definition changes here but not in `COWORK.md`. A new exclusion gets added to the GPT prompt but never makes it into a rule. Corrections happen in chat and are lost. Six months later, three different sources claim three different things, and Dustin spends time arbitrating instead of selling.

**Stagnation** — the system stops absorbing learning. Real projects produce real corrections, but the corrections live in Dustin's head or in scattered chats. The engine never improves. The GPT keeps making the same mistakes the rules were supposed to prevent.

This document defines the disciplines that prevent both.

---

## 2. The Three Cadences

Governance runs on three timescales. Each has a specific purpose, a specific output, and a scheduled task that fires the prompt.

### Weekly — Engine Health Check (Fridays 4:30 PM)

**Purpose:** Catch drift early. Detect any disconnect between what the repo says and what's actually happening on real projects.

**Trigger:** Cowork scheduled task `htx-engine-weekly-health` (Friday 4:30 PM MST).

**Procedure (Cowork executes):**

1. `cd C:\Users\dusti\htx-prewire-engine-git && git pull origin main` — sync the latest state.
2. Check open PRs at `https://github.com/dustinwkerns-GTG/htx-prewire-engine/pulls`. List title, age, author, status.
3. Read `prompt_versions/Prompt_Update_Log.md` and note the date of the most recent entry.
4. List rule files in `rules/` modified in the last 7 days (`git log --since='1 week ago' --pretty=format:'%h %ad %s' --date=short -- rules/`).
5. Compare `mappings/lanes.yaml` § lanes against `COWORK.md` § Project Lanes — note any discrepancy in lane names, primary sources, or workbook tabs.
6. Check active projects in `01 Clients & Projects\Active\` for any project missing the seven control files or the lane field in `PROJECT_CONTEXT.md`.
7. Produce a one-paragraph briefing for Dustin's morning briefing the next business day.

**Output:** One paragraph in Saturday or Monday morning briefing summarizing PRs, Log activity, stale items, drift findings.

**Halt conditions:** If a drift between `COWORK.md` and the repo is detected, surface as a Critical item — do not silently note. If a PR has been open more than 14 days, flag it.

### Monthly — Rule Library Audit (1st of each month, 9:00 AM)

**Purpose:** Identify rules that have stopped earning their place. Identify rules that need consolidation. Identify gaps where rules should exist but don't.

**Trigger:** Cowork scheduled task `htx-engine-monthly-audit` (1st of each month at 9 AM MST).

**Procedure (Cowork executes):**

1. Pull the latest repo.
2. For every rule file in `rules/`:
   - Last modified date
   - Last project that triggered or relied on it (from `AI_HANDOFF_LOG.md` entries — if accessible)
   - Cross-references in or out (does anything point to it; does it point to anything that's gone?)
3. List rules that haven't been modified in 90+ days.
4. List rules that are referenced 0 times by other rules and 0 times by skills.
5. Summarize the corrections-to-rules in the last 30 days from `Prompt_Update_Log.md`.
6. Identify any "Planned for vX.Y.Z" notes in `README.md` that are now overdue (>60 days past mention).
7. Produce a report and file it as `reports/engine-audit-YYYY-MM.md` in the repo (open as a PR).

**Output:** A monthly audit report PR. Dustin reviews, merges (or requests changes), and acts on any retirement / consolidation recommendations.

**Halt conditions:** None — this is a reflective report, not an enforcement.

### Quarterly — Architecture Review (1st of Jan, Apr, Jul, Oct at 9:00 AM)

**Purpose:** Look at the system end-to-end. Is the engine doing what it was set up to do? Is Cowork's process layer still aligned? Are the three lanes still the right lanes? Is the correction-to-rule workflow producing meaningful improvement, or has it become checkbox-compliance?

**Trigger:** Cowork scheduled task `htx-engine-quarterly-review`.

**Procedure (Cowork prompts Dustin):**

1. Cowork sends a calendar event with a 30-minute block: "Quarterly HTX Prewire Engine Architecture Review."
2. Cowork compiles a pre-meeting brief with:
   - Volume metrics: how many projects ran through the engine; how many corrections became rules; how many PRs Dustin merged
   - Trend metrics: lane distribution, common Review Needed categories, recurring corrections
   - Open architectural questions surfaced in the last quarter
3. Dustin reviews the brief and decides whether structural changes are needed:
   - Should a lane be added, retired, or renamed?
   - Should the engine take on a function currently in Cowork (or vice versa)?
   - Should new skills replace manual GPT-driven workflows?
   - Should the workbook structure evolve?

**Output:** Either no change (system is working) or a v0.X.0+ feature plan logged as a feature-request issue in the repo.

**Halt conditions:** None — strategic, not operational.

---

## 3. Drift Detection

Drift is the single biggest threat to a system like this. Specific drift surfaces to monitor:

### Between this repo and Cowork (`COWORK.md`)

The shared concepts are:
- The four Project Lanes (definitions, primary sources, workbook tabs)
- The seven required control files (`PROJECT_CONTEXT.md`, `INTAKE_FORM.md`, `AI_RUN_PACKAGE_PreWire_v1.md`, `AI_HANDOFF_LOG.md`, `ASSUMPTIONS_AND_DECISIONS.md`, `REVIEW_NEEDED.md`, `FINAL_SCOPE_NOTES.md`)
- The 12-subfolder project structure
- AI_RUN_PACKAGE reference blocks per lane

The canonical bridge is `mappings/lanes.yaml`. Both `COWORK.md` and the repo's rules should agree with the YAML. When they don't, the YAML wins until updated through a PR.

**Detection:** Weekly engine health check compares `mappings/lanes.yaml` against `COWORK.md` § Project Lanes. Any discrepancy is a Critical Review Needed.

**Resolution:** Open an issue using `.github/ISSUE_TEMPLATE/correction.md`. Codex (or Dustin manually) updates whichever side is wrong.

### Between this repo and the Prewire Helper GPT

The GPT has a system prompt that was the original engine. As rules in this repo evolve, the GPT's prompt should evolve too — otherwise the GPT keeps producing outputs based on stale rules.

**Detection:** When the cumulative volume of rule changes since the last GPT-prompt update exceeds a threshold (heuristic: 5+ rule changes or 60+ days), surface in the monthly audit.

**Resolution:** Generate a flattened "GPT knowledge snapshot" from the current rules and upload to the GPT. This is planned automation (`exports/gpt_knowledge_snapshot.md`, v0.3.0+). Until then, Dustin manually updates the GPT prompt or its knowledge files.

### Between rules and the actual project workbook

The `mappings/item_codes.yaml` (planned v0.3.0) and the Legend Database tab of `HTX_Prewire_Calculator_BLANK_MASTER_REV6_Optimized_Benchmark_5.14.26.xlsx` must agree. When item codes are added or changed in the workbook, the YAML must be updated.

**Detection:** Manual for now. Planned: an automated diff between workbook Legend Database and `mappings/item_codes.yaml` in the monthly audit.

**Resolution:** Update both via PR. Update the workbook in OneDrive; update the YAML mirror; log in `Prompt_Update_Log.md`.

### Between rules and real project outputs

The hardest drift to detect: rules say one thing, but on a real project the engine produced something different and no one noticed. The defense is the correction-to-rule workflow — corrections that go through it close this gap. Corrections that happen in chat and aren't captured are the failure mode.

**Detection:** Cowork should ask, after every Dustin Review Brief, "Were there any corrections to engine output that should become rules?" If yes, open an issue using `.github/ISSUE_TEMPLATE/correction.md` before the project is filed to `11 Final Approved\`.

---

## 4. Rule Retirement

Rules accumulate. Without retirement discipline, the library bloats and Dustin's review time goes up.

A rule is a **retirement candidate** when:

- Last modified date is 90+ days ago AND
- Cross-references to it are 0 from other rules AND
- It hasn't been triggered or referenced in any project in the last quarter AND
- The original need has been absorbed into a different rule

A rule is **retired** when:

1. The monthly audit identifies it as a candidate
2. Dustin confirms retirement is appropriate
3. A PR moves the rule file to `rules/99-archived/[YYYY-MM]-[rule-name].md` (preserves the history)
4. `Prompt_Update_Log.md` logs the retirement with the rationale

Retirement is reversible. Archived rules can be reactivated by moving them back to `rules/` and adding a new Prompt_Update_Log entry.

**Never retire a rule that's been triggered in the last 90 days, even if it hasn't been modified.** Stable rules that just work are the most valuable rules.

---

## 5. Audit Trail

Every decision that affects engine behavior leaves a trail. The trail elements:

| Decision type | Logged in |
|---|---|
| New rule | `prompt_versions/Prompt_Update_Log.md` + git commit + PR description |
| Rule change | Same |
| Rule retirement | Same |
| New skill | Same + skill SKILL.md frontmatter `description` |
| Skill change | Same |
| YAML mapping update | Same |
| AGENTS.md change | Same |
| COWORK.md change | `COWORK.md` § Update Log + `Prompt_Update_Log.md` (if engine-relevant) |
| GPT prompt update | `Prompt_Update_Log.md` + the GPT's own version notes |
| Per-project correction (not yet promoted to rule) | `09 AI - Working\AI_HANDOFF_LOG.md` of that project |
| Per-project Dustin Review Brief decisions | `10 Review Needed\` and `11 Final Approved\` of that project |

When in doubt, log it. Over-documentation is recoverable; under-documentation isn't.

---

## 6. Continuity Planning

The engine should keep working if Dustin is unavailable for a short period, and should be recoverable if any part of the stack fails.

### Short-term continuity (Dustin away 1-7 days)

- Cowork continues its daily rhythm via scheduled tasks. Morning briefings, email triage, EOD wrap-ups run on cron.
- New `[HTX PROJECT]` emails create folders and control files automatically per `COWORK.md` § Project Trigger System.
- Spreadsheet quirks scans run when invoked.
- Codex is available for repo edits but should not merge PRs without Dustin's review.
- Open PRs queue up for Dustin's return.

### Medium-term continuity (Dustin away 1-4 weeks)

- Designate a delegate (Rhonda, if she's in role) for project intake review.
- The delegate uses the Dustin Review Brief and the seven control files to make routine decisions, but does not override engine guardrails (lighting control in base, exterior entertainment in base, etc.).
- Quarterly review postpones if it falls in this window.
- All architectural changes wait for Dustin.

### Long-term continuity (Dustin handoff or extended absence)

- The repo plus `COWORK.md` plus the Prewire Helper GPT prompt is the documented system. A successor with normal AV/integration knowledge can run it.
- The handoff procedure (planned `templates/Handoff.template.md`) covers credentials, repo access, Codex authorization, Cowork installation, OneDrive structure, builder/client relationships, and pending decisions.

### Recovery — repo loss

- The repo lives on GitHub. As long as GitHub is online, the repo is recoverable.
- A backup clone exists at `C:\dev\htx-prewire-engine\` on Dustin's machine.
- If both are lost, the OneDrive copy at `C:\Users\dusti\OneDrive\Documentos\HomeTroniX of Colorado\HTX Prewire Engine\` is a third copy (without git history, but with all current rule and doc content).
- Recovery procedure: clone fresh from GitHub. If GitHub itself is lost, push the OneDrive copy as a new repo and re-establish.

### Recovery — Cowork unavailable

- Cowork is one of three runtimes that can consume this repo (Cowork + GPT + Codex). If Cowork goes offline, the GPT continues running with its current prompt; corrections accumulate; once Cowork is back, the accumulated corrections become PRs.

### Recovery — Codex unavailable

- Dustin or a developer can edit the repo locally and push without Codex. The correction-to-rule workflow continues; only the autonomous PR creation is gone.

### Recovery — GPT unavailable

- The repo is the source of truth. The engine continues operating via Cowork-invoked skills. The GPT is the backup; losing it does not lose the engine.

---

## 7. Per-Project Quality Gates

Every project should pass these gates before it's marked complete (`11 Final Approved\`):

| Gate | Owner | When |
|---|---|---|
| Address Cross-Check section #4 produced and clear | Engine | Intake |
| File classification complete; no Unknown files unresolved | Engine | Intake |
| Contamination check clean | Engine | Intake |
| Spreadsheet Quirks scan run (if Lighting Control Only) | Engine | Intake |
| Lutron generation check clean (if lighting in scope) | Engine | Intake |
| Lane explicitly set in `PROJECT_CONTEXT.md` | Cowork | Intake |
| Seven control files present | Cowork | Intake |
| Correction-to-rule prompt asked after Dustin Review | Cowork | Pre-final-approval |
| Definition of Done items per lane all met | Cowork + Dustin | Final approval |
| Approved files moved to `11 Final Approved\` only after Dustin approval | Cowork | Final approval |

Skipping a gate without explicit Dustin override is a process bug. Surface in the next weekly engine health check.

---

## 8. Volunteer-Friendly Onboarding

If Dustin ever brings someone onto this system (Rhonda, Jake, a new hire, a consultant), they should be able to start contributing without a long ramp.

The onboarding path:

1. **Day 1:** Read `AGENTS.md` end-to-end. Read `COWORK.md` end-to-end. Read this file. Read `CODEX.md` if doing repo work.
2. **Day 1:** Skim `rules/01-project-lanes.md` and `rules/00-non-negotiables.md`.
3. **Day 2:** Pick a recent project and read its seven control files. Compare to the rules. Build mental model of how intake → output flows.
4. **Day 2-3:** Shadow Dustin on one Dustin Review Brief.
5. **Week 2:** Take one low-risk task — perhaps reviewing a Codex PR or auditing a project's File Inventory.
6. **Week 4:** Trusted enough to handle Routine Triage items on the morning briefing (without writing client-facing communication).

No volunteer / new contributor merges PRs that change `rules/00-non-negotiables.md` or `AGENTS.md` non-negotiables (§ 5) without Dustin's explicit OK.

---

## 9. When Governance Itself Should Change

This file is governance. It can also drift.

Review GOVERNANCE.md every quarter as part of the architecture review. Specifically check:

- Are the three cadences (weekly, monthly, quarterly) still the right cadences? Should they be more frequent, less frequent, different timing?
- Is rule retirement happening, or is the library only growing?
- Are continuity plans actually exercised, or are they paper-only? Try a one-day simulation of "Dustin is offline."
- Is the audit trail tight, or are corrections leaking around it?

Update GOVERNANCE.md when the answers change. Log changes in `Prompt_Update_Log.md` like any other behavior change.

---

## 10. Observation Artifact Rule

Files written to `reports/` or `codex-handoffs/` by any agent (Cowork, Hermes, or Codex)
are advisory only. They do not change engine behavior by themselves.

**An observation artifact becomes a rule change only when:**
1. Codex converts it into a PR on a focused branch following CODEX.md discipline
2. The PR includes a test or markdown fixture, a Prompt_Update_Log entry, and a clear
   before/after description
3. Dustin reviews and merges the PR

No agent may treat a finding in `reports/` or a handoff in `codex-handoffs/` as an
active rule until the PR is merged. When in doubt, surface to Dustin.

---

## 11. Anti-Patterns to Avoid

- **Bureaucracy.** Adding governance steps that slow down real work without preventing real failures. If a check has never caught a real issue in 6 months, retire it.
- **Performative reporting.** Reports that nobody reads. Either Dustin reads the monthly audit and it influences decisions, or the audit stops being produced.
- **Over-automation.** Don't automate Dustin out of the loop on decisions he should be making. Cowork should surface and ask, not silently decide.
- **Drift without escalation.** If drift is detected and just logged but never fixed, the logging is theater. Drift findings should always include a clear next action.

---

## 12. Skills Layer Governance

The engine governs **rules** — the behavioral logic that determines what the prewire engine produces. Cowork governs **skills** — the scheduled SKILL.md automation tasks that run daily operations (triage, briefings, project trigger, governance check, EOD wrap-up). These are parallel governance concerns. The disciplines below apply to the skills layer only; they do not replace or overlap with the engine cadences in § 2.

### Skills vs. Rules — What's Different

Rules are promoted from real project corrections and change slowly. Skills are operational scripts that run every day and must be updated whenever the workflow they support changes. Skills fail silently when they're wrong — the automation fires but produces a bad or missing result. Rules fail loudly — the engine output is visibly incorrect. This makes skill drift harder to catch and more important to monitor proactively.

### Daily Feedback Loop

Every weekday, the EOD Wrap-Up skill (htx-eod-wrapup STEP 7B) scans the day's activity for manual interventions — tasks done by hand that a skill should have handled automatically. Any finding is flagged as a SKILL ENHANCEMENT CANDIDATE and appended to the persistent log at:

`C:\Users\dusti\OneDrive\Documentos\HomeTroniX of Colorado\09 Internal Operations & SOPs\SKILL_ENHANCEMENT_LOG.md`

Every weekday morning, the Daily Governance Check (htx-daily-governance-check CHECK 4B) reads that log and surfaces:
- **ESCALATIONS** — the same skill flagged 3+ consecutive days → requires Dustin action
- **PATTERNS** — the same skill flagged 2+ times in 14 days (non-consecutive) → worth monitoring
- **SKILL HEALTH** — the open candidate count surfaces in both the governance feed and the daily report

### Acting on Candidates

When a SKILL ENHANCEMENT CANDIDATE is escalated or reviewed by Dustin:

1. Dustin reviews the log entry and determines: update the skill, defer, or mark Won't Fix
2. If updating: edit the SKILL.md file and update the scheduled task via the scheduled-tasks MCP
3. Log the resolution in `SKILL_ENHANCEMENT_LOG.md` — update Status to "Resolved" and add the Resolution note
4. If the change affects a workflow documented in COWORK.md, update COWORK.md § Update Log
5. If the change has engine implications (affects rules, lane definitions, or COWORK.md ↔ repo contract), log in `prompt_versions/Prompt_Update_Log.md` using the standard entry format

### Skill Retirement

A skill is a **retirement candidate** when all three conditions are met:
- It has not fired in 90+ days
- Its function has been absorbed into another skill or made unnecessary by a workflow change
- Dustin confirms retirement is appropriate

Retirement procedure: move the SKILL.md file to an archive location, delete or disable the scheduled task, log the retirement in `prompt_versions/Prompt_Update_Log.md`, update COWORK.md § Scheduled Automation to remove the entry.

### STEP STRUCTURE Documentation Standard

Every multi-step SKILL.md file must include a STEP STRUCTURE block immediately before STEP 1. This block documents:
- Which steps are required on every run vs. optional (with condition)
- Step dependencies (step B cannot run before step A)
- Exception routing (when step 2 triggers alternate paths)
- Valid STOP conditions (when it is correct to exit early without completing all steps)

This standard was applied to all active htx-* skills on 2026-05-15. New skills must include this block from the start.

### Audit Trail for Skill Changes

Skill changes follow the same audit trail discipline as rule changes (§ 5), with these specific entries:

| Decision type | Logged in |
|---|---|
| New skill created | `prompt_versions/Prompt_Update_Log.md` + COWORK.md § Scheduled Automation |
| Skill updated (behavioral change) | `prompt_versions/Prompt_Update_Log.md` + COWORK.md § Update Log |
| Skill retired | Same + scheduled task deleted |
| Enhancement candidate resolved | `SKILL_ENHANCEMENT_LOG.md` Resolution column |
| STEP STRUCTURE documentation added | `prompt_versions/Prompt_Update_Log.md` |

---

## 13. Cross-references

- `AGENTS.md` — operating agreement
- `CODEX.md` — Codex's operating instructions
- `COWORK.md` (Dustin's machine) — Cowork's operating doc
- `rules/05-cowork-integration.md` — Cowork ↔ repo contract
- `rules/99-corrections-workflow.md` — the correction-to-rule process
- `prompt_versions/Prompt_Update_Log.md` — the audit trail
- `mappings/lanes.yaml` — shared lane definitions
- `SKILL_ENHANCEMENT_LOG.md` (09 Internal Operations & SOPs) — daily skills feedback log

---

*This file is the governance discipline. Detailed enforcement happens in the cadences and the audit trail. Edit GOVERNANCE.md when the discipline changes; edit specific rules when specific behaviors change.*
