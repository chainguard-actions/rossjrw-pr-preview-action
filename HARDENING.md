<!-- markdownlint-disable -->

# Hardening Report: rossjrw--pr-preview-action/v1.7.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rossjrw--pr-preview-action/v1.7.1** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in action.yml directly interpolate GitHub Actions expressions inside shell command strings. The 'Wait for preview deployment on GitHub Pages' step calls `wait_for_pages_deployment "${{ inputs.deploy-repository }}" "${{ steps.deployed-commit.outputs.deployed_commit_sha }}" "${{ inputs.preview-branch }}" "${{ inputs.token }}"` and the 'Wait for preview removal on GitHub Pages' step does the same with `${{ steps.removed-commit.outputs.deployed_commit_sha }}`. Any of these values (especially `inputs.deploy-repository` and `inputs.preview-branch`) can contain shell metacharacters injected by an attacker, leading to arbitrary command execution. These should be moved to `env:` variables and double-quoted in the shell.

Locations:

- `action.yml:175`
- `action.yml:237`

### github-env-injection (severity: high)

lib/main.sh writes multiple values derived from user-controlled inputs to $GITHUB_ENV (lines ~41-53) and $GITHUB_OUTPUT (lines ~55-64) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The affected variables include: `pages_base_url` (from `inputs.pages-base-url` / `inputs.custom-url`), `preview_file_path` (from `inputs.umbrella-dir` and `inputs.pr-number` / `github.event.number`), `preview_url_path` (derived from the above), `deployment_action` (from `inputs.action`), and `github_action_repository` (from `github.action_repository` / `github.repository`). These are all set via the calling step's `env:` block from `inputs.*` and `github.*` context. An attacker can inject newlines into these values to poison $GITHUB_ENV or $GITHUB_OUTPUT with arbitrary key-value pairs.

Locations:

- `lib/main.sh:41`
- `lib/main.sh:55`

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

1. script-injection / static-inline-injection in action.yml: Both 'Wait for preview deployment on GitHub Pages' and 'Wait for preview removal on GitHub Pages' steps had ${{ inputs.deploy-repository }}, ${{ inputs.preview-branch }}, ${{ inputs.token }}, and ${{ steps.*.outputs.deployed_commit_sha }} expressions directly interpolated in run: shell strings. Moved all four expressions to env: blocks (DEPLOY_REPO, DEPLOYED_COMMIT_SHA, PREVIEW_BRANCH, DEPLOY_TOKEN) and updated the shell commands to reference plain environment variables.

2. github-env-injection in lib/main.sh: User-controlled values (deployment_action, preview_file_path, pages_base_url, preview_url_path, github_action_repository, action_version, action_start_time) were written directly to $GITHUB_ENV and $GITHUB_OUTPUT without newline sanitization. Added sanitization using `printf '%s' "$VAR" | tr -d '\n\r'` for each user-controlled value before writing to the environment files, storing results in safe_* prefixed variables.

