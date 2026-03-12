# SKILL: Pattern Validation

## Purpose
Review a pattern card or full pattern document for completeness, internal consistency, and readiness for the next governance stage. Produces a structured validation report with actionable findings, not just a pass/fail. The goal is to help the author fix problems, not gatekeep.

## When to Activate
- Architect says "validate this" or "is this ready for review?"
- A pattern card is tagged `ready-for-review`
- A full pattern document is being promoted from `draft` to `under-review`
- After a pattern edit, to check nothing broke

## Behavior

### Autonomy Level
Run all checks autonomously and produce a full report. Do not ask questions during validation — record gaps as findings and let the architect decide what to address. Exception: if the document is ambiguous in a way that makes a check impossible to run, note it as `CANNOT ASSESS` rather than guessing.

---

## Pattern Card Validation

Run these checks on a Pattern Card before promotion:

| Check | What to Look For |
|---|---|
| **Completeness** | Are all non-optional sections filled in? Flag any that contain only the prompt text or are empty. |
| **Problem clarity** | Does §Problem make sense to someone who doesn't already know the answer? Would a new team member understand what pain this solves? |
| **Anti-pattern alternatives** | Every "don't use it when" row must name a concrete alternative, not just say "don't do this." |
| **Tool specificity** | Are tools named specifically (Fivetran, Dagster) or vaguely ("an ETL tool")? Vague tool references block promotion. |
| **Non-negotiables** | Are there at least 3 non-negotiables? Are any of them actually preferences rather than requirements? |
| **Open questions** | Are open questions blockers or acceptable deferments? Flag any that touch security or data handling as potentially blocking. |

### Pattern Card Report Format

```
## Pattern Card Validation Report
Card: [Name]  |  Date: [Date]  |  Reviewer: Claude

### ✅ Ready to Promote
[List any sections that are complete and well-formed]

### ⚠️ Fix Before Promotion (Blocking)
1. [Section]: [Finding] — [Specific suggestion]

### 💬 Consider Improving (Non-Blocking)
1. [Section]: [Finding] — [Suggestion]

### Promotion Recommendation
READY / NOT READY — [one sentence rationale]
```

---

## Full Pattern Document Validation

Run these checks on a full ADP document before status change:

### Structure Checks
- [ ] All 18 sections present and numbered correctly
- [ ] Frontmatter complete: `pattern_id`, `version`, `status`, `lifecycle_stage`, `owner`, `domain` all populated
- [ ] Pattern ID matches filename
- [ ] `next_review_due` is set and in the future
- [ ] `composed_of` patterns have their options resolved in §2.3
- [ ] No `_TODO:` or `_[placeholder]` text remaining in the document

### Constraint Checks
- [ ] Every constraint has an ID in format `C-NNN`
- [ ] Constraint IDs are sequential with no gaps or duplicates
- [ ] Every MUST/MUST NOT constraint uses RFC 2119 language precisely — flag any that hedge ("MUST, in most cases")
- [ ] Every constraint is one clear sentence — flag run-ons
- [ ] Tool constraints (§8.2) match the tool table in §9 — no tools in constraints that aren't in the tools table

### Traceability Checks
- [ ] Every `C-NNN` constraint in §8 has either a `DA-CNNN` assertion in §15, or an inline `# no-assert: [reason]` comment
- [ ] Every anti-pattern in §7 references a specific alternative (pattern ID or named approach)
- [ ] Every integration in §11 appears in at least one diagram in §13
- [ ] RBAC roles in §10.3 are referenced in at least one constraint in §8.3 or §10.4

### Consistency Checks
- [ ] `status` and `lifecycle_stage` are compatible (experimental patterns cannot be `approved`)
- [ ] If `extends:` is set, the parent pattern's constraints are not contradicted here
- [ ] If `requires:` is set, those patterns exist (or are noted as pending)
- [ ] Variant IDs in §5 are referenced by use cases in §6
- [ ] Drift assertions in §15 reference valid constraint IDs that exist in §8

### Diagram Checks
- [ ] At least one mermaid diagram present
- [ ] All tools in §9 appear in the Container diagram (or there's a reason they don't)
- [ ] Sequence diagram shows credential/secret retrieval if Vault or secrets store is in scope

### Full Pattern Report Format

```
## Pattern Validation Report
Pattern: [ID] — [Name]  |  Version: [x.y.z]  |  Date: [Date]

### Summary
[X] blocking issues  |  [Y] warnings  |  [Z] checks passed

### 🔴 Blocking Issues (must fix before status change)
1. [Rule]: [Finding] — [How to fix]

### 🟡 Warnings (should fix, non-blocking)
1. [Rule]: [Finding] — [Suggestion]

### ✅ Passed
[Collapsed list of passing checks]

### Traceability Matrix
| Constraint | Drift Assertion | Status |
|---|---|---|
| C-001 | DA-C001 | ✅ Traced |
| C-002 | — | ⚠️ Missing assertion |

### Status Change Recommendation
APPROVED TO ADVANCE / NOT READY
Target status: [under-review / approved]
[One paragraph rationale]
```

---

## What Validation Does NOT Do
- Does not verify that diagrams render correctly (run `mermaid --validate` in CI for that)
- Does not check that external tool references (version numbers, doc URLs) are current — invoke the Research skill for that
- Does not make governance decisions — it surfaces findings for human review
- Does not approve a pattern — that is always a human sign-off in §16
