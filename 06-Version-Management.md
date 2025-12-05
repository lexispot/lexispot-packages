# 06 - Version Management Guide

## Managing Releases and Versions

This guide covers how to manage different versions of your language packages, when to create new versions, and best practices.

---

## Version Strategy

### Current Setup

- **Repository**: lexispot/lexispot-packages
- **Current Version**: v1.0.0
- **Languages**: 34 packages
- **Release Date**: December 2025

---

## When to Create New Versions

### Minor Version (v1.X.0)

Create a new minor version when:
- ✅ Adding new language packages
- ✅ Updating multiple existing packages
- ✅ Significant content improvements
- ✅ Want to maintain version history

**Example**: v1.0.0 → v1.1.0 (added 10 languages)

### Patch Version (v1.0.X)

Create a patch version when:
- ✅ Fixing bugs in specific packages
- ✅ Small corrections
- ✅ Metadata updates

**Example**: v1.1.0 → v1.1.1 (fixed typo in French package)

### Major Version (vX.0.0)

Create a major version when:
- ✅ Breaking changes to format
- ✅ Complete rebuild of packages
- ✅ Changing compression format
- ✅ Incompatible with older app versions

**Example**: v1.9.0 → v2.0.0 (new SQLite schema)

---

## Creating a New Version

### Method 1: Using add_languages.sh

```bash
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
./add_languages.sh

# Choose: 1 (Create new version)
# Enter: v1.1.0
```

### Method 2: Manual Creation

```bash
# 1. Update manifest generator
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
# Edit generate_manifest.py, change VERSION = "v1.1.0"

# 2. Generate new manifest
python3 generate_manifest.py

# 3. Create release
gh release create v1.1.0 \
  /Users/valerio/Documents/commands/dict-builder/packages/*.zst \
  /Users/valerio/Documents/commands/dict-builder/packages/manifest.json \
  --repo lexispot/lexispot-packages \
  --title "LexiSpot v1.1.0" \
  --notes "## What's New
- Added 5 new languages
- Updated French translations
- Performance improvements

## Languages
Now includes 39 language packages!"
```

---

## Updating iOS App for New Version

### Update ManifestConfiguration

```swift
// File: LexiSpot/Models/LanguageManifest.swift

struct ManifestConfiguration {
    static let manifestURL = URL(string: 
        "https://github.com/lexispot/lexispot-packages/releases/download/v1.1.0/manifest.json"
    )!
    //                                                                      ^^^^^^^ Update version here
    
    static let fallbackManifestURL = URL(string: 
        "https://github.com/lexispot/lexispot-packages/releases/download/v1.1.0/manifest.json"
    )!
}
```

### Rebuild and Deploy

```bash
# 1. Update version in Xcode
# Project Settings → General → Version: 1.1.0

# 2. Build
xcodebuild -project LexiSpot.xcodeproj -scheme LexiSpot build

# 3. Deploy to TestFlight / App Store
```

---

## Version History Example

```
v1.0.0 (Dec 2025)
├── Initial release
├── 34 languages
└── Basic functionality

v1.1.0 (Jan 2026)
├── Added 5 new languages (Italian, Portuguese, Russian, Japanese, Korean)
├── Updated French package
├── Fixed German translations
└── 39 languages total

v1.1.1 (Jan 2026)
├── Fixed crash on iOS 17
├── Corrected Spanish checksums
└── 39 languages (no new languages)

v1.2.0 (Feb 2026)
├── Added 10 more languages
├── Improved search performance
├── Updated English package
└── 49 languages total

v2.0.0 (Mar 2026)
├── New SQLite schema (breaking change)
├── Added example sentences
├── Better compression (30% smaller)
└── Requires iOS app v2.0+
```

---

## Release Notes Best Practices

### Good Release Notes

```markdown
## What's New in v1.1.0

### New Languages 🌍
- Italian (Italiano)
- Portuguese (Português)
- Russian (Русский)
- Japanese (日本語)
- Korean (한국어)

### Improvements ✨
- Updated French translations with 500 new terms
- Fixed German compound words
- Improved search accuracy

### Stats
- Total languages: 39
- Total entries: 2.5M+
- Total size: 50MB compressed

### Technical
- Package format: SQLite + Zstandard
- Checksums: SHA256
- CDN: GitHub Releases (global)
```

