# Hardening Report: rossjrw--pr-preview-action/v1.7.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **rossjrw--pr-preview-action/v1.7.2** was hardened automatically. 15 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Wait for preview deployment on GitHub Pages' (deploy) run: block directly interpolates attacker-controlled inputs expressions into shell command arguments without first assigning them to environment variables. Specifically, `${{ inputs.deploy-repository }}`, `${{ inputs.preview-branch }}`, and `${{ inputs.token }}` are passed directly as arguments to the `wait_for_pages_deployment` shell function. An attacker controlling these inputs could inject arbitrary shell commands.

Locations:

- `action.yml:192`

### script-injection (severity: high)

The 'Generate comment content for deployment' run: block directly interpolates attacker-controlled expressions into shell command arguments: `${{ inputs.preview-branch }}`, `${{ github.server_url }}`, and `${{ inputs.deploy-repository }}` are passed directly as arguments to `generate-comment.sh`. An attacker controlling these inputs could inject arbitrary shell commands.

Locations:

- `action.yml:203`

### script-injection (severity: high)

The 'Wait for preview removal on GitHub Pages' (remove) run: block directly interpolates attacker-controlled inputs expressions into shell command arguments without first assigning them to environment variables. Specifically, `${{ inputs.deploy-repository }}`, `${{ inputs.preview-branch }}`, and `${{ inputs.token }}` are passed directly as arguments to the `wait_for_pages_deployment` shell function.

Locations:

- `action.yml:249`

### script-injection (severity: high)

The 'Generate comment content for removal' run: block directly interpolates attacker-controlled expressions into shell command arguments: `${{ inputs.preview-branch }}`, `${{ github.server_url }}`, and `${{ inputs.deploy-repository }}` are passed directly as arguments to `generate-comment.sh`. An attacker controlling these inputs could inject arbitrary shell commands.

Locations:

- `action.yml:260`

### github-env-injection (severity: high)

lib/main.sh writes multiple attacker-controlled values to $GITHUB_ENV and $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` applied before writes). The values written include: `umbrella_path` (from inputs.umbrella-dir), `pr_number` (from inputs.pr-number / github.event.number), `deployment_repository` (from inputs.deploy-repository), `pages_base_url` (derived from deployment_repository), `preview_file_path`, `preview_url_path`, and `preview_url` (all derived from attacker-controlled inputs). A newline injection in any of these values could allow an attacker to set arbitrary environment variables or action outputs.

Locations:

- `lib/main.sh:44`
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

Fixed all 15 findings across action.yml and lib/main.sh:

1. action.yml - 'Wait for preview deployment on GitHub Pages' (deploy): Moved ${{ inputs.deploy-repository }}, ${{ inputs.preview-branch }}, ${{ inputs.token }} to env: block as DEPLOY_REPOSITORY, PREVIEW_BRANCH, DEPLOY_TOKEN. Shell script now references plain env vars.

2. action.yml - 'Generate comment content for deployment': Moved ${{ inputs.preview-branch }}, ${{ github.server_url }}, ${{ inputs.deploy-repository }} to env: block as INPUT_PREVIEW_BRANCH, INPUT_SERVER_URL, INPUT_DEPLOY_REPOSITORY. Other args ($action_repository, $action_version, $preview_url, $action_start_time) are already available as env vars from the setup step via GITHUB_ENV.

3. action.yml - 'Wait for preview removal on GitHub Pages' (remove): Same fix as the deploy wait step, using DEPLOY_REPOSITORY, REMOVED_COMMIT_SHA, PREVIEW_BRANCH, DEPLOY_TOKEN env vars.

4. action.yml - 'Generate comment content for removal': Same fix as the deploy comment step.

5. lib/main.sh - Added sanitization of all attacker-controlled values (umbrella_path, pr_number, deployment_repository, pages_base_url, preview_file_path, preview_url_path, preview_url, deployment_action, github_action_repository, action_version, action_start_time, action_start_timestamp) using `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_ENV and $GITHUB_OUTPUT.

