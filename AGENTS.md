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
# .beads/ is tracked in castrojo/common fork (main branch only)
# .beads/.gitignore excludes runtime files (*.db, daemon.*, etc.)
# .gitattributes configures beads merge driver for collaboration
# Fork main branch has full beads tracking for shared task management
```

**2. Branch Strategy:**
```bash
# Main branch: Contains beads + AGENTS.md (pushed to castrojo/common)
git checkout main  # Has .beads/, .gitattributes, AGENTS.md tracked in fork

# Feature branches: Clean, no beads metadata
git checkout -b feat/add-firefox-config
# Work tracked in beads on main, but beads files NOT in feature branch
# Only commit actual feature changes (system_files/, etc.)
```

**3. PR Creation Rules:**
- Feature branches must NOT contain `.beads/`, `.gitattributes`, or `AGENTS.md`
- Only commit actual changes relevant to the feature
- Beads tracking syncs via fork's main branch
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
6. **Collaboration:** Beads data syncs via fork for team task visibility

### AGENTS.md Updates

**This file (AGENTS.md):**
- Lives in `castrojo/common` fork only
- Updated frequently during development
- Pushed to fork periodically when requested
- **NEVER** included in PRs to projectbluefin/common
- Evolves independently from production changes

## Multi-Repository Beads Tracking

**CRITICAL:** This repository (`castrojo/common`) is the **central beads database** for ALL Bluefin work across multiple repositories.

### Architecture Overview

```
UPSTREAM REPOSITORIES → WORKING FORKS (with beads)
├── projectbluefin/common          → castrojo/common (BEADS HUB - full database)
├── ublue-os/bluefin               → castrojo/bluefin (redirect to hub)
├── ublue-os/bluefin-lts           → castrojo/bluefin-lts (redirect to hub)
├── projectbluefin/dakota          → castrojo/dakota (redirect to hub)
├── projectbluefin/documentation   → castrojo/documentation (redirect to hub)
├── projectbluefin/iso             → castrojo/iso (manual reference)
├── projectbluefin/egg             → castrojo/egg (manual reference)
├── projectbluefin/brew            → castrojo/brew (manual reference)
├── projectbluefin/testfin         → castrojo/testfin (manual reference)
├── projectbluefin/finpilot        → castrojo/finpilot (manual reference)
├── projectbluefin/branding        → castrojo/branding (manual reference)
└── projectbluefin/website         → castrojo/website (manual reference)
```

### Tracked Repositories

This beads database tracks work across these Bluefin repositories (as documented in monthly progress reports at `projectbluefin/documentation/reports/`):

**Tier 1 - Primary Development (with beads redirect):**
- **ublue-os/bluefin** - Main Bluefin distribution (Fedora-based)
- **ublue-os/bluefin-lts** - LTS variant built on CentOS with bootc
- **projectbluefin/dakota** - Distroless Bluefin variant
- **projectbluefin/documentation** - User documentation and guides
- **projectbluefin/common** - This repo (shared OCI configuration layer)

**Tier 2 - Infrastructure (manual reference):**
- **projectbluefin/iso** - ISO builder for installation media
- **projectbluefin/egg** - Buildstream for building Bluefin
- **projectbluefin/brew** - Homebrew tarball generation and integration
- **projectbluefin/testfin** - Testing infrastructure
- **projectbluefin/finpilot** - Custom Bluefin builder

**Tier 3 - Supporting (manual reference):**
- **projectbluefin/branding** - Branding assets (logos, wallpapers)
- **projectbluefin/website** - Public website
- **ublue-os/homebrew-tap** - Production Homebrew packages (if forked)
- **ublue-os/homebrew-experimental-tap** - Experimental packages (if forked)

### Setup: Beads Redirect in Working Forks

For Tier 1 repositories where you actively develop, set up beads redirect to share the central database.

**Assumed directory structure:**
```
~/src/
├── bluefin-common/    (beads hub - full .beads/ database)
├── bluefin/           (redirects to ../bluefin-common/.beads)
├── bluefin-lts/       (redirects to ../bluefin-common/.beads)
├── dakota/            (redirects to ../bluefin-common/.beads)
└── documentation/     (redirects to ../bluefin-common/.beads)
```

**Initial setup (per Tier 1 repository):**
```bash
# In castrojo/bluefin
cd ~/src/bluefin
bd init --redirect ../bluefin-common/.beads

# In castrojo/bluefin-lts
cd ~/src/bluefin-lts
bd init --redirect ../bluefin-common/.beads

# In castrojo/dakota
cd ~/src/dakota
bd init --redirect ../bluefin-common/.beads

# In castrojo/documentation
cd ~/src/documentation
bd init --redirect ../bluefin-common/.beads
```

**Verify redirect:**
```bash
# In any redirected repo:
bd list
# Should show issues from central database

