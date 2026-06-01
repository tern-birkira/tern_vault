# Code Review

*How to give and receive code review at Tern*
*See also: [[Pull Request Guide]] | [[Git Workflow]]*

---

## As a Reviewer

### What to Look For

| Priority | Check |
|----------|-------|
| 🔴 Must fix | Bugs, correctness issues, security problems, broken logic |
| 🟡 Should fix | Design issues, performance problems, missing error handling |
| 🟢 Nice to have | Style, naming, clarity suggestions |

Make the priority clear in your comment. Don't block a merge over 🟢 items.

### How to Comment

- Be specific: point to the exact line, explain *why* it's a problem
- Suggest the fix when possible
- Distinguish questions ("why did you choose X?") from blockers ("this will crash if Y")
- Review the whole MR before commenting — context matters

### Before Approving

- [ ] Understand what the code does
- [ ] CI is green
- [ ] No obvious bugs
- [ ] Code is readable and maintainable
- [ ] Tests cover the new behavior

---

## As the Author

### Receiving Feedback

- Treat comments as questions, not attacks
- Respond to every comment — even "done" or "agreed"
- If you disagree, explain why — don't just silently ignore
- Ask for clarification if a comment is unclear

### Incorporating Feedback

```bash
# Make changes, then
git add -p
git commit -m "fix: address review feedback on XmlImporter"
git push
```

Don't squash commits mid-review — reviewers need to see what changed.

---

## Review Etiquette

- **Fast turnaround** — aim to review within 1 business day
- **Don't approve your own MR** — always need at least one other approval
- **Don't merge without CI green**
- Small MRs = faster reviews = faster iteration

---

*Related: [[Pull Request Guide]] | [[Git Workflow]] | [[CI Pipeline]]*
