# Hardening Report: rossjrw--pr-preview-action/v1.7.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **rossjrw--pr-preview-action/v1.7.3** was hardened automatically. 12 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Four run: blocks in action.yml directly interpolate ${{ inputs.* }} and ${{ github.* }} expressions into shell commands without first assigning them to environment variables. (1) The 'Wait for preview deployment on GitHub Pages' step passes ${{ inputs.deploy-repository }}, ${{ inputs.preview-branch }}, and ${{ inputs.token }} directly as shell arguments to wait_for_pages_deployment. (2) The 'Generate comment content for deployment' step passes ${{ inputs.preview-branch }}, ${{ github.server_url }}, and ${{ inputs.deploy-repository }} directly as shell arguments to generate-comment.sh. (3) The 'Wait for preview removal on GitHub Pages' step has the same pattern as (1). (4) The 'Generate comment content for removal' step has the same pattern as (2). An attacker controlling these inputs via a malicious PR could inject arbitrary shell commands.

Locations:

- `action.yml:175`
- `action.yml:188`
- `action.yml:230`
- `action.yml:243`

### github-env-injection (severity: high)

Two run: blocks write attacker-controlled values to $GITHUB_OUTPUT without sanitization. The 'Generate comment content for deployment' step constructs $CONTENT from shell arguments that include ${{ inputs.preview-branch }}, ${{ github.server_url }}, and ${{ inputs.deploy-repository }}, then writes it to $GITHUB_OUTPUT via a heredoc (echo content<<EOF ... >> $GITHUB_OUTPUT). The 'Generate comment content for removal' step has the same pattern. A newline injected via any of these inputs could break out of the heredoc and inject arbitrary environment variables or outputs. The required sanitization (printf '%s' ... | tr -d '\n\r') is absent.

Locations:

- `action.yml:188`
- `action.yml:243`

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed all 12 findings across 4 steps in action.yml:

1. 'Wait for preview deployment on GitHub Pages': Added env: block with DEPLOY_REPOSITORY, DEPLOYED_COMMIT_SHA, PREVIEW_BRANCH, DEPLOY_TOKEN. Replaced all ${{ inputs.* }} and ${{ steps.*.outputs.* }} expressions in run: with env var references.

2. 'Generate comment content for deployment': Added env: block with INPUT_PREVIEW_BRANCH, INPUT_SERVER_URL, INPUT_DEPLOY_REPOSITORY. Added sanitization via `printf '%s' | tr -d '\n\r'` for each input before passing to generate-comment.sh. Quoted $GITHUB_OUTPUT.

3. 'Wait for preview removal on GitHub Pages': Same fix as #1 but for the removal step (REMOVED_COMMIT_SHA instead of DEPLOYED_COMMIT_SHA).

4. 'Generate comment content for removal': Same fix as #2 but for the removal step.

All ${{ inputs.* }} and ${{ github.* }} expressions have been moved out of run: blocks into env: maps, and attacker-controlled values are sanitized before being used as shell arguments or written to $GITHUB_OUTPUT.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed lib/main.sh by: (1) adding a sanitize() helper function that uses `printf '%s' "$1" | tr -d '\n\r'` to strip newlines and carriage returns; (2) sanitizing all attacker-controlled values (deployment_action, preview_file_path, pages_base_url, preview_url_path, preview_url, action_repository, action_version, action_start_time, action_start_timestamp) into safe_* variables before writing them to $GITHUB_ENV and $GITHUB_OUTPUT. This prevents newline injection attacks where attacker-controlled inputs (inputs.umbrella-dir, inputs.pages-base-url, inputs.pages-base-path, inputs.deploy-repository, inputs.pr-number, github.event.number, github.repository, github.ref) could inject arbitrary environment variable definitions or outputs.

