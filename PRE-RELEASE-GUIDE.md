# Pre-Release (RC) User Guide

A complete guide to using pre-release tags for testing and controlled deployments.

## Overview

Pre-releases (Release Candidates/RC) allow you to:
- Develop features and get them tested before full release
- Create stable snapshots for QA/UAT testing
- Iterate on feedback without affecting production releases
- Maintain parallel development while testing is ongoing

## Workflow Summary

```
Development → RC Release → Testing → Full Release
     ↓            ↓           ↓          ↓
Feature Branch → v2.3.0-rc.1 → QA/UAT → v2.3.0
```

## Prerequisites

- Repository configured with release-please workflows
- Understanding of [Conventional Commits](https://www.conventionalcommits.org/)
- Access to create branches and trigger workflows

## Step-by-Step Process

### 1. Develop Your Features

Work normally on feature branches with conventional commit messages:

```bash
# Create feature branch
git checkout -b feature/new-api-endpoint

# Make your changes with conventional commits
git commit -m "feat: add user profile API endpoint"
git commit -m "fix: handle edge case in validation"
git commit -m "docs: update API documentation"

# Push and create PR
git push origin feature/new-api-endpoint
```

**Important**

If squashing your commits, please ensure to to comment using the [conventional commits](https://www.conventionalcommits.org/en/v1.0.0/?ref=amarjanica.com). Otherwise, your release won't be created.

- ✅ `feat: add user authentication system`
- ✅ `fix: resolve payment processing bug`
- ❌ `Update login feature` (won't trigger releases) 

### 2. Create RC Release

**Manual Trigger**

```bash
# Via GitHub UI
# 1. Go to Actions → Pre-Release (RC Tag)
# 2. Click "Run workflow"
# 3. Select the feature-branch
# 4. Enter base_version: 2.3.0
# 5. Add optional notes for testers
```

### 3. RC Creation Results

The workflow automatically:
- ✅ Creates RC tag: `v2.3.0-rc.1`
- ✅ Creates GitHub pre-release with release notes
- ✅ Marks it clearly as "Pre-release" (not production)
- ✅ Includes all changes since last release

### 4. Distribute for Testing

**Provide to QA/Testing Team:**

```markdown
## RC Ready for Testing

**Version**: v2.3.0-rc.1
**Release**: https://github.com/yourorg/repo/releases/tag/v2.3.0-rc.1

### What's New:
- Added user profile API endpoint
- Fixed validation edge cases  
- Updated API documentation

### Deployment:
```bash
git checkout v2.3.0-rc.1
# Deploy to staging environment
```

**Testing Deadline**: [Date]
**Report Issues**: [Link to issue tracker]
```

### 5. Handle Testing Feedback

**If Issues Found:**
```bash
# Switch to RC branch
git checkout rc/v2.3.0

# Fix the issues
git commit -m "fix: resolve QA feedback on validation"
git commit -m "fix: handle null pointer in API response"

# Push changes - creates new RC automatically
git push origin rc/v2.3.0
```

This creates `v2.3.0-rc.2` with your fixes.

**Continue Iteration:**
- `v2.3.0-rc.1` → QA finds bugs
- `v2.3.0-rc.2` → QA finds edge case  
- `v2.3.0-rc.3` → QA approves ✅

### 5. Promote to Full Release

Once testing is complete and approved:

```bash
# Merge to main branch (triggers full release)
git checkout main
git pull origin main

# Option A: Merge RC branch
git merge rc/v2.3.0
git push origin main

# Option B: Cherry-pick specific commits
git cherry-pick <commit-hash>
git push origin main

# Option C: Create new PR from Github GUI
# (Recommended for audit trail)
```

This triggers the normal release workflow → creates `v2.3.0` (final release).

## Complete Example Timeline

```bash
# Monday: Development
git checkout -b feature/payment-system
git commit -m "feat: add payment processing"
git commit -m "feat: add receipt generation"
git push origin feature/payment-system
# → Dry run shows: "Next Tag: v2.4.0"

# Tuesday: Create RC for testing
git checkout -b rc/v2.4.0
git push origin rc/v2.4.0
# → Creates v2.4.0-rc.1

# Wednesday-Thursday: QA testing v2.4.0-rc.1
# QA reports: "Payment fails for international cards"

# Friday: Fix and new RC
git checkout rc/v2.4.0
git commit -m "fix: support international payment cards"
git push
# → Creates v2.4.0-rc.2

# Monday: QA approves v2.4.0-rc.2

# Tuesday: Release to production
git checkout main
git merge rc/v2.4.0
git push
# → Creates v2.4.0 (final release)
```

## Best Practices

### RC Branch Naming
- ✅ `rc/v2.3.0` - Clear version indication
- ✅ `rc/payment-feature` - Feature-based naming
- ❌ `testing` - Too generic
- ❌ `rc-branch` - No version info

### Version Prediction
- Always check dry-run output before creating RC
- Account for other PRs that might merge before yours
- Consider semantic versioning impact of breaking changes

### Communication
- Send clear RC announcements to testing team
- Include what's new, how to deploy, and testing deadline
- Provide direct links to GitHub releases
- Set expectations for feedback timeline

### RC Iteration
- Keep RC branches focused on the planned release
- Avoid adding new features during RC phase
- Only fix bugs found during testing
- Document what changed between RC versions

### Environment Management
- **Staging/Test**: Deploy RC tags (`v*-rc.*`)
- **Production**: Deploy only final tags (`v*`)
- **Local Dev**: Use branch names or latest commits

## Troubleshooting

### RC Workflow Not Triggering
- Verify branch name follows `rc/*` pattern
- Check workflow permissions (contents: write)
- Ensure base version follows semantic versioning

### Wrong Version Generated
- Check conventional commit format in recent commits
- Verify no existing release PR conflicts
- Review release-please configuration

### RC Not Creating GitHub Release
- Check repository permissions
- Verify workflow completed successfully
- Look for rate limiting or API errors in workflow logs
- If the create-release flag is set to false, it won't be released.

### Testing Team Can't Access RC
- Ensure RC created as GitHub pre-release
- Check repository visibility settings
- Verify release assets are uploaded correctly

## Advanced Usage

### Custom RC Notes
```bash
gh workflow run pre-release-rc.yml \
  -f base_version=2.3.0 \
  -f notes="
**Focus Areas for Testing:**
- Payment processing with various card types
- API rate limiting under high load
- Mobile app compatibility

**Known Issues:**
- Debug logging still enabled (will be removed in final)
"
```

### Deployment Automation
Configure your CI/CD to automatically deploy RC tags to staging:

```yaml
# .github/workflows/deploy-staging.yml
on:
  release:
    types: [prereleased]  # Triggers on RC releases

jobs:
  deploy:
    if: contains(github.event.release.tag_name, '-rc.')
    # Deploy to staging environment
```

### Multi-Team Coordination
For larger teams, consider:
- RC creation approval process
- Scheduled RC creation windows
- Automated testing integration
- Release notes review process

## Integration with Existing Workflow

This pre-release process integrates seamlessly with your existing release-please setup:
- Regular development continues normally
- Release PRs still work as expected
- Final releases follow standard process
- No changes needed to existing automation

The RC workflow is additive - it doesn't replace your normal release process, it enhances it with controlled testing phases.