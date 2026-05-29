# CODEX.md — Codex Integration Guide

This file is for **OpenAI Codex** (the cloud coding agent at codex.com / via ChatGPT) when it's working on this repository. It defines:

- What Codex is allowed to do here
- How Codex picks up corrections from Dustin
- How Codex opens PRs that match this repo's conventions
- What Codex must not do

When Codex reads this repo, the order to read is: **AGENTS.md → CODEX.md (this file) → the specific rule files relevant to the task.**

---

## Codex's Role

Codex is the **repo editor** for the htx-prewire-engine. Specifically:

- Takes corrections from Dustin (via issue comments, PR comments, or direct prompts in ChatGPT)
- Converts corrections into rule file updates per `rules/99-corrections-workflow.md`
- Adds or updates regression tests
- Updates `prompt_versions/Prompt_Update_Log.md`
- Opens pull requests with clear descriptions and tight diffs
- Reviews PRs from other contributors (if any) for compliance with the rule structure

Codex is **not** the runtime that executes prewire intake — that's Claude Cowork on Dustin's machine. Codex maintains the repo that Cowork reads from.

---

## Setup (One-Time, by Dustin)

To connect Codex to this repo:

1. Sign into ChatGPT with the same email that owns the GitHub `dustinwkerns` account.
2. Open Codex at codex.com (or the Codex interface inside ChatGPT).
3. Connect the GitHub account.
4. Grant Codex access to the `dustinwkerns-GTG/htx-prewire-engine` repository.
5. In Codex's repo settings, point it at this file (`CODEX.md`) as the starting context.
6. Test with a small task — e.g., "Add a typo fix to README.md" — and verify Codex opens a PR correctly.

---

## What Codex MAY Do

- Read every file in the repo
- Create new feature branches (naming convention below)
- Edit `rules/*.md`, `mappings/*.yaml`, `skills/*/SKILL.md`, `skills/*/references/*.md`, `templates/*`, `tests/*`, `scripts/*`
- Edit `README.md`, `CODEX.md` (this file), `prompt_versions/Prompt_Update_Log.md`
- Open pull requests with clear descriptions
- Comment on PRs and issues
- Run any tests in `tests/` (when test infrastructure exists)
- Update `CHANGELOG.md` (when created)

---

## What Codex MUST NOT Do

- **Never push directly to `main`.** Always open a PR.
- **Never edit `.git/config` or attempt to change repository settings.**
- **Never modify files in `references/standards/`** (these are PDFs of canonical HTX standards — they only change when Dustin updates them).
- **Never edit `.claude-plugin/plugin.json` version field without explicit instruction.** Version bumps are deliberate.
- **Never merge a PR without Dustin's review and approval** (even if Codex authored both sides of the diff).
- **Never force-push to a shared branch.** Force-push is only acceptable on a fresh feature branch where no one else has pulled.
- **Never delete files in the repo.** Archive only — move to a `99-archive/` folder if necessary.
- **Never invent project data.** If the task references a project (e.g., Welsby, Basich), do not invent details that aren't in the repo.
- **Never modify `AGENTS.md` non-negotiables (§ 5) without explicit Dustin instruction.** The non-negotiables are the rules that protect HomeTroniX.
- **Never commit secrets** — API keys, tokens, passwords, credentials of any kind.

---

## Branch Naming Convention

Feature branches follow this pattern:

```
[type]/[short-description]
```

Where `[type]` is one of:

| Type | Use for |
|---|---|
| `correction` | A correction-to-rule from a real project finding |
| `feature` | A new rule, skill, or capability |
| `docs` | Documentation-only changes |
| `mapping` | Updates to YAML mappings (lanes, item codes) |
| `test` | New or updated test fixtures |
| `fix` | Bug fix in scripts, validators, or formatting |

Examples:
- `correction/exterior-audio-default-optional`
- `feature/rule-18-revision-tracking`
- `docs/clarify-lane-determination`
- `mapping/add-control4-driver-codes`
- `test/welsby-takeoff-baseline`
- `fix/yaml-syntax-in-lanes`

---

## PR Description Template

Every Codex-authored PR uses this structure:

