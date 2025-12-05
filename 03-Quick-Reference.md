# 03 - Quick Reference

## Essential Commands

### Check Status

```bash
# View current release
gh release view v1.0.0 --repo lexispot/lexispot-packages

# List all releases
gh release list --repo lexispot/lexispot-packages

# Count packages
ls /Users/valerio/Documents/commands/dict-builder/packages/*.zst | wc -l
```

### Add New Languages

```bash
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
./add_languages.sh
```

### Regenerate Manifest

```bash
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
python3 generate_manifest.py
```

### Test Manifest URL

```bash
curl -L https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json | jq .
```

### Download Package Test

```bash
curl -L -o test.zst https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/en.sqlite.zst
ls -lh test.zst
rm test.zst
```

---

## File Locations

### Scripts
```
/Users/valerio/Documents/AppsWidgets/LexiSpot/
├── setup_github_cdn.sh        # Initial setup
├── generate_manifest.py       # Generate manifest
└── add_languages.sh           # Add languages
```

### Packages
```
/Users/valerio/Documents/commands/dict-builder/packages/
├── *.sqlite                   # Uncompressed
├── *.sqlite.zst              # Compressed (uploaded)
└── manifest.json             # Package metadata
```

### iOS App
```
/Users/valerio/Documents/AppsWidgets/LexiSpot/LexiSpot/
└── Models/
    └── LanguageManifest.swift # CDN configuration
```

### Guides
```
/Users/valerio/Documents/commands/lexispot-dict-builder/Guides/
├── 00-README.md
├── 01-Initial-Setup.md
├── 02-Adding-Languages.md
├── 03-Quick-Reference.md      # This file
├── 04-Troubleshooting.md
├── 05-iOS-Integration.md
└── 06-Version-Management.md
```

---

## GitHub URLs

### Repository
```
https://github.com/lexispot/lexispot-packages
```

### Releases
```
https://github.com/lexispot/lexispot-packages/releases
```

### Manifest (Current)
```
https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json
```

### Package URL Pattern
```
https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/LANGUAGE_CODE.sqlite.zst
```

Examples:
- `en.sqlite.zst` - English
- `fr.sqlite.zst` - French
- `es.sqlite.zst` - Spanish
- `de.sqlite.zst` - German
- `it.sqlite.zst` - Italian
- `pt.sqlite.zst` - Portuguese
- `ru.sqlite.zst` - Russian
- `ja.sqlite.zst` - Japanese
- `ko.sqlite.zst` - Korean
- `zh-CN.sqlite.zst` - Chinese (Simplified)

---

## Common Tasks

### Add Single Language

```bash
# 1. Generate NEW_LANG.sqlite.zst with your dict-builder

# 2. Add to CDN
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
./add_languages.sh
# Choose: 2 (Update existing)
```

### Add Multiple Languages

```bash
# 1. Generate multiple .sqlite.zst files

# 2. Add all at once
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
./add_languages.sh
# Script auto-detects all new packages
```

### Create New Version

```bash
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
./add_languages.sh
# Choose: 1 (New version)
# Enter: v1.1.0

# Then update iOS app:
# Edit: LexiSpot/Models/LanguageManifest.swift
# Change: v1.0.0 → v1.1.0
```

### Delete Package from Release

```bash
gh release delete-asset v1.0.0 LANGUAGE.sqlite.zst \
  --repo lexispot/lexispot-packages --yes
```

### Delete Entire Release

```bash
gh release delete v1.0.0 --repo lexispot/lexispot-packages --yes
```

### Re-upload Everything

```bash
cd /Users/valerio/Documents/AppsWidgets/LexiSpot

# Delete old release
gh release delete v1.0.0 --repo lexispot/lexispot-packages --yes

# Re-run setup
./setup_github_cdn.sh
```

---

## Verification Commands

### Check Package Count

```bash
# On GitHub
gh release view v1.0.0 --repo lexispot/lexispot-packages --json assets -q '.assets | length'

# Local directory
ls /Users/valerio/Documents/commands/dict-builder/packages/*.zst | wc -l
```

### Verify Checksums

```bash
# Calculate checksum
shasum -a 256 /Users/valerio/Documents/commands/dict-builder/packages/en.sqlite.zst

# Compare with manifest
cat /Users/valerio/Documents/commands/dict-builder/packages/manifest.json | jq '.packages[] | select(.language_code=="en") | .checksum'
```

### Test zstd Compression

```bash
# Verify file is valid
zstd -t /Users/valerio/Documents/commands/dict-builder/packages/en.sqlite.zst

# Decompress test
zstd -d /Users/valerio/Documents/commands/dict-builder/packages/en.sqlite.zst -o /tmp/test.sqlite
file /tmp/test.sqlite
rm /tmp/test.sqlite
```

### Check Package Sizes

```bash
# Individual sizes
ls -lh /Users/valerio/Documents/commands/dict-builder/packages/*.zst

# Total size
du -sh /Users/valerio/Documents/commands/dict-builder/packages/

# Largest files
ls -lhS /Users/valerio/Documents/commands/dict-builder/packages/*.zst | head
```

---

## iOS App Commands

### Open Project

```bash
open /Users/valerio/Documents/AppsWidgets/LexiSpot/LexiSpot.xcodeproj
```

### Build & Run

```bash
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
xcodebuild -project LexiSpot.xcodeproj -scheme LexiSpot -destination 'platform=iOS Simulator,name=iPhone 15' build
```

### Clean Build

```bash
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
xcodebuild clean -project LexiSpot.xcodeproj -scheme LexiSpot
```

---

## GitHub CLI Tips

