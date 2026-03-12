# SKILL: Pattern Diagramming

## Purpose
Generate syntactically valid Mermaid diagrams for architectural pattern documents from plain-language descriptions, existing pattern card content, or data flow narratives. Produces both a Lite (C4 Level 1) and Full (C4 Level 2) view plus a sequence diagram. Validates that diagrams render before inserting into a document.

## When to Activate
- Architect says "generate the diagram" or "draw this out"
- A pattern document has the §13 diagram scaffolds but no actual diagram content
- Architect pastes a flow description and asks for it to be visualized
- A diagram exists but is out of sync with updated pattern prose

## Behavior

### Autonomy Level
Generate diagrams autonomously from available context. Present each diagram with a plain-English description of what it shows. Ask for corrections in terms of the architecture ("should Vault be inside or outside the system boundary?"), not in terms of Mermaid syntax.

### Diagram Selection Logic

| Request | Diagrams to Generate |
|---|---|
| Full pattern promotion | All three: Context + Container + Sequence |
| "Quick diagram" / lite request | Context (C4 L1) only |
| "Show the data flow" | Sequence diagram only |
| "Show the system" | Container (C4 L2) only |
| Pattern Card promotion | Start with Context, add others after architect confirms scope |

### Generation Rules

**C4 Context Diagram (Lite)**
- Maximum 5–6 nodes. If more are needed, the pattern is probably two patterns.
- Include: primary human actor, the core system, 2–3 external systems
- Use `C4Context` not generic flowchart — this is the canonical lite view
- Label relationships with protocol where known (HTTPS, SQL, OTLP, etc.)

**C4 Container Diagram (Full)**
- Show every tool as a Container or ContainerDb within the system boundary
- External systems (IDP, observability, secrets) go outside the boundary
- Service accounts and auth mechanisms go on the relationship arrows, not as separate nodes
- Include the `UpdateLayoutConfig` directive to control density

**Sequence Diagram**
- Always start with the initiating trigger (scheduler, human, event) as an `actor`
- Show credential/secret retrieval as explicit steps — don't hide auth
- Include the observability emission step at the end
- Add a `Note` for the most important failure mode or SLA constraint

### Mermaid Syntax Guardrails
These are common rendering failures to avoid:

```
❌ Avoid: Node IDs with spaces or special chars
   Container(my node, ...)  →  ✅ Container(myNode, ...)

❌ Avoid: Missing quotes around labels with commas
   Rel(a, b, Read, Write)  →  ✅ Rel(a, b, "Read, Write")

❌ Avoid: C4Context with ContainerDb nodes (wrong diagram type)

❌ Avoid: sequenceDiagram participant names with spaces
   participant My Service  →  participant MyService as "My Service"

❌ Avoid: Very long Rel labels — Confluence truncates > ~40 chars
```

### Output Format
Present each diagram with:

```markdown
### [Diagram Name]

> What this shows: [1-sentence plain English description]

[mermaid code block]

> Review notes:
> - [Node X] is shown outside the boundary — move it inside if [condition]
> - [Relationship Y] uses [protocol] — confirm this is correct
```

### Sync Check
After generating diagrams, compare them against the prose in §11 (Integration Points) and §12 (Data Flow). Flag any integration point in the table that doesn't appear in the diagram, and any node in the diagram that doesn't correspond to a documented integration point. These are documentation defects.
