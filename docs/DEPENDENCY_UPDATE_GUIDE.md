# Dependency Update Guide

This guide explains all the files you need to update when increasing the version of a dependency package in LiteLLM. This applies to backward-compatible dependency updates.

## Files to Update

### 1. Core Dependency Files

#### `pyproject.toml`
- **Location**: `/pyproject.toml`
- **Purpose**: Main package dependencies using Poetry format
- **What to update**: 
  - Required dependencies in `[tool.poetry.dependencies]`
  - Optional dependencies (marked with `optional = true`)
  - Version constraints (e.g., `^3.9.7` or `>=0.23.0`)
- **Example**:
  ```toml
  [tool.poetry.dependencies]
  orjson = {version = "^3.9.7", optional = true}
  ```
  Change to:
  ```toml
  [tool.poetry.dependencies]
  orjson = {version = "^3.11.2", optional = true}
  ```

#### `requirements.txt`
- **Location**: `/requirements.txt`
- **Purpose**: Pinned versions for proxy and all dependencies (used in production)
- **What to update**: Exact version pins
- **Example**:
  ```
  orjson==3.10.12 # fast /embedding responses
  ```
  Change to:
  ```
  orjson==3.11.2 # fast /embedding responses
  ```

#### `poetry.lock`
- **Location**: `/poetry.lock`
- **Purpose**: Lock file with exact dependency versions and hashes
- **What to update**: Run `poetry lock --no-update` or `poetry update <package-name>` to regenerate
- **Note**: This file is auto-generated, don't manually edit

### 2. Additional Package Directories

#### `enterprise/pyproject.toml` and `enterprise/poetry.lock`
- **Location**: `/enterprise/`
- **Purpose**: Enterprise package dependencies
- **What to update**: If the dependency is used in enterprise features, update both files
- **How**: 
  1. Update `enterprise/pyproject.toml` with new version
  2. Run `cd enterprise && poetry lock --no-update` or `poetry update <package-name>`

#### `litellm-proxy-extras/pyproject.toml` and `litellm-proxy-extras/poetry.lock`
- **Location**: `/litellm-proxy-extras/`
- **Purpose**: Proxy extras dependencies (reduces main package size)
- **What to update**: If the dependency is part of proxy extras, update both files
- **How**: 
  1. Update `litellm-proxy-extras/pyproject.toml` with new version
  2. Run `cd litellm-proxy-extras && poetry lock --no-update` or `poetry update <package-name>`

### 3. CI/CD and Docker Files

#### `.circleci/requirements.txt`
- **Location**: `/.circleci/requirements.txt`
- **Purpose**: Dependencies for CircleCI builds and tests
- **What to update**: Update version pins to match main requirements
- **Example**:
  ```
  orjson==3.10.12 # fast /embedding responses
  ```
  Change to:
  ```
  orjson==3.11.2 # fast /embedding responses
  ```

#### `docker/build_from_pip/requirements.txt`
- **Location**: `/docker/build_from_pip/requirements.txt`
- **Purpose**: Dependencies for Docker images built from pip
- **What to update**: If the dependency is critical for Docker deployments, ensure it's compatible
- **Note**: This typically references the main package, so direct updates may not be needed

#### `Makefile`
- **Location**: `/Makefile`
- **Purpose**: Development commands and CI-compatible installations
- **What to update**: Hardcoded version pins (if any)
- **Example**: Lines with `pip install openai==1.99.5`
- **Commands to check**:
  - `install-dev-ci`
  - `install-proxy-dev-ci`
  - `install-test-deps`

### 4. Testing and Compliance Files

#### `tests/code_coverage_tests/liccheck.ini`
- **Location**: `/tests/code_coverage_tests/liccheck.ini`
- **Purpose**: License compliance configuration for dependencies
- **What to update**: Add or update package version constraints in `[Authorized Packages]` section
- **Example**:
  ```ini
  [Authorized Packages]
  orjson: >=3.10.12 # Unknown license
  ```
  Change to:
  ```ini
  [Authorized Packages]
  orjson: >=3.11.2 # Unknown license
  ```
- **Note**: You may need to verify the license hasn't changed between versions

#### `tests/local_testing/test_basic_python_version.py`
- **Location**: `/tests/local_testing/test_basic_python_version.py`
- **Purpose**: Tests that validate dependency configuration
- **What to update**: Typically auto-validates by reading pyproject.toml, so no direct changes needed
- **What to verify**: Run the test to ensure optional dependencies are still properly configured

### 5. Documentation (Recommended)

#### `docs/my-website/release_notes/vX.X.X/index.md`
- **Location**: `/docs/my-website/release_notes/` (create new version directory)
- **Purpose**: Document dependency updates for users
- **What to add**: Entry under **Dependencies** section in release notes
- **Example**:
  ```markdown
  #### Bugs
  
  - **Dependencies**
      - Bumped `orjson` version to "3.11.2" - [PR #XXXXX](https://github.com/BerriAI/litellm/pull/XXXXX)
  ```

## Update Process

### Step-by-Step Guide

1. **Identify the dependency to update**
   - Check current version in `pyproject.toml`
   - Review changelog of the dependency package for breaking changes
   - Ensure the update is backward compatible

2. **Update main package dependencies**
   ```bash
   # Update pyproject.toml manually
   # Then update the lock file
   poetry update <package-name>
   
   # Or to update without changing other dependencies
   poetry lock --no-update
   ```

