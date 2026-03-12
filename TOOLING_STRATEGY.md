# Architectural Pattern Document — Tooling Strategy

> Covers: Copier scaffolding · AI-assisted authoring · CI/CD validation linting  
> Phase 3 (Extraction) and Phase 4 (Drift Detection Engine) are referenced but scoped separately.

---

## 1. Overview

Three tools work together to reduce the cost of creating and maintaining high-quality pattern documents:

| Tool | Phase | Purpose |
|---|---|---|
| **Copier Template** | Creation | Scaffold a new pattern skeleton from answered prompts |
| **Claude Authoring Assistant** | Creation / Update | AI-assisted drafting of pattern sections from structured Q&A |
| **Pattern Linter (CI)** | Validation | Enforce template completeness and constraint traceability before merge |

---

## 2. Copier Scaffolding Template

### 2.1 What It Does

A Copier project template that generates a properly structured pattern markdown file when invoked. The architect answers a series of prompts and receives a pre-populated skeleton — frontmatter filled, section headers in place, placeholder constraint IDs allocated, and relevant boilerplate for the declared pattern type.

### 2.2 Repository Structure

```
adp-template/                        ← Copier template repo
├── copier.yaml                       ← Prompt definitions and logic
├── template/
│   ├── docs/
│   │   └── patterns/
│   │       └── {{pattern_id}}-{{pattern_slug}}.md.jinja   ← Jinja2 template
│   └── drift/
│       └── {{pattern_id}}-assertions.yaml.jinja            ← Drift detection scaffold
└── scripts/
    └── post_generate.py             ← Post-generation hooks (index update, git branch)
```

### 2.3 copier.yaml — Prompt Definitions

```yaml
# copier.yaml
_min_copier_version: "9.0"
_jinja_extensions:
  - copier_templates_extensions.TemplateExtensionLoader

# ── Identity ──────────────────────────────────────────────────────────────
pattern_id:
  type: str
  help: "Pattern ID (format: ADP-DOMAIN-NNN, e.g. ADP-DM-001)"
  validator: "{% if not pattern_id | regex_search('^ADP-[A-Z]+-\\d{3}$') %}Invalid format{% endif %}"

pattern_name:
  type: str
  help: "Human-readable pattern name (e.g. SaaS ELT Data Movement)"

pattern_type:
  type: str
  help: "Pattern type"
  choices:
    - enterprise
    - solution
    - composite
  default: solution

domain:
  type: str
  help: "Domain this pattern belongs to"
  choices:
    - data-movement
    - identity
    - integration
    - security
    - compute
    - observability
    - networking
    - application

lifecycle_stage:
  type: str
  help: "Initial lifecycle stage"
  choices:
    - experimental
    - incubating
    - adopted
  default: experimental

owner:
  type: str
  help: "Owning team or individual (e.g. Data Platform Team)"

# ── Relationships ─────────────────────────────────────────────────────────
extends:
  type: str
  help: "Parent pattern ID this specializes (leave blank if none)"
  default: ""

requires:
  type: str
  help: "Comma-separated pattern IDs that must be in place (leave blank if none)"
  default: ""

is_composite:
  type: bool
  help: "Is this a composite pattern assembled from sub-patterns?"
  default: false

composed_of:
  type: str
  help: "Comma-separated component pattern IDs (only if composite)"
  default: ""
  when: "{{ is_composite }}"

# ── Tools ─────────────────────────────────────────────────────────────────
primary_tools:
  type: str
  help: "Comma-separated primary tools used (e.g. Fivetran, Dagster, Snowflake)"
  default: ""

# ── Security ──────────────────────────────────────────────────────────────
has_pii:
  type: bool
  help: "Does this pattern handle PII or Restricted data?"
  default: false

idp:
  type: str
  help: "Identity provider used"
  choices:
    - Okta
    - Azure AD
    - AWS IAM Identity Center
    - Other
  default: Okta

# ── Variants ──────────────────────────────────────────────────────────────
variant_count:
  type: int
  help: "How many variants does this pattern have?"
  default: 1
  validator: "{% if variant_count < 1 or variant_count > 6 %}Must be 1–6{% endif %}"

# ── Derived values ────────────────────────────────────────────────────────
pattern_slug:
  type: str
  help: "URL-safe slug for the pattern (e.g. saas-elt-data-movement)"
```

### 2.4 Jinja2 Template Excerpt

