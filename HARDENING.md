# Hardening Report: rossjrw--pr-preview-action/v1.7.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

Action **rossjrw--pr-preview-action/v1.7.1** was hardened automatically. 8 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two run: blocks in action.yml directly interpolate inputs.* expressions inside shell commands without first assigning them to environment variables. The 'Wait for preview deployment on GitHub Pages' step passes ${{ inputs.deploy-repository }}, ${{ inputs.preview-branch }}, and ${{ inputs.token }} directly as shell arguments to wait_for_pages_deployment. The 'Wait for preview removal on GitHub Pages' step does the same. An attacker controlling these inputs (e.g. via a pull_request event) could inject arbitrary shell commands.

Locations:

- `action.yml:193`
- `action.yml:261`

### github-env-injection (severity: high)

lib/main.sh writes attacker-controlled values derived from inputs.* (umbrella_path from inputs.umbrella-dir, pr_number from inputs.pr-number, pages_base_url from inputs.pages-base-url, pages_base_path from inputs.pages-base-path, deployment_repository from inputs.deploy-repository, deprecated_custom_url from inputs.custom-url) directly to $GITHUB_ENV and $GITHUB_OUTPUT using plain echo, without the required sanitization step (printf '%s' "$VAR" | tr -d '\n\r'). A malicious input containing newlines could inject arbitrary environment variable definitions into subsequent steps.

Locations:

- `lib/main.sh:43`
- `lib/main.sh:54`

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

1. action.yml - 'Wait for preview deployment on GitHub Pages' and 'Wait for preview removal on GitHub Pages' steps: Moved ${{ inputs.deploy-repository }}, ${{ inputs.preview-branch }}, and ${{ inputs.token }} out of run: shell blocks and into env: blocks (as DEPLOY_REPOSITORY, PREVIEW_BRANCH, DEPLOY_TOKEN, DEPLOYED_COMMIT_SHA/REMOVED_COMMIT_SHA). Shell commands now reference these as plain $ENV_VAR variables.

2. lib/main.sh - Added sanitization of all attacker-controlled input-derived values (umbrella_path, pr_number, pages_base_url, pages_base_path, deployment_repository, deprecated_custom_url, preview_file_path, preview_url_path) using `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_ENV and $GITHUB_OUTPUT. All echo statements to those files now use the safe_* sanitized variables.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In lib/main.sh, added sanitization for the `github_action_repository` variable by creating `safe_github_action_repository=$(printf '%s' "$github_action_repository" | tr -d '\n\r')` alongside the other safe_* variables, and replaced the unsanitized `echo "action_repository=$github_action_repository"` with `echo "action_repository=$safe_github_action_repository"` when writing to $GITHUB_ENV. This follows the same pattern already used for all other attacker-controlled inputs (umbrella_path, pr_number, pages_base_url, etc.) and prevents newline injection attacks via a crafted repository name.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

In lib/main.sh, added sanitization for the `deployment_action` variable by creating `safe_deployment_action=$(printf '%s' "$deployment_action" | tr -d '\n\r')` alongside the other safe_* variable definitions. Replaced both unsanitized `echo "deployment_action=$deployment_action"` writes (lines 52 and 63) with `echo "deployment_action=$safe_deployment_action"` in the GITHUB_ENV and GITHUB_OUTPUT write blocks. This prevents newline injection attacks where an attacker could inject arbitrary entries into the GitHub environment file via the `inputs.action` parameter.

