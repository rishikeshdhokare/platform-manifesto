# Contributing to the {Company} Platform Manifesto

Thank you for your interest in contributing to the Platform Engineering Manifesto. Whether you are fixing a typo, proposing a new section, or adopting this manifesto for your own organization, this guide covers the process.

## Table of Contents

- [How to Contribute](#how-to-contribute)
- [Writing Style](#writing-style)
- [Document Structure](#document-structure)
- [What Makes a Good PR](#what-makes-a-good-pr)
- [Adopting and Customizing for Your Organization](#adopting-and-customizing-for-your-organization)
- [Proposing New Sections or Major Changes](#proposing-new-sections-or-major-changes)
- [Code of Conduct](#code-of-conduct)
- [License](#license)

---

## How to Contribute

We use a standard fork-and-PR workflow.

### 1. Fork the repository

Click **Fork** on GitHub to create your own copy.

### 2. Create a feature branch

```bash
git checkout -b your-branch-name
```

Use a descriptive branch name: `add-caching-patterns`, `fix-broken-links`, `update-ci-practices`.

### 3. Make your changes

Edit or add files following the [writing style](#writing-style) and [document structure](#document-structure) rules below. Run through the [PR checklist](#pr-checklist) before committing.

### 4. Commit and push

```bash
git add .
git commit -m "Add caching patterns to developer guides"
git push origin your-branch-name
```

Write commit messages that explain **what** changed and **why**.

### 5. Open a Pull Request

Open a PR against `main`. Fill out every section of the [PR template](./.github/pull_request_template.md). At least one Staff Engineer or Principal must approve before merging.

---

## Writing Style

These rules keep the manifesto consistent and machine-parseable. For the full set of rules (including diagram conventions and naming standards), see [AGENTS.md](./AGENTS.md).

| Rule | Details |
|:-----|:--------|
| **No em-dashes or en-dashes** | Never use `—` or `–`. Use a hyphen-minus with spaces (` - `) or rewrite the sentence |
| **Straight quotes only** | Use `"` and `'`, never curly or smart quotes |
| **Company placeholders** | Use `{Company}` (title case) in prose and headings. Use `{company}` (lowercase) in URLs, packages, and technical identifiers |
| **Voice** | Write in second person ("you") or first person plural ("we") |
| **Be direct** | State rules as rules. "All services must..." not "Services should ideally..." |
| **No trailing whitespace** | Files end with a single newline |
| **No sensitive content** | No real company names, employee names, internal URLs, API keys, or passwords |

---

## Document Structure

Every numbered `.md` file inside a section folder must follow this structure. Root-level files (`README.md`, `GLOSSARY.md`, `CONTRIBUTING.md`, etc.) are exempt from these requirements.

### Required elements

1. **Title with emoji** - `# <emoji> Document Title`
2. **Shields.io badges** (on one line) - status badge, owner (purple), updated year (green)
3. **Numbered sections with emoji headers** - `## <emoji> 1. Overview`
4. **Centered navigation footer**:

```markdown
---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
```

### File naming

- Content docs: `NN-kebab-case-name.md` (e.g., `01-tech-stack.md`)
- Section folders: `NN-kebab-case/` (e.g., `01-platform-standards/`)
- Root files: UPPERCASE (e.g., `README.md`, `GLOSSARY.md`)

### Index updates (critical)

When you add, remove, or rename any document, you **must**:

1. Update the **section README.md** file table
2. Update the **root README.md** collapsible section table
3. Search all `.md` files for references to old filenames and update them
4. Verify no broken internal links remain

---

## What Makes a Good PR

### Content standards

- Adds value that helps engineers ship better, faster, or more reliably
- Follows the writing style and document structure rules above
- Does not duplicate content already covered in another section
- Cross-references related documents using relative markdown links

### PR checklist

Before submitting, verify every item. This mirrors the [PR template](./.github/pull_request_template.md).

- [ ] All internal markdown links resolve
- [ ] Section and root README indexes updated (if docs were added, removed, or renamed)
- [ ] No em-dashes or en-dashes in changed files
- [ ] No curly or smart quotes
- [ ] No company-specific terminology or forbidden content
- [ ] Shields.io badge year is current (if applicable)
- [ ] Mermaid diagrams follow repository conventions (if applicable)
- [ ] New documents have emoji title, badges, numbered sections, and footer

---

## Adopting and Customizing for Your Organization

This manifesto is designed to be forked and adapted. The principles are universal; the specific tools are a reference implementation.

### Step 1: Fork the repository

```bash
gh repo fork {company}/platform-manifesto --clone
```

### Step 2: Replace company placeholders

The manifesto uses two placeholders throughout:

| Placeholder | Usage | Example |
|:------------|:------|:--------|
| `{Company}` | Headings, prose, display names | "Welcome to {Company}" |
| `{company}` | URLs, packages, Git orgs, technical identifiers | `com.{company}.orders` |

Find and replace both with your organization name:

```bash
# Preview what will change
grep -r "{Company}" --include="*.md" -l
grep -r "{company}" --include="*.md" -l

# Replace (use your preferred tool - sed, IDE find/replace, etc.)
```

### Step 3: Swap in your tech stack

Search for specific tool names (e.g., "Spring Boot", "Aurora", "EKS") and replace them with your equivalents. The [Customize This Manifesto](./README.md#-customize-this-manifesto) section in the root README provides a mapping table of universal principles to reference tools.

### Step 4: Select your sections

Not every section applies to every organization. You can:

- **Remove entire section folders** you do not need (e.g., `11-domain-catalog/` if you have your own service docs)
- **Add new sections** following the `NN-kebab-case/` naming convention
- **Renumber sections** if you remove or reorder them (update all cross-references)

After removing or adding sections, update the root `README.md` and verify all internal links.

### Step 5: Wire it into your agents

If you use AI agents in your engineering workflows, index the manifesto into your agent orchestration layer. See [Context Engineering](./12-ai-engineering/01-context-engineering.md) for guidance on AGENTS.md files, cursor rules, and RAG pipelines.

---

## Proposing New Sections or Major Changes

Small fixes (typos, broken links, clarifications) can go straight to a PR. For larger changes, use the RFC process.

### When to write an RFC

- Adding a new section to the manifesto
- Changing a fundamental principle or non-negotiable
- Proposing a major restructure of existing content
- Introducing a new technology standard that affects multiple sections

### RFC process

1. **Open a GitHub Issue** describing the proposed change, its motivation, and the sections it affects
2. **Draft an RFC** following the process in [RFC Process](./07-ways-of-working/05-rfc-process.md)
3. **Discuss** in `#engineering-discussions` or an Architecture Clinic session
4. **Get approval** from at least one Staff Engineer or Principal
5. **Implement** the approved changes via PR

For questions or informal discussion before writing an RFC, use `#engineering-discussions` on Slack or open a GitHub Discussion.

---

## Code of Conduct

We are committed to providing a welcoming and inclusive environment. All contributors are expected to:

- Be respectful and constructive in all interactions
- Welcome newcomers and help them contribute
- Focus feedback on the content, not the person
- Accept constructive criticism gracefully

Unacceptable behavior includes harassment, trolling, personal attacks, and publishing others' private information. Maintainers have the right and responsibility to remove contributions that violate these standards.

Report concerns to the Platform Engineering team leads.

---

## License

This manifesto is licensed under [Creative Commons Attribution 4.0 International (CC BY 4.0)](./LICENSE).

You are free to:

- **Share** - copy and redistribute the material in any medium or format
- **Adapt** - remix, transform, and build upon the material for any purpose, including commercial use

Under the following terms:

- **Attribution** - you must give appropriate credit, provide a link to the license, and indicate if changes were made

See the full [LICENSE](./LICENSE) file for details.
