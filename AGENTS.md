# Bluefin-Common Copilot Instructions

This document provides essential information for coding agents working with the bluefin-common repository.

## Repository Overview

**Bluefin-Common** is a shared OCI layer containing common configuration files used across all Bluefin variants (bluefin, bluefin-dx, bluefin-lts).

- **Type**: Minimal OCI container layer (system files only)
- **Purpose**: Centralize shared configuration to reduce duplication across bluefin and bluefin-lts
- **Base**: Built from scratch with COPY directive
- **Languages**: Configuration files (JSON, shell scripts, markdown)
- **Build System**: GitHub Actions with podman/buildah

## Repository Structure

### Root Directory Files
- `Containerfile` - Multi-stage build (scratch → ctx stage with system_files)
- `cosign.pub` - Container signing public key (shared with bluefin/bluefin-lts)
- `README.md` - Basic repository description

### Key Directories
- `system_files/` - All configuration files that get copied into bluefin images
  - `etc/ublue-os/` - System configuration files (bling.json, fastfetch.json, setup.json)
  - `usr/share/ublue-os/` - User-space configurations
    - `firefox-config/` - Firefox default settings
    - `flatpak-overrides/` - Flatpak app overrides
    - `just/` - Just recipe additions
    - `motd/` - Message of the day templates and tips
    - `privileged-setup.hooks.d/` - Privileged setup hooks
    - `system-setup.hooks.d/` - System setup hooks
    - `user-setup.hooks.d/` - User setup hooks

### GitHub Actions
- `.github/workflows/build.yml` - Simple build workflow using podman/buildah

## Build Instructions

### Prerequisites
This repository requires minimal tooling:
- **podman** and **buildah** (usually pre-installed on development systems)
- No Just, no pre-commit, no complex build dependencies

### Build Commands

**Build locally:**
```bash
# Build the container
buildah build -t bluefin-common:latest -f ./Containerfile .

# Inspect the built image
podman images bluefin-common
```

**Test the image:**
```bash
# Copy files from the container to verify structure
podman create --name test bluefin-common:latest
podman cp test:/system_files ./test-output
podman rm test
tree ./test-output
```

### Build Process
1. GitHub Actions triggers on push to main or PR
2. `buildah build` creates image from `Containerfile`
3. Image is pushed to `ghcr.io/projectbluefin/bluefin-common:latest`
4. Bluefin and bluefin-lts reference this image with `COPY --from=ghcr.io/ublue-os/bluefin-common:latest`

## Usage in Downstream Projects

Bluefin and bluefin-lts use this layer in their Containerfiles:

```dockerfile
FROM ghcr.io/ublue-os/bluefin-common:latest AS bluefin-common

# Later in the build:
COPY --from=bluefin-common /system_files /desired/destination
```

## Making Changes

### Modifying Configuration Files

1. **Edit files in `system_files/`** - Maintain the existing directory structure
2. **Test locally** with buildah to ensure no syntax errors
3. **Create PR** - GitHub Actions will build and validate

### Adding New Configuration Files

1. Place files in the appropriate subdirectory under `system_files/`
2. Follow the existing path conventions:
   - System configs: `system_files/etc/ublue-os/`
   - User configs: `system_files/usr/share/ublue-os/`
3. Ensure file permissions are correct (executables for scripts)

### Common Modification Patterns
- **Firefox configs**: Edit `system_files/usr/share/ublue-os/firefox-config/`
- **Setup hooks**: Modify scripts in `system_files/usr/share/ublue-os/*-setup.hooks.d/`
- **System settings**: Update JSON files in `system_files/etc/ublue-os/`
- **Just recipes**: Add/modify `.just` files in `system_files/usr/share/ublue-os/just/`

## Validation

### Manual Validation
```bash
# Check Containerfile syntax
buildah build --dry-run -f ./Containerfile .

# Validate JSON files
find system_files -name "*.json" -exec sh -c 'echo "Checking {}"; cat {} | jq . > /dev/null' \;

# Check shell script syntax
find system_files -name "*.sh" -exec bash -n {} \;
```

### GitHub Actions
The build workflow automatically:
- Builds the container with buildah
- Pushes to GHCR on merge to main
- Validates build succeeds on PRs

## Development Guidelines

### Making Changes
1. **Keep it simple** - This repo contains only configuration files
2. **Maintain structure** - Follow existing directory patterns
3. **Test locally** - Build with buildah before pushing
4. **No complex dependencies** - This is intentionally minimal

### File Editing Best Practices
- **JSON files**: Validate syntax with `jq` before committing
- **Shell scripts**: Check syntax with `bash -n script.sh`
- **Keep files small** - Each file should have a single, clear purpose
- **Document changes** - Update comments in configuration files

