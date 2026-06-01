# Rule 99 — Correction-to-Rule Workflow

When Dustin corrects an output, that correction must turn into a rule update — not just a one-time fix. This workflow defines exactly how that happens, who does what, and how the change reaches production without drift.

This is the single most important rule for long-term improvement. Without this discipline, the engine plateaus.

---

## When This Workflow Runs

This workflow fires every time:
- Dustin says "that's wrong" / "this should be different" / "we don't do it that way" about engine output
- Dustin amends a proposal in a way that contradicts what the engine produced
- A field tech reports the engine got something wrong
- A builder/client correction reveals a pattern the engine missed
- A new HTX standard or process supersedes existing engine behavior
- A new edge case is discovered that existing rules don't address

If a correction happens *outside* this workflow (e.g., Dustin tells the engine in chat and moves on), the correction is lost. **Every correction goes through this workflow or it doesn't count.**

---

## The Seven Steps

### Step 1 — Capture the correction precisely

Document the correction in Dustin's exact words (or paraphrased with confirmation):

```
Correction (verbatim or paraphrased + confirmed):
"[Dustin's exact statement of what was wrong]"

Original output (what the engine produced):
"[The specific output that was wrong]"

Project context:
- Project name
- Lane
- Date
- Which rule/skill produced the wrong output
```

This goes into a working scratchpad — either as a draft rule update, a GitHub issue, or a note in `09 AI - Working\` for the affected project.

### Step 2 — Identify the rule(s) affected

Map the correction to specific rule files:

```
Affected rules:
- rules/[NN]-[topic].md § [section]
- rules/[NN]-[topic].md § [section]

Affected skills:
- skills/[skill-name]/SKILL.md
- skills/[skill-name]/references/[file].md

Affected mappings:
- mappings/[file].yaml (if applicable)
```

If the correction maps to no existing rule file, this may be a new rule. Create the new file in the next step.

### Step 3 — Draft the reusable rule

Convert the correction into a rule statement that future projects will follow. Use this format:

```
Reusable rule:
"[New rule statement — clear, specific, testable]"

Before example (what the engine would have done under old rules):
"[Original output]"

After example (what the engine should do under the new rule):
"[Corrected output]"

Edge cases / exceptions:
- [Any cases where the new rule does not apply]
```

Good reusable rules are:
- Specific enough to be testable
- General enough to apply across projects
- Limited in scope (don't try to fix five things in one rule)

### Step 4 — Update the affected files

In the local clone at `C:\dev\htx-prewire-engine\`, on a feature branch:

```powershell
cd C:\dev\htx-prewire-engine
git checkout -b correction/[short-description]

# Edit the affected rule files, skills, mappings
# Make focused changes — only what the correction requires
```

Keep diffs small. One correction = one focused PR.

### Step 5 — Add or update a regression test

Add a test that would have caught the original mistake. For now, this is a manual test fixture documented in `tests/` (planned for v0.3.0+). When the test infrastructure exists, this is a `pytest` function.

For v0.2.0, the test is a markdown file in `tests/`:

```markdown
# Test — [Correction description]

**Date:** [YYYY-MM-DD]
**Origin:** Correction from [Project Name]
**Rule:** [rules/NN-topic.md]

## Input

[Synthetic input that demonstrates the situation]

## Expected output

[What the engine should produce]

## Anti-output

[What the engine should NOT produce — the old wrong behavior]
```

### Step 6 — Update `Prompt_Update_Log.md`

Every behavior change gets an entry:

```markdown
## [YYYY-MM-DD] — v0.X.Y — [Short description]

**Summary:** [One-paragraph description]

**Reason:** [Why the change was needed; reference the originating project/correction]

**Files changed:**
- rules/[file].md — [what changed]
- skills/[skill]/[file].md — [what changed]
- mappings/[file].yaml — [what changed]
- tests/[test].md — [test added]

**Affected workflow / lane:** [Full Prewire / Lighting Control Only / All]

**Tests added or updated:**
- [test name]

