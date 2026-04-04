# AI Agent Rules for Platform Manifesto

These rules apply to any AI agent (Cursor, Copilot, or similar) making changes to this repository.

---

## Writing Style

- NEVER use em-dashes (" - "). Use regular dashes with spaces (" - ") or rewrite the sentence.
- Use straight quotes, not curly/smart quotes.
- Use `{Company}` as the company name placeholder everywhere. Never use a real company name.
- Write in second person ("you") or first person plural ("we").
- Be direct and opinionated. State rules as rules: "All services must..." not "Services should ideally..."
- No trailing whitespace. Files end with a single newline.

---

## Document Structure (every content doc)

Every numbered `.md` file in a section folder MUST have:

1. **Title with emoji**: `# 🏷️ Document Title`
2. **Shields.io badges** (one line): status (Mandated=blue, Guidance=orange), owner (purple), updated year (green)
3. **Numbered sections with emoji headers**: `## 1. 🎯 Overview`
4. **Centered navigation footer**:
```markdown
---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
```

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

---

## File Naming

- Content docs: `NN-kebab-case-name.md` (e.g., `01-tech-stack.md`)
- Section folders: `NN-kebab-case/` (e.g., `01-platform-standards/`)
- Root files: UPPERCASE (`README.md`, `GLOSSARY.md`, `ONBOARDING.md`, `AGENTS.md`)

---

## Forbidden Content

- No real company names, employee names, or internal URLs
- No ride-hailing terminology: Jeeny, rider, ride-hail, fare, pickup, dropoff, Trips Service, Matching Engine, Surge Pricing
- No API keys, passwords, or tokens (even examples must be obviously fake)
- No region-specific references tied to a particular company

---

## Section README Format

```markdown
# 📐 NN -- Section Title

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

## Pre-Commit Checklist

- [ ] No em-dashes (" - ") in changed files
- [ ] No ride-hailing or company-specific terminology
- [ ] All internal markdown links resolve
- [ ] Section README lists all files in the directory
- [ ] Shields.io badge year is current
- [ ] Every new diagram follows the mermaid rules above