## Trust These Instructions

**This repository is intentionally simple.** It contains only:
- Configuration files in `system_files/`
- A minimal Containerfile
- A simple GitHub Actions workflow

There are no complex build systems, no package management, no multi-stage builds beyond the scratch→ctx pattern.

## GitHub Label Structure

This repository follows the CNCF label pattern with `/` separators and color-coded groups.

- When asked to make github labels consistent across bluefin use this set of repositories. Do not touch any other repos:
  - @projectbluefin/common
  - @projectbluefin/distroless
  - @ublue-os/bluefin
  - @ublue-os/bluefin-lts 
  - Ensure that the colors remain consistent

This file is the source of truth for labels in Bluefin.

NEVER touch the issues themselves, only rename the labels.

### Label Categories

**kind/** - Issue types (color: `#a6e3a1` light green)
- `kind/bug` - Something isn't working
- `kind/enhancement` - New feature requests
- `kind/documentation` - Documentation improvements
- `kind/question` - Support questions
- `kind/tech-debt` - Refactoring and maintenance
- `kind/duplicate` - Duplicate issues
- `kind/invalid` - Invalid issues
- `kind/wontfix` - Won't be implemented
- `kind/github-action` - CI/CD automation
- `kind/renovate` - Dependency updates
- `kind/parity` - LTS/Bluefin differences
- `kind/automation` - Workflow automation

**area/** - Configuration areas (Catppuccin Mocha colors by domain)
- Desktop (pink `#f5c2e7`): `area/gnome`, `area/aurora`, `area/bling`
- Development (sky `#89dceb`): `area/dx`, `area/buildstream`, `area/finpilot`
- Package management (peach `#eba0ac`): `area/brew`, `area/just`
- System services (lavender `#b4befe`): `area/services`, `area/policy`
- Infrastructure (teal `#94e2d5`): `area/iso`, `area/upstream`, `area/bluespeed`

**size/** - PR size (color: `#3fb950` dark green)
- `size/XS`, `size/S`, `size/M`, `size/L`, `size/XL`, `size/XXL`

**Other labels** (keep original colors)
- `good first issue` (#7057ff)
- `help wanted` (#008672)
- `lgtm` (#238636)
- `dependencies` (#0366d6)
- `stale` (#dadada)
- `aarch64` (#a8f908)

### Modifying Labels

Use `gh label edit` to rename or recolor labels:

```bash
# Rename and recolor
gh label edit "old-name" --name "new-name" --color "a6e3a1"

# Update only color for kind/ labels
gh label edit "kind/example" --color "a6e3a1"

# Update area/ label with appropriate domain color
gh label edit "area/example" --color "f5c2e7"  # Use domain color
```

When adding new labels, follow the prefix/color pattern above:
- All `kind/` labels use `#a6e3a1` (light green)
- All `size/` labels use `#3fb950` (dark green)
- `area/` labels use domain-specific Catppuccin Mocha colors

## Beads Isolation Strategy

**CRITICAL:** This repository uses beads for local task tracking, but beads metadata **MUST NEVER** be pushed to projectbluefin/common.

### Repository Structure
- **Upstream:** `projectbluefin/common` - Clean, production repository (NO beads)
- **Fork:** `castrojo/common` - Development fork with beads tracking
- **Local:** Full beads configuration in `.beads/` directory

### What Gets Pushed Where

**To projectbluefin/common (via PR):**
- ✅ Configuration files in `system_files/`
- ✅ `Containerfile` changes
- ✅ `.github/workflows/` updates
- ✅ `README.md` documentation
- ❌ **NEVER** `.beads/` directory
- ❌ **NEVER** `.gitattributes` (contains beads merge config)
- ❌ **NEVER** `AGENTS.md` (this file stays in fork only)

**To castrojo/common (fork):**
- ✅ All beads configuration (`.beads/` directory)
- ✅ `.gitattributes` (for beads merge strategy)
- ✅ `AGENTS.md` (agent instructions - **periodically** when requested)
- ✅ Local development branches

### Isolation Enforcement

**1. Git Configuration:**
```bash
# .beads/ is tracked in your fork for collaboration
# .beads/.gitignore already excludes runtime files (*.db, daemon.*, etc.)
# .gitattributes configures beads merge driver

# When creating PRs, these files are excluded from feature branches
```

**2. Branch Strategy:**
```bash
# Main branch (local): Contains beads + AGENTS.md
git checkout main  # Has .beads/, .gitattributes, AGENTS.md

# Feature branches: Clean, no beads metadata
git checkout -b feat/add-firefox-config
# Work tracked in beads, but beads files NOT committed to feature branch
# Only commit actual feature changes (system_files/, etc.)
```

**3. PR Creation Rules:**
- Feature branches must NOT contain `.beads/`, `.gitattributes`, or `AGENTS.md`
- Only commit actual changes relevant to the feature
- Beads tracking happens on main branch in your fork
- PRs to projectbluefin/common are minimal and clean

### Workflow Integration

**Starting a new feature:**
```bash
# On main branch (has beads)
bd create --title="Add Firefox config" --type=feature --priority=2
bd update bluefin-common-1 --status=in_progress

# Create clean feature branch
git checkout -b feat/add-firefox-config

# Make ONLY the feature changes
# Do NOT commit .beads/, .gitattributes, or AGENTS.md

# Commit feature changes
git add system_files/usr/share/ublue-os/firefox-config/
git commit -m "feat: add Firefox privacy defaults"

# Return to main to update beads
git checkout main
bd close bluefin-common-1
git commit -m "docs: close beads issue for Firefox config"

# Back to feature branch for PR
git checkout feat/add-firefox-config
```

**Verification Before PR:**
```bash
# Ensure feature branch is clean
git diff main feat/add-firefox-config -- .beads .gitattributes AGENTS.md
# Should show NO differences (these files not in feature branch)

git log feat/add-firefox-config --oneline
# Should show ONLY feature commits, no beads updates
```

### Why This Matters

1. **Clean PRs:** Upstream maintainers see only relevant changes
2. **No confusion:** projectbluefin/common stays beads-free
3. **Full tracking:** You maintain complete task history in fork
4. **Isolation:** Beads workflow doesn't pollute production repo
5. **Flexibility:** AGENTS.md updates pushed to fork when ready, not with every PR

### AGENTS.md Updates

**This file (AGENTS.md):**
- Lives in `castrojo/common` fork only
- Updated frequently during development
- Pushed to fork periodically when requested
- **NEVER** included in PRs to projectbluefin/common
- Evolves independently from production changes

## Pull Request Workflow

**When creating a PR to projectbluefin/common**, follow this workflow:

### 1. Create Feature Branch
```bash
git checkout -b feat/your-feature-name
# Use conventional commit prefixes: feat/, fix/, docs/, ci/, refactor/
```

### 2. Make Changes and Track with Beads
```bash
bd create --title="Your task" --type=feature --priority=2
bd update <id> --status=in_progress
# ... make your changes ...
bd close <id>
```

### 3. Squash Before Push (MANDATORY)
**CRITICAL:** Always squash your commits before pushing to your fork.

```bash
# Interactive rebase to squash all commits
git rebase -i main

# In the editor, keep first 'pick', change others to 'squash' or 's'
# Edit the final commit message to follow conventional commits

# Verify single commit
git log --oneline -5
```

### 4. Push to Your Fork
```bash
# Push squashed commit to your fork
git push origin feat/your-feature-name

# If you had previously pushed, force push the squashed version
git push origin feat/your-feature-name --force-with-lease
```

### 5. Open PR in Browser (MANDATORY)
**CRITICAL:** Always open the PR in the browser so the user can edit it themselves.

```bash
# Open PR creation page in browser
gh pr create --web --repo projectbluefin/common
```

**DO NOT** use `gh pr create` with `--title` or `--body` flags. The user must have full control to edit the PR description, title, and details in the GitHub web interface.

### PR Guidelines
- **Commit message format:** Follow [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/#specification)
- **Single commit:** PRs should contain exactly one squashed commit
- **Clear description:** User will add context in web interface
- **Reference issues:** Link related beads issues if applicable

## Other Rules

- Use [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/#specification) for all commits and PR titles
- Keep changes minimal and surgical
- This layer is used by both bluefin (ublue-os/bluefin) and bluefin-lts (ublue-os/bluefin-lts)
- Changes here affect all downstream Bluefin variants

## Attribution Requirements

AI agents must disclose what tool and model they are using in the "Assisted-by" commit footer:

```text
Assisted-by: [Model Name] via [Tool Name]
```

Example:

```text
Assisted-by: Claude 3.5 Sonnet via GitHub Copilot
```

## Landing the Plane (Session Completion)

**When ending a work session**, you MUST complete ALL steps below. Work is NOT complete until `git push` succeeds.

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **PUSH TO REMOTE** - This is MANDATORY:
   ```bash
   git pull --rebase
   bd sync
   git push
   git status  # MUST show "up to date with origin"
   ```
5. **Clean up** - Clear stashes, prune remote branches
6. **Verify** - All changes committed AND pushed
7. **Hand off** - Provide context for next session

**CRITICAL RULES:**
- Work is NOT complete until `git push` succeeds
- NEVER stop before pushing - that leaves work stranded locally
- NEVER say "ready to push when you are" - YOU must push
- If push fails, resolve and retry until it succeeds