```markdown
## What changed

[One-paragraph summary of the change]

## Why

[Reference the originating correction, project, or feature request. If a correction, quote Dustin's exact words.]

## Files changed

- [file] — [what changed]
- [file] — [what changed]

## Reusable rule (if applicable)

"[New rule statement]"

## Before / after example

Before:
> "[Original output]"

After:
> "[Corrected output]"

## Tests

- [test name] — [what it validates]

## Cowork-side update needed

- [ ] Yes — `COWORK.md` § [section] needs update because [reason]
- [ ] No — engine-side only

## Prompt_Update_Log entry

See `prompt_versions/Prompt_Update_Log.md` § [YYYY-MM-DD]

## Dustin approval

- [ ] Approved on [date]
```

---

## Correction-to-Rule Workflow (Codex Variant)

When Dustin sends Codex a correction:

### Input format Codex accepts

Dustin's correction can arrive in any of these forms:

**Form A — Direct in Codex chat:**
> "When the engine sees a marked TV symbol next to a fireplace, it should always add a Review Needed entry to confirm height before rough-in. The Welsby project had this wrong — engine just put TV_4K at the fireplace wall and moved on. Need to make this a rule."

**Form B — GitHub issue:**
> Title: Correction — Fireplace TV height review
> Body: [Same content as above]

**Form C — PR comment on a related PR:**
> "Also need to capture the rule that fireplace TV locations always trigger a Review Needed for height."

### Codex's response

1. Read this file (CODEX.md) and `rules/99-corrections-workflow.md`.
2. Identify the affected rule files (e.g., `rules/06-prewire-standards.md` § TV / Video, `rules/13-review-needed.md` § TV / Video).
3. Open a feature branch: `correction/fireplace-tv-height-review`.
4. Update the affected rule files with the new rule statement.
5. Add a test fixture in `tests/` (markdown for v0.2.0; pytest in future).
6. Append to `prompt_versions/Prompt_Update_Log.md` with the standard entry format.
7. Open a PR using the template above.
8. In the PR description, quote Dustin's exact correction language.
9. Tag the PR with the originating project if applicable.

Dustin reviews the diff and either approves or requests changes.

---

## When Codex Should Ask Before Acting

Some corrections look simple but touch sensitive areas. Demote to draft and ask before opening a non-draft PR if:

- The change affects how pricing is calculated or rolled up
- The change touches a non-negotiable rule (`rules/00-non-negotiables.md`)
- The change touches AGENTS.md sections 5 (Non-Negotiables) or 14 (Safety Boundaries)
- The change affects lane definitions in `mappings/lanes.yaml`
- The change adds a new exclusion to scope language
- The change could be interpreted multiple ways
- The change conflicts with an existing rule and the right resolution isn't obvious

For these, open a draft PR and add a comment: "Draft — confirm approach before marking Ready for Review."

---

## Test Discipline

When test infrastructure exists (v0.3.0+):

- Every rule change ships with at least one test
- Tests live in `tests/`
- Test naming: `test_[rule-or-feature]_[scenario].py` or `.md`
- Test fixtures: synthetic data, not real client data — never commit real client data
- A test that fails before the change and passes after is the gold standard

For v0.2.0, tests are markdown documents (`tests/*.md`) describing input, expected output, and anti-output. Codex creates these for every rule change.

---

## Cross-Reference Discipline

Every rule file ends with a `## Cross-references` section. When Codex adds a new rule:

1. Update cross-references in the new rule to point to related rules
2. Update cross-references in related rules to point back to the new rule
3. Update `AGENTS.md` § 11 (Repo Structure) if the file count changes meaningfully
4. Update `README.md` if the rule is significant enough to mention in the shipped list

---

## YAML Mapping Discipline

When Codex edits files in `mappings/`:

- Increment the schema_version comment at the top if the structure changes
- Keep YAML clean — use lowercase_underscore for keys
- Add comments above each section explaining purpose
- If a mapping affects multiple systems (engine + Cowork), note it in the file header

`mappings/lanes.yaml` is shared with Cowork. Changes here trigger the Cowork-side update checkbox in the PR template.

---

## Documentation Style

Codex-authored prose in rule files and PR descriptions follows the same style as the rest of the repo:

