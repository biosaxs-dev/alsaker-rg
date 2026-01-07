# PyPI Package Build and Upload Guide

This guide explains how to build and upload the `alsaker-rg` package to PyPI.

## Prerequisites

1. **Install build tools:**
```bash
pip install build twine
```

2. **Create PyPI account:**
   - Sign up at https://pypi.org/account/register/
   - Optionally: Sign up at https://test.pypi.org/ for testing

3. **Configure credentials:**
Create `~/.pypirc` (Linux/Mac) or `%USERPROFILE%\.pypirc` (Windows):
```ini
[distutils]
index-servers =
    pypi
    testpypi

[pypi]
username = __token__
password = pypi-YOUR-API-TOKEN-HERE

[testpypi]
username = __token__
password = pypi-YOUR-TEST-API-TOKEN-HERE
```

## Building the Package

1. **Clean previous builds:**
```bash
rm -rf dist/ build/ src/alsaker_rg.egg-info/
# Or on Windows:
# Remove-Item -Recurse -Force dist, build, src/alsaker_rg.egg-info -ErrorAction SilentlyContinue
```

2. **Build the package:**
```bash
python -m build
```

This creates:
- `dist/alsaker_rg-1.0.0-py3-none-any.whl` (wheel)
- `dist/alsaker-rg-1.0.0.tar.gz` (source distribution)

3. **Check the build:**
```bash
twine check dist/*
```

## Testing Locally

Install the package locally to test:

```bash
# Install from wheel
pip install dist/alsaker_rg-1.0.0-py3-none-any.whl

# Or install in editable mode for development
pip install -e .
```

Test it:
```bash
python -c "from alsaker_rg import estimate_Rg; print('Import successful!')"
python examples/example_single_replicate.py
```

## Uploading to Test PyPI (Recommended First)

Test your package on Test PyPI before uploading to production:

```bash
twine upload --repository testpypi dist/*
```

Install from Test PyPI to verify:
```bash
pip install --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple/ alsaker-rg
```

(The `--extra-index-url` allows dependencies to be installed from regular PyPI)

## Uploading to PyPI (Production)

Once you've tested everything:

```bash
twine upload dist/*
```

Users can then install with:
```bash
pip install alsaker-rg
```

## Version Management

Before each release:

1. **Update version** in `src/alsaker_rg/__init__.py`:
```python
__version__ = "1.0.1"  # Increment appropriately
```

2. **Update version** in `pyproject.toml`:
```toml
version = "1.0.1"
```

3. **Create git tag:**
```bash
git tag -a v1.0.1 -m "Release version 1.0.1"
git push origin v1.0.1
```

## Semantic Versioning

Follow semantic versioning (MAJOR.MINOR.PATCH):

- **MAJOR**: Breaking changes (e.g., 2.0.0)
- **MINOR**: New features, backwards compatible (e.g., 1.1.0)
- **PATCH**: Bug fixes, backwards compatible (e.g., 1.0.1)

## Pre-release Versions

For alpha/beta/rc releases:

```python
__version__ = "1.1.0a1"  # Alpha
__version__ = "1.1.0b1"  # Beta
__version__ = "1.1.0rc1"  # Release candidate
```

## Troubleshooting

### Issue: "File already exists"

PyPI doesn't allow re-uploading the same version. Solutions:
- Increment version number
- Delete and rebuild if it was a mistake

### Issue: "Invalid package"

Run checks:
```bash
twine check dist/*
python -m build --sdist --wheel --outdir dist/ .
```

### Issue: "Import errors after install"

Check package structure:
```bash
tar -tzf dist/alsaker-rg-1.0.0.tar.gz | grep "\.py$"
```

Verify `__init__.py` exists and exports correctly.

## GitHub Actions (Optional)

Create `.github/workflows/publish.yml` for automated releases:

```yaml
name: Publish to PyPI

on:
  release:
    types: [published]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: '3.x'
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install build twine
      - name: Build package
        run: python -m build
      - name: Publish to PyPI
        env:
          TWINE_USERNAME: __token__
          TWINE_PASSWORD: ${{ secrets.PYPI_API_TOKEN }}
        run: twine upload dist/*
```

## References

- [Python Packaging Guide](https://packaging.python.org/)
- [PyPI Help](https://pypi.org/help/)
- [PEP 517](https://peps.python.org/pep-0517/) - Build system interface
- [PEP 518](https://peps.python.org/pep-0518/) - pyproject.toml
