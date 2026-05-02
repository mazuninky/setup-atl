# setup-atl

GitHub Action that installs [`atl`](https://github.com/mazuninky/atl) — the unified CLI for Atlassian Confluence and Jira — on a workflow runner and adds it to `PATH`.

## Usage

```yaml
- uses: mazuninky/setup-atl@v1
  with:
    version: latest          # or a specific version like "2026.18.4"

- run: atl --version
- run: atl jira issue list --jql "assignee = currentUser()" --format json
  env:
    ATL_API_TOKEN: ${{ secrets.ATL_TOKEN }}
    ATL_EMAIL:     ${{ secrets.ATL_EMAIL }}
    ATL_SITE:      your-site.atlassian.net
```

Pin to a specific version for reproducible builds:

```yaml
- uses: mazuninky/setup-atl@v1
  with:
    version: 2026.18.4
```

## Inputs

| Name      | Required | Default          | Description |
|-----------|----------|------------------|-------------|
| `version` | no       | `latest`         | Calendar version (`2026.18.4`), tag (`v2026.18.4`), or `latest`. |
| `token`   | no       | `${{ github.token }}` | Token used to query the GitHub API and download release assets. |

## Outputs

| Name       | Description |
|------------|-------------|
| `version`  | Resolved version that was installed (e.g. `2026.18.4`). |
| `bin-path` | Absolute path to the directory containing the `atl` binary. |

## Supported runners

| OS              | Arch  | Notes |
|-----------------|-------|-------|
| `ubuntu-*`      | x86_64 | `x86_64-unknown-linux-gnu` |
| `macos-*`       | arm64  | `aarch64-apple-darwin` (M1+) |
| `windows-*`     | x86_64 | `x86_64-pc-windows-msvc` |

`atl` does not currently ship Linux arm64 or macOS x86_64 builds, so the action will fail fast on those runners.

## Authentication

`atl` reads the API token from the `ATL_API_TOKEN` environment variable in headless environments — the OS keyring is unavailable on runners. Pass your token through workflow secrets:

```yaml
env:
  ATL_API_TOKEN: ${{ secrets.ATL_TOKEN }}
  ATL_EMAIL:     ${{ secrets.ATL_EMAIL }}
  ATL_SITE:      your-site.atlassian.net
```

## How it works

The action is a thin composite wrapper:

1. Resolves the version (queries `releases/latest` if `version: latest`).
2. Detects the runner OS/arch and picks the matching release asset.
3. Downloads the tarball (or zip on Windows) and verifies its SHA256 sidecar.
4. Extracts the binary into `$RUNNER_TOOL_CACHE/atl/<version>/<target>/`.
5. Appends that directory to `$GITHUB_PATH`.

No build step, no Node dependencies — just bash.

## License

MIT.