### What to Include

- ✅ What's new
- ✅ What changed
- ✅ What was fixed
- ✅ Statistics (package count, size)
- ✅ Breaking changes (if any)
- ✅ Migration notes (for major versions)

---

## Managing Multiple Versions

### Keep Old Versions Available

Old releases remain accessible:
```
v1.0.0 → https://github.com/lexispot/lexispot-packages/releases/tag/v1.0.0
v1.1.0 → https://github.com/lexispot/lexispot-packages/releases/tag/v1.1.0
v1.2.0 → https://github.com/lexispot/lexispot-packages/releases/tag/v1.2.0
```

This allows:
- Users on old app versions to still download
- Rollback if issues found
- A/B testing different versions

### Latest Release

GitHub's "latest" tag automatically points to newest:
```
https://github.com/lexispot/lexispot-packages/releases/latest/download/manifest.json
```

**However**: Don't use "latest" in production, use specific versions for stability.

---

## Rolling Updates (Single Version)

Alternative strategy: Keep updating v1.0.0 instead of creating new versions.

### Pros
- ✅ Simpler - no iOS app updates needed
- ✅ Users always get latest
- ✅ Single URL never changes

### Cons
- ⚠️ No version history
- ⚠️ Can't rollback easily
- ⚠️ No A/B testing

### How To

```bash
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
./add_languages.sh

# Choose: 2 (Update existing version)
```

This re-uploads to v1.0.0 with new packages.

---

## Deprecating Old Versions

When to mark a version as deprecated:

### Add Deprecation Notice

```bash
gh release edit v1.0.0 --repo lexispot/lexispot-packages \
  --notes "**⚠️ DEPRECATED**: Please update to v1.2.0 or later

This version is no longer maintained. New app versions require v1.2.0+.

[View Latest Release](https://github.com/lexispot/lexispot-packages/releases/latest)"
```

### Don't Delete Old Releases

- Users on old app versions need them
- GitHub has no storage limits
- Keeps history intact

---

## Beta Releases

### Pre-release for Testing

```bash
gh release create v1.2.0-beta \
  /Users/valerio/Documents/commands/dict-builder/packages/*.zst \
  /Users/valerio/Documents/commands/dict-builder/packages/manifest.json \
  --repo lexispot/lexispot-packages \
  --title "LexiSpot v1.2.0 Beta" \
  --notes "Beta release for testing. Not for production use." \
  --prerelease
```

### Testing Beta in App

```swift
// Use beta URL for testing
static let manifestURL = URL(string: 
    "https://github.com/lexispot/lexispot-packages/releases/download/v1.2.0-beta/manifest.json"
)!
```

### Promote Beta to Stable

After testing:
```bash
# 1. Delete beta
gh release delete v1.2.0-beta --repo lexispot/lexispot-packages --yes

# 2. Create stable
gh release create v1.2.0 \
  # ... same packages ...
  --repo lexispot/lexispot-packages
```

---

## Hotfix Workflow

When you need to quickly fix a critical bug:

### 1. Create Hotfix Release

```bash
# Fix the issue in your source files
# Then create patch version

gh release create v1.1.2 \
  /Users/valerio/Documents/commands/dict-builder/packages/*.zst \
  /Users/valerio/Documents/commands/dict-builder/packages/manifest.json \
  --repo lexispot/lexispot-packages \
  --title "LexiSpot v1.1.2 (Hotfix)" \
  --notes "## Hotfix Release

### Fixed
- Critical crash on app launch
- Corrupted French package

### No New Features
This is a maintenance release only."
```

### 2. Update App Urgently

```swift
// Update to hotfix version
static let manifestURL = URL(string: 
    "https://github.com/lexispot/lexispot-packages/releases/download/v1.1.2/manifest.json"
)!
```

### 3. Submit to App Store

- Request expedited review if critical
- Explain urgency in review notes

---

## Automated Versioning

### Using Git Tags

```bash
# Tag your commit
git tag -a v1.1.0 -m "Release v1.1.0"
git push origin v1.1.0

# Then create GitHub release from tag
gh release create v1.1.0 --notes-file RELEASE_NOTES.md
```

### Semantic Versioning

