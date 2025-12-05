# 02 - Adding New Languages Guide

## Overview

This guide shows you how to add more language packages to your CDN after the initial setup.

**Use this when**: You've generated new dictionary packages and want to make them available in the app.

---

## Quick Process (3 Steps)

```bash
# 1. Generate new dictionaries (your existing process)
cd /Users/valerio/Documents/commands/dict-builder
# ... run your dict-builder commands ...

# 2. Run the automated script
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
./add_languages.sh

# 3. Done! (optionally update iOS app if you created a new version)
```

That's it! The script handles everything automatically.

---

## Detailed Walkthrough

### Step 1: Generate New Dictionary Packages

Use your existing dict-builder to create new `.sqlite.zst` files:

```bash
cd /Users/valerio/Documents/commands/dict-builder

# Your dict-builder process creates files like:
# - it.sqlite.zst (Italian)
# - pt.sqlite.zst (Portuguese)
# - ru.sqlite.zst (Russian)
# etc.
```

All new `.sqlite.zst` files go into:
```
/Users/valerio/Documents/commands/dict-builder/packages/
```

### Step 2: Run the Add Languages Script

```bash
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
./add_languages.sh
```

The script will:

1. **Detect Changes**
   ```
   🔍 Detecting changes...
   
   📊 Package Summary:
      Current release: 34 packages
      Local directory: 37 packages
      New packages: 3
   
   🆕 New packages to add:
      ✅ it.sqlite.zst
      ✅ pt.sqlite.zst
      ✅ ru.sqlite.zst
   ```

2. **Ask for Strategy**
   ```
   📋 Choose update strategy:
      1. Create NEW version (v1.0.1, v1.1.0, etc.) - Recommended
      2. Update EXISTING version (v1.0.0) - Simpler, but no history
   
   Enter choice (1 or 2):
   ```

3. **Upload to GitHub**
   - Regenerates manifest with fresh checksums
   - Uploads new packages
   - Updates or creates release

---

## Two Update Strategies

### Strategy 1: Create New Version (Recommended)

**When to use**: Major updates, want version history, adding many languages

**Process**:
```bash
./add_languages.sh
# Choose: 1
# Enter new version: v1.1.0
```

**What happens**:
- ✅ Creates new release v1.1.0
- ✅ Uploads ALL packages (old + new)
- ✅ Keeps version history
- ⚠️ Requires iOS app URL update

**Update iOS app**:
```swift
// In: LexiSpot/Models/LanguageManifest.swift
// Change:
static let manifestURL = URL(string: "https://github.com/lexispot/lexispot-packages/releases/download/v1.1.0/manifest.json")!
//                                                                                                      ^^^^^^^ Update version
```

**Benefits**:
- Clean version history
- Can rollback to previous versions
- Professional release management
- Old versions remain available

---

### Strategy 2: Update Existing Version

**When to use**: Adding 1-2 languages, quick update, don't want to update app

**Process**:
```bash
./add_languages.sh
# Choose: 2
```

**What happens**:
- ✅ Adds new packages to v1.0.0
- ✅ Updates manifest.json
- ✅ **No iOS app changes needed!**
- ⚠️ No version history

**Benefits**:
- Simplest option
- No app updates required
- Users see new languages immediately
- Same URL keeps working

---

## Example: Adding Italian, Portuguese, Russian

### Before
```
Current packages: 34
- English, French, German, Spanish, etc.
```

### Generate New Dictionaries
```bash
cd /Users/valerio/Documents/commands/dict-builder
# Your process creates:
# - it.sqlite.zst
# - pt.sqlite.zst  
# - ru.sqlite.zst
```

### Run Script
```bash
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
./add_languages.sh
```

**Output**:
```
🌍 LexiSpot - Add New Languages
================================

🔍 Detecting changes...

📊 Package Summary:
   Current release: 34 packages
   Local directory: 37 packages
   New packages: 3

🆕 New packages to add:
   ✅ it.sqlite.zst
   ✅ pt.sqlite.zst
   ✅ ru.sqlite.zst

📋 Choose update strategy:
   1. Create NEW version (v1.0.1, v1.1.0, etc.) - Recommended
   2. Update EXISTING version (v1.0.0) - Simpler, but no history

Enter choice (1 or 2): 2

📝 Regenerating manifest with updated URLs...
✅ Manifest generated

📤 Uploading new packages to v1.0.0...
   Uploading it.sqlite.zst...
   Uploading pt.sqlite.zst...
   Uploading ru.sqlite.zst...
   Uploading manifest.json...

✅ Release v1.0.0 updated!

📱 No iOS app changes needed!
   Users will automatically see new languages on next app launch

🎉 Done! New languages are now available on your CDN.
```

### After
```
Total packages: 37
- English, French, German, Spanish, Italian, Portuguese, Russian, etc.
```

---

## Automatic App Updates

Your iOS app has **automatic update checking** built-in!

### How It Works

1. **App Launch**: Fetches manifest.json
2. **Comparison**: Compares with cached version
3. **Detection**: Finds new packages automatically
4. **Display**: Shows new languages in "Available" list
5. **Download**: Users can download immediately

### User Experience

**Existing users** (Strategy 2):
- Open app → New languages appear
- No app update required
- Instant availability

**New version** (Strategy 1):
- Need to update app from App Store
- Or release new build to TestFlight
- Then new languages appear

---

## Language Name Mapping

The script automatically maps 100+ language codes to proper names.

### Already Supported

