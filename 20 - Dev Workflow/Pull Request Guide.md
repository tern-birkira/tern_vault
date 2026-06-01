# Pull Request Guide

*How to contribute code at Tern via pull/merge requests*
*See also: [[Git Workflow]] | [[Code Review]] | [[CI Pipeline]]*

---

## The PR Lifecycle

```
feature branch
    → open MR/PR
        → CI runs automatically
            → peer review
                → address feedback
                    → approved → merged
```

> **Note:** GitLab uses "Merge Request" (MR), GitHub uses "Pull Request" (PR). Same concept.

---

## Opening a Good MR

### Title
Short, imperative, describes the *change*:
- ✅ `Add XmlImporter for parsing <instances> blocks`
- ✅ `Fix namespace prefix loss during XML serialization`
- ❌ `Fixed some bugs` / `WIP`

### Description Template

```markdown
## What
Brief description of what this MR changes.

## Why
Why is this change needed? What problem does it solve?

## How
Key implementation decisions, design choices, anything a reviewer should know.

## Testing
How did you verify this works? (ran locally, added test, manual test steps)

## Checklist
- [ ] Compiles without warnings
- [ ] No debug output left
- [ ] Self-reviewed diff
- [ ] Tests added/updated
```

---

## Size Guidelines

- **Small MRs are better.** Max ~400 lines changed is a good target.
- If a feature is large, split into: data model → parser → UI → tests
- Draft MR = share early for feedback before it's ready to merge

---

## Responding to Review Comments

- Address every comment — either fix it or reply explaining why not
- Use "resolve" only after the change is made or discussion is closed
- Don't force-push after reviewer has commented (makes it hard to re-review)
  - Exception: cleaning up before final approval is OK

---

## Common Mistakes to Avoid

| Mistake | Better approach |
|---------|----------------|
| Mixing unrelated changes | One branch, one concern |
| Giant commits | Commit each logical step |
| Pushing broken code | Build + test locally first |
| No description | Fill out the template |
| Resolving comments before fixing | Fix first, then resolve |

---

## When to Request Review

- CI is green
- Self-review done (read your own diff as a stranger would)
- Description is complete
- You're not planning more major changes

---

*Related: [[Git Workflow]] | [[Code Review]] | [[CI Pipeline]]*
