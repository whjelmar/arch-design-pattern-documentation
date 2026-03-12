# Pattern Card: [Pattern Name]

> **Fill this out first.** This is a draft document. It graduates to a full Architectural Design Pattern after EA review.  
> ⏱️ Target: 30–45 minutes to complete a first draft.

---

## The Basics

| | |
|---|---|
| **Pattern Name** | |
| **Your Name / Team** | |
| **Date** | |
| **Domain** | `data-movement` / `identity` / `integration` / `security` / `compute` / `other` |
| **Replaces / Related To** | _(Any existing approach this supersedes or sits next to)_ |

---

## What Problem Does This Solve?

> In 3–5 sentences: What keeps happening that this pattern fixes? Who feels the pain?

_Write here._

---

## When Should Someone Use This?

> Describe 2–3 situations where this is the right choice.

- **Use it when:** 
- **Use it when:** 
- **Use it when:** 

---

## When Should Someone NOT Use This?

> Describe situations where this looks applicable but isn't. Name what to use instead.

| Don't use it when... | Use this instead |
|---|---|
| | |
| | |

---

## The Tools

> List every tool involved. One row per tool.

| Tool | What It Does in This Pattern |
|---|---|
| | |
| | |

---

## How Data or Control Flows

> Draw or describe the flow. A simple left-to-right narrative is fine.  
> The AI assistant can convert this into a proper diagram — just write it plainly.

```
Example: 
Salesforce → [Fivetran extracts nightly] → Snowflake LANDING table 
→ [Dagster asset runs] → Snowflake CURATED table → BI tool
```

_Your flow here:_

---

## Who Can Do What

> List the key roles (human and system/service) and what they're allowed to do.

| Role | Human or Service? | What They Can Do |
|---|---|---|
| | | |
| | | |

---

## The Non-Negotiables

> What are the 3–5 things that MUST be true for any use of this pattern to be valid?  
> Plain language is fine — the AI assistant will formalize these into RFC 2119 language.

1. 
2. 
3. 
4. _(optional)_
5. _(optional)_

---

## The Hard Nos

> What are 2–3 things that would make an implementation NON-compliant, even if it looks like this pattern?

1. 
2. 
3. 

---

## Open Questions

> What are you unsure about? What decisions haven't been made yet?

- [ ] 
- [ ] 

---

## Ready to Escalate?

> When this card is filled out, tag it for EA review. The AI authoring assistant will promote this to a full pattern document.

- [ ] All sections above completed (blanks are OK for Open Questions)
- [ ] At least one other person has read this and it makes sense to them
- [ ] Tagged `status: ready-for-review` in frontmatter below

```yaml
---
pattern_card_id: PC-000
status: draft          # draft | ready-for-review
created: YYYY-MM-DD
owner: ""
domain: ""
---
```

---

_Pattern Card v1.0 — promotes to ADP template after EA review._