```jinja2
{# template/docs/patterns/{{pattern_id}}-{{pattern_slug}}.md.jinja #}
---
pattern_id: {{ pattern_id }}
pattern_name: "{{ pattern_name }}"
version: "0.1.0"
status: draft
lifecycle_stage: {{ lifecycle_stage }}
pattern_type: {{ pattern_type }}
domain: {{ domain }}
owner: "{{ owner }}"
reviewers: []
created_date: "{{ '%Y-%m-%d' | strftime }}"
last_reviewed: ""
next_review_due: ""
extends: [{% if extends %}"{{ extends }}"{% endif %}]
requires: [{% for r in requires.split(',') if r.strip() %}"{{ r.strip() }}"{% if not loop.last %}, {% endif %}{% endfor %}]
composed_of: [{% if is_composite %}{% for c in composed_of.split(',') if c.strip() %}"{{ c.strip() }}"{% if not loop.last %}, {% endif %}{% endfor %}{% endif %}]
conflicts_with: []
tags: [{{ domain }}]
---

# {{ pattern_name }} — Architectural Design Pattern

> **RFC 2119 Notice:** The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHALL**, **SHALL NOT**, **SHOULD**, **SHOULD NOT**, **RECOMMENDED**, **MAY**, and **OPTIONAL** in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

---

## 1. Pattern Identity

| Field | Value |
|---|---|
| **Pattern ID** | `{{ pattern_id }}` |
| **Name** | {{ pattern_name }} |
| **Version** | `0.1.0` |
| **Status** | `draft` |
| **Lifecycle Stage** | `{{ lifecycle_stage }}` |
| **Pattern Type** | `{{ pattern_type }}` |
| **Domain** | `{{ domain }}` |
| **Owner** | {{ owner }} |
| **Created** | {{ '%Y-%m-%d' | strftime }} |

{# ... remaining sections pre-populated from template ... #}

## 8. RFC 2119 Constraints

> **Note for author:** Assign constraint IDs sequentially starting from `C-001`.
> Every constraint MUST have a corresponding drift detection assertion in §15 or a documented rationale for why automated detection is not feasible.

### 8.1 Implementation Constraints

| ID | Constraint | Level |
|---|---|---|
| `C-001` | _TODO: Define first MUST constraint_ | MUST |

{# Generate variant tables based on variant_count #}
## 5. Pattern Variants

| Variant ID | Name | Status | Use When | Constraints Added |
|---|---|---|---|---|
{% for i in range(variant_count) -%}
| `VAR-0{{ i + 1 }}` | _TODO: Variant {{ i + 1 }} name_ | `draft` | _TODO: When to use_ | _TODO: Any additional constraints_ |
{% endfor %}
```

### 2.5 Post-Generate Hook

```python
# scripts/post_generate.py
import subprocess
import json
from pathlib import Path

def update_pattern_index(context):
    """Add the new pattern to the pattern index JSON."""
    index_path = Path("docs/patterns/_index.json")
    index = json.loads(index_path.read_text()) if index_path.exists() else {"patterns": []}
    
    index["patterns"].append({
        "id": context["pattern_id"],
        "name": context["pattern_name"],
        "domain": context["domain"],
        "status": "draft",
        "lifecycle_stage": context["lifecycle_stage"],
        "owner": context["owner"],
        "created": context["_now"].strftime("%Y-%m-%d"),
        "file": f"docs/patterns/{context['pattern_id']}-{context['pattern_slug']}.md"
    })
    
    index_path.write_text(json.dumps(index, indent=2))
    print(f"✅ Added {context['pattern_id']} to pattern index")

def create_git_branch(context):
    """Create a feature branch for the new pattern."""
    branch = f"pattern/{context['pattern_id'].lower()}-{context['pattern_slug']}"
    subprocess.run(["git", "checkout", "-b", branch], check=True)
    print(f"✅ Created branch: {branch}")

if __name__ == "__main__":
    # Copier injects context via environment
    import os, json
    context = json.loads(os.environ.get("COPIER_ANSWERS_FILE_CONTENT", "{}"))
    update_pattern_index(context)
    create_git_branch(context)
```

### 2.6 Usage

```bash
# Install Copier
pip install copier

# Generate a new pattern document
copier copy gh:your-org/adp-template ./

# Or point to an internal GitLab instance
copier copy git+https://gitlab.internal.com/ea/adp-template.git ./
```

---

## 3. AI-Assisted Authoring

### 3.1 What It Does

A Claude-powered authoring assistant that conducts a structured interview with the architect and drafts pattern sections. It can be invoked as a standalone artifact (browser-based) or as a CLI tool for integration into the terminal workflow.

### 3.2 Authoring Flow

```
1. Architect invokes assistant with pattern_id and domain
2. Assistant loads the section-by-section question bank
3. Architect answers questions conversationally
4. Assistant drafts each section in compliant template format
5. Architect reviews, edits, and approves each section
6. Assistant assembles the full document and outputs it
7. Output can be committed directly or piped to the Copier template
```

