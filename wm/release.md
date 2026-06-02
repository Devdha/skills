---
name: wm-release
description: Use when releasing a new version of wm, publishing a release, or user says "릴리즈", "release", "배포"
---

# WM Release

Automate the wm release process.

## Steps

When the user says "release" or provides a version number:

### 1. Determine Version

- If version provided (e.g., "0.2.3"), use it
- If not, check latest tag and suggest next version:
  ```bash
  git tag --list --sort=-v:refname | head -1
  ```

### 2. Verify Pre-conditions

```bash
# Ensure on main branch with clean state
git status --porcelain
git branch --show-current

# Ensure all tests pass
go build ./... && go test ./...
```

If dirty or not on main, warn the user before proceeding.

### 3. Create Git Tag & GitHub Release

```bash
# Get commits since last release for changelog
PREV_TAG=$(git tag --list --sort=-v:refname | head -1)
git log ${PREV_TAG}..HEAD --oneline

# Tag and push
git tag v${VERSION}
git push origin v${VERSION}

# Create GitHub release with changelog
gh release create v${VERSION} --title "v${VERSION}" --notes "changelog here"
```

### 4. Wait for GoReleaser CI

```bash
# Find the release workflow run triggered by the tag push
gh run list --workflow=release.yml --limit 1

# Wait for it to complete
gh run watch <run-id> --exit-status
```

### 5. Provide npm Publish Command

After CI completes successfully, output this for the user:

```
Release v${VERSION} complete! Binaries uploaded.

To publish to npm (requires OTP):
  cd /Users/donghun/personal/wm && ./scripts/publish-npm-local.sh ${VERSION}
```

**DO NOT attempt to run the npm publish script.** It requires interactive OTP input that cannot be handled in automated environments.