```
af=Afrikaans, sq=Albanian, am=Amharic, ar=Arabic, hy=Armenian,
az=Azerbaijani, eu=Basque, be=Belarusian, bn=Bengali, bs=Bosnian,
bg=Bulgarian, ca=Catalan, zh-CN=Chinese (Simplified), 
zh-TW=Chinese (Traditional), hr=Croatian, cs=Czech, da=Danish,
nl=Dutch, en=English, eo=Esperanto, et=Estonian, fi=Finnish,
fr=French, gl=Galician, ka=Georgian, de=German, el=Greek,
gu=Gujarati, ht=Haitian Creole, he=Hebrew, hi=Hindi, hu=Hungarian,
is=Icelandic, id=Indonesian, ga=Irish, it=Italian, ja=Japanese,
ko=Korean, la=Latin, lv=Latvian, lt=Lithuanian, mk=Macedonian,
ms=Malay, mt=Maltese, no=Norwegian, fa=Persian, pl=Polish,
pt=Portuguese, ro=Romanian, ru=Russian, sr=Serbian, sk=Slovak,
sl=Slovenian, es=Spanish, sw=Swahili, sv=Swedish, th=Thai,
tr=Turkish, uk=Ukrainian, ur=Urdu, vi=Vietnamese, cy=Welsh,
... and 50+ more!
```

### Adding New Language Names

If a language shows as "XX" instead of proper name:

1. Edit `generate_manifest.py`
2. Add to `language_name_map()` function:

```python
def language_name_map():
    return {
        # ... existing entries ...
        "NEW": ("Language Name", "Native Name"),
    }
```

3. Run `python3 generate_manifest.py` again

---

## Testing New Languages

### 1. Test Locally First

```bash
# Check file exists
ls -lh /Users/valerio/Documents/commands/dict-builder/packages/NEW_LANG.sqlite.zst

# Verify it's valid zstd
zstd -t /Users/valerio/Documents/commands/dict-builder/packages/NEW_LANG.sqlite.zst

# Check checksum
shasum -a 256 /Users/valerio/Documents/commands/dict-builder/packages/NEW_LANG.sqlite.zst
```

### 2. Test in iOS Simulator

After upload:
1. Run app in Xcode
2. Go to Languages tab
3. Find new language
4. Download it
5. Go to Search and test translations

### 3. Verify on GitHub

```bash
# Check release
gh release view v1.0.0 --repo lexispot/lexispot-packages

# Test download
curl -L -o test.zst https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/NEW_LANG.sqlite.zst
```

---

## Manual Addition (If Script Fails)

If `add_languages.sh` has issues, add manually:

```bash
# 1. Regenerate manifest
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
python3 generate_manifest.py

# 2. Upload specific package
gh release upload v1.0.0 \
  /Users/valerio/Documents/commands/dict-builder/packages/NEW_LANG.sqlite.zst \
  --repo lexispot/lexispot-packages \
  --clobber

# 3. Update manifest
gh release upload v1.0.0 \
  /Users/valerio/Documents/commands/dict-builder/packages/manifest.json \
  --repo lexispot/lexispot-packages \
  --clobber
```

---

## Batch Adding Multiple Languages

The script handles multiple languages automatically:

```bash
# Your dict-builder creates:
# - it.sqlite.zst
# - pt.sqlite.zst
# - ru.sqlite.zst
# - ja.sqlite.zst
# - ko.sqlite.zst

# Run once:
./add_languages.sh

# It detects all 5 new packages
# Uploads all in one operation
```

---

## Version Numbering Best Practices

### Semantic Versioning

```
v1.0.0 → Initial 34 languages
v1.1.0 → Added 10 more languages (minor update)
v1.1.1 → Fixed bug in French package (patch)
v1.2.0 → Added 5 languages + updated 3 existing (minor update)
v2.0.0 → Major rebuild with new format (major update)
```

### Increment Rules

- **Patch (v1.0.X)**: Bug fixes, small corrections
- **Minor (v1.X.0)**: New languages, improvements
- **Major (vX.0.0)**: Breaking changes, format changes

---

## Rollback (If Needed)

If something goes wrong:

### Delete Recent Release
```bash
gh release delete v1.1.0 --repo lexispot/lexispot-packages --yes
```

### Revert iOS App
```swift
// Change back to:
static let manifestURL = URL(string: "https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json")!
```

### Previous Releases Remain
Old releases are never deleted automatically. Users can always access previous versions.

---

## Troubleshooting

### "No new packages detected"
```bash
# Check files exist
ls /Users/valerio/Documents/commands/dict-builder/packages/*.zst

# Make sure they're not already uploaded
gh release view v1.0.0 --repo lexispot/lexispot-packages
```

### "Checksum mismatch in app"
```bash
# Regenerate manifest
python3 generate_manifest.py

# Re-upload both
./add_languages.sh
```

### "Language not showing in app"
```bash
# Verify manifest has it
curl -L https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json | jq '.packages[] | select(.language_code=="NEW_LANG")'

# Force app to refresh (pull to refresh in app)
```

---

## Cost Impact

Adding languages = **$0.00** additional cost!

- ✅ No storage limits
- ✅ No bandwidth fees
- ✅ No per-file charges
- ✅ Add 100 languages → Still free

---

## Next Steps

After adding languages:
1. Test downloads in app
2. Update app description with new language count
3. Announce to users (if relevant)
4. Consider updating App Store screenshots

See `03-Quick-Reference.md` for command shortcuts!