### 3.3 System Prompt Design

The assistant uses a structured system prompt that:

- Knows the full template schema (sections, required fields, constraint ID format)
- Knows the approved tool list and technology radar status for each tool
- Understands RFC 2119 constraint phrasing rules
- Can generate syntactically valid Mermaid C4 diagrams from described architectures
- Generates drift detection YAML assertions from constraints it helps define
- Flags when a described use case matches an existing anti-pattern

```markdown
SYSTEM PROMPT (abbreviated):

You are an Enterprise Architecture assistant helping author pattern documents
for a PE firm's L&A portfolio architecture team. You follow the approved pattern
template strictly. When authoring constraints, always assign an ID in the format
C-NNN and use RFC 2119 language (MUST, MUST NOT, SHOULD, etc.) precisely.

For every constraint you author, generate a corresponding drift detection
assertion in the §15 YAML format before moving to the next section.

Approved tools by domain (current as of [date]):
- data-movement: Fivetran, Dagster, Snowflake, HighTouch, dbt Core
- [other domains...]

When the architect describes an integration or data flow, generate both the
C4 Container mermaid diagram AND the sequence diagram. Ask clarifying questions
if the described architecture is ambiguous.

Never author a pattern with status 'approved' — always output 'draft'. Approval
is a human governance step.
```

### 3.4 CLI Tool Design

```bash
# Installation
npm install -g @your-org/adp-author   # or pip install adp-author

# Invoke for a new pattern
adp-author new --id ADP-DM-002 --domain data-movement

# Continue authoring an existing draft
adp-author continue --file docs/patterns/ADP-DM-002-custom-elt.md

# Draft a specific section only
adp-author section --file ADP-DM-002-custom-elt.md --section "constraints"

# Validate after authoring (calls linter)
adp-author validate --file ADP-DM-002-custom-elt.md
```

### 3.5 Claude API Integration

```typescript
// adp-author/src/claude-client.ts
const CLAUDE_API_URL = "https://api.anthropic.com/v1/messages";
const MODEL = "claude-sonnet-4-20250514";

interface AuthoringSession {
  patternId: string;
  domain: string;
  conversationHistory: Message[];
  currentSection: string;
  draftSections: Record<string, string>;
}

async function authorSection(
  session: AuthoringSession,
  userInput: string
): Promise<{ response: string; draftContent?: string }> {
  
  const systemPrompt = await loadSystemPrompt(); // loads from adp-author/prompts/system.md
  
  session.conversationHistory.push({
    role: "user",
    content: userInput
  });

  const response = await fetch(CLAUDE_API_URL, {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({
      model: MODEL,
      max_tokens: 4096,
      system: systemPrompt,
      messages: session.conversationHistory
    })
  });

  const data = await response.json();
  const assistantMessage = data.content[0].text;
  
  session.conversationHistory.push({
    role: "assistant",
    content: assistantMessage
  });

  // Extract drafted content if present (wrapped in ```pattern ... ``` blocks)
  const draftMatch = assistantMessage.match(/```pattern\n([\s\S]*?)```/);
  
  return {
    response: assistantMessage,
    draftContent: draftMatch?.[1]
  };
}
```

---

## 4. Pattern Linter (CI Validation)

### 4.1 What It Does

A CI job (GitLab CI / GitHub Actions) that validates committed pattern documents against the template schema before merge. It catches structural defects, missing required sections, orphaned constraint IDs (constraints without drift assertions), and frontmatter inconsistencies.

### 4.2 Validation Rules

| Rule ID | Rule | Severity | Description |
|---|---|---|---|
| `LINT-001` | Required frontmatter fields | Error | All frontmatter fields in the template must be present and non-empty |
| `LINT-002` | Pattern ID format | Error | Must match `ADP-[A-Z]+-\d{3}` |
| `LINT-003` | Status/lifecycle consistency | Error | `experimental` or `incubating` patterns MUST NOT have status `approved` |
| `LINT-004` | Required sections present | Error | All 18 template sections must be present |
| `LINT-005` | Constraint ID format | Error | All constraint IDs must match `C-\d{3}` and be sequential |
| `LINT-006` | Constraint/assertion traceability | Warning | Every `C-NNN` constraint must have a corresponding `DA-C-NNN` assertion in §15, or a `# no-assert: [reason]` inline comment |
| `LINT-007` | Diagram present | Warning | §13 must contain at least one mermaid code block |
| `LINT-008` | Anti-pattern alternatives | Error | Every anti-pattern in §7 must reference a named alternative (pattern ID or approach) |
| `LINT-009` | Composition optionality resolved | Error | If `composed_of` is non-empty, §2.3 must document options resolved for each component |
| `LINT-010` | Review date not overdue | Warning | `next_review_due` must not be in the past for `approved` or `adopted` patterns |
| `LINT-011` | Drift assertion severity valid | Error | All drift assertion `severity` values must be one of: `critical`, `high`, `medium`, `low` |
| `LINT-012` | No placeholder text | Warning | Detects `_TODO:` or `_[` placeholders left in document |

