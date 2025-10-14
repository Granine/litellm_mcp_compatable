# Dependency Update Flow Diagram

**👉 New to dependency updates? Start with [DEPENDENCY_UPDATE_README.md](./DEPENDENCY_UPDATE_README.md) for an overview!**

---

This document provides a visual reference for the dependency update process in LiteLLM.

## File Dependency Map

```
Dependency Update
│
├── Core Package Files
│   ├── pyproject.toml ──────────────► Update version constraint
│   ├── poetry.lock ─────────────────► Regenerate (poetry lock)
│   └── requirements.txt ────────────► Update pinned version
│
├── Sub-Packages (if applicable)
│   ├── enterprise/
│   │   ├── pyproject.toml ──────────► Update version constraint
│   │   └── poetry.lock ─────────────► Regenerate (poetry lock)
│   └── litellm-proxy-extras/
│       ├── pyproject.toml ──────────► Update version constraint
│       └── poetry.lock ─────────────► Regenerate (poetry lock)
│
├── CI/CD Files
│   ├── .circleci/requirements.txt ──► Update pinned version
│   ├── Makefile ────────────────────► Update hardcoded versions
│   └── docker/build_from_pip/
│       └── requirements.txt ────────► Review compatibility
│
├── Testing & Compliance
│   ├── tests/code_coverage_tests/
│   │   └── liccheck.ini ────────────► Update version constraint
│   └── tests/local_testing/
│       └── test_basic_python_version.py ► Auto-validates
│
└── Documentation
    └── docs/my-website/release_notes/
        └── vX.X.X/index.md ─────────► Document the update
```

## Update Process Flow

```
Step 1: Preparation
┌────────────────────────────────────┐
│ Review dependency changelog        │
│ Verify backward compatibility      │
│ Check for license changes          │
└────────────────┬───────────────────┘
                 │
                 ▼
Step 2: Update Core Files
┌────────────────────────────────────┐
│ 1. Edit pyproject.toml             │
│ 2. Run: poetry update <package>    │
│ 3. Edit requirements.txt           │
└────────────────┬───────────────────┘
                 │
                 ▼
Step 3: Update Sub-packages (if needed)
┌────────────────────────────────────┐
│ For enterprise:                    │
│   cd enterprise                    │
│   poetry update <package>          │
│                                    │
│ For proxy-extras:                  │
│   cd litellm-proxy-extras          │
│   poetry update <package>          │
└────────────────┬───────────────────┘
                 │
                 ▼
Step 4: Update CI/CD
┌────────────────────────────────────┐
│ 1. Update .circleci/requirements   │
│ 2. Update Makefile (if needed)     │
│ 3. Review Docker requirements      │
└────────────────┬───────────────────┘
                 │
                 ▼
Step 5: Update Compliance
┌────────────────────────────────────┐
│ Update liccheck.ini                │
└────────────────┬───────────────────┘
                 │
                 ▼
Step 6: Testing
┌────────────────────────────────────┐
│ ✓ License compliance check         │
│ ✓ Dependency validation test       │
│ ✓ Import safety check              │
│ ✓ Unit tests                       │
│ ✓ Linting                          │
└────────────────┬───────────────────┘
                 │
                 ▼
Step 7: Documentation
┌────────────────────────────────────┐
│ Create/update release notes        │
│ Document reason for update         │
└────────────────┬───────────────────┘
                 │
                 ▼
Step 8: Commit & PR
┌────────────────────────────────────┐
│ Create PR with clear description   │
│ Ensure CI/CD passes                │
└────────────────────────────────────┘
```

## Testing Validation Flow

