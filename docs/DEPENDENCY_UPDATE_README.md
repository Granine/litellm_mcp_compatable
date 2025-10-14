# Dependency Update Documentation - Quick Start

This is the entry point for understanding how to update dependencies in LiteLLM.

## Which Document Should I Use?

Choose based on your needs:

### 📚 [DEPENDENCY_UPDATE_GUIDE.md](./DEPENDENCY_UPDATE_GUIDE.md)
**Use when:** You're updating a dependency for the first time or need detailed explanations
- **Length:** ~300 lines (10-15 min read)
- **Content:** Complete guide with examples, troubleshooting, and detailed explanations
- **Best for:** Learning the process, reference documentation

### ✅ [DEPENDENCY_UPDATE_CHECKLIST.md](./DEPENDENCY_UPDATE_CHECKLIST.md)
**Use when:** You know the process and need a quick reference
- **Length:** ~130 lines (5 min read)
- **Content:** Checklist format, commands, file locations, quick troubleshooting
- **Best for:** Quick updates, copy-paste commands, ensuring nothing is missed

### 🔄 [DEPENDENCY_UPDATE_FLOW.md](./DEPENDENCY_UPDATE_FLOW.md)
**Use when:** You want to visualize the process or understand file relationships
- **Length:** ~220 lines (10 min read)
- **Content:** Visual diagrams, flow charts, dependency maps, priority matrix
- **Best for:** Understanding workflow, visual learners, training

## Quick Answer to Your Question

**Q: I want to increase the version of a dependency package (backward compatible). What are all the files I need to change?**

**A:** Here's the minimum set of files (13 total):

### Must Always Update:
1. `pyproject.toml` - Update version constraint
2. `poetry.lock` - Regenerate with `poetry update <package>`
3. `requirements.txt` - Update pinned version
4. `tests/code_coverage_tests/liccheck.ini` - Update version constraint

### CI/CD Files:
5. `.circleci/requirements.txt` - Update pinned version
6. `Makefile` - Update if package has hardcoded version

### Sub-packages (If Applicable):
7. `enterprise/pyproject.toml` - If used in enterprise features
8. `enterprise/poetry.lock` - Regenerate
9. `litellm-proxy-extras/pyproject.toml` - If used in proxy extras
10. `litellm-proxy-extras/poetry.lock` - Regenerate

### Optional But Recommended:
11. `docker/build_from_pip/requirements.txt` - Review for Docker compatibility
12. `docs/my-website/release_notes/vX.X.X/index.md` - Document the update
13. `tests/local_testing/test_basic_python_version.py` - Run to validate (no changes needed)

## Testing Requirements

After updating files, run these tests in order:

```bash
# 1. License compliance
poetry run python tests/code_coverage_tests/check_licenses.py

# 2. Dependency validation
poetry run pytest tests/local_testing/test_basic_python_version.py::test_package_dependencies -v

# 3. Import safety
make check-import-safety

# 4. Unit tests
make test-unit

# 5. Linting
make lint
```

## Typical Update Commands

```bash
# 1. Update the dependency
poetry update <package-name>

# 2. Update requirements.txt manually or export
poetry export -f requirements.txt --without-hashes --with proxy-dev --extras "proxy extra_proxy" > requirements.txt

# 3. Update sub-packages if needed
cd enterprise && poetry update <package-name> && cd ..
cd litellm-proxy-extras && poetry update <package-name> && cd ..

# 4. Update other files manually (see list above)

# 5. Run tests
make test-unit
make lint
```

## Common Examples

### Example 1: Updating `orjson` from 3.10.12 to 3.11.2

Files to update:
- `pyproject.toml`: `orjson = {version = "^3.11.2", optional = true}`
- `poetry.lock`: `poetry update orjson`
- `requirements.txt`: `orjson==3.11.2`
- `.circleci/requirements.txt`: `orjson==3.11.2`
- `liccheck.ini`: `orjson: >=3.11.2`

### Example 2: Updating `openai` from 1.99.5 to 1.99.9

Files to update:
- `pyproject.toml`: `openai = ">=1.99.9"`
- `poetry.lock`: `poetry update openai`
- `requirements.txt`: `openai==1.99.9`
- `.circleci/requirements.txt`: Update version
- `Makefile`: Update all instances of `openai==1.99.5`
- `docker/build_from_pip/requirements.txt`: Update version
- `liccheck.ini`: `openai: >=1.99.9`

### Example 3: Updating `pydantic` (Be Careful!)

⚠️ **Warning:** Pydantic v1 to v2 is NOT backward compatible!

For minor version updates (e.g., 2.10.1 to 2.10.2):
- Same files as Example 1
- Extra testing required due to type system changes

## File Update Matrix

| File | Priority | When | Auto-Gen |
|------|----------|------|----------|
| pyproject.toml | HIGH | Always | No |
| poetry.lock | HIGH | Always | Yes |
| requirements.txt | HIGH | Always | No |
| liccheck.ini | HIGH | Always | No |
| .circleci/requirements.txt | MEDIUM | Always | No |
| Makefile | LOW | If hardcoded | No |
| enterprise/* | MEDIUM | If used | Partial |
| litellm-proxy-extras/* | MEDIUM | If used | Partial |
| docker/* | LOW | Review | No |
| release notes | LOW | Recommended | No |

## Common Mistakes to Avoid

❌ **Don't:**
- Forget to regenerate `poetry.lock`
- Update `pyproject.toml` but forget `requirements.txt`
- Skip updating `liccheck.ini`
- Skip running tests
- Update to a non-backward-compatible version without careful review

✅ **Do:**
- Check the dependency's changelog first
- Update all related files consistently
- Run all test suites
- Document the update in release notes
- Verify license compatibility

## Need More Help?

1. **Detailed Guide:** Read [DEPENDENCY_UPDATE_GUIDE.md](./DEPENDENCY_UPDATE_GUIDE.md)
2. **Visual Reference:** See [DEPENDENCY_UPDATE_FLOW.md](./DEPENDENCY_UPDATE_FLOW.md)
3. **Quick Checklist:** Use [DEPENDENCY_UPDATE_CHECKLIST.md](./DEPENDENCY_UPDATE_CHECKLIST.md)
4. **Contributing:** See [../CONTRIBUTING.md](../CONTRIBUTING.md)

## Summary

For a backward-compatible dependency update:

1. **Update 4-6 core files** (pyproject.toml, poetry.lock, requirements.txt, liccheck.ini, .circleci/requirements.txt, Makefile)
2. **Update 0-4 sub-package files** (if dependency is used in enterprise or proxy extras)
3. **Update 0-2 optional files** (Docker requirements, release notes)
4. **Run 5 test suites** (license, dependency validation, import safety, unit tests, linting)
5. **Create PR** with clear description

Total time: 10-30 minutes depending on complexity and testing requirements.
