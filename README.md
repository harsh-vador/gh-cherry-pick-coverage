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
cd tools/gh-cherry-pick-coverage
./gh-cherry-pick-coverage --help
```

## Prerequisites

- `gh` CLI authenticated (`gh auth login`)
- `git` on `PATH`
- Python 3.9+

## Usage

### Interactive (no flags)

```bash
gh cherry-pick-coverage
```

Prompts you to:
1. Pick repos from a numbered list of built-in defaults (multi-select, e.g.
   `1,2` or `a` for all).
2. Type branches (comma-separated, e.g. `1.13,1.12.10`).
3. Choose whether to filter to your own PRs.

### Scripted

```bash
gh cherry-pick-coverage \
  --repo open-metadata/openmetadata-collate \
  --repo open-metadata/OpenMetadata \
  --branch 1.13 \
  --branch 1.12.10 \
  --workdir /tmp/cpc-cache \
  --open
```

| Flag | Notes |
|---|---|
| `--repo OWNER/REPO` | Repeatable. Omit (with `--branch`) to pick interactively. |
| `--branch BRANCH` | Repeatable. Same set applies to every repo. |
| `--mine` | Only PRs you authored (shortcut for `--author @me`). |
| `--author USER` | Only PRs by USER (GitHub login or `@me`). Excludes `--mine`. |
| `--label LABEL` | Defaults to `"To release"`. |
| `--out PATH` | HTML output (default `cherry-pick-coverage.html`). |
| `--json PATH` | Also write raw report JSON. |
| `--workdir DIR` | Persistent clone cache. Reuse → faster. |
| `--open` | Open dashboard in default browser. |
| `-v` | Verbose logs. |

### Examples

```bash
# Only your own missing cherry-picks on 1.13
gh cherry-pick-coverage --repo open-metadata/openmetadata-collate \
  --branch 1.13 --mine --open

# Filter by a different user
gh cherry-pick-coverage --repo open-metadata/OpenMetadata \
  --branch 1.13 --author pmbrull
```

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
