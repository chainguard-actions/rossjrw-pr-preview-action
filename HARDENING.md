<!-- markdownlint-disable -->

# Hardening Report: rossjrw--pr-preview-action/v1.7.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rossjrw--pr-preview-action/v1.7.1** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate `${{ }}` expressions inside shell command strings. The 'Wait for preview deployment on GitHub Pages' step calls `wait_for_pages_deployment "${{ inputs.deploy-repository }}" "${{ steps.deployed-commit.outputs.deployed_commit_sha }}" "${{ inputs.preview-branch }}" "${{ inputs.token }}"` — all four arguments are `${{ }}` expressions expanded by the YAML template engine before the shell ever sees them, enabling script injection via attacker-controlled inputs. The 'Wait for preview removal on GitHub Pages' step has the identical pattern with `${{ steps.removed-commit.outputs.deployed_commit_sha }}`.

Locations:

- `action.yml:177`
- `action.yml:228`

### github-env-injection (severity: high)

lib/main.sh writes values derived from caller-controlled (untrusted) env vars to $GITHUB_ENV and $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The env vars written — including `$deployment_action` (from `inputs.action`), `$umbrella_path` (from `inputs.umbrella-dir`), `$pr_number` (from `inputs.pr-number || github.event.number`), `$pages_base_url` (from `inputs.pages-base-url` or `inputs.custom-url`), `$preview_file_path`, `$preview_url_path`, and `$preview_url` — are all set from `inputs.*` and `github.*` context values in the calling step's `env:` block, making them untrusted. A newline injected into any of these values could add arbitrary entries to GITHUB_ENV or GITHUB_OUTPUT.

Locations:

- `lib/main.sh:44`
- `lib/main.sh:52`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Wait for preview deployment on GitHub Pages"; move to env: map

Locations:

- `action.yml:177`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Wait for preview deployment on GitHub Pages"; move to env: map

Locations:

- `action.yml:177`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step "Wait for preview deployment on GitHub Pages"; move to env: map

Locations:

- `action.yml:177`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:227`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:227`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:227`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Fixed two categories of issues:

1. script-injection / static-inline-injection in action.yml: Both 'Wait for preview deployment on GitHub Pages' and 'Wait for preview removal on GitHub Pages' steps had ${{ }} expressions (inputs.deploy-repository, steps.*.outputs.deployed_commit_sha, inputs.preview-branch, inputs.token) directly interpolated in run: blocks. Moved all four expressions to env: blocks (WAIT_DEPLOY_REPO, WAIT_DEPLOYED_COMMIT_SHA/WAIT_REMOVED_COMMIT_SHA, WAIT_PREVIEW_BRANCH, WAIT_TOKEN) and updated the shell script to reference the env vars instead.

2. github-env-injection in lib/main.sh: Added a _safe() helper function using 'printf "%s" "$1" | tr -d "\n\r"' to sanitize all values before writing to $GITHUB_ENV and $GITHUB_OUTPUT. Applied sanitization to deployment_action, preview_file_path, pages_base_url, preview_url_path, preview_url (both URL components), github_action_repository, action_version, action_start_time, and action_start_timestamp.

