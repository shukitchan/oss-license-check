# oss-license-check

A GitHub Actions workflow that runs **weekly** to analyze all your GitHub repositories using the
[OSS Review Toolkit (ORT)](https://github.com/oss-review-toolkit/ort), and produces a per-repository license
inventory.

## How it works

The workflow (`weekly-license-scan.yml`) is made up of three jobs:

1. **List Repositories** – calls the GitHub API to enumerate every non-archived, non-fork
   repository owned by the authenticated user (up to 256).
2. **Scan** (parallel matrix) – for each repository:
   - **ORT Analyzer** – resolves all package-manager dependencies and writes
     `analyzer-result.yml`.
   - **ORT Reporter** – generates three report formats from the analyzer result:
     - `StaticHtml` – human-readable HTML page with a full license table
     - `SpdxDocument` – machine-readable SPDX 2.x document
     - `CycloneDx` – machine-readable CycloneDx BOM
   - Per-repository artifacts are uploaded and retained for **90 days**.
3. **Summarize** – downloads every per-repository artifact and writes a consolidated status
   table to the workflow's Step Summary.

## Prerequisites

| Requirement | Details |
|---|---|
| **`GH_PAT` secret** *(private repos only)* | A [Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) with the **`repo`** scope. Required only to clone private repositories during the analyze step. Add it under **Settings → Secrets and variables → Actions → New repository secret**. If all your repositories are public this secret is not needed. |
| **Docker** | The runner uses `ghcr.io/oss-review-toolkit/ort` Docker image. `ubuntu-latest` GitHub-hosted runners include Docker by default. |

## Setup

1. Fork or copy this repository into your GitHub account.
2. The workflow uses the built-in `GITHUB_TOKEN` to list repositories via the GitHub API — no
   extra configuration is needed for public repositories.
3. *(Optional – private repos)* Create a Personal Access Token (PAT) with the `repo` scope and
   add it as a repository secret named **`GH_PAT`**. The analyze step will use this token to clone
   private repositories.
4. The workflow runs automatically every Sunday at 00:00 UTC, or you can trigger it manually
   via **Actions → Weekly OSS License Scan → Run workflow**.

## Configuration

| Option | Location | Default | Description |
|---|---|---|---|
| Schedule | `on.schedule.cron` | `0 0 * * 0` (weekly) | Standard cron expression |
| ORT version | `env.ORT_VERSION` | `latest` | Pin to a specific release tag (e.g. `25.0.0`) for reproducible scans |
| Max parallel scans | `jobs.scan.strategy.max-parallel` | `5` | Concurrent scan jobs |
| Include forks | `.filter(r => !r.fork)` in the script | forks excluded | Remove the `!r.fork` condition to include forks |
| Include archived repos | `.filter(r => !r.archived)` in the script | archived excluded | Remove the `!r.archived` condition to include archived repos |
| Report formats | `jobs.scan` → Reporter step `-f` flag | `StaticHtml,SpdxDocument,CycloneDx` | Any ORT-supported format |

## Artifacts

After each run, the following artifacts are available on the workflow run page:

- **`ort-results-<repo-name>`** – raw ORT output for a single repository (analyzer result
  and reports directory).
- **`all-license-reports-<run-id>`** – consolidated archive of every repository's results.

Artifacts are retained for **90 days**.

## License

[MIT](LICENSE)
