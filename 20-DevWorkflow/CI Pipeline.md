# CI Pipeline

*Automated checks that run on every push/MR in `polaris-asd-editor`*
*See also: [[Pull Request Guide]] | [[Git Workflow]]*

---

## What CI Does

Every push to a branch triggers the pipeline automatically. It checks:

1. **Build** — does the code compile? (CMake + make/ninja)
2. **Tests** — do all unit tests pass?
3. **Static Analysis** — linting / clang-tidy (if configured)
4. **Packaging** — produces build artifacts (if on main)

A green pipeline = safe to review and merge.

---

## Pipeline Stages

> Fill this in once you have access to the GitLab CI config (`.gitlab-ci.yml`)

```
build → test → [package]
```

| Stage | What Runs | Time (approx) |
|-------|-----------|---------------|
| build | CMake configure + compile | ? |
| test | Unit tests | ? |
| package | Create installer/archive | ? (main only) |

---

## Checking Pipeline Status

On GitLab:
- MR page shows pipeline status inline
- Click the status icon to see full job logs
- Failed job → click → scroll to the red section

---

## Running CI Locally (Pre-Push Check)

```bash
# Quick sanity check before pushing
cd build
cmake ..
make -j$(nproc)
ctest --output-on-failure
```

Saves time vs. waiting for remote CI.

---

## Common CI Failures

| Failure | Cause | Fix |
|---------|-------|-----|
| Compile error | Code doesn't build on CI's toolchain | Check exact error, may be compiler version difference |
| Linker error | Missing lib or wrong target name | Check `CMakeLists.txt` |
| Test failure | Test env differs from local | Look at test output in job log |
| Timeout | Build too slow | Usually infra issue, re-run |

---

## Re-Running a Failed Pipeline

On GitLab: MR page → pipeline status → "Retry" button on failed job.

---

*Related: [[Pull Request Guide]] | [[Git Workflow]] | [[Code Review]]*
