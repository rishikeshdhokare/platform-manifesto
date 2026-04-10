# AI Agent Rules for Platform Manifesto

These rules apply to any AI agent (Cursor, Copilot, or similar) making changes to this repository.

> **This manifesto is agent-native.** Beyond governing edits to this repo, the manifesto is designed to be consumed as organizational context by AI agents operating across your engineering organization. Index it into your agent orchestration layer (RAG, context window, or tool retrieval) so agents follow the same standards as human engineers. See [Agent-Native by Design](./README.md#-agent-native-by-design) for details.

---

## Writing Style

- NEVER use em-dashes (`—`, Unicode U+2014) or en-dashes (`–`, Unicode U+2013). Use a hyphen-minus with spaces (` - `) or a plain hyphen. Rewrite the sentence if needed.
- Use straight quotes, not curly/smart quotes.
- Use placeholders for the company name. Never use a real company name.
  - `{Company}` (title case) for display names and prose (e.g., "Welcome to {Company}").
  - `{company}` (lowercase) for URLs, packages, Git orgs, and technical identifiers (e.g., `com.{company}.orders`, `api.{company}.com`).
- Write in second person ("you") or first person plural ("we").
- Be direct and opinionated. State rules as rules: "All services must..." not "Services should ideally..."
- No trailing whitespace. Files end with a single newline.

---

## Document Structure (every content doc)

Every numbered `.md` file in a section folder MUST have:

1. **Title with emoji**: `# 🏷️ Document Title`
2. **Shields.io badges** (one line): status badge, owner (purple), updated year (green). Valid statuses: Mandated (blue), Guidance (orange), Reference (blue), Active (green), Active Program (blue)
3. **Numbered sections with emoji headers**: `## 🎯 1. Overview`
4. **Centered navigation footer**:
```markdown
---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
```

**Accessibility note:** Emoji in headings improves visual scanning but can affect screen readers. For organizations requiring strict accessibility compliance, consider a plain-title variant. The emoji convention is a style choice, not a structural requirement.

---

## Index Updates (CRITICAL)

When adding, removing, or renaming ANY document:

1. Update the **section README.md** file table
2. Update the **root README.md** collapsible section table
3. Search ALL .md files for references to old filenames and update them
4. Verify no broken internal links

---

## Mermaid Diagrams

- Max **10-12 nodes** per diagram. Split if larger.
- Max **3 words** per node label. camelCase IDs.
- Max **2 words** per edge label. Wrap specials in quotes.
- Max **1 level** of subgraph nesting.
- ZERO `style`, `classDef`, or `:::`. Let theme handle colors.
- Shapes: `[rectangle]` for steps, `[(cylinder)]` for DBs, `{diamond}` for decisions.
- Add `**Visual overview:**` label before each diagram.

**Exception:** The root `README.md` landing page may use larger overview diagrams for navigation and orientation purposes. These diagrams are exempt from the node count and subgraph limits above, but must still avoid `style`, `classDef`, or `:::` directives.

---

## File Naming

- Content docs: `NN-kebab-case-name.md` (e.g., `01-tech-stack.md`)
- Section folders: `NN-kebab-case/` (e.g., `01-platform-standards/`)
- Root files: UPPERCASE (`README.md`, `GLOSSARY.md`, `ONBOARDING.md`, `AGENTS.md`)

---

## Forbidden Content

- No real company names, employee names, or internal URLs. Templated URLs using the `{company}` placeholder (e.g., `https://backstage.internal.{company}.com`) are allowed and encouraged.
- No ride-hailing terminology: Jeeny, rider, ride-hail, fare, pickup, dropoff, Trips Service, Matching Engine, Surge Pricing
- No API keys, passwords, or tokens (even examples must be obviously fake)
- No region-specific references tied to a particular company

---

## Section README Format

```markdown
# 📐 NN - Section Title

> One-line description.

| Document | Description |
|----------|-------------|
| [01-filename.md](./01-filename.md) | Brief description |

---
<div align="center">

🏠 [Back to root](../README.md)

</div>
```

---

## Pull Request Descriptions

> **Scope:** This rule applies to PRs in **this manifesto repository** only, because the manifesto is vendor-neutral documentation. Application repositories have different disclosure requirements - see the [code review guide](./03-engineering-practices/06-code-review-guide.md) for agent-authored code provenance standards.

When creating a pull request in this repository, the title and body must NEVER reference AI tooling:

- Do not mention Cursor, Copilot, ChatGPT, Claude, or any AI assistant by name
- Do not use phrases like "AI-generated", "auto-generated", or "assisted by"
- Write the PR as a human would - focus on **what** changed and **why**

A PR template exists at `.github/pull_request_template.md`. Populate every section of the template when creating PRs.

---

## Pre-Commit Checklist

- [ ] No em-dashes (`—`) or en-dashes (`–`) in changed files
- [ ] No ride-hailing or company-specific terminology
- [ ] All internal markdown links resolve
- [ ] Section README lists all files in the directory
- [ ] Shields.io badge year is current
- [ ] Every new diagram follows the mermaid rules above
