# 04 - Troubleshooting Guide

## Common Issues and Solutions

---

## Setup Issues

### "gh: command not found"

**Problem**: GitHub CLI not installed

**Solution**:
```bash
brew install gh
gh --version  # Verify installation
```

---

### "jq: command not found"

**Problem**: jq JSON processor not installed

**Solution**:
```bash
brew install jq
jq --version  # Verify installation
```

---

### "Authentication required"

**Problem**: Not logged into GitHub

**Solution**:
```bash
gh auth status  # Check status
gh auth login   # Login if needed
```

**Steps**:
1. Choose: GitHub.com
2. Choose: HTTPS
3. Choose: Login with a web browser
4. Copy code and paste in browser
5. Authorize GitHub CLI

---

### "Repository is empty" when creating release

**Problem**: GitHub repository has no commits

**Solution**:
```bash
cd ~/Desktop
git clone https://github.com/lexispot/lexispot-packages.git temp_repo
cd temp_repo

# Create README
echo "# LexiSpot Language Packages" > README.md
git add README.md
git commit -m "Initial commit"
git push origin main

cd ..
rm -rf temp_repo

# Now run setup again
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
./setup_github_cdn.sh
```

---

### "Permission denied" when running scripts

**Problem**: Scripts not executable

**Solution**:
```bash
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
chmod +x setup_github_cdn.sh
chmod +x generate_manifest.py
chmod +x add_languages.sh
```

Or run with bash/python directly:
```bash
bash setup_github_cdn.sh
python3 generate_manifest.py
```

---

## Upload Issues

### Upload stuck or very slow

**Problem**: Network issues or large files

**Solution**:

1. **Check your internet connection**
   ```bash
   ping github.com
   ```

2. **Resume from where it left off** (script handles this)
   ```bash
   ./setup_github_cdn.sh  # It will skip already uploaded files
   ```

3. **Upload manually in smaller batches**
   ```bash
   # Upload first 10 files
   for file in /Users/valerio/Documents/commands/dict-builder/packages/*.zst | head -10; do
     gh release upload v1.0.0 "$file" --repo lexispot/lexispot-packages --clobber
   done
   ```

---

### "HTTP 422: Validation Failed"

**Problem**: Release or file already exists, or validation error

**Solutions**:

**If release exists**:
```bash
# Delete and recreate
gh release delete v1.0.0 --repo lexispot/lexispot-packages --yes
./setup_github_cdn.sh
```

**If repository is empty**:
See "Repository is empty" solution above.

---

### Upload fails midway

**Problem**: Network interruption

**Solution**:
```bash
# The script with --clobber will resume
./setup_github_cdn.sh

# Or manually upload remaining files
gh release upload v1.0.0 \
  /Users/valerio/Documents/commands/dict-builder/packages/MISSING_FILE.zst \
  --repo lexispot/lexispot-packages \
  --clobber
```

---

## Manifest Issues

### "Cannot iterate over null (null)" jq error

**Problem**: Manifest has wrong structure or field name

**Solution**: The manifest structure is correct (uses `.packages` not `.languages`). This was already fixed in the script.

If you see this again:
```bash
# Verify manifest structure
cat /Users/valerio/Documents/commands/dict-builder/packages/manifest.json | jq '.packages | length'

# Should show a number, not error
```

---

### Manifest not found

**Problem**: Manifest file doesn't exist

**Solution**:
```bash
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
python3 generate_manifest.py
```

---

### Wrong checksums in manifest

**Problem**: Files changed after manifest generation

**Solution**:
```bash
# Regenerate manifest with fresh checksums
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
python3 generate_manifest.py

# Re-upload manifest
gh release upload v1.0.0 \
  /Users/valerio/Documents/commands/dict-builder/packages/manifest.json \
  --repo lexispot/lexispot-packages \
  --clobber
```

---

## iOS App Issues

### App can't fetch manifest

**Problem**: URL incorrect or network issue

**Diagnose**:
```bash
# Test manifest URL
curl -I https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json

# Should return: HTTP/2 200
```

**Solutions**:

1. **Check iOS app configuration**
   ```swift
   // In: LexiSpot/Models/LanguageManifest.swift
   // Verify URL is correct:
   static let manifestURL = URL(string: "https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json")!
   ```

2. **Check release exists**
   ```bash
   gh release view v1.0.0 --repo lexispot/lexispot-packages
   ```

3. **Test in browser**: Open URL in Safari
   - Should show JSON, not 404

---

### "Checksum mismatch" error in app

