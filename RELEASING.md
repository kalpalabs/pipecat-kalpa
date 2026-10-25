# Publishing a release

Releases use GitHub Actions trusted publishing. No PyPI API token is needed.
Create `pipecat-kalpa` from the `kalpalabs` PyPI organization's Projects page,
then configure the project's trusted publisher to use the GitHub repository
`kalpalabs/pipecat-kalpa`, workflow `release.yml`, and environment `pypi`.
The PyPI organization must be active before it can create or manage projects.

Restrict the GitHub `pypi` environment to the `main` branch and `v*` tags.
Only releases whose source is already merged into `main` can be published.

1. Update the version in `pyproject.toml`, run `uv lock`, and update
   `CHANGELOG.md`. Merge the changes after CI passes.
2. Check the release locally:

   ```bash
   uv sync --locked
   uv run pytest
   uv run ruff check .
   uv run ruff format --check .
   uv build
   uvx --from twine twine check --strict dist/pipecat_kalpa-X.Y.Z*
   uv run --isolated --no-project --with ./dist/pipecat_kalpa-X.Y.Z-py3-none-any.whl \
     python -I -c "from pipecat_kalpa import KalpaTTSService, KalpaHttpTTSService"
   ```

   Replace `X.Y.Z` with the release version. These checks validate the tests,
   metadata, README markup, and imports from an isolated wheel installation.
3. Tag the same tested commit and push the tag:

   ```bash
   git tag vX.Y.Z <tested-commit>
   git push origin vX.Y.Z
   ```

   The workflow verifies that the tag matches the package version, runs tests
   and package checks, then publishes the wheel and source archive to PyPI.
4. Verify `uv add pipecat-kalpa==X.Y.Z` in a clean project and import both
   `KalpaTTSService` and `KalpaHttpTTSService`.

To publish an existing tag that predates this workflow, run it from `main`:

```bash
gh workflow run release.yml --ref main -f ref=vX.Y.Z -f repository=pypi
```

PyPI does not let you replace an uploaded release file. Fix a published
package by releasing a new version; never move an existing release tag.

## Optional TestPyPI rehearsal

TestPyPI is a separate index with separate accounts. It can verify the upload
flow and rendered package page before publication, but is not required.
To use it, configure a trusted publisher for the same repository and workflow
with environment `testpypi`, and restrict that GitHub environment to `main`.

```bash
gh workflow run release.yml --ref main -f ref=main -f repository=testpypi
```

Inspect <https://test.pypi.org/project/pipecat-kalpa/>. Install the package
itself from TestPyPI with `--no-deps`; resolve dependencies from normal PyPI.
TestPyPI does not contain every dependency and should not be used as an
extra index for general dependency resolution.
