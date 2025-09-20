## Release Process

Using [release-please](https://github.com/google-github-actions/release-please-action) to cut releases automatically when changes land on `main` with [Conventional Commit](https://www.conventionalcommits.org/en/v1.0.0/?ref=amarjanica.com) messages.

### TL;DR

1. Write commits using the Conventional Commit prefix (e.g. `feat:`, `fix:`).
2. Merge to `main`.
3. The workflow calculates the new version, updates the changelog & creates tags (`vX.Y.Z`, plus floating `vX` & `vX.Y`).

### How version bumps are chosen

| Commit style (examples) | Effect on version | Notes |
|-------------------------|-------------------|-------|
| `fix: handle null input` | Patch bump (X.Y.Z → X.Y.(Z+1)) | Bug fixes only |
| `feat: add user export`  | Minor bump (X.Y.Z → X.(Y+1).0) | New backwards‑compatible feature |
| <br> `feat!: change auth flow` <br><br> `refactor!: drop legacy API` <br><br> `BREAKING CHANGE: removed field` | Major bump ((X+1).0.0) | Use `!` after type or add a `BREAKING CHANGE:` footer |
| <br> `docs: documentation changes` <br><br> `chore: miscellaneous tasks like build processes or auxiliary tools` <br><br> `refactor: code refactoring without adding features or fixing bugs` <br><br> `style: code style changes, linter fixes (formatting, missing semicolons, etc.)` <br><br> `ci: changes to continuous integration scripts` | No release by itself | Unless combined with a feat/fix or you include a manual release trigger |

If no commits with a bump-triggering type are merged, no new release is created.

### Minimal commit examples

```text
feat: add export endpoint
fix: correct off-by-one in pagination
feat!: remove deprecated /v1/login route

BREAKING CHANGE: Clients must use /v2/login
```

### Tips

* Want to force a release for a maintenance change? Convert the commit to `fix:` (if it is a bug) or add a small `feat:` where appropriate.
* Group related small fixes into a single `fix:` commit when possible to avoid noisy patch versions.
* Use clear, user‑facing language in the first line—this becomes part of the changelog.

If you don’t follow the Conventional Commit format, the automation will skip releasing.

## Do NOT Consume / reference `@main`

Teams should **never reference this repository’s `main` branch directly** in reusable workflow or action calls, e.g.

```yaml
# ❌ Avoid
uses: your-org/repo/.github/workflows/ecs-deploy.yml@main
```

#### Why `@main` is unsafe

* Moving target – contents can change at any time; a re-run tomorrow may produce different behavior.
* No stability contract – we reserve the right to refactor, rename inputs, or adjust defaults on `main` before a release is cut.
* Hard to reproduce incidents – you cannot easily recreate an historical CI run if the referenced commit has moved on.
* Security / supply‑chain risk – audits and SBOMs require an immutable reference (tag or commit SHA).
* Hidden breakages – consumers silently inherit changes they did not test.

#### What to use instead

Pick the tag that matches the stability you need:

| Need | Use | Example |
|------|-----|---------|
| Always latest compatible (features + fixes within same major) | `vX` | `@v1` |
| Only latest patches for a chosen minor | `vX.Y` | `@v1.2` |
| Fully locked / reproducible | `vX.Y.Z` | `@v1.2.3` |

```yaml
# ✅ Recommended (tracks non-breaking updates)
uses: your-org/test-workflow-versioning/.github/workflows/deploy.yml@v1

# ✅ Locked to a minor (only patch bumps)
uses: your-org/test-workflow-versioning/.github/workflows/deploy.yml@v1.0

# ✅ Fully pinned (no movement)
uses: your-org/test-workflow-versioning/.github/workflows/deploy.yml@v1.0.3
```

#### Migrating existing consumers

1. Search your repos for `test-workflow-versioning@main`.
2. Replace with the appropriate tag (usually `@v1`).
3. Commit with: `chore: stop consuming workflow via @main`.
4. (Optional) For critical pipelines, pin initially to a full version, then relax to `v1` after validation.

> We may introduce a guard failing calls that still use `@main` after a grace period.

## Version Tags Explained

Each release creates one immutable semantic version tag and two *floating* convenience tags. Example after the first release:

```text
v1.0.0   # exact, never moves
v1.0     # floating "major.minor" tag – always the latest patch in the 1.0 line
v1       # floating "major" tag – always the latest 1.x.y release
```

### Why three tags?

| Tag        | Moves? | Purpose | Typical Consumer |
|------------|--------|---------|------------------|
| `v1.0.0`   | No     | Reproducible builds, auditing, SBOMs | CI pipelines needing locked version |
| `v1.0`     | Yes (on patch releases) | Always pick up the newest bug fix for that minor | Internal automation, users wanting only safe patch upgrades |
| `v1`       | Yes (on any new minor/patch in major 1) | Track all non‑breaking feature & patch updates | Reusable workflow / action references |

### How the workflow creates / updates them

1. `release-please` determines the next semantic version (initially `1.0.0`) and creates tag + GitHub Release.
2. A follow‑up step force (re)tags the floating refs:
   * Deletes any remote `v<major>` and `v<major>.<minor>` tags if they exist.
   * Recreates them pointing at the new release commit.
3. Result: `v1.0.0` is permanent; `v1` and `v1.0` advance over time.

When version `1.1.0` is released:

```text
v1.1.0  (new immutable tag)
v1.1    (now points to 1.1.0)
v1      (moves from latest 1.0.x to 1.1.0)
v1.0    (continues to point at latest 1.0.x patch until no more 1.0 patches)
```

### Consuming the tags

#### In another GitHub workflow (reusable action / composite / workflow)

```yaml
uses: your-org/test-workflow-versioning@v1       # auto-updates to new minor & patch releases (no breaking changes)
# uses: your-org/test-workflow-versioning@v1.0   # only patch updates within 1.0.x
# uses: your-org/test-workflow-versioning@v1.0.3 # exact version (recommended for production reproducibility)
```

#### Cloning source locally

```bash
git clone https://github.com/your-org/test-workflow-versioning.git --branch v1 --single-branch
# or for exact bits
git clone https://github.com/your-org/test-workflow-versioning.git --branch v1.0.0 --single-branch
```

#### Updating local floating tags

Because `v1` / `v1.0` move, fetch with `--force` to update:

```bash
git fetch --tags --force
```

#### Referencing Docker images (if you later publish them)



#### Example Commit messages

Push images under the same scheme for user expectations:


```text
repo/image:1.0.0
repo/image:1.0    # floating minor
repo/image:1       # floating major
```


feat: represents a new feature, and correlates to a SemVer minor.

### Which tag should I use?

| Goal | Recommended Tag |
|------|-----------------|
| Absolute reproducibility / audit trail | `vX.Y.Z` |
| Automatic bug‑fixes only for chosen minor | `vX.Y` |
| Automatic new (non‑breaking) features & patches | `vX` |

> Security note: For critical production pipelines pin to a full tag (or commit SHA). Floating tags are force‑pushed by design.

### Forcing a major or minor bump

Because the workflow uses Conventional Commits, bump types depend on commit messages merged to `main`:

* `feat:` triggers a **minor** bump (e.g., 1.0.0 → 1.1.0) unless you configure a different release-type.
* `fix:` triggers a **patch** bump (1.0.0 → 1.0.1).
* Add `!` (e.g., `feat!:`) or include a `BREAKING CHANGE:` footer to trigger a **major** bump (1.x.y → 2.0.0). After that, floating tags `v2` and `v2.0` start being created/updated; the old `v1` / `v1.*` remain where they last pointed.

### Verifying what a floating tag points to

```bash
git fetch --tags --force
git rev-parse v1
git rev-parse v1.0
git describe --tags --abbrev=0 v1   # shows the full semver currently referenced
```

---

In short: use the immutable tag for locked builds, and the floating tags for convenient upgrade channels aligned with Semantic Versioning guarantees.

## Release Dry Run Workflow (Version Prediction)

The `Release (Dry Run)` workflow predicts the *next* semantic version **without** opening a release PR or creating tags. It’s an early warning so reviewers can confirm whether a PR will cause a release (and what type) before merging to `main`.

### Why it helps

* Avoid surprise major/minor bumps.
* Verify docs / chore changes won’t trigger a release.
* Surface GHES API / permission issues independently of real releases.

### Operation (high‑level)

1. Checkout with full history (needs prior tags).
2. Run `release-please` CLI with `--dry-run`, `--release-type simple`.
3. Parse log lines for a proposed version (multiple regex patterns).
4. If parsing fails (CLI error, wording change, API mismatch) a fallback heuristic inspects commit messages since the last tag:
   * `BREAKING CHANGE` / `!:` → major++ (reset minor & patch)
   * `feat:` → minor++ (reset patch)
   * `fix:` / `perf:` (only if no feat) → patch++
   * Otherwise → no bump (outputs empty)
5. Expose outputs: `predicted_version`.

### Outputs Cheat Sheet

| Situation | Outputs | Meaning |
|-----------|---------|---------|
| All empty | (none) | No qualifying commits → no release.
| `predicted_version = 1.2.0` | minor bump | At least one `feat:` (no breaking marker).
| `predicted_version = 1.1.5` | patch bump | Only fixes/perf since last tag.
| `predicted_version = 2.0.0` | major bump | Breaking marker (`!:` or footer) detected.
| Non‑zero `final_exit_code` + version present | tolerated error | CLI failed but parsing/fallback succeeded.


### Fallback Limitations

The heuristic is intentionally minimal (e.g. multiple `feat:` still equals one minor bump; no changelog text). If it triggers often, fix the underlying CLI/API issue rather than relying on it.

### Possible Enhancements (Future)

* Add `used_fallback=true` output for visibility.
* Upload JSON artifact (raw log + parsed metadata) once GHES artifact limitations are resolved.
* ✅ ~~Enforce label / approval if a major bump is predicted.~~ **IMPLEMENTED**

### Major Bump Protection

The release dry-run workflow now includes automatic protection for major version bumps (breaking changes):

**When a major bump is detected, the workflow automatically:**
1. 🏷️ **Adds labels**: `major-release` and `requires-approval` 
2. 📝 **Posts warning comment** with breaking change checklist
3. 👥 **Requests reviews** from CODEOWNERS (if configured)
4. ⚠️ **Shows visual warnings** in the standard dry-run comment

**Configuration:**
```yaml
- uses: ./.github/actions/release-dry-run
  with:
    enforce-major-approval: true  # default: true
```

**To set up automatic review requests:**
1. Create `.github/CODEOWNERS` file with senior maintainers
2. Configure branch protection rules requiring CODEOWNERS approval
3. Major bumps will automatically request reviews from specified users

**Example CODEOWNERS:**
```
# Global owners
* @senior-maintainer @release-manager

# Workflows require DevOps approval  
/.github/ @devops-team
```

This ensures breaking changes receive proper review and communication before release.

---
Use the dry run summary during review to answer: *Will this PR release, and to what version?*