Follow semver.org:
- **MAJOR**: Breaking changes (v2.0.0)
- **MINOR**: New features (v1.1.0)
- **PATCH**: Bug fixes (v1.0.1)

### Version Bumping

```bash
# Bump version automatically
# (requires npm-version or similar tool)

npm version minor  # 1.0.0 → 1.1.0
npm version patch  # 1.1.0 → 1.1.1
npm version major  # 1.1.1 → 2.0.0
```

---

## Changelog

Keep a CHANGELOG.md in your repository:

```markdown
# Changelog

All notable changes to LexiSpot language packages.

## [1.2.0] - 2026-02-15

### Added
- 10 new languages
- Example sentences for common phrases
- Audio pronunciation data (beta)

### Changed
- Improved French translations
- Updated German compound words
- Better search indexing

### Fixed
- Spanish accent marks
- Chinese character encoding

## [1.1.0] - 2026-01-20

### Added
- Italian language support
- Portuguese language support
- Russian language support

### Changed
- Updated manifest format with entry counts

## [1.0.0] - 2025-12-05

### Added
- Initial release with 34 languages
```

---

## Version Tagging Strategy

### Tags to Use

```
Production:
- v1.0.0
- v1.1.0
- v1.2.0

Beta/Testing:
- v1.1.0-beta
- v1.2.0-rc1

Experimental:
- v2.0.0-alpha
```

### Don't Use

```
❌ latest (GitHub manages this)
❌ dev (too vague)
❌ test (not descriptive)
```

---

## Rollback Procedure

If a release has critical issues:

### 1. Revert iOS App

```swift
// Change back to previous working version
static let manifestURL = URL(string: 
    "https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json"
)!
```

### 2. Mark Release as Broken

```bash
gh release edit v1.1.0 --repo lexispot/lexispot-packages \
  --notes "**⚠️ DO NOT USE**: This release has critical issues

Please use v1.0.0 instead:
https://github.com/lexispot/lexispot-packages/releases/tag/v1.0.0

Issues:
- Corrupted packages
- Checksum mismatches

We're working on v1.1.1 to fix these issues."
```

### 3. Don't Delete

- Keep for reference
- Shows what went wrong
- Prevents version number reuse

### 4. Release Fixed Version

```bash
# Release v1.1.1 with fixes
gh release create v1.1.1 ...
```

---

## Version Management Tools

### GitHub CLI

```bash
# List all releases
gh release list --repo lexispot/lexispot-packages

# View specific release
gh release view v1.0.0 --repo lexispot/lexispot-packages

# Compare releases
diff <(gh release view v1.0.0 --json assets) \
     <(gh release view v1.1.0 --json assets)
```

### Git

```bash
# List tags
git tag -l

# Show tag details
git show v1.0.0

# Create annotated tag
git tag -a v1.1.0 -m "Release v1.1.0"

# Push tags
git push origin --tags
```

---

## Multi-Environment Strategy

### Development

```
Repository: lexispot-packages-dev
URL: https://github.com/lexispot/lexispot-packages-dev/releases/download/v1.0.0/manifest.json
```

### Staging

```
Repository: lexispot-packages-staging
URL: https://github.com/lexispot/lexispot-packages-staging/releases/download/v1.0.0/manifest.json
```

### Production

```
Repository: lexispot-packages
URL: https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json
```

---

## Best Practices Summary

### Do ✅

- Use semantic versioning
- Write clear release notes
- Keep old versions available
- Test before releasing
- Tag releases in git
- Document breaking changes
- Increment versions properly

### Don't ❌

- Delete old releases
- Reuse version numbers
- Skip version numbers
- Use vague tags like "latest" in prod
- Make breaking changes in minor versions
- Forget to update iOS app URL
- Release without testing

---

## Version Checklist

Before creating a new version:

- [ ] All packages generated correctly
- [ ] Manifest has correct checksums
- [ ] Tested downloads locally
- [ ] Release notes written
- [ ] Version number incremented properly
- [ ] Breaking changes documented (if any)
- [ ] iOS app URL updated (if needed)
- [ ] TestFlight build ready (if needed)

---

**Related Guides**:
- `02-Adding-Languages.md` - Adding new content
- `03-Quick-Reference.md` - Version commands
- `04-Troubleshooting.md` - Version issues