bd stats
# Should show same stats as bluefin-common
```

**Git isolation:**
- `.beads/redirect` is automatically in `.git/info/exclude`
- Feature branches won't see `.beads/` directory
- PRs to upstream remain clean (no beads metadata)

### Naming Convention: Repository Prefixes

Use repository name prefixes in issue titles to indicate which repo the work belongs to:

**Format:** `[repo-name] Description of work`

**Examples:**
```bash
bd create --title="[bluefin] Add Firefox privacy defaults" --type=task
bd create --title="[bluefin-lts] Port privacy config to LTS" --type=task
bd create --title="[common] Update shared flatpak overrides" --type=task
bd create --title="[dakota] Add distroless variant support" --type=task
bd create --title="[iso] Implement Dakota installer" --type=feature
bd create --title="[docs] Document privacy features" --type=task
bd create --title="[multi] Cross-repo testing infrastructure" --type=epic
```

**Repository prefix mapping:**
- `[bluefin]` - ublue-os/bluefin
- `[bluefin-lts]` - ublue-os/bluefin-lts
- `[common]` - projectbluefin/common (this repo)
- `[dakota]` - projectbluefin/dakota
- `[iso]` - projectbluefin/iso
- `[egg]` - projectbluefin/egg
- `[brew]` - projectbluefin/brew
- `[docs]` - projectbluefin/documentation
- `[website]` - projectbluefin/website
- `[multi]` - Work spanning multiple repositories

### Workflow: Creating Issues Across Repositories

**From Tier 1 repos (with redirect):**
```bash
# In any repo with redirect, bd commands update central database
cd ~/src/bluefin
bd create --title="[bluefin] Implement hardware video decoding" \
  --description="Update Firefox config to enable HW decode" \
  --type=task --priority=2

# Issue is created in central database
# Visible from all repos with redirects
```

**From Tier 2/3 repos (manual reference):**
```bash
# Always create in common repo
cd ~/src/bluefin-common
bd create --title="[iso] Add production secrets to ISO repo" \
  --description="Related to projectbluefin/iso#78
  
  Add secrets needed for ISO signing and publishing." \
  --type=task --priority=1
```

**Linking to external issues/PRs:**
```bash
bd create --title="[bluefin] Fix rebase-helper for testing branches" \
  --description="Implements ublue-os/bluefin#1234
  
  Context: Rebase-helper currently doesn't handle testing branches.
  Need to detect :testing suffix and adjust logic." \
  --type=bug --priority=1
```

### Workflow: Working on Multi-Repo Issues

**1. Create issue in central database:**
```bash
# From any redirected repo OR common
bd create --title="[bluefin] Add privacy defaults" --type=task --priority=2
bd update bluefin-common-abc --status=in_progress
```

**2. Do work in appropriate repository:**
```bash
cd ~/src/bluefin
git checkout -b feat/firefox-privacy

# Make changes
vim system_files/etc/firefox/...
git add system_files/
git commit -m "feat: add Firefox privacy defaults

Assisted-by: Claude 3.5 Sonnet via GitHub Copilot"
```

**3. Squash and push to fork:**
```bash
git rebase -i main  # Squash commits
git push origin feat/firefox-privacy
```

**4. Create PR with beads reference:**
```bash
gh pr create --web --repo ublue-os/bluefin
```

**In PR description:**
```markdown
## Summary
Add privacy-focused Firefox defaults including tracking protection

## Beads Tracking
Tracked in castrojo/common: `bluefin-common-abc`
```

**5. After PR merges, close beads issue:**
```bash
# From any redirected repo OR common
bd close bluefin-common-abc --reason="Merged in ublue-os/bluefin#1234"

# Sync beads to git (in common repo)
cd ~/src/bluefin-common
bd sync
git add .beads/
git commit -m "track: close bluefin privacy issue"
git push origin main
```

### Cross-Repository Epics

Epics can span multiple repositories:

```bash
bd create --title="Epic: Enhanced Privacy & Security Configuration" \
  --description="Comprehensive privacy and security hardening across Bluefin

Repositories involved:
- [bluefin] Firefox defaults, flatpak overrides, system policies
- [bluefin-lts] Port all privacy defaults to LTS
- [common] Shared privacy configuration layer
- [docs] Privacy features documentation

Related upstream issues:
- ublue-os/bluefin#456
- projectbluefin/common#789
- projectbluefin/documentation#123" \
  --type=epic --priority=1
```

**Create child tasks for each repository:**
```bash
# Epic child tasks
bd create --title="[bluefin] Add Firefox privacy defaults" --type=task --priority=1
bd create --title="[bluefin] Add flatpak privacy overrides" --type=task --priority=1
bd create --title="[bluefin-lts] Port privacy defaults to LTS" --type=task --priority=1
bd create --title="[common] Create shared privacy config layer" --type=task --priority=1
bd create --title="[docs] Document privacy features" --type=task --priority=2

