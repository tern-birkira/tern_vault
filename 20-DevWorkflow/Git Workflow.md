# Git Workflow

*How work is organized in the `polaris-asd-editor` repo*
*See also: [[Pull Request Guide]] | [[CI Pipeline]]*

---

## Branch Strategy

```
main / master
  └── feature/your-feature-name
  └── fix/short-description
  └── chore/maintenance-task
```

- Never commit directly to `main`
- One branch per feature or fix
- Branch from latest `main`

---

## Branch Naming

| Type | Pattern | Example |
|------|---------|---------|
| Feature | `feature/short-name` | `feature/track-label-import` |
| Fix | `fix/short-name` | `fix/xml-namespace-parse` |
| Chore | `chore/short-name` | `chore/update-cmake-deps` |

---

## Commit Messages

Keep commits atomic — one logical change per commit.

```
<type>: <short description>

<optional body — why, not what>
```

Types: `feat`, `fix`, `refactor`, `test`, `chore`, `docs`

Examples:
```
feat: add XmlImporter class for parsing <instances> blocks
fix: preserve ${property} placeholders during serialization
refactor: split XsdParser into parser and type-mapper
```

---

## Daily Workflow

```bash
# Start of day — sync with main
git checkout main
git pull

# Create feature branch
git checkout -b feature/my-feature

# Work, commit often
git add -p          # stage hunks selectively
git commit -m "feat: ..."

# Push and open PR when ready
git push -u origin feature/my-feature
```

---

## Keeping Branch Up To Date

```bash
git fetch origin
git rebase origin/main    # preferred over merge — keeps history clean
```

If conflicts: resolve, then `git rebase --continue`.

---

## Before Pushing

- [ ] Code compiles without warnings
- [ ] No debug prints left in code
- [ ] New code has unit tests (if applicable)
- [ ] Self-review your diff (`git diff main`)

---

*Related: [[Pull Request Guide]] | [[CI Pipeline]] | [[Code Review]]*
