# Dependency Update Quick Reference Checklist

Use this checklist when updating a dependency package version in LiteLLM.

## Files to Update

### Core Files (Always Required)
- [ ] `pyproject.toml` - Update version constraint in `[tool.poetry.dependencies]`
- [ ] `poetry.lock` - Regenerate with `poetry lock --no-update` or `poetry update <package>`
- [ ] `requirements.txt` - Update exact version pin

### Sub-packages (If Applicable)
- [ ] `enterprise/pyproject.toml` - If dependency is used in enterprise features
- [ ] `enterprise/poetry.lock` - Regenerate with `cd enterprise && poetry lock --no-update`
- [ ] `litellm-proxy-extras/pyproject.toml` - If dependency is part of proxy extras
- [ ] `litellm-proxy-extras/poetry.lock` - Regenerate with `cd litellm-proxy-extras && poetry lock --no-update`

### CI/CD Files
- [ ] `.circleci/requirements.txt` - Update version to match requirements.txt
- [ ] `Makefile` - Check for hardcoded version pins (search for package name)
- [ ] `docker/build_from_pip/requirements.txt` - Review if needed for Docker builds

### Testing & Compliance
- [ ] `tests/code_coverage_tests/liccheck.ini` - Update version constraint in `[Authorized Packages]`

### Documentation (Recommended)
- [ ] `docs/my-website/release_notes/vX.X.X/index.md` - Document the update in release notes

## Commands to Run

### Update Commands
```bash
# 1. Update main package
poetry update <package-name>
# OR to keep other deps unchanged:
# poetry lock --no-update

# 2. Update requirements.txt (if using poetry export)
poetry export -f requirements.txt --without-hashes --with proxy-dev --extras "proxy extra_proxy" > requirements.txt

# 3. Update enterprise (if applicable)
cd enterprise && poetry update <package-name> && cd ..

# 4. Update proxy extras (if applicable)
cd litellm-proxy-extras && poetry update <package-name> && cd ..
```

### Test Commands (Run in Order)
```bash
# 1. License compliance check
poetry run python tests/code_coverage_tests/check_licenses.py

# 2. Dependency validation test
poetry run pytest tests/local_testing/test_basic_python_version.py::test_package_dependencies -v

# 3. Import safety check
make check-import-safety

# 4. Unit tests
make test-unit

# 5. Linting
make lint

# 6. Integration tests (if needed)
make test-integration
```

## Quick File Locations

| File Type | Path |
|-----------|------|
| Main dependencies | `/pyproject.toml` |
| Main lock file | `/poetry.lock` |
| Pinned versions | `/requirements.txt` |
| Enterprise deps | `/enterprise/pyproject.toml` |
| Proxy extras deps | `/litellm-proxy-extras/pyproject.toml` |
| CircleCI deps | `/.circleci/requirements.txt` |
| Docker deps | `/docker/build_from_pip/requirements.txt` |
| License config | `/tests/code_coverage_tests/liccheck.ini` |
| Makefile | `/Makefile` |
| Release notes | `/docs/my-website/release_notes/` |

## Common Package Updates

### openai
Files: `pyproject.toml`, `requirements.txt`, `Makefile` (multiple places), `.circleci/requirements.txt`, `liccheck.ini`

### orjson
Files: `pyproject.toml`, `requirements.txt`, `.circleci/requirements.txt`, `liccheck.ini`

### pydantic
Files: `pyproject.toml`, `requirements.txt`, `.circleci/requirements.txt`, `liccheck.ini`
⚠️ **Warning**: v1 to v2 is NOT backward compatible

### fastapi/uvicorn/starlette
Files: `pyproject.toml`, `requirements.txt`, `.circleci/requirements.txt`, `liccheck.ini`

### prisma
Files: `pyproject.toml`, `requirements.txt`, `.circleci/requirements.txt`, `liccheck.ini`

## Troubleshooting

| Issue | Solution |
|-------|----------|
| License check fails | Update `liccheck.ini` with new version in `[Authorized Packages]` |
| test_package_dependencies fails | Ensure optional deps are in extras groups in `pyproject.toml` |
| Poetry lock fails | Check for version conflicts; may need to update multiple deps together |
| Import errors | Review dependency changelog for breaking changes |

## Pre-Update Checklist

Before updating:
- [ ] Review dependency's changelog for breaking changes
- [ ] Verify update is backward compatible
- [ ] Check if dependency has new license requirements
- [ ] Identify which files need updating (use this checklist)
- [ ] Create a branch for the update

## Post-Update Checklist

After updating:
- [ ] All tests pass
- [ ] License compliance verified
- [ ] Release notes created/updated
- [ ] PR created with clear description of changes
- [ ] CI/CD pipelines pass

---

For detailed information, see:
- [DEPENDENCY_UPDATE_GUIDE.md](./DEPENDENCY_UPDATE_GUIDE.md) - Complete guide with examples
- [DEPENDENCY_UPDATE_FLOW.md](./DEPENDENCY_UPDATE_FLOW.md) - Visual flow diagrams and reference
