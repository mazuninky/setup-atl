# setup-atl

GitHub Action that installs [`atl`](https://github.com/mazuninky/atl) — the unified CLI for Atlassian Confluence and Jira — on a workflow runner and adds it to `PATH`.

## Usage

```yaml
- uses: mazuninky/setup-atl@v1
  with:
    version: latest          # or a specific version like "2026.47.2"

- run: atl --version
# A runner has no atl profile: write one without the token (see Authentication).
- run: |
    cat > atl.toml <<'EOF'
    default_profile = "ci"

    [profiles.ci.jira]
    domain = "https://your-site.atlassian.net"
    email = "you@example.com"
    EOF
- run: atl jira search "assignee = currentUser()" -F json
  env:
    ATL_CONFIG: atl.toml
    ATL_API_TOKEN: ${{ secrets.ATL_TOKEN }}
```

Pin to a specific version for reproducible builds:

```yaml
- uses: mazuninky/setup-atl@v1
  with:
    version: 2026.47.2
```

## Inputs

| Name      | Required | Default          | Description |
|-----------|----------|------------------|-------------|
| `version` | no       | `latest`         | Calendar version (`2026.47.2`), tag (`v2026.47.2`), or `latest`. |
| `token`   | no       | `${{ github.token }}` | Token used to query the GitHub API and download release assets. |

## Outputs

| Name       | Description |
|------------|-------------|
| `version`  | Resolved version that was installed (e.g. `2026.47.2`). |
| `bin-path` | Absolute path to the directory containing the `atl` binary. |

## Supported runners

| OS              | Arch  | Notes |
|-----------------|-------|-------|
| `ubuntu-*`      | x86_64 | `x86_64-unknown-linux-gnu` |
| `ubuntu-*-arm`  | arm64  | `aarch64-unknown-linux-gnu` (atl 2026.47.2 and later) |
| `macos-*`       | arm64  | `aarch64-apple-darwin` (M1+) |
| `macos-*-intel` | x86_64 | `x86_64-apple-darwin` (atl 2026.47.2 and later) |
| `windows-*`     | x86_64 | `x86_64-pc-windows-msvc` |

## Authentication

The OS keyring is unavailable on runners. `atl` takes the site and the email from a profile in its config file and the API token from the `ATL_API_TOKEN` environment variable, which overrides the profile. Point `ATL_CONFIG` at a config file with the profile (as in [Usage](#usage)), keep the token out of it, and pass the token through workflow secrets:

```yaml
env:
  ATL_CONFIG: atl.toml
  ATL_API_TOKEN: ${{ secrets.ATL_TOKEN }}
```

`ATL_PROFILE` picks a profile other than `default_profile`.

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
