# Reusable Release Actions

This repository provides battle-tested GitHub Actions for semantic versioning and release management using release-please.

## 🎯 Three Independent, Powerful Actions

**Complete Release Pipeline:**
1. **`release-dry-run`** - Zero-config version prediction and validation
2. **`pre-release-rc`** - Release candidate management for testing
3. **`release`** - Enterprise-grade production release pipeline

All actions are designed for **maximum reusability** with **minimal configuration** required!

## 🔄 Complete Independence - Mix and Match

Each action works **completely independently** - use one, two, or all three based on your needs:

### **Use Case 1: Only PR Validation**
Perfect for teams that just want version prediction on pull requests:
```yaml
# .github/workflows/pr-check.yml
name: PR Validation
on:
  pull_request:
    branches: [main]
jobs:
  validate:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: your-org/release-please/.github/actions/release-dry-run@v1.0.0
```

### **Use Case 2: Only RC Management**
Perfect for teams that manually create releases but want RC automation:
```yaml
# .github/workflows/rc.yml
name: Release Candidates
on:
  push:
    branches: [rc/**]
jobs:
  rc:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: your-org/release-please/.github/actions/pre-release-rc@v1.0.0
        with:
          notes: "Testing new features"
```

### **Use Case 3: Only Production Releases**
Perfect for teams that want automated production releases:
```yaml
# .github/workflows/release.yml
name: Production Release
on:
  push:
    branches: [main]
jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: your-org/release-please/.github/actions/release@v1.0.0
```

### **Use Case 4: Complete Pipeline (All Actions in One Workflow)**
The most comprehensive approach - all three actions working together:
```yaml
# .github/workflows/complete-release.yml
name: Complete Release Pipeline
on:
  pull_request:
    branches: [main]
  push:
    branches: [main, 'rc/**']

jobs:
  # Job 1: Always validate PRs
  validate-pr:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Predict and validate version
        uses: your-org/release-please/.github/actions/release-dry-run@v1.0.0

  # Job 2: Create RCs on rc/* branches
  release-candidate:
    if: github.event_name == 'push' && startsWith(github.ref, 'refs/heads/rc/')
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Create release candidate
        uses: your-org/release-please/.github/actions/pre-release-rc@v1.0.0
        with:
          notes: "Release candidate for testing"

  # Job 3: Production release on main
  production-release:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Create production release
        id: release
        uses: your-org/release-please/.github/actions/release@v1.0.0
      - name: Deploy to production
        if: steps.release.outputs.release_created == 'true'
        run: echo "🚀 Deploying ${{ steps.release.outputs.version }}"
```

### **Use Case 5: Sequential Pipeline (One Job, Multiple Actions)**
All actions in sequence within a single job:
```yaml
# .github/workflows/sequential-release.yml
name: Sequential Release Pipeline
on:
  push:
    branches: [main]

jobs:
  complete-pipeline:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      # Step 1: Validate what would be released
      - name: Validate release
        id: validate
        uses: your-org/release-please/.github/actions/release-dry-run@v1.0.0
      
      # Step 2: Create RC first (optional)
      - name: Create RC
        if: steps.validate.outputs.bump_type != 'none'
        uses: your-org/release-please/.github/actions/pre-release-rc@v1.0.0
        with:
          notes: "Pre-release validation"
      
      # Step 3: Create production release
      - name: Production release
        id: release
        uses: your-org/release-please/.github/actions/release@v1.0.0
      
      - name: Summary
        run: |
          echo "Predicted: ${{ steps.validate.outputs.predicted_version }}"
          echo "Released: ${{ steps.release.outputs.version }}"
```

## 📊 Choose What You Need

| Team Type | Actions Needed | Workflow Strategy |
|-----------|---------------|-------------------|
| **Simple Projects** | `release` only | Single workflow, main branch only |
| **Testing-Heavy** | `release-dry-run` + `pre-release-rc` | Combined workflow or separate |
| **Enterprise** | All three | **Single comprehensive workflow** |
| **Manual Release** | `release-dry-run` only | PR-only workflow |
| **RC-Only Teams** | `pre-release-rc` only | RC branch workflow |

## 🔄 Workflow Strategies

### **Strategy 1: Single Comprehensive Workflow** ⭐ *Recommended*
- All three actions in one workflow file
- Different jobs triggered by different events
- Cleanest approach for teams using all actions

### **Strategy 2: Separate Workflows**
- Each action in its own workflow file
- Good for teams adopting actions incrementally
- Easier to manage permissions per action

### **Strategy 3: Sequential Pipeline**
- All actions in sequence within one job
- Useful for validation → RC → release flow
- Good for testing and staged releases

## Actions Available

### 🚀 Pre-Release RC Action (Independent)
Creates or increments release candidate tags (vX.Y.Z-rc.N) with GitHub prereleases.