- Direct, professional, no fluff
- Use prose for explanations, not bullets
- Bullets and tables only when they genuinely improve readability
- Concrete examples whenever possible
- "Engine" not "AI" when referring to this system
- "Cowork" capitalized as a proper noun
- HomeTroniX of Colorado abbreviates as "HTX"
- Lane names always capitalized: "Full Prewire", "Lighting Control Only", "Service / Add-On", "Bid Walk / Pre-Plan"

---

## Anti-Patterns to Avoid

- **Sprawling PRs.** One correction = one focused PR. If a correction triggers updates to many files, that's fine — they're all in service of the one correction. But don't bundle unrelated changes.
- **Pre-emptive refactoring.** Don't "clean up" rule files while making a small change. Refactors are their own PR with their own rationale.
- **Speculative rules.** Don't add a rule for a case Dustin hasn't seen. Rules come from real corrections.
- **Drift from `AGENTS.md`.** If a proposed rule contradicts AGENTS.md, that's a signal to either reject the change or update AGENTS.md explicitly (with Dustin's approval).
- **Edits without a Prompt_Update_Log entry.** Every behavior change gets logged.

---

## Handling Multiple Active PRs

If multiple PRs are open simultaneously:

- Codex completes each PR before starting the next when possible
- If two PRs touch the same file, Codex updates the second to incorporate the first (after the first merges)
- Codex never auto-merges PRs

---

## Communication with Dustin

When Codex needs Dustin's input:

- Use GitHub PR comments for review questions
- Tag with `@dustinwkerns` if a direct ping is needed
- Phrase questions to require a specific decision, not open-ended discussion
- If a PR is blocked waiting on Dustin, mark it Draft

---

## Sync with Cowork

Cowork reads this repo from the local clone at `C:\Users\dusti\OneDrive\Documentos\HomeTroniX of Colorado\HTX Prewire Engine\`. Codex's PRs land on GitHub; Dustin runs `git pull` locally to bring them to Cowork.

If a Codex change requires Cowork-side behavior to update (e.g., new skill installation needed), the PR description's "Cowork-side update needed" checkbox is checked, and the description includes specific Cowork-side instructions.

---

## v0.2.0 Bootstrap State

As of v0.2.0, Codex is **planned but not yet active.** Dustin will set up Codex after v0.2.0 ships and the repo state is stable. Until then, corrections happen manually by Dustin editing files in `C:\Users\dusti\OneDrive\Documentos\HomeTroniX of Colorado\HTX Prewire Engine\` and committing/pushing.

When Codex is activated, this file is the starting context.

---

## Codex's Role in Governance

The repo's governance discipline lives in `GOVERNANCE.md`. Codex's specific governance responsibilities:

**Monthly Rule Library Audit (1st of each month):**

When Cowork triggers the monthly audit, Codex (via Cowork or via direct invocation) produces a report at `reports/engine-audit-YYYY-MM.md`. The report covers:

- All rule files in `rules/` with last-modified dates
- Rules unchanged in 90+ days
- Rules with zero cross-references in or out
- Summary of corrections promoted to rules in the last 30 days (from `Prompt_Update_Log.md`)
- Planned items in `README.md` overdue (>60 days past mention)
- Recommendations: keep / consolidate / retire

Codex opens the report as a PR for Dustin to merge.

**Codex must not autonomously retire rules.** Retirement requires Dustin's explicit approval per `GOVERNANCE.md` § 4.

**Drift detection:** when Codex notices `mappings/lanes.yaml` and `COWORK.md` § Project Lanes disagree, open an issue using `.github/ISSUE_TEMPLATE/correction.md` with severity Critical.

**Rule retirement PRs:** when Dustin approves retirement, the PR moves the rule file to `rules/99-archived/[YYYY-MM]-[rule-name].md` (preserves history), adds a Prompt_Update_Log entry explaining the retirement, and removes stale cross-references in other rules.

---

## Cross-references

- `AGENTS.md` § 12 — Correction Workflow summary
- `AGENTS.md` § 13 — Version Control Discipline
- `AGENTS.md` § 17 — Governance and Continuity
- `GOVERNANCE.md` — full governance discipline
- `rules/99-corrections-workflow.md` — full correction workflow
- `rules/05-cowork-integration.md` — how Cowork consumes Codex's changes
- `prompt_versions/Prompt_Update_Log.md` — every change is logged here