### 4.3 Linter Implementation

```python
# adp-linter/linter.py
import re
import sys
import yaml
from pathlib import Path
from dataclasses import dataclass, field
from typing import List, Literal

@dataclass
class LintResult:
    rule_id: str
    severity: Literal["error", "warning"]
    message: str
    line: int | None = None

def lint_pattern(filepath: Path) -> List[LintResult]:
    results = []
    content = filepath.read_text()
    lines = content.splitlines()
    
    # Extract frontmatter
    fm_match = re.match(r'^---\n(.*?)\n---', content, re.DOTALL)
    if not fm_match:
        return [LintResult("LINT-001", "error", "No frontmatter found", 1)]
    
    try:
        frontmatter = yaml.safe_load(fm_match.group(1))
    except yaml.YAMLError as e:
        return [LintResult("LINT-001", "error", f"Frontmatter YAML parse error: {e}", 1)]
    
    # LINT-001: Required frontmatter fields
    required_fields = [
        "pattern_id", "pattern_name", "version", "status",
        "lifecycle_stage", "pattern_type", "domain", "owner"
    ]
    for f in required_fields:
        if not frontmatter.get(f):
            results.append(LintResult(
                "LINT-001", "error",
                f"Required frontmatter field '{f}' is missing or empty"
            ))
    
    # LINT-002: Pattern ID format
    pid = frontmatter.get("pattern_id", "")
    if not re.match(r'^ADP-[A-Z]+-\d{3}$', pid):
        results.append(LintResult(
            "LINT-002", "error",
            f"Pattern ID '{pid}' does not match format ADP-[A-Z]+-NNN"
        ))
    
    # LINT-003: Status/lifecycle consistency
    status = frontmatter.get("status", "")
    lifecycle = frontmatter.get("lifecycle_stage", "")
    if lifecycle in ("experimental", "incubating") and status == "approved":
        results.append(LintResult(
            "LINT-003", "error",
            f"Pattern with lifecycle_stage '{lifecycle}' cannot have status 'approved'"
        ))
    
    # LINT-004: Required sections
    required_sections = [
        "## 1. Pattern Identity",
        "## 2. Pattern Relationships",
        "## 3. Summary",
        "## 4. Context & Problem Statement",
        "## 5. Pattern Variants",
        "## 6. Acceptable Use Cases",
        "## 7. Unacceptable Use Cases",
        "## 8. RFC 2119 Constraints",
        "## 9. Tools",
        "## 10. Authentication & Authorization",
        "## 11. Integration Points",
        "## 12. Data Flow",
        "## 13. Architecture Diagrams",
        "## 14. Application Lifecycle Integration",
        "## 15. Drift Detection Specification",
        "## 16. Review & Approval",
        "## 17. References",
        "## 18. Changelog"
    ]
    for section in required_sections:
        if section not in content:
            results.append(LintResult(
                "LINT-004", "error",
                f"Required section '{section}' not found in document"
            ))
    
    # LINT-005: Constraint ID format and sequential ordering
    constraint_ids = re.findall(r'`(C-\d{3})`', content)
    for cid in constraint_ids:
        if not re.match(r'^C-\d{3}$', cid):
            results.append(LintResult("LINT-005", "error", f"Malformed constraint ID: {cid}"))
    
    # LINT-006: Every constraint must have a drift assertion or exemption comment
    unique_constraints = set(constraint_ids)
    drift_assertions = set(re.findall(r'constraint_ref:\s*(C-\d{3})', content))
    no_assert_exemptions = set(re.findall(r'#\s*no-assert:\s*(C-\d{3})', content))
    
    for cid in unique_constraints:
        if cid not in drift_assertions and cid not in no_assert_exemptions:
            results.append(LintResult(
                "LINT-006", "warning",
                f"Constraint {cid} has no drift detection assertion in §15. "
                f"Add a DA-{cid} assertion or add '# no-assert: {cid} [reason]' inline."
            ))
    
    # LINT-007: Diagram present
    if "```mermaid" not in content:
        results.append(LintResult(
            "LINT-007", "warning",
            "No mermaid diagram found in document. §13 requires at least one diagram."
        ))
    
    # LINT-012: Placeholder text detection
    placeholder_lines = [
        (i + 1, line) for i, line in enumerate(lines)
        if "_TODO:" in line or re.search(r'_\[.+\]_', line)
    ]
    for lineno, line in placeholder_lines:
        results.append(LintResult(
            "LINT-012", "warning",
            f"Placeholder text found: '{line.strip()}'",
            lineno
        ))
    
    return results