**Cowork-side update needed:** [Yes / No — if yes, what to update in COWORK.md or Cowork's behavior]

**Dustin approval status:** [Draft pending review / Approved on YYYY-MM-DD]
```

### Step 7 — Open a pull request

Push the branch and open a PR. The PR description includes:

- Link to the correction (project, date, what was wrong)
- The reusable rule
- Before / after example
- Files changed (auto-shown by GitHub)
- Test added
- Cowork-side update checkbox (if Cowork's behavior also needs updating)

Dustin reviews the diff and merges. Codex may handle steps 4–7 autonomously when Dustin approves.

---

## Codex's Role

When Codex (the OpenAI cloud agent connected to this GitHub repo) is set up to take corrections:

1. Dustin sends a correction to Codex as an issue or comment.
2. Codex performs steps 2–6 above on a feature branch.
3. Codex opens the PR with all the documentation in place.
4. Dustin reviews and merges (or requests changes).
5. The merged change flows to Cowork on the next local pull.

For now (v0.2.0), Codex is not yet integrated. Corrections happen manually via local git commits. See `CODEX.md` for the planned integration.

---

## What Counts as a "Behavior Change"

Yes — these are behavior changes and need the full workflow:
- Adding a new rule
- Changing how an existing rule applies
- Updating an item code mapping
- Adding a new pattern to a checklist
- Changing the priority assignment of a Review Needed item type
- Adding or changing a skill's procedure

No — these are not behavior changes (still need a commit but no Prompt_Update_Log entry):
- Fixing typos in rule files
- Reformatting tables
- Updating cross-references to match a renamed file
- Updating the README
- Adding examples that illustrate existing rules without changing them

When in doubt, treat it as a behavior change and log it. Over-documentation is recoverable.

---

## Demotion Rule

If a proposed rule change feels risky or touches a sensitive area (pricing, scope boundaries, legal protections), demote to a draft and ask Dustin before applying:

- Draft the rule update
- Open the PR as a Draft (not Ready for Review)
- Ask Dustin in chat: "Proposed rule change at [file]. Before I mark this Ready for Review, want to confirm the approach?"
- Apply only after Dustin's explicit OK

Better to interrupt once than ship a wrong rule.

---

## Standing Correction Categories

Some correction types come up repeatedly. Have templates ready:

| Category | Typical correction | Typical fix |
|---|---|---|
| Item code update | "We don't use that code anymore" | Update `mappings/item_codes.yaml` + Legend Database tab |
| Default behavior change | "We don't assume X anymore" | Update `rules/08-unmarked-assumptions.md` or relevant rule |
| New exclusion | "We need to explicitly exclude Y in scope language" | Update `rules/14-scope-writing.md` § Exclusions |
| Lane behavior change | "We do X differently on lighting-only projects" | Update `rules/02-lighting-control-only-lane.md` |
| Confidence threshold change | "That should have been Low confidence, not Medium" | Update `rules/00-non-negotiables.md` § Confidence + relevant rule |
| Review Needed criteria | "That should trigger a Critical, not High" | Update `rules/13-review-needed.md` or `rules/10/11/16/17` |
| Cowork integration shift | "Cowork should do X before invoking the skill" | Update `rules/05-cowork-integration.md` + COWORK.md |

---

## Template

Save this as `templates/Correction_To_Rule.template.md` (planned for v0.3.0+):

```markdown
# Correction to Rule — [YYYY-MM-DD]

## Step 1 — Capture

**Correction (Dustin's words):**
"..."

**Original output (what the engine produced):**
"..."

**Project context:**
- Project: [Name]
- Lane: [Full Prewire / Lighting Control Only / Service / Bid Walk]
- Date: [YYYY-MM-DD]
- Source skill/rule: [skill or rule that produced the output]

## Step 2 — Identify affected files

- rules/[NN]-[topic].md § [section]
- skills/[name]/SKILL.md
- mappings/[file].yaml

## Step 3 — Reusable rule

**Rule:** "..."

**Before example:** "..."

**After example:** "..."

**Edge cases:** "..."

## Step 4 — Branch and edit

```bash
git checkout -b correction/[short-description]
# edit affected files
```

## Step 5 — Test fixture

See `tests/[test-name].md`

## Step 6 — Log entry

See `prompt_versions/Prompt_Update_Log.md` § [YYYY-MM-DD]

## Step 7 — PR

PR link: [URL after open]
Dustin approval status: [Pending / Approved on YYYY-MM-DD]
```

---

## Cross-references

- `prompt_versions/Prompt_Update_Log.md` — every change is logged here
- `CODEX.md` — Codex integration plan (when ready, Codex automates steps 2–7)
- `AGENTS.md` § 12 (Correction Workflow) — the AGENTS.md summary
- `rules/05-cowork-integration.md` § Sync Discipline — keeping repo and Cowork aligned