**Standalone minimal usage:**
```yaml
- uses: your-org/release-please/.github/actions/pre-release-rc@v1.0.0
  with:
    notes: "Testing new features"
```

**All parameters:**
```yaml
- uses: your-org/release-please/.github/actions/pre-release-rc@v1.0.0
  with:
    target-branch: main          # optional, defaults to repo default
    base-version: "1.2.0"        # optional, auto-detected if not provided
    release-type: simple         # optional, defaults to "simple"
    package-name: my-package     # optional, defaults to "workflows"
    path: "."                    # optional, defaults to "."
    notes: "Custom notes"        # optional
    create-release: "true"       # optional, defaults to "true"
```

### 📦 Release Action (Independent)
Complete release management with state reconciliation, floating tags, and artifact uploads.

**Standalone minimal usage:**
```yaml
- uses: your-org/release-please/.github/actions/release@v1.0.0
```

**Advanced usage:**
```yaml
- uses: your-org/release-please/.github/actions/release@v1.0.0
  with:
    release-type: simple               # optional, defaults to "simple"
    artifact-path: "dist/app.zip"      # optional, upload build artifacts
    artifact-name: "my-app.zip"        # optional, custom artifact name
    create-floating-tags: "true"       # optional, create vX, vX.Y tags
    reconcile-state: "true"            # optional, fix orphaned tags/releases
    package-name: my-package           # optional, for monorepos
    path: "."                          # optional, package path
```

### 📊 Release Dry Run Action (Independent)
Predicts the next semantic version without creating releases or tags. **ZERO configuration required** - works perfectly out of the box!

**Standalone minimal usage (literally nothing required):**
```yaml
- uses: your-org/release-please/.github/actions/release-dry-run@v1.0.0
```

**Advanced usage with all parameters:**
```yaml
- uses: your-org/release-please/.github/actions/release-dry-run@v1.0.0
  with:
    target-branch: main                    # optional, defaults to "main"
    release-type: simple                   # optional, defaults to "simple"
    package-name: my-package               # optional, defaults to "workflows"
    path: "."                              # optional, defaults to "."
    pr-comment: "true"                     # optional, post prediction in PR comments
    enforce-major-approval: "true"         # optional, require approval for major bumps
    artifact-name: "my-dry-run-log"       # optional, custom artifact name
    unique-suffix: "true"                  # optional, prevent artifact conflicts
    pr-comment-marker: "custom-marker"    # optional, custom PR comment marker
    show-raw-exit-code: "false"           # optional, show raw exit codes in PR
```

**Rich outputs available:**
- `predicted_tag`: Next release tag (vX.Y.Z)
- `predicted_version`: Next semantic version (X.Y.Z)
- `major`, `minor`, `patch`: Individual version components
- `current_tag`: Current latest tag
- `next_tag`: Tag after PR merge
- `bump_type`: Type of bump (none|patch|minor|major)
- `major_bump_detected`: True if major bump with protection applied
- `final_exit_code`: Raw release-please exit code

## Complete Workflow Examples

### Feature Branch Validation (Zero Config!)
```yaml
name: Validate Changes
on:
  pull_request:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: your-org/release-please/.github/actions/release-dry-run@v1.0.0
```

### Release Candidate Creation (Independent)
```yaml
name: Release Candidate
on:
  push:
    branches: [rc/**]

jobs:
  rc:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - uses: your-org/release-please/.github/actions/pre-release-rc@v1.0.0
        with:
          notes: "Release candidate for testing"
```

### Production Release (Independent)
```yaml
name: Release
on:
  push:
    branches: [main]

jobs:
  release:
    runs-on: ubuntu-latest
    permissions:
      contents: write
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      # Simple release with all enterprise features
      - name: Release
        id: release
        uses: your-org/release-please/.github/actions/release@v1.0.0
        with:
          release-type: simple
      
      # Use release outputs for downstream jobs
      - name: Deploy to production
        if: steps.release.outputs.release_created == 'true'
        run: |
          echo "Deploying version ${{ steps.release.outputs.version }}"
          echo "Release tag: ${{ steps.release.outputs.tag_name }}"
```

### Conditional Logic Based on Version Bump (Dry Run Only)
```yaml
name: Smart Deployment
on:
  pull_request:
    branches: [main]

jobs:
  check-and-deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      - name: Predict version
        id: version
        uses: your-org/release-please/.github/actions/release-dry-run@v1.0.0
      
      - name: Skip deployment for patch releases
        if: steps.version.outputs.bump_type == 'patch'
        run: echo "⏭️ Skipping deployment for patch release"
      
      - name: Notify team of major release
        if: steps.version.outputs.bump_type == 'major'
        run: |
          echo "🚨 Major release detected: ${{ steps.version.outputs.predicted_version }}"
          echo "This will be a breaking change!"
      
      - name: Deploy preview for minor/major releases
        if: contains(fromJSON('["minor", "major"]'), steps.version.outputs.bump_type)
        run: echo "🚀 Deploying preview for ${{ steps.version.outputs.predicted_version }}"
```

