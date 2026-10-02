<!-- markdownlint-disable -->

# Hardening Report: rossjrw--pr-preview-action/v1.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rossjrw--pr-preview-action/v1.8.1** was hardened automatically. 16 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): The 'Wait for preview deployment on GitHub Pages' run: block directly interpolates ${{ inputs.deploy-repository }}, ${{ steps.deployed-commit.outputs.deployed_commit_sha }}, ${{ inputs.preview-branch }}, and ${{ inputs.token }} inside the shell command string. YAML template substitution occurs before the shell parses the command, allowing an attacker-controlled value to inject arbitrary shell commands. Offending line: `wait_for_pages_deployment "${{ inputs.deploy-repository }}" "${{ steps.deployed-commit.outputs.deployed_commit_sha }}" "${{ inputs.preview-branch }}" "${{ inputs.token }}"`

Locations:

- `action.yml:196`

### script-injection (severity: high)

Rule (a): The 'Generate comment content for deployment' run: block directly interpolates multiple ${{ }} expressions inside shell command strings, including ${{ env.action_repository }}, ${{ env.action_version }}, ${{ env.preview_url }}, ${{ inputs.preview-branch }}, ${{ github.server_url }}, ${{ inputs.deploy-repository }}, ${{ env.action_start_time }}, and ${{ inputs.qr-code }}. YAML template substitution occurs before the shell parses the command, allowing attacker-controlled values to inject arbitrary shell commands.

Locations:

- `action.yml:207`

### script-injection (severity: high)

Rule (a): The 'Wait for preview removal on GitHub Pages' run: block directly interpolates ${{ inputs.deploy-repository }}, ${{ steps.removed-commit.outputs.deployed_commit_sha }}, ${{ inputs.preview-branch }}, and ${{ inputs.token }} inside the shell command string. YAML template substitution occurs before the shell parses the command, allowing an attacker-controlled value to inject arbitrary shell commands. Offending line: `wait_for_pages_deployment "${{ inputs.deploy-repository }}" "${{ steps.removed-commit.outputs.deployed_commit_sha }}" "${{ inputs.preview-branch }}" "${{ inputs.token }}"`

Locations:

- `action.yml:265`

### script-injection (severity: high)

Rule (a): The 'Generate comment content for removal' run: block directly interpolates multiple ${{ }} expressions inside shell command strings, including ${{ env.action_repository }}, ${{ env.action_version }}, ${{ env.preview_url }}, ${{ inputs.preview-branch }}, ${{ github.server_url }}, ${{ inputs.deploy-repository }}, ${{ env.action_start_time }}, and ${{ env.qr_code_provider }}. YAML template substitution occurs before the shell parses the command, allowing attacker-controlled values to inject arbitrary shell commands.

Locations:

- `action.yml:276`

### github-env-injection (severity: high)

lib/main.sh writes multiple values derived from inherited env vars (set from inputs.* and github.* context via the env: block of the calling step) to $GITHUB_ENV and $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Variables written include: pages_base_url (from inputs.pages-base-url), preview_url (constructed from pages_base_url), action_repository (from github.action_repository/github.repository), action_version (from action_ref), action_start_time, deployment_action (from inputs.action), preview_url_path, and preview_file_path. An attacker-controlled input containing newlines could inject arbitrary environment variable assignments into $GITHUB_ENV or output entries into $GITHUB_OUTPUT.

Locations:

- `lib/main.sh:43`
- `lib/main.sh:53`

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

Fixed all 16 findings across action.yml and lib/main.sh:

1. action.yml - 'Wait for preview deployment on GitHub Pages': Moved ${{ inputs.deploy-repository }}, ${{ steps.deployed-commit.outputs.deployed_commit_sha }}, ${{ inputs.preview-branch }}, ${{ inputs.token }} to env: block as WAIT_DEPLOY_REPO, WAIT_DEPLOYED_SHA, WAIT_PREVIEW_BRANCH, WAIT_TOKEN.

2. action.yml - 'Generate comment content for deployment': Moved all 8 ${{ }} expressions to env: block with GC_ prefix (GC_ACTION_REPOSITORY, GC_ACTION_VERSION, GC_PREVIEW_URL, GC_PREVIEW_BRANCH, GC_SERVER_URL, GC_DEPLOY_REPOSITORY, GC_ACTION_START_TIME, GC_QR_CODE).

3. action.yml - 'Wait for preview removal on GitHub Pages': Same fix as #1 but for removal step, using WAIT_REMOVED_SHA.

4. action.yml - 'Generate comment content for removal': Same fix as #2 but for removal step, using GC_QR_CODE_PROVIDER for env.qr_code_provider.

5. lib/main.sh - Added sanitization of all values written to $GITHUB_ENV and $GITHUB_OUTPUT using printf '%s' "$VAR" | tr -d '\n\r' to prevent newline injection attacks from attacker-controlled inputs.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed both 'Generate comment content for deployment' (line 175) and 'Generate comment content for removal' (line 250) steps in action.yml. Replaced the static `EOF` heredoc delimiter used when writing to $GITHUB_OUTPUT with a randomly-generated delimiter: `DELIM="EOF_$(openssl rand -hex 16)"`. This prevents an attacker from injecting arbitrary key=value pairs into $GITHUB_OUTPUT by crafting user-controlled inputs (preview-branch, deploy-repository, qr-code) that contain a newline followed by a line matching the delimiter.

