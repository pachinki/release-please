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

**Important**: Ensure your PR title follows conventional commits format:
- ✅ `feat: add user authentication system`
- ✅ `fix: resolve payment processing bug`
- ❌ `Update login feature` (won't trigger releases)

### 2. Predict the Next Version

Before creating an RC, determine what version will be released:

**Option A: Use Dry Run Workflow**
1. Create PR for your feature branch
2. Check the dry-run comment for: "Next Tag After Merge: v2.3.0"
3. Note this version for your RC

**Option B: Manual Analysis**
Based on your commits since the last release:
- `feat:` or `feat!:` → Minor or Major bump
- `fix:`, `perf:` → Patch bump
- `docs:`, `style:`, `chore:` → Usually no bump

### 3. Create RC Release

**Method A: Automatic (via RC Branch)**
```bash
# Create RC branch - triggers workflow automatically
git checkout -b rc/v2.3.0  # Use predicted version
git push origin rc/v2.3.0
```

**Method B: Manual Trigger**
```bash
# Via GitHub UI
# 1. Go to Actions → Pre-Release (RC Tag)
# 2. Click "Run workflow"
# 3. Enter base_version: 2.3.0
# 4. Add optional notes for testers

# Via GitHub CLI (if authenticated)
gh workflow run pre-release-rc.yml \
  -f base_version=2.3.0 \
  -f notes="Ready for QA testing - new API endpoints"
```

### 4. RC Creation Results

The workflow automatically:
- ✅ Creates RC tag: `v2.3.0-rc.1`
- ✅ Creates GitHub pre-release with release notes
- ✅ Marks it clearly as "Pre-release" (not production)
- ✅ Includes all changes since last release

### 5. Distribute for Testing

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

### 6. Handle Testing Feedback

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

### 7. Promote to Full Release

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

# Option C: Create new PR from RC branch
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

### Testing Team Can't Access RC
- Ensure RC created as GitHub pre-release
- Check repository visibility settings
- Verify release assets are uploaded correctly

## FAQ - Common Scenarios

### Q: What happens if someone else releases while I'm testing an RC?

**Scenario:** You created `v4.1.0-rc.1` and `v4.1.0-rc.2` on your feature branch, but someone else merged their changes and released `v4.1.0` before your PR was ready. Now the dry-run says your PR will create `v4.2.0`.

**Solution (Recommended - Rebase Approach):**

1. **Rebase your feature branch onto the new release:**
   ```bash
   git checkout your-feature-branch
   git rebase v4.1.0
   ```

2. **Continue with RC workflow for the new version:**
   ```bash
   gh workflow run pre-release-rc.yml \
     --ref your-feature-branch \
     --field base-version=4.2.0
   ```
   This creates `v4.2.0-rc.1` with your changes properly based on `v4.1.0`.

3. **Clean up orphaned RC tags (optional):**
   ```bash
   git tag -d v4.1.0-rc.1 v4.1.0-rc.2
   git push origin :refs/tags/v4.1.0-rc.1
   git push origin :refs/tags/v4.1.0-rc.2
   ```

4. **Complete your testing and merge** - This will create `v4.2.0`.

**Why this happens:** This is normal in concurrent development. Multiple teams can work on features simultaneously, and whoever merges first gets that version number.

**Alternative (Merge Approach):**
```bash
git checkout your-feature-branch
git merge v4.1.0
# Then continue with v4.2.0 RC workflow
```

### Q: Should RC tags be deleted after the final release?

**Answer:** **Keep them!** RC tags serve as valuable historical records:
- 🔍 **Audit trail** - Complete history of what was tested
- 🐛 **Debugging** - Can check out exact RC version if issues arise  
- 📋 **Compliance** - Enterprise environments require traceability
- 🔄 **Rollback** - Reference specific RC if emergency rollback needed

**Example healthy tag structure:**
```
v4.0.0
v4.1.0-rc.1  ← Keep these
v4.1.0-rc.2  ← Keep these  
v4.1.0       ← Final release
v4.2.0-rc.1  ← Keep these
v4.2.0       ← Final release
```

### Q: Can I create RCs for patch releases?

**Answer:** Yes! Use the same workflow:
```bash
gh workflow run pre-release-rc.yml \
  --ref your-hotfix-branch \
  --field base-version=4.1.1
```

This is especially useful for critical hotfixes that need testing before production deployment.

### Q: What if my RC workflow fails?

**Common solutions:**
1. **Check version conflicts** - Ensure no newer version already exists
2. **Verify branch permissions** - Make sure you can create tags on the branch
3. **Review base version** - Ensure it's higher than the latest existing tag
4. **Check for typos** - Verify version format (e.g., `4.1.0`, not `v4.1.0`)

**Debug workflow:**
```bash
# Check latest tag
git describe --tags --abbrev=0

# Check what the dry-run suggests
# Run dry-run workflow on your branch first
```

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