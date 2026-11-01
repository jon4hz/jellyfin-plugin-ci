# jellyfin-plugin-ci

Reusable GitHub Actions workflows shared by my Jellyfin plugins.

| Workflow | What it does |
|---|---|
| `ci.yml` | `dotnet build` with warnings as errors, plus pre-commit (re-runnable on a PR via the `pre-commit ci run` label) |
| `release.yml` | sets the version in the csproj, `dotnet publish` (the csproj's `CreatePluginPackage` target zips the plugin), creates the GitHub release and prepends the version to `manifest.json` on `main` |
| `update-pre-commit.yml` | weekly `pre-commit autoupdate` PR |

Conventions a plugin repo has to follow: one `*.sln`, `global.json` and `manifest.json` in the root, the project at `<Name>/<Name>.csproj` with `AssemblyVersion`/`FileVersion`/`PluginVersion` properties, and a package named `bin/<repo-name>-<version>.zip`. `targetAbi` is derived from the `Jellyfin.Controller` package version.

Callers (copy into `.github/workflows/`):

```yaml
# ci.yml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    types: [labeled, opened, reopened, synchronize]
jobs:
  ci:
    uses: jon4hz/jellyfin-plugin-ci/.github/workflows/ci.yml@main
    permissions:
      contents: read
      pull-requests: write
```

```yaml
# release.yml
name: Release
on:
  push:
    tags: ['v*.*.*']
  workflow_dispatch:
    inputs:
      version:
        description: 'Version to build (e.g. 1.0.0)'
        required: true
        type: string
jobs:
  release:
    uses: jon4hz/jellyfin-plugin-ci/.github/workflows/release.yml@main
    permissions:
      contents: write
    with:
      version: ${{ inputs.version || github.ref_name }}
```

```yaml
# update-pre-commit.yml
name: Update pre-commit hooks
on:
  schedule:
    - cron: '0 9 * * 1'
  workflow_dispatch:
jobs:
  update:
    uses: jon4hz/jellyfin-plugin-ci/.github/workflows/update-pre-commit.yml@main
    permissions:
      contents: write
      pull-requests: write
```