# Add dependencies
bd dep add bluefin-common-child1 bluefin-common-epic  # Child depends on epic planning
bd dep add bluefin-common-lts bluefin-common-bluefin  # LTS depends on bluefin implementation
bd dep add bluefin-common-docs bluefin-common-bluefin  # Docs depend on implementation
```

### Area Labels and Monthly Reports

Monthly progress reports at `projectbluefin/documentation/reports/` categorize work by area labels. When creating beads issues, note the relevant area in the description:

**Area label mapping:**
- `area/gnome` - GNOME desktop environment
- `area/aurora` - Aurora variant (KDE)
- `area/bling` - Terminal/CLI enhancements and MOTD
- `area/dx` - Developer tools and IDE integrations
- `area/brew` - Homebrew packages and integration
- `area/just` - Just recipes and task automation
- `area/iso` - Installation and ISO building
- `area/policy` - System policies and security
- `area/bluespeed` - AI/ML integration
- `area/services` - System services and daemons
- `area/upstream` - Upstream contributions
- `area/flatpak` - Flatpak applications and overrides
- `area/nvidia` - NVIDIA GPU support

**Example with area notation:**
```bash
bd create --title="[bluefin] Add GNOME extension" \
  --description="Area: gnome

Adds SearchLight extension to default GNOME config" \
  --type=task --priority=2
```

### Fork Structure Requirements

**Each fork in castrojo namespace must have:**

1. **Upstream remote configured:**
   ```bash
   git remote -v
   # origin: git@github.com:castrojo/repo.git (your fork)
   # upstream: git@github.com:projectbluefin/repo.git (or ublue-os)
   ```

2. **Main branch tracks fork:**
   ```bash
   git branch -vv
   # * main  abc1234 [origin/main] Latest work
   ```

3. **Beads redirect (Tier 1 repos only):**
   - File: `.beads/redirect` contains relative path to central database
   - Automatically git-ignored (in `.git/info/exclude`)
   - Never committed to feature branches

4. **Feature branch workflow:**
   - Always branch from main
   - No beads metadata in feature branches
   - Squash before push to fork
   - Open PR in browser (never use `--title` or `--body` flags)
   - PRs to upstream are minimal and clean

### Beads Sync Strategy

**Daily workflow:**
```bash
# Work happens in any Tier 1 repo (with redirect)
cd ~/src/bluefin
bd create "..."
bd update "..."
bd close "..."

# Periodically sync beads to git (in common repo)
cd ~/src/bluefin-common
bd sync --from-main  # Pull any remote beads changes
# ... work with beads ...
bd sync              # Export to JSONL and prepare commit
git add .beads/
git commit -m "track: update beads issues"
git push origin main  # Push to castrojo/common fork
```

**All beads commits stay in `castrojo/common` fork main branch. Never pushed to upstream.**

### Why Multi-Repository Tracking?

**Benefits:**
1. **Single source of truth** - All Bluefin work tracked in one place
2. **Cross-repo visibility** - See dependencies between repositories
3. **Seamless workflow** - `bd` commands work in all Tier 1 repos
4. **Unified epics** - Epics span multiple repos naturally
5. **Clean PRs** - Beads never leaks to any upstream repository
6. **Progress reports** - Monthly reports aggregate from beads data
7. **Team collaboration** - Shared task tracking across all repos

**Isolation guarantees:**
1. `.beads/redirect` is automatically git-ignored
2. Feature branches in ALL repos are clean (no beads)
3. PRs to ANY upstream contain zero beads metadata
4. Only `castrojo/common` main branch has `.beads/` committed
5. Beads data syncs via fork, never to upstream

## Pull Request Workflow (Universal for ALL Repositories)

**CRITICAL:** This workflow applies to ALL PRs from `castrojo/*` forks to ANY upstream repository (projectbluefin/*, ublue-os/*, etc.)

**When creating a PR to ANY upstream repository**, follow this workflow:

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
# Open PR creation page in browser for the appropriate upstream
gh pr create --web --repo <upstream-org>/<upstream-repo>

# Examples:
gh pr create --web --repo projectbluefin/common
gh pr create --web --repo ublue-os/bluefin
gh pr create --web --repo ublue-os/bluefin-lts
gh pr create --web --repo projectbluefin/dakota
gh pr create --web --repo projectbluefin/documentation
```

**DO NOT** use `gh pr create` with `--title` or `--body` flags. The user must have full control to edit the PR description, title, and details in the GitHub web interface.

### PR Guidelines
- **Commit message format:** Follow [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/#specification)
- **Single commit:** PRs should contain exactly one squashed commit
- **Clear description:** User will add context in web interface
- **Reference issues:** Link related beads issues if applicable

**This workflow applies to ALL upstream repositories:**
- projectbluefin/* (common, dakota, iso, egg, brew, documentation, website, etc.)
- ublue-os/* (bluefin, bluefin-lts, etc.)

**Every PR from castrojo namespace MUST be squashed before push.**

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
