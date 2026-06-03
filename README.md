# gh cherry-pick-coverage

Single-file `gh` CLI extension that audits PRs labeled `To release` (or any
label) merged into `main` but missing from a set of release branches.

Stdlib-only Python (no `pip install`), portable, shareable via gist or as a
proper `gh` extension repo.

## Install

### As a gh extension (recommended)

```bash
gh extension install <owner>/gh-cherry-pick-coverage
gh cherry-pick-coverage --help
```

### From a gist

```bash
curl -sL https://gist.githubusercontent.com/<user>/<gist-id>/raw \
  -o /usr/local/bin/gh-cherry-pick-coverage
chmod +x /usr/local/bin/gh-cherry-pick-coverage
gh cherry-pick-coverage --help
```

### Local copy in this repo (no install)

```bash
cd tools/cherry-pick-dashboard
./gh-cherry-pick-coverage --help
```

## Prerequisites

- `gh` CLI authenticated (`gh auth login`)
- `git` on `PATH`
- Python 3.9+

## Usage

```bash
gh cherry-pick-coverage \
  --repo open-metadata/openmetadata-collate \
  --repo open-metadata/OpenMetadata \
  --branch 1.13 \
  --branch 1.12.10 \
  --workdir /tmp/cpc-cache \
  --open
```

| Flag | Required | Notes |
|---|---|---|
| `--repo OWNER/REPO` | yes | Repeatable. |
| `--branch BRANCH` | yes | Repeatable. Same set applies to every repo. |
| `--label LABEL` | no | Defaults to `"To release"`. |
| `--out PATH` | no | HTML output (default `cherry-pick-coverage.html`). |
| `--json PATH` | no | Also write raw report JSON. |
| `--workdir DIR` | no | Persistent clone cache. Reuse → faster. |
| `--open` | no | Open dashboard in default browser. |
| `-v` | no | Verbose logs. |

Output is a self-contained HTML file — open it locally, attach to a Slack
message, or host anywhere.

## How "missing" is decided

For each `(repo, branch, PR)`:

1. PR merged into `main`, labeled, merged after the branch cut.
2. Branch does NOT contain the merge commit as an ancestor.
3. Branch does NOT contain a `(cherry picked from commit <M>)` trailer.
4. Branch does NOT contain a commit subject ending in `(#<PR>)` — catches
   manual cherry-picks landed as separate PRs.

If all four are true → PR is missing.

The most recent `auto-cherry-pick-labeled-prs.yaml` run for the PR's branch
HEAD (matched via head_sha) is surfaced as the likely reason.

## Security notes

- Token is supplied via `git -c http.extraheader=...` so it is never written
  to `.git/config` (safe even when the cache dir is shared / uploaded).
- All untrusted PR data is escaped when embedded in the HTML `<script>` block
  (`<`, `>`, `&`, U+2028, U+2029).
- No third-party dependencies — stdlib only.
