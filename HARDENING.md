<!-- markdownlint-disable -->

# Hardening Report: rossjrw--pr-preview-action/v1.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rossjrw--pr-preview-action/v1.8.1** was hardened automatically. 13 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Four `run:` blocks in action.yml directly interpolate `${{ }}` expressions inside shell command strings, enabling script injection. (1) The 'Wait for preview deployment on GitHub Pages' (deploy) step passes `${{ inputs.deploy-repository }}`, `${{ steps.deployed-commit.outputs.deployed_commit_sha }}`, `${{ inputs.preview-branch }}`, and `${{ inputs.token }}` directly as shell arguments to `wait_for_pages_deployment`. (2) The 'Generate comment content for deployment' step passes `${{ env.action_repository }}`, `${{ env.action_version }}`, `${{ env.preview_url }}`, `${{ inputs.preview-branch }}`, `${{ github.server_url }}`, `${{ inputs.deploy-repository }}`, `${{ env.action_start_time }}`, and `${{ inputs.qr-code }}` directly as shell arguments. (3) The 'Wait for preview removal on GitHub Pages' (remove) step has the same pattern as (1) with `${{ steps.removed-commit.outputs.deployed_commit_sha }}`. (4) The 'Generate comment content for removal' step has the same pattern as (2). Any of these inputs could contain shell metacharacters injected by an attacker via a pull request or workflow_dispatch event.

Locations:

- `action.yml:200`
- `action.yml:212`
- `action.yml:262`
- `action.yml:274`

### github-env-injection (severity: high)

In `lib/main.sh`, multiple variables derived from user-controlled inputs (set via the `env:` block in action.yml from `inputs.*` and `github.*` context values) are written to `$GITHUB_ENV` and `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Variables written include: `umbrella_path` (from `inputs.umbrella-dir`), `pages_base_url` (from `inputs.pages-base-url` or `inputs.custom-url`), `pr_number` (from `inputs.pr-number` or `github.event.number`), `deployment_repository` (from `inputs.deploy-repository`), `preview_file_path`, `preview_url_path`, `preview_url`, `action_repository` (from `github.action_repository`), `action_version`, `action_start_time`. A newline in any of these values could inject arbitrary environment variables into subsequent steps.

Locations:

- `lib/main.sh:44`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Wait for preview deployment on GitHub Pages"; move to env: map

Locations:

- `action.yml:186`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Wait for preview deployment on GitHub Pages"; move to env: map

Locations:

- `action.yml:186`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step "Wait for preview deployment on GitHub Pages"; move to env: map

Locations:

- `action.yml:186`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Generate comment content for deployment"; move to env: map

Locations:

- `action.yml:200`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Generate comment content for deployment"; move to env: map

Locations:

- `action.yml:202`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.qr-code }}" appears directly in run: block of step "Generate comment content for deployment"; move to env: map

Locations:

- `action.yml:205`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:257`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:257`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:257`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Generate comment content for removal"; move to env: map

Locations:

- `action.yml:271`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Generate comment content for removal"; move to env: map

Locations:

- `action.yml:273`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Fixed all script injection findings in action.yml by moving ${{ }} expressions from run: shell strings into env: blocks and referencing them as plain environment variables. Fixed github-env-injection in lib/main.sh by sanitizing all user-controlled values with `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_ENV and $GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed both heredoc injection vulnerabilities in action.yml at the 'Generate comment content for deployment' and 'Generate comment content for removal' steps. Replaced the static 'EOF' heredoc delimiter with a cryptographically random 32-character hex string generated via `DELIM=$(openssl rand -hex 16)`. This prevents an attacker from injecting a line exactly matching 'EOF' in user-controlled inputs (qr-code, preview-branch, deploy-repository) to terminate the heredoc early and inject arbitrary key=value pairs into $GITHUB_OUTPUT. Also fixed $GITHUB_OUTPUT to be properly quoted in both steps.