**Problem**: Downloaded file doesn't match expected checksum

**Diagnose**:
```bash
# Calculate actual checksum
shasum -a 256 /Users/valerio/Documents/commands/dict-builder/packages/en.sqlite.zst

# Check manifest checksum
curl -sL https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json | \
  jq '.packages[] | select(.language_code=="en") | .checksum'

# Should match!
```

**Solutions**:

1. **Regenerate manifest**
   ```bash
   cd /Users/valerio/Documents/AppsWidgets/LexiSpot
   python3 generate_manifest.py
   ```

2. **Re-upload both package and manifest**
   ```bash
   gh release upload v1.0.0 \
     /Users/valerio/Documents/commands/dict-builder/packages/en.sqlite.zst \
     /Users/valerio/Documents/commands/dict-builder/packages/manifest.json \
     --repo lexispot/lexispot-packages \
     --clobber
   ```

---

### Package downloads but won't install

**Problem**: Corrupted file or decompression issue

**Diagnose**:
```bash
# Test file is valid zstd
zstd -t /Users/valerio/Documents/commands/dict-builder/packages/en.sqlite.zst

# Test decompression
zstd -d /Users/valerio/Documents/commands/dict-builder/packages/en.sqlite.zst -o /tmp/test.sqlite
file /tmp/test.sqlite  # Should say: SQLite 3.x database
rm /tmp/test.sqlite
```

**Solution**:
```bash
# Re-compress the source file
cd /Users/valerio/Documents/commands/dict-builder/packages
zstd -f en.sqlite -o en.sqlite.zst

# Regenerate manifest and re-upload
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
python3 generate_manifest.py
gh release upload v1.0.0 en.sqlite.zst manifest.json --repo lexispot/lexispot-packages --clobber
```

---

### New language not showing in app

**Problem**: Manifest not updated or app not refreshing

**Diagnose**:
```bash
# Check if language is in manifest
curl -sL https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json | \
  jq '.packages[] | select(.language_code=="NEW_LANG")'

# Should return package info, not empty
```

**Solutions**:

1. **Verify upload**
   ```bash
   gh release view v1.0.0 --repo lexispot/lexispot-packages
   # Check if NEW_LANG.sqlite.zst is listed
   ```

2. **Re-upload manifest**
   ```bash
   gh release upload v1.0.0 \
     /Users/valerio/Documents/commands/dict-builder/packages/manifest.json \
     --repo lexispot/lexispot-packages \
     --clobber
   ```

3. **Force app refresh**
   - In app: Go to Settings
   - Pull to refresh
   - Or: Kill and relaunch app

---

## Package Issues

### Package too large (>2GB)

**Problem**: GitHub has 2GB per file limit

**Solution**:
```bash
# Check file size
ls -lh /Users/valerio/Documents/commands/dict-builder/packages/LARGE.sqlite.zst

# If >2GB, you need to:
# 1. Optimize the source SQLite database
# 2. Use better compression
# 3. Or split into multiple packages
```

For very large packages:
- Consider using SQLite VACUUM to optimize
- Use maximum zstd compression: `zstd -19 file.sqlite`
- Split large languages into multiple files

---

### Wrong language name displayed

**Problem**: Language code not in name mapping

**Solution**:
```bash
# Edit generate_manifest.py
# Add to language_name_map() function:

def language_name_map():
    return {
        # ... existing entries ...
        "NEW": ("Proper Name", "Native Name"),
    }

# Regenerate and re-upload
python3 generate_manifest.py
gh release upload v1.0.0 manifest.json --repo lexispot/lexispot-packages --clobber
```

---

### Package showing as 0 KB or corrupted

**Problem**: Upload failed or file corruption

**Solution**:
```bash
# Delete the asset
gh release delete-asset v1.0.0 PACKAGE.sqlite.zst \
  --repo lexispot/lexispot-packages --yes

# Re-upload
gh release upload v1.0.0 \
  /Users/valerio/Documents/commands/dict-builder/packages/PACKAGE.sqlite.zst \
  --repo lexispot/lexispot-packages
```

---

## Script Issues

### add_languages.sh shows no new packages

**Problem**: Files already uploaded or wrong directory

**Diagnose**:
```bash
# Check local files
ls /Users/valerio/Documents/commands/dict-builder/packages/*.zst

# Check GitHub
gh release view v1.0.0 --repo lexispot/lexispot-packages --json assets -q '.assets[].name' | grep '.zst$'

# Compare
```

**Solution**:
Make sure new files exist locally and aren't on GitHub yet.

---

### Python script fails with import error

