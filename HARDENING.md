<!-- markdownlint-disable -->

# Hardening Report: rossjrw--pr-preview-action/v1.7.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rossjrw--pr-preview-action/v1.7.2** was hardened automatically. 12 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Four `run:` blocks in action.yml directly interpolate GitHub Actions expressions (`${{ ... }}`) into shell command strings, enabling script injection. (1) 'Wait for preview deployment on GitHub Pages': `${{ inputs.deploy-repository }}`, `${{ steps.deployed-commit.outputs.deployed_commit_sha }}`, `${{ inputs.preview-branch }}`, and `${{ inputs.token }}` are interpolated directly as shell arguments to `wait_for_pages_deployment`. (2) 'Generate comment content for deployment': `${{ env.action_repository }}`, `${{ env.action_version }}`, `${{ env.preview_url }}`, `${{ inputs.preview-branch }}`, `${{ github.server_url }}`, `${{ inputs.deploy-repository }}`, `${{ env.action_start_time }}` are interpolated as shell arguments. (3) 'Wait for preview removal on GitHub Pages': same pattern as (1) but using `${{ steps.removed-commit.outputs.deployed_commit_sha }}`. (4) 'Generate comment content for removal': same pattern as (2). All of these should be routed through `env:` variables and then double-quoted in the shell script.

Locations:

- `action.yml:185`
- `action.yml:200`
- `action.yml:255`
- `action.yml:270`

### github-env-injection (severity: high)

lib/main.sh writes multiple variables derived from user-controlled inputs to `$GITHUB_ENV` and `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). The variables written include: `pages_base_url` (sourced from `inputs.pages-base-url` or `inputs.custom-url`), `preview_file_path` (constructed from `inputs.umbrella-dir` and `inputs.pr-number`), `preview_url_path` (derived from the above), `deployment_action` (from `inputs.action`), `action_repository` (from `github.action_repository`), and `preview_url` (constructed from user-controlled base URL and path). An attacker could inject newlines into any of these inputs to poison the `GITHUB_ENV` or `GITHUB_OUTPUT` files and set arbitrary environment variables or outputs for subsequent steps.

Locations:

- `lib/main.sh:43`
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

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Generate comment content for deployment"; move to env: map

Locations:

- `action.yml:191`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Generate comment content for deployment"; move to env: map

Locations:

- `action.yml:193`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:247`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:247`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step "Wait for preview removal on GitHub Pages"; move to env: map

Locations:

- `action.yml:247`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.preview-branch }}" appears directly in run: block of step "Generate comment content for removal"; move to env: map

Locations:

- `action.yml:261`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.deploy-repository }}" appears directly in run: block of step "Generate comment content for removal"; move to env: map

Locations:

- `action.yml:263`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Fixed all script injection findings in action.yml by moving ${{ }} expressions from run: blocks into env: blocks and referencing them as double-quoted shell variables. Fixed four steps: 'Wait for preview deployment on GitHub Pages', 'Generate comment content for deployment', 'Wait for preview removal on GitHub Pages', and 'Generate comment content for removal'. Fixed github-env-injection in lib/main.sh by sanitizing all user-controlled values with `printf '%s' "$VAR" | tr -d '\n\r'` before writing them to $GITHUB_ENV and $GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed both github-env-injection findings in action.yml. In both 'Generate comment content for deployment' and 'Generate comment content for removal' steps, added sanitization of the user-controlled inputs GC_PREVIEW_BRANCH and GC_DEPLOY_REPOSITORY using `printf '%s' "$VAR" | tr -d '\n\r'` before passing them to generate-comment.sh. The sanitized values (SAFE_PREVIEW_BRANCH and SAFE_DEPLOY_REPOSITORY) are then used in the generate-comment.sh invocation, preventing newline injection that could break the heredoc EOF delimiter and inject arbitrary key=value pairs into $GITHUB_OUTPUT.