```
Testing Phase
│
├── License Compliance
│   └── poetry run python tests/code_coverage_tests/check_licenses.py
│       ├── PASS ──► Continue
│       └── FAIL ──► Update liccheck.ini
│
├── Dependency Configuration
│   └── pytest tests/local_testing/test_basic_python_version.py::test_package_dependencies
│       ├── PASS ──► Continue
│       └── FAIL ──► Fix pyproject.toml extras
│
├── Import Safety
│   └── make check-import-safety
│       ├── PASS ──► Continue
│       └── FAIL ──► Fix imports
│
├── Unit Tests
│   └── make test-unit
│       ├── PASS ──► Continue
│       └── FAIL ──► Fix code or revert
│
├── Linting
│   └── make lint
│       ├── PASS ──► Continue
│       └── FAIL ──► Fix linting issues
│
└── Integration Tests (Optional)
    └── make test-integration
        ├── PASS ──► Ready to merge
        └── FAIL ──► Fix or document known issues
```

## Common Dependency Categories

### Category 1: Core Dependencies (Always in pyproject.toml)
- openai, anthropic, httpx, pydantic, etc.
- **Impact**: Main package functionality
- **Files**: All core files + liccheck.ini

### Category 2: Optional Dependencies (proxy, etc.)
- orjson, fastapi, uvicorn, prisma, etc.
- **Impact**: Proxy and optional features
- **Files**: All core files + liccheck.ini
- **Note**: Must be in extras group in pyproject.toml

### Category 3: Development Dependencies
- pytest, black, ruff, mypy, etc.
- **Impact**: Development and testing
- **Files**: pyproject.toml [tool.poetry.dev-dependencies]
- **Note**: Not in requirements.txt

### Category 4: Enterprise/Proxy Extras
- litellm-enterprise, litellm-proxy-extras
- **Impact**: Enterprise features or proxy extras
- **Files**: Core files + respective sub-package

## File Update Priority Matrix

| File | Priority | When to Update | Auto-Generated |
|------|----------|----------------|----------------|
| pyproject.toml | HIGH | Always | No |
| poetry.lock | HIGH | Always | Yes (poetry) |
| requirements.txt | HIGH | Always | No |
| enterprise/pyproject.toml | MEDIUM | If used in enterprise | No |
| enterprise/poetry.lock | MEDIUM | If used in enterprise | Yes (poetry) |
| litellm-proxy-extras/pyproject.toml | MEDIUM | If proxy extra | No |
| litellm-proxy-extras/poetry.lock | MEDIUM | If proxy extra | Yes (poetry) |
| .circleci/requirements.txt | MEDIUM | Always | No |
| Makefile | LOW | If hardcoded version | No |
| docker/*/requirements.txt | LOW | Review for Docker | No |
| liccheck.ini | HIGH | Always | No |
| release notes | LOW | Recommended | No |

## Backward Compatibility Checklist

Before updating, verify:

- [ ] **No breaking API changes** - Review dependency changelog
- [ ] **License unchanged** - Or compatible license
- [ ] **Python version compatibility** - Matches project requirements (>=3.8.1, <4.0)
- [ ] **No new dependencies introduced** - Or acceptable new dependencies
- [ ] **Security fixes** - Document if security-related
- [ ] **Performance impact** - Consider for critical path dependencies

## Quick Reference: Common Commands

```bash
# Update single dependency
poetry update <package-name>

# Update without changing other dependencies
poetry lock --no-update

# Export to requirements.txt
poetry export -f requirements.txt --without-hashes --with proxy-dev --extras "proxy extra_proxy" > requirements.txt

# Test license compliance
poetry run python tests/code_coverage_tests/check_licenses.py

# Test dependency configuration
poetry run pytest tests/local_testing/test_basic_python_version.py::test_package_dependencies -v

# Run all tests
make test-unit

# Lint check
make lint
```

## See Also

- [DEPENDENCY_UPDATE_GUIDE.md](./DEPENDENCY_UPDATE_GUIDE.md) - Detailed guide
- [DEPENDENCY_UPDATE_CHECKLIST.md](./DEPENDENCY_UPDATE_CHECKLIST.md) - Quick checklist
- [CONTRIBUTING.md](../CONTRIBUTING.md) - General contributing guidelines
