# 👀 Code Review Guide

![Status: Guidance](https://img.shields.io/badge/status-guidance-orange?style=flat-square) ![Owner: Engineering Leadership](https://img.shields.io/badge/owner-Engineering_Leadership-purple?style=flat-square) ![Updated: 2025](https://img.shields.io/badge/updated-2025-green?style=flat-square)

---

## 🎯 1. Why Code Review Matters

Code review is the most effective quality gate we have — not because it catches all bugs (it doesn't), but because it:
- Spreads knowledge about the codebase across the team
- Catches design problems before they're baked in
- Ensures standards are applied consistently
- Builds shared ownership — everyone is responsible for the codebase, not just the author

Done well, code review is a **collaborative and educational** experience. Done poorly, it is a source of friction, resentment, and delay.

---

## 📏 2. For the Author: Before You Raise a PR

### 2.1 Review Your Own Code First

Read every line of your diff before requesting review. Ask yourself:
- Is this the simplest possible solution?
- Have I left any debug code, commented-out code, or TODOs I don't intend to address?
- Would a colleague understand this without asking me?
- Have I tested the edge cases?

### 2.2 Keep PRs Small

The single most impactful thing you can do for your reviewers is to keep PRs small.

Research consistently shows: reviewers find fewer bugs in large PRs, spend less time per line, and give worse feedback — not because they're less diligent, but because reviewing 800 lines is cognitively exhausting.

**Target:** < 400 lines changed. **Hard limit:** 800 lines (requires justification in PR description).

If your change is large, break it into a sequence of smaller PRs:
```
PR 1: Add the database migration (20 lines)
PR 2: Add the domain model and unit tests (150 lines)
PR 3: Add the service logic and integration tests (200 lines)
PR 4: Add the API endpoint (100 lines)
```

### 2.3 Write a Useful PR Description

Don't make the reviewer figure out what you've changed and why. Tell them:

```markdown
## What
Adds price calculation for multi-stop orders. Previously, multi-stop orders 
used a flat rate regardless of total distance. Now, price is calculated 
per leg and summed.

## Why
Required for RIDE-1892. Providers reported flat-rate multi-stop orders were 
undercompensated for long routes.

## How it was tested
- Unit tests for PriceCalculationService cover: single-leg, two-leg, 
  three-leg orders; dynamic pricing interactions
- Integration test confirms persistence and event publishing
- Tested manually in dev with a 3-stop order: price was correctly summed

## Notes for reviewer
The PriceCalculationStrategy interface is new — if you think the strategy 
pattern is overkill here, happy to discuss. I added it because we'll 
likely need a different strategy for shared orders (RIDE-2001).
```

### 2.4 Respond to Every Comment

- If you've fixed it: "Done."
- If you disagree: Explain why, respectfully. Don't just silently ignore it.
- If you don't understand it: Ask for clarification.
- If you've discussed it offline: "Discussed with @reviewer — we agreed to [outcome]."

---

## 📏 3. For the Reviewer: How to Review Well

### 3.1 Review SLA

- **First review within 1 business day** — unreviewed PRs block your colleague's work
- If you're too busy: leave a comment saying when you'll review, or ask someone else to take it
- Stale PRs (no activity for 3 days) are auto-flagged to the team lead

### 3.2 What to Review

**Do review:**
- Correctness — does it do what it says?
- Test coverage — is the behaviour tested? Are edge cases covered?
- Design — is the solution appropriately simple? Are responsibilities in the right place?
- Security — no secrets, no injection risks, no overly permissive access
- API contracts — does it follow the [API standards](../02-architecture-and-api/02-api-standards.md)?
- Observability — does it log key events? Does it emit metrics for business-critical paths?
- Error handling — are errors handled correctly, not swallowed?

**Don't review (automated tools do this):**
- Formatting — Checkstyle handles this
- Style (indentation, spaces) — automated
- Import order — automated
- Coverage percentage — SonarCloud enforces this

### 3.3 How to Leave Good Feedback

The tone of a comment matters as much as its content. Code review comments should be:
- **Specific** — reference the exact code, not a vague concern
- **Actionable** — say what you'd change, not just that something is wrong
- **Explained** — say *why* the change matters
- **Kind** — critique the code, not the author

Use prefixes to signal the weight of your comment:

| Prefix | Meaning |
|--------|---------|
| **[blocking]** | Must be addressed before merge |
| **[suggestion]** | Take it or leave it — my preference, not a requirement |
| **[question]** | I don't understand this — help me |
| **[nit]** | Very minor; address if you want; don't feel obligated |
| **[praise]** | This is good — called out explicitly |

---

## 💡 4. Comment Examples — Good and Bad

### Correctness

```
// ❌ Bad comment — vague; author doesn't know what to fix
"This doesn't look right"

// ✅ Good comment
[blocking] If order.getProviderId() is null (which it can be before the MATCHED state),
this will throw a NullPointerException. Suggest:
    if (order.getProviderId() == null) {
        throw new InvalidOrderStateException("No provider assigned to order " + orderId);
    }
```

### Design

```
// ❌ Bad comment — dismissive; no alternative offered
"This is over-engineered"

// ✅ Good comment
[suggestion] The PriceCalculationStrategy interface is a nice pattern, but given we 
currently only have one implementation, it adds indirection without current benefit. 
YAGNI applies here. Happy to revisit when RIDE-2001 is scoped — at that point the 
pattern will pay off. What do you think?
```

### Testing

```
// ❌ Bad comment — generic; not actionable
"Needs more tests"

// ✅ Good comment
[blocking] I see tests for the happy path, but there's no test for when 
pricingPort.calculateFinalPrice() throws an exception. Based on our error handling 
standards, this should propagate and cause the order to remain IN_PROGRESS. 
Can you add a test for that case?
```

### Praise (Don't Forget This)

```
// ✅ Good — call out good work specifically; builds culture
[praise] The use of the Outbox pattern here is exactly right for this use case. 
The atomic save + outbox write guarantees we never lose the event. Well done.
```

### Nits

```
// ✅ Good nit — minor; flagged clearly; author can use judgment
[nit] Variable name `d` could be more descriptive — maybe `providerId`?
```

---

## 💡 5. Common Review Scenarios

### "I disagree with the reviewer's suggestion"

Don't silently ignore it. Don't get defensive. Have the conversation:

```
Author: "Thanks for the suggestion. My thinking was X because Y. 
         Do you see a risk I'm missing, or is this a preference?"

Reviewer: "Fair point on X. My concern was specifically Z — but if 
           your reasoning accounts for that, I'm happy to defer."
```

If you can't agree, escalate to the Tech Lead. Don't let a PR sit blocked indefinitely over a preference.

### "This PR is too large to review properly"

It's acceptable to say:

```
This PR is 900 lines and I'm not comfortable giving it the review it deserves 
in one pass. Could you split it into:
  1. The domain model changes
  2. The service logic
  3. The API layer

I'll review each as they come in. Happy to pair on the split if it would help.
```

### "The CI is failing but the author wants to merge anyway"

The CI gates exist for a reason. **Never approve a PR with a failing CI gate**, even if the author explains it away. If the gate is genuinely wrong (false positive), fix the gate first.

### "I approved but then the author pushed more commits"

Your approval is invalidated by new commits (branch protection enforces this). Re-review the diff since your last review:

```bash
# See what changed since you last reviewed
git diff {last-reviewed-sha}..HEAD
```

---

## 🔒 6. Reviewing Security-Sensitive Changes

Apply extra scrutiny when reviewing:
- Authentication / authorisation changes
- Anything that touches user data (PII)
- New external HTTP endpoints
- Changes to payment flows
- Database migrations on production tables
- New AWS IAM permissions

For these, ask specifically:
- Could a malicious actor exploit this endpoint?
- Does this expose data to a user who shouldn't see it?
- Does this store or log sensitive data?
- Is this permission the minimum necessary?

---

## 📏 7. The Approval Decision

**Approve** when: You're satisfied the code is correct, tested, safe, and follows standards. Minor nits are fine to note even on approval.

**Request changes** when: There is at least one [blocking] issue that must be addressed.

**Comment without approval** when: You have questions or suggestions but want to let someone else make the approval decision.

**Don't:**
- Approve PRs you haven't read to unblock a teammate — this defeats the point
- Block a PR indefinitely over a [nit] or [suggestion]
- Add [blocking] tags to preference-based feedback

---

## 📋 8. Self-Review Checklist (Run This Before Every PR)

```
Code
[ ] No debug code, commented-out code, or orphaned TODOs
[ ] No hardcoded values that should be config
[ ] No secrets, credentials, or PII in code or comments
[ ] Error cases handled (not just the happy path)
[ ] Null/empty inputs handled

Tests
[ ] Unit tests for new logic (especially edge cases)
[ ] Integration test if DB or Kafka is involved
[ ] Tests are meaningful — they would catch the bug they're testing for

Standards
[ ] Follows coding standards (./04-coding-standards.md)
[ ] Follows hexagonal architecture (../02-architecture-and-api/03-hexagonal-architecture.md)
[ ] API changes follow API standards (../02-architecture-and-api/02-api-standards.md)
[ ] DB migration follows migration guide (../06-developer-guides/03-database-migrations.md)

PR
[ ] Title follows Conventional Commits format
[ ] Description explains what, why, and how tested
[ ] PR is < 400 lines (or justified in description)
[ ] Linked to Jira ticket
```

---
<div align="center">

⬅️ [Back to section](./README.md) · 🏠 [Back to root](../README.md)

</div>