### Login / Logout

```bash
# Check status
gh auth status

# Logout
gh auth logout

# Login again
gh auth login
```

### View Release in Browser

```bash
gh release view v1.0.0 --repo lexispot/lexispot-packages --web
```

### Download All Assets

```bash
mkdir -p ~/Desktop/lexispot-backup
cd ~/Desktop/lexispot-backup
gh release download v1.0.0 --repo lexispot/lexispot-packages
```

### Create Release Manually

```bash
gh release create v1.0.1 \
  /Users/valerio/Documents/commands/dict-builder/packages/*.zst \
  /Users/valerio/Documents/commands/dict-builder/packages/manifest.json \
  --repo lexispot/lexispot-packages \
  --title "LexiSpot v1.0.1" \
  --notes "Updated language packages"
```

---

## Manifest Manipulation

### View Package List

```bash
curl -sL https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json | \
  jq -r '.packages[] | "\(.language_code): \(.name)"'
```

### Get Package Count

```bash
curl -sL https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json | \
  jq '.packages | length'
```

### Find Specific Language

```bash
curl -sL https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json | \
  jq '.packages[] | select(.language_code=="en")'
```

### Total Compressed Size

```bash
curl -sL https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json | \
  jq '[.packages[].compressed_size] | add'
```

---

## Backup Commands

### Backup Repository

```bash
cd ~/Desktop
git clone https://github.com/lexispot/lexispot-packages.git lexispot-backup
cd lexispot-backup
gh release download v1.0.0
```

### Backup Source Files

```bash
# Backup to external drive
cp -r /Users/valerio/Documents/commands/dict-builder/packages/ \
  /Volumes/Backup/lexispot-packages-$(date +%Y%m%d)/
```

### Backup iOS Project

```bash
cd /Users/valerio/Documents/AppsWidgets/
tar -czf LexiSpot-backup-$(date +%Y%m%d).tar.gz LexiSpot/
```

---

## Monitoring

### Watch Upload Progress

```bash
# In another terminal while uploading
watch -n 5 'gh release view v1.0.0 --repo lexispot/lexispot-packages --json assets -q ".assets | length"'
```

### Check GitHub API Rate Limit

```bash
gh api rate_limit
```

### View Repository Stats

```bash
gh api repos/lexispot/lexispot-packages
```

---

## Keyboard Shortcuts (Xcode)

| Action | Shortcut |
|--------|----------|
| Build & Run | Cmd+R |
| Stop | Cmd+. |
| Clean Build | Cmd+Shift+K |
| Open Quickly | Cmd+Shift+O |
| Build | Cmd+B |
| Test | Cmd+U |

---

## Environment Variables

```bash
# Set for session
export GITHUB_USER="lexispot"
export GITHUB_REPO="lexispot-packages"

# Use in commands
gh release view v1.0.0 --repo $GITHUB_USER/$GITHUB_REPO
```

---

## One-Liner Commands

### Quick Status Check

```bash
echo "Packages on GitHub: $(gh release view v1.0.0 --repo lexispot/lexispot-packages --json assets -q '.assets | length')" && echo "Packages locally: $(ls /Users/valerio/Documents/commands/dict-builder/packages/*.zst | wc -l | xargs)"
```

### Quick Add Language

```bash
cd /Users/valerio/Documents/AppsWidgets/LexiSpot && python3 generate_manifest.py && gh release upload v1.0.0 /Users/valerio/Documents/commands/dict-builder/packages/NEW_LANG.sqlite.zst /Users/valerio/Documents/commands/dict-builder/packages/manifest.json --repo lexispot/lexispot-packages --clobber
```

### Quick Test Download

```bash
curl -sL https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/en.sqlite.zst | zstd -d | file -
```

---

## Aliases (Add to ~/.zshrc)

```bash
# LexiSpot shortcuts
alias lsp-cd='cd /Users/valerio/Documents/AppsWidgets/LexiSpot'
alias lsp-pkg='cd /Users/valerio/Documents/commands/dict-builder/packages'
alias lsp-status='gh release view v1.0.0 --repo lexispot/lexispot-packages'
alias lsp-add='cd /Users/valerio/Documents/AppsWidgets/LexiSpot && ./add_languages.sh'
alias lsp-manifest='cd /Users/valerio/Documents/AppsWidgets/LexiSpot && python3 generate_manifest.py'
alias lsp-open='open /Users/valerio/Documents/AppsWidgets/LexiSpot/LexiSpot.xcodeproj'

# Reload shell
source ~/.zshrc
```

Then use:
```bash
lsp-cd        # Go to LexiSpot project
lsp-pkg       # Go to packages directory
lsp-status    # Check release status
lsp-add       # Add new languages
lsp-manifest  # Regenerate manifest
lsp-open      # Open Xcode project
```

---

## Emergency Recovery

### If Something Breaks

```bash
# 1. Check what's wrong
gh release view v1.0.0 --repo lexispot/lexispot-packages

# 2. Delete broken release
gh release delete v1.0.0 --repo lexispot/lexispot-packages --yes

# 3. Regenerate manifest
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
python3 generate_manifest.py

# 4. Re-upload everything
./setup_github_cdn.sh
```

### If iOS App Won't Download

```bash
# Test manifest URL
curl -I https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json

# Test package URL
curl -I https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/en.sqlite.zst

# Check checksums match
shasum -a 256 /Users/valerio/Documents/commands/dict-builder/packages/en.sqlite.zst
curl -sL https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json | jq '.packages[] | select(.language_code=="en") | .checksum'
```

---

**Need more help?** See `04-Troubleshooting.md`