3. **Update requirements.txt**
   ```bash
   # Manually edit requirements.txt with new version
   # Or regenerate from poetry
   poetry export -f requirements.txt --without-hashes --with proxy-dev --extras "proxy extra_proxy" > requirements.txt
   ```

4. **Update sub-packages (if applicable)**
   ```bash
   # For enterprise
   cd enterprise
   # Update enterprise/pyproject.toml manually
   poetry update <package-name>
   cd ..
   
   # For litellm-proxy-extras
   cd litellm-proxy-extras
   # Update litellm-proxy-extras/pyproject.toml manually
   poetry update <package-name>
   cd ..
   ```

5. **Update CI/CD files**
   - Update `.circleci/requirements.txt` with new version
   - Check `Makefile` for hardcoded version pins
   - Review `docker/build_from_pip/requirements.txt` if needed

6. **Update license compliance**
   - Update `tests/code_coverage_tests/liccheck.ini`
   - Verify the license hasn't changed: `poetry run python tests/code_coverage_tests/check_licenses.py`

7. **Run tests**
   ```bash
   # Test dependency validation
   poetry run pytest tests/local_testing/test_basic_python_version.py::test_package_dependencies -v
   
   # Run license checks
   poetry run python tests/code_coverage_tests/check_licenses.py
   
   # Run relevant unit tests
   make test-unit
   
   # Run integration tests if needed
   make test-integration
   ```

8. **Update documentation**
   - Create or update release notes in `docs/my-website/release_notes/`
   - Document the reason for the update (bug fix, security, new feature support)

## Testing Requirements

After updating a dependency, you should:

### 1. License Compliance Test
```bash
poetry run python tests/code_coverage_tests/check_licenses.py
```
This ensures the updated package still has an acceptable license.

### 2. Dependency Configuration Test
```bash
poetry run pytest tests/local_testing/test_basic_python_version.py::test_package_dependencies -v
```
This validates that optional dependencies are properly configured in extras.

### 3. Import Safety Test
```bash
make check-import-safety
```
This ensures imports still work correctly.

### 4. Unit Tests
```bash
make test-unit
```
Run unit tests to ensure the dependency update doesn't break existing functionality.

### 5. Integration Tests (if applicable)
```bash
make test-integration
```
For critical dependencies, run integration tests to verify end-to-end functionality.

### 6. Linting
```bash
make lint
```
Ensure code style and type checking still pass.

## Common Dependency Updates

### OpenAI Package
Files to update:
- `pyproject.toml`
- `requirements.txt`
- `Makefile` (multiple hardcoded references)
- `.circleci/requirements.txt`
- `docker/build_from_pip/requirements.txt`
- `tests/code_coverage_tests/liccheck.ini`

### orjson Package
Files to update:
- `pyproject.toml` (optional dependency)
- `requirements.txt`
- `.circleci/requirements.txt`
- `tests/code_coverage_tests/liccheck.ini`

### Pydantic Package
Files to update:
- `pyproject.toml` (core dependency)
- `requirements.txt`
- `.circleci/requirements.txt`
- `tests/code_coverage_tests/liccheck.ini`
- Note: Pydantic v1 to v2 would NOT be backward compatible

## Checklist

Use this checklist when updating a dependency:

- [ ] Updated `pyproject.toml` with new version constraint
- [ ] Regenerated `poetry.lock` with `poetry lock --no-update` or `poetry update <package>`
- [ ] Updated `requirements.txt` with exact version pin
- [ ] Updated `enterprise/pyproject.toml` and regenerated `enterprise/poetry.lock` (if applicable)
- [ ] Updated `litellm-proxy-extras/pyproject.toml` and regenerated lock file (if applicable)
- [ ] Updated `.circleci/requirements.txt`
- [ ] Checked and updated `Makefile` for hardcoded versions
- [ ] Updated `tests/code_coverage_tests/liccheck.ini`
- [ ] Ran license compliance check: `poetry run python tests/code_coverage_tests/check_licenses.py`
- [ ] Ran dependency validation test: `poetry run pytest tests/local_testing/test_basic_python_version.py::test_package_dependencies -v`
- [ ] Ran unit tests: `make test-unit`
- [ ] Ran linting: `make lint`
- [ ] Created/updated release notes in `docs/my-website/release_notes/`
- [ ] Verified the update is backward compatible

## Troubleshooting

### Issue: License check fails
**Solution**: Update `tests/code_coverage_tests/liccheck.ini` with the new version constraint in the `[Authorized Packages]` section.

### Issue: test_package_dependencies fails
**Solution**: Ensure all optional dependencies in `pyproject.toml` are listed in at least one extras group.

### Issue: Poetry lock fails
**Solution**: Check for version conflicts. You may need to update multiple dependencies together or adjust version constraints.

### Issue: Import errors after update
**Solution**: The update may have introduced breaking changes. Review the dependency's changelog and update your code accordingly.

## Additional Resources

- [Dependency Update Flow Diagram](./DEPENDENCY_UPDATE_FLOW.md) - Visual reference for the update process
- [Dependency Update Checklist](./DEPENDENCY_UPDATE_CHECKLIST.md) - Quick reference checklist
- [Poetry Documentation](https://python-poetry.org/docs/)
- [LiteLLM Contributing Guide](../CONTRIBUTING.md)
- [Release Notes Generation Instructions](../cookbook/misc/RELEASE_NOTES_GENERATION_INSTRUCTIONS.md)
