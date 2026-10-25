# Publishing a release

Releases use GitHub Actions trusted publishing. No PyPI API token is needed.
Configure `pipecat-kalpa` on each index to trust the GitHub repository
`kalpalabs/pipecat-kalpa`, workflow `release.yml`, and its matching environment:
`pypi` on PyPI or `testpypi` on TestPyPI. For a new project, add a pending
publisher from the account's Publishing page.

Restrict both GitHub environments to the `main` branch, and also allow `v*`
tags in the `pypi` environment. Only releases whose source is already merged
into `main` can be published.

1. Update the version in `pyproject.toml`, run `uv lock`, and update
   `CHANGELOG.md`. Merge the changes after CI passes.
2. Rehearse from `main` on TestPyPI:

   ```bash
   gh workflow run release.yml --ref main -f ref=main -f repository=testpypi
   ```

   Inspect the rendered README and release files at
   <https://test.pypi.org/project/pipecat-kalpa/>. Install the package itself
   from TestPyPI with `--no-deps`; resolve its dependencies from normal PyPI.
   TestPyPI does not contain every dependency and should not be used as an
   extra index for general dependency resolution.
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