def main():
    errors = 0
    warnings = 0
    
    files = [Path(f) for f in sys.argv[1:] if f.endswith(".md")]
    
    for filepath in files:
        print(f"\n📄 Linting: {filepath}")
        results = lint_pattern(filepath)
        
        if not results:
            print("  ✅ All checks passed")
            continue
            
        for r in results:
            icon = "❌" if r.severity == "error" else "⚠️"
            loc = f" (line {r.line})" if r.line else ""
            print(f"  {icon} [{r.rule_id}]{loc} {r.message}")
            if r.severity == "error":
                errors += 1
            else:
                warnings += 1
    
    print(f"\n{'─'*60}")
    print(f"Results: {errors} error(s), {warnings} warning(s)")
    
    if errors > 0:
        print("❌ Linting failed — fix errors before merge")
        sys.exit(1)
    else:
        print("✅ Linting passed")
        sys.exit(0)

if __name__ == "__main__":
    main()
```

### 4.4 GitLab CI Integration

```yaml
# .gitlab-ci.yml (pattern validation job)

pattern-lint:
  stage: validate
  image: python:3.12-slim
  before_script:
    - pip install pyyaml --quiet
  script:
    - |
      # Find all changed pattern files in this MR
      CHANGED_PATTERNS=$(git diff --name-only origin/$CI_MERGE_REQUEST_TARGET_BRANCH_NAME...HEAD \
        | grep '^docs/patterns/ADP-.*\.md$' || true)
      
      if [ -z "$CHANGED_PATTERNS" ]; then
        echo "No pattern files changed — skipping lint"
        exit 0
      fi
      
      echo "Linting changed patterns:"
      echo "$CHANGED_PATTERNS"
      python adp-linter/linter.py $CHANGED_PATTERNS
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
  allow_failure: false

pattern-index-check:
  stage: validate
  image: python:3.12-slim
  script:
    - python adp-linter/check_index.py  # Verifies pattern index is up to date
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      changes:
        - docs/patterns/**/*.md
```

---

## 5. Rendering Compatibility Notes

| Feature | Confluence | GitLab | Obsidian |
|---|---|---|---|
| YAML frontmatter | ⚠️ Not rendered (ignored) | ✅ Rendered as metadata | ✅ Native support |
| Mermaid C4 diagrams | ✅ via Mermaid macro or plugin | ✅ Native (```` ```mermaid ```` ) | ✅ via Mermaid plugin |
| Mermaid sequence diagrams | ✅ | ✅ | ✅ |
| Tables | ✅ | ✅ | ✅ |
| Code blocks (YAML) | ✅ | ✅ | ✅ |
| Internal links (`[ADP-XXX]`) | ⚠️ Requires Confluence linking | ✅ GitLab wiki links | ✅ Native wikilinks |
| Checkboxes in tables | ❌ | ⚠️ Partial | ✅ |

**Confluence note:** The Mermaid for Confluence app (marketplace) or the Mermaid Diagrams plugin is required for diagram rendering. Frontmatter can be surfaced via a Confluence page property macro if desired.

**Obsidian note:** Install the Mermaid plugin. Frontmatter is natively indexed by Obsidian Dataview — this enables a live pattern catalogue query across all pattern documents.

```dataview
TABLE pattern_name, status, lifecycle_stage, domain, owner
FROM "docs/patterns"
WHERE pattern_id
SORT lifecycle_stage ASC, domain ASC
```

---

## 6. Phase Roadmap

| Phase | Scope | Status |
|---|---|---|
| **Phase 1** | Template + Copier scaffolding + AI authoring assistant + CI linter | 🟢 Defined here |
| **Phase 2** | Drift detection engine — ingests assertion YAML, runs checks against infrastructure, routes alerts | 🔵 Design pending |
| **Phase 3** | Extraction — reads existing Terraform / Helm / IaC and infers candidate pattern docs | 🔵 Future |
| **Phase 4** | Pattern catalogue UI — searchable, filterable, cross-referenced pattern registry | 🔵 Future |

---

_Tooling strategy version: 1.0.0 — Maintained by the Enterprise Architecture team._
