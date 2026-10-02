<!-- markdownlint-disable -->

# Hardening Report: rossjrw--pr-preview-action/v1.7.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rossjrw--pr-preview-action/v1.7.2** was hardened automatically. 12 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Four run: blocks in action.yml directly interpolate ${{ }} expressions inside shell command strings, enabling script injection.

1. 'Wait for preview deployment on GitHub Pages' step (line 177): `wait_for_pages_deployment "${{ inputs.deploy-repository }}" "${{ steps.deployed-commit.outputs.deployed_commit_sha }}" "${{ inputs.preview-branch }}" "${{ inputs.token }}"` — inputs.deploy-repository, inputs.preview-branch, inputs.token, and steps output are interpolated directly into the shell command.

2. 'Generate comment content for deployment' step (lines 187–194): Multiple ${{ env.action_repository }}, ${{ env.action_version }}, ${{ env.preview_url }}, ${{ inputs.preview-branch }}, ${{ github.server_url }}, ${{ inputs.deploy-repository }}, ${{ env.action_start_time }} are interpolated directly as shell arguments to generate-comment.sh.

3. 'Wait for preview removal on GitHub Pages' step (line 250): Same pattern as #1 with ${{ inputs.deploy-repository }}, ${{ steps.removed-commit.outputs.deployed_commit_sha }}, ${{ inputs.preview-branch }}, ${{ inputs.token }}.

4. 'Generate comment content for removal' step (lines 260–267): Same pattern as #2 with the same set of ${{ }} expressions as shell arguments.

An attacker controlling any of these inputs (e.g. via a malicious PR branch name or deploy-repository value) can inject arbitrary shell commands.

Locations:

- `action.yml:177`
- `action.yml:187`
- `action.yml:250`
- `action.yml:260`

### github-env-injection (severity: high)

lib/main.sh writes values derived from untrusted inputs to $GITHUB_ENV and $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r').

The 'Setup preview environment' step sets env vars (umbrella_path, pr_number, pages_base_url, pages_base_path, deployment_repository, deprecated_custom_url) directly from inputs.* and github.* context values. lib/main.sh then uses these to compute preview_file_path, preview_url_path, preview_url, pages_base_url, action_version, etc., and writes them unsanitized to $GITHUB_ENV (lines 44–52) and $GITHUB_OUTPUT (lines 54–62).

For example:
  echo "preview_url=https://$pages_base_url/$preview_url_path/" >> "$GITHUB_ENV"
  echo "preview_file_path=$preview_file_path" >> "$GITHUB_ENV"

Since pages_base_url and preview_file_path are derived from inputs.pages-base-url, inputs.umbrella-dir, and inputs.pr-number (all caller-controlled), a newline injected into any of these inputs can write arbitrary key=value pairs into GITHUB_ENV, allowing environment variable hijacking in subsequent steps.

Locations:

- `lib/main.sh:44`
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

Fixed all 12 findings across action.yml and lib/main.sh:

1. action.yml - 'Wait for preview deployment on GitHub Pages' step: Moved inputs.deploy-repository, steps.deployed-commit.outputs.deployed_commit_sha, inputs.preview-branch, and inputs.token into an env: block (WAIT_DEPLOY_REPO, WAIT_DEPLOY_SHA, WAIT_PREVIEW_BRANCH, WAIT_TOKEN) and referenced them as shell variables.

2. action.yml - 'Generate comment content for deployment' step: Moved all 7 ${{ }} expressions (env.action_repository, env.action_version, env.preview_url, inputs.preview-branch, github.server_url, inputs.deploy-repository, env.action_start_time) into an env: block (GC_* prefixed vars) and referenced them as shell variables.

3. action.yml - 'Wait for preview removal on GitHub Pages' step: Same fix as #1 but for the removal step (using steps.removed-commit.outputs.deployed_commit_sha).

4. action.yml - 'Generate comment content for removal' step: Same fix as #2 but for the removal step.

5. lib/main.sh - Added a _safe() helper function using printf '%s' | tr -d '\n\r' and wrapped all values written to $GITHUB_ENV and $GITHUB_OUTPUT with _safe() to prevent newline injection attacks from caller-controlled inputs.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed both 'Generate comment content for deployment' (line 175) and 'Generate comment content for removal' (line 248) steps in action.yml. Two mitigations applied to each step: (1) sanitized user-controlled inputs GC_PREVIEW_BRANCH and GC_DEPLOY_REPOSITORY with `printf '%s' "$VAR" | tr -d '\n\r'` before passing them to generate-comment.sh; (2) replaced the static 'EOF' heredoc delimiter with a cryptographically random delimiter `GCEOF_$(openssl rand -hex 16)` to prevent an attacker from injecting a matching delimiter line. Also quoted $GITHUB_OUTPUT properly.

