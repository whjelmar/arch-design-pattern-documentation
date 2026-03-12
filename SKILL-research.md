# SKILL: Pattern Research

## Purpose
Fetch authoritative information about tools and vendors referenced in a pattern card or pattern document. Populates the Tools section, authentication details, and integration constraints with real, current information from vendor documentation rather than requiring the architect to manually research each tool.

## When to Activate
- Architect mentions a tool by name and wants its capabilities, auth model, or pricing tier documented
- A pattern card has tool names in §Tools but no descriptions
- Architect asks "what does [tool] support for auth?" or "what are [tool]'s connector limits?"
- Architect is comparing two tools for a role in a pattern

## Behavior

### Autonomy Level
Fetch and summarize **autonomously**. Present findings for human review before writing them into constraint language. Never write a MUST/MUST NOT constraint from research alone — flag it as a candidate constraint and ask the architect to confirm.

### Research Sequence
For each tool identified, execute in this order:

1. **Homepage + product overview**  
   `https://[tool-website].com` — confirms current product positioning and major capabilities

2. **Security / Trust documentation**  
   Look for: `/security`, `/trust`, `/compliance`, or search `[tool name] security documentation`  
   Extract: auth mechanisms supported, encryption standards, compliance certifications (SOC2, ISO27001, HIPAA)

3. **Integration / connector documentation**  
   Look for: `/docs/integrations`, `/connectors`, `/api`  
   Extract: supported sources/targets, protocol (REST/gRPC/webhook), rate limits, schema change behavior

4. **Pricing / tier page** _(if relevant to license tier column)_  
   Extract: which features are Enterprise vs. free tier — flag any pattern constraints that require paid features

### Output Format
Present research as a structured briefing before inserting into any document:

```
## Research Brief: [Tool Name]

**Source URLs visited:**
- [url1]
- [url2]

**Auth mechanisms supported:**
- [finding]

**Relevant certifications:**
- [finding]

**Key integration behaviors:**
- [finding]

**Candidate constraints (needs architect confirmation):**
- CANDIDATE: Implementations SHOULD use [auth mechanism] because [reason from docs]
- CANDIDATE: [Tool] MUST be version >= [x.y] because older versions lack [feature]

**Gaps / couldn't find:**
- [anything that needs manual lookup]
```

### Specific Sites to Prioritize
When researching tools common to the L&A data platform domain, prioritize these doc roots:

| Tool | Primary Doc URL |
|---|---|
| Fivetran | https://fivetran.com/docs |
| Dagster | https://docs.dagster.io |
| Snowflake | https://docs.snowflake.com |
| HighTouch | https://hightouch.com/docs |
| dbt | https://docs.getdbt.com |
| Okta | https://developer.okta.com/docs |
| HashiCorp Vault | https://developer.hashicorp.com/vault/docs |
| Temporal | https://docs.temporal.io |
| n8n | https://docs.n8n.io |

### What NOT to Do
- Do not cite blog posts or community forums as authoritative sources for constraint language
- Do not pull pricing as a constraint without confirming with the architect
- Do not invent version numbers — if a minimum version can't be confirmed from docs, leave a `[NEEDS CONFIRMATION]` placeholder
- Do not fetch internal company URLs without explicit instruction