**Problem**: Python version or missing module

**Solution**:
```bash
# Check Python version (need 3.x)
python3 --version

# Script uses only standard library, no pip install needed
# If still fails, run with verbose errors:
python3 -v generate_manifest.py
```

---

### Script hangs or freezes

**Problem**: Waiting for input or large file processing

**Solution**:
- Check if script is waiting for input (press Enter)
- For large files, give it time
- Check Activity Monitor for CPU usage
- Ctrl+C to cancel, then try again

---

## GitHub Issues

### Rate limit exceeded

**Problem**: Too many API requests

**Check**:
```bash
gh api rate_limit
```

**Solution**:
Wait for rate limit to reset (usually 1 hour), or:
```bash
# Authenticate to get higher limits
gh auth login
```

---

### "Resource not accessible by personal access token"

**Problem**: Token permissions insufficient

**Solution**:
```bash
# Re-authenticate with full permissions
gh auth logout
gh auth login
# Make sure to grant all requested permissions
```

---

### Can't delete release

**Problem**: Permission issue or release in use

**Solution**:
```bash
# Force delete
gh release delete v1.0.0 --repo lexispot/lexispot-packages --yes

# If still fails, delete from web:
# https://github.com/lexispot/lexispot-packages/releases
```

---

## Storage Issues

### Low disk space

**Problem**: Not enough space for packages

**Check**:
```bash
df -h
du -sh /Users/valerio/Documents/commands/dict-builder/packages/
```

**Solution**:
- Delete old/temporary files
- Clean Xcode derived data: `rm -rf ~/Library/Developer/Xcode/DerivedData/*`
- Empty trash
- Or use external drive for packages

---

### Can't write to directory

**Problem**: Permission issue

**Solution**:
```bash
# Check permissions
ls -la /Users/valerio/Documents/commands/dict-builder/packages/

# Fix if needed
chmod -R u+w /Users/valerio/Documents/commands/dict-builder/packages/
```

---

## Network Issues

### Slow uploads

**Problem**: Slow internet or large files

**Solutions**:
- Use wired connection instead of WiFi
- Upload during off-peak hours
- Upload in smaller batches
- Check: `speedtest-cli` (install with `brew install speedtest-cli`)

---

### Connection timeout

**Problem**: Network interruption

**Solution**:
```bash
# The upload scripts use --clobber to resume
# Just run again:
./setup_github_cdn.sh

# Or manually resume:
gh release upload v1.0.0 REMAINING_FILES --repo lexispot/lexispot-packages --clobber
```

---

## Complete Reset (Nuclear Option)

If everything is broken and you want to start fresh:

```bash
# 1. Delete GitHub release
gh release delete v1.0.0 --repo lexispot/lexispot-packages --yes

# 2. Regenerate manifest
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
python3 generate_manifest.py

# 3. Re-run complete setup
./setup_github_cdn.sh

# 4. Verify in iOS app
open LexiSpot.xcodeproj
```

---

## Getting Help

### Check Logs

**GitHub CLI**:
```bash
gh release view v1.0.0 --repo lexispot/lexispot-packages
```

**iOS App**: 
- Xcode console output
- Device logs in Console.app

### Verify Everything

```bash
# 1. Check GitHub authentication
gh auth status

# 2. Check repository exists
gh repo view lexispot/lexispot-packages

# 3. Check release exists
gh release view v1.0.0 --repo lexispot/lexispot-packages

# 4. Check manifest is valid
curl -sL https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json | jq .

# 5. Check packages are accessible
curl -I https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/en.sqlite.zst
```

### Test Components Individually

```bash
# Test manifest generation
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
python3 generate_manifest.py

# Test single file upload
gh release upload v1.0.0 \
  /Users/valerio/Documents/commands/dict-builder/packages/en.sqlite.zst \
  --repo lexispot/lexispot-packages \
  --clobber

# Test download
curl -L -o /tmp/test.zst https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/en.sqlite.zst
zstd -t /tmp/test.zst
rm /tmp/test.zst
```

---

## Still Stuck?

1. **Check other guides**:
   - `01-Initial-Setup.md` - Setup process
   - `02-Adding-Languages.md` - Adding languages
   - `03-Quick-Reference.md` - Commands

2. **Check GitHub Docs**:
   - https://cli.github.com/manual/
   - https://docs.github.com/en/repositories/releasing-projects-on-github

3. **Verify URLs**: All URLs should be accessible in a browser

4. **Start Fresh**: Use the Complete Reset steps above

---

**Last Updated**: December 2025