## Key Features

### 🎯 Zero Configuration Required
The `release-dry-run` action works perfectly with **no inputs** - just add it to your workflow and it works!

### 🛡️ Built-in Safety Features
- **Major Version Protection**: Automatically labels and blocks major version bumps until approved
- **Smart PR Comments**: Shows predicted version changes directly in pull requests
- **Artifact Logging**: Preserves debug logs for troubleshooting

### 🔧 Universal Compatibility
- Works with any repository using [Conventional Commits](https://conventionalcommits.org/)
- Supports all release-please release types (simple, node, python, manifest, etc.)
- Handles monorepos and single-package repos equally well

### 📈 Rich Output Data
Both actions provide comprehensive outputs for building complex workflows:
- Version predictions and bump types
- Current and next version information
- Safety flags for major releases
- Debug information and exit codes

## Prerequisites

- Repository using [Conventional Commits](https://conventionalcommits.org/)
- Node.js 20+ available in runner
- Appropriate permissions:
  - `contents: read` for dry-run
  - `contents: write` for RC creation
  - `pull-requests: write` for PR comments

## Outputs

Both actions provide rich outputs for further workflow steps:

**Pre-Release RC outputs:**
- `base_version`: The semantic version (without RC suffix)
- `rc_number`: The RC increment number
- `rc_tag`: Full RC tag (vX.Y.Z-rc.N)
- `predicted_version`: Predicted next release version

**Release Action outputs:**
- `release_created`: Whether a release was created (true/false)
- `tag_name`: Name of the created tag (vX.Y.Z)
- `version`: Complete version string (X.Y.Z)
- `major`, `minor`, `patch`: Individual version components
- `sha`: SHA of the release commit
- `upload_url`: Upload URL for release assets

**Dry Run outputs:**
- `predicted_tag`: Predicted next tag (vX.Y.Z) 
- `predicted_version`: Predicted version (X.Y.Z)
- `major`, `minor`, `patch`: Individual version components
- `current_tag`: Current latest tag (vX.Y.Z)
- `next_tag`: Tag after PR merge (vX.Y.Z)
- `bump_type`: Semantic bump type (none|patch|minor|major)
- `major_bump_detected`: True if major bump with protection applied
- `final_exit_code`: Raw release-please exit code

## Why These Actions Are Exceptionally Reusable

### ✅ Perfect Defaults
Every input has sensible defaults that work for 95% of repositories:
- `target-branch: "main"` - Standard default branch
- `release-type: "simple"` - Universal release-please type  
- `package-name: "workflows"` - Generic fallback name
- `path: "."` - Root directory (most common case)

### ✅ Zero Learning Curve
- Copy one line, get full functionality
- No configuration files required
- No setup or initialization needed
- Works immediately with Conventional Commits

### ✅ Complete Independence AND Perfect Integration
- **Independent**: Each action works standalone without dependencies
- **Combinable**: Actions work beautifully together in single workflows
- **Flexible Triggers**: Use conditions to run different actions based on events
- **Sequential or Parallel**: Run actions in sequence or separate jobs
- **Event-Driven**: Different actions for different GitHub events (PR, push, etc.)

### ✅ Best of Both Worlds
- **Adopt Incrementally**: Start with one action, add others later
- **Combine When Ready**: Move to single comprehensive workflow
- **No Vendor Lock-in**: Use what you need, when you need it
- **Easy Migration**: Move between workflow strategies without changing actions

### ✅ Perfect for Different Team Workflows
- **Simple Teams**: Use `release` action only for automated production releases
- **Testing-Heavy Teams**: Use `release-dry-run` + `pre-release-rc` for validation and testing
- **Enterprise Teams**: Use all three for complete release pipeline management
- **Manual Release Teams**: Use `release-dry-run` only for PR validation
- **RC-Only Teams**: Use `pre-release-rc` only for candidate management

### ✅ DRY Principle in Action
Instead of copying 150+ lines of complex bash scripts:
```yaml
# Before: Complex workflow with embedded logic
- name: Reconcile release state
  run: |
    # 50+ lines of bash...
- name: Release Please  
  # More steps...
- name: Create floating tags
  run: |
    # 30+ lines of bash...

# After: Simple action call
- uses: your-org/release-please/.github/actions/release@v1.0.0
```

This demonstrates **perfect separation of concerns** - complex logic lives in reusable actions, workflows orchestrate simple steps.