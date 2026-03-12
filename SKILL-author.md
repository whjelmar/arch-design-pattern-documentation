# SKILL: Pattern Authoring

## Purpose
Convert a completed Pattern Card into a full Architectural Design Pattern document, or draft individual sections of an in-progress pattern. Works section by section, presenting drafts for review before committing. Never produces an `approved` status document — that is a human governance action.

## When to Activate
- Architect says "promote this card to a full pattern"
- Architect says "draft the constraints section" or "write up the RBAC table"
- A pattern document has placeholder text (`_TODO:`) that needs filling in
- Architect wants to add a new variant or anti-pattern to an existing doc

## Behavior

### Autonomy Level
Draft content autonomously from available context. **Always present a section draft for review before moving to the next section.** Ask one clarifying question per section if something critical is ambiguous — don't ask multiple questions at once.

For constraint language specifically: present candidate constraints, explain the reasoning, and ask the architect to confirm or adjust the RFC 2119 level (MUST vs. SHOULD vs. MAY) before finalizing.

### Promotion Sequence (Pattern Card → Full ADP)

Work through these sections in order. Each section is a checkpoint — get confirmation before proceeding.

```
1. Identity & Relationships  (frontmatter + §1 + §2)
2. Summary & Problem         (§3 + §4)
3. Variants                  (§5) — ask architect to enumerate variants first
4. Use Cases & Anti-Patterns (§6 + §7)
5. Constraints               (§8) — derive from card's Non-Negotiables + Hard Nos
6. Tools                     (§9) — pull from Research skill output if available
7. Auth & RBAC               (§10) — derive from card's Who Can Do What
8. Integration Points        (§11)
9. Data Flow                 (§12) — derive from card's flow narrative
10. Diagrams                 (§13) — invoke Diagram skill
11. ALM Integration          (§14)
12. Drift Detection          (§15) — derive from §8 constraints
13. Review, References, Log  (§16–18)
```

### Constraint Authoring Rules
When converting plain-language non-negotiables from a Pattern Card into RFC 2119 constraints:

- Use **MUST** when violation would make the implementation non-compliant or create a security/compliance risk
- Use **MUST NOT** for explicit prohibitions — especially around credentials, data handling, and tool substitution
- Use **SHOULD** for strong recommendations with legitimate edge-case exceptions
- Use **MAY** for optional capabilities the pattern explicitly permits
- Assign IDs sequentially: `C-001`, `C-002`, etc. — never reuse or skip IDs
- Every MUST/MUST NOT constraint gets a corresponding drift detection assertion in §15 — author them together
- Every constraint gets exactly one sentence. If it needs two sentences, it's two constraints.

### Handling Ambiguity
If the Pattern Card is unclear on a critical point, use this resolution order:
1. Check if the Research skill has already fetched relevant vendor documentation
2. Ask one targeted question to the architect
3. Insert a `[NEEDS DECISION: description]` placeholder and move on — don't block

### Pattern Relationship Inference
When the architect describes a pattern, actively look for implied relationships:
- If the pattern uses Vault → suggest `requires: ADP-SEC-001`
- If the pattern emits telemetry → suggest `requires: ADP-OBS-001`  
- If a similar pattern already exists in the index → flag potential conflict or extension opportunity
- If the pattern is a specialization of a broader approach → suggest `extends:` candidate

### Tone and Language
- Write for an audience of architects and senior engineers, not executives
- Avoid passive voice in constraint language ("Data MUST be encrypted" not "Encryption of data is required")
- Avoid hedging in MUST constraints — if it's a MUST, say it plainly
- Tables over prose where structure exists; prose over tables for narrative sections (§3, §4, §12.1)
