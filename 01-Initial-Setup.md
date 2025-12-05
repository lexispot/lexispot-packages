# 01 - Initial CDN Setup Guide

## Overview

This guide covers the **one-time setup** of your free GitHub CDN for LexiSpot language packages.

**Status**: ✅ COMPLETED (You've already done this!)

---

## What Was Set Up

### GitHub Repository
- **Account**: lexispot
- **Repository**: lexispot-packages
- **URL**: https://github.com/lexispot/lexispot-packages
- **Visibility**: Public (required for free CDN)

### Initial Release
- **Version**: v1.0.0
- **Packages**: 34 language files (.sqlite.zst)
- **Manifest**: manifest.json with SHA256 checksums
- **CDN**: GitHub + Fastly global network

### iOS App Configuration
- **File**: `/Users/valerio/Documents/AppsWidgets/LexiSpot/LexiSpot/Models/LanguageManifest.swift`
- **Manifest URL**: `https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json`

---

## Setup Steps (What You Did)

### 1. Installed Tools
```bash
brew install gh jq
```

- **gh**: GitHub CLI for uploading releases
- **jq**: JSON processor for manifest validation

### 2. GitHub Authentication
```bash
gh auth login
```

Logged in with GitHub account: **lexispot**

### 3. Generated Manifest
```bash
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
python3 generate_manifest.py
```

This created `manifest.json` with:
- SHA256 checksums for all 34 packages
- GitHub CDN URLs
- Package metadata (size, language names, etc.)

### 4. Created Repository
```bash
# Cloned empty repo
git clone https://github.com/lexispot/lexispot-packages.git temp_repo
cd temp_repo

# Added README
echo "# LexiSpot Language Packages" > README.md
git add README.md
git commit -m "Initial commit"
git push origin main
```

### 5. Uploaded Packages
```bash
cd /Users/valerio/Documents/AppsWidgets/LexiSpot
./setup_github_cdn.sh
```

This script:
- Created release v1.0.0
- Uploaded all 34 .sqlite.zst files
- Uploaded manifest.json
- Total upload time: ~10 minutes

---

## What You Have Now

### Free Global CDN
- **Storage**: Unlimited
- **Bandwidth**: Unlimited  
- **Locations**: Global (Fastly CDN)
- **Cost**: $0.00/month forever
- **Uptime**: 99.9% SLA

### 34 Language Packages Available
```
Afrikaans, Albanian, Amharic, Arabic, Armenian, Azerbaijani,
Basque, Belarusian, Bengali, Bosnian, Bulgarian, Catalan,
Cebuano, Chichewa, Chinese (Simplified), Chinese (Traditional),
Corsican, Croatian, Czech, Danish, Dutch, English, Esperanto,
Estonian, Filipino, Finnish, French, Frisian, Galician, Georgian,
German, Greek, Gujarati, Haitian Creole
```

### Public URLs
All packages accessible at:
```
https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/LANGUAGE.sqlite.zst
```

Examples:
- English: `en.sqlite.zst`
- French: `fr.sqlite.zst`
- Spanish: `es.sqlite.zst`

---

## Verification

### Check Release
```bash
gh release view v1.0.0 --repo lexispot/lexispot-packages
```

### Test Manifest URL
```bash
curl -L https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json | jq .
```

Should return JSON with 34 packages.

### Test Package Download
```bash
curl -L -o test.zst https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/en.sqlite.zst
ls -lh test.zst
rm test.zst
```

---

## iOS App Integration

### How It Works

1. **App Launch**: App fetches manifest.json from GitHub
2. **Package List**: Displays 34 available languages
3. **User Downloads**: Taps language → app downloads .zst file
4. **Verification**: App verifies SHA256 checksum
5. **Installation**: Decompresses and installs to local storage
6. **Translation**: All lookups happen offline

### Configuration Location
```swift
// File: LexiSpot/Models/LanguageManifest.swift

struct ManifestConfiguration {
    static let manifestURL = URL(string: 
        "https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json"
    )!
}
```

---

## File Structure

### On GitHub
```
lexispot-packages/
├── README.md (in main branch)
└── Releases/
    └── v1.0.0/
        ├── af.sqlite.zst
        ├── am.sqlite.zst
        ├── ar.sqlite.zst
        ├── ... (31 more)
        └── manifest.json
```

### On Your Mac
```
/Users/valerio/Documents/
├── commands/
│   └── dict-builder/
│       └── packages/
│           ├── *.sqlite (uncompressed)
│           ├── *.sqlite.zst (compressed)
│           └── manifest.json
└── AppsWidgets/
    └── LexiSpot/
        ├── setup_github_cdn.sh
        ├── generate_manifest.py
        ├── add_languages.sh
        └── LexiSpot/ (iOS app)
```

---

## Cost Analysis

| Service | Monthly Cost | Annual Cost |
|---------|-------------|-------------|
| GitHub Repository | $0.00 | $0.00 |
| Storage (Unlimited) | $0.00 | $0.00 |
| Bandwidth (Unlimited) | $0.00 | $0.00 |
| CDN (Fastly) | $0.00 | $0.00 |
| **Total** | **$0.00** | **$0.00** |

### Comparison with Alternatives

| Provider | Storage | Bandwidth | Monthly Cost |
|----------|---------|-----------|--------------|
| **GitHub Releases** | ∞ | ∞ | **$0.00** |
| Cloudflare R2 | 10 GB free | ∞ | $0.00 (then $0.015/GB) |
| AWS S3 | Pay per GB | $0.09/GB | ~$5-50 |
| Google Cloud Storage | Pay per GB | $0.12/GB | ~$10-100 |

---

## Security Features

### Package Verification
- **Checksums**: SHA256 for every package
- **Integrity**: App verifies before installation
- **HTTPS**: All downloads encrypted

### Manifest Signature (Optional)
Current: Placeholder signatures (dev mode)

For production:
```bash
# Generate ED25519 key pair
ssh-keygen -t ed25519 -f lexispot_signing_key

# Sign manifest (implement signing script)
# Update ManifestConfiguration.publicKeyBase64
```

---

## Monitoring

### View Download Stats
```bash
gh release view v1.0.0 --repo lexispot/lexispot-packages
```

Shows download count for each file.

### Check Repository Traffic
1. Go to: https://github.com/lexispot/lexispot-packages
2. Click "Insights" → "Traffic"
3. View visitors and download stats

---

## Next Steps

✅ Initial setup complete!

Now you can:
1. **Test the app** - Download packages and verify they work
2. **Add languages** - See `02-Adding-Languages.md`
3. **Deploy to TestFlight** - Test with real users
4. **Submit to App Store** - Launch LexiSpot!

---

## Backup Information

### Repository Backup
GitHub automatically backs up your repository. But you can also:

```bash
# Clone for local backup
git clone https://github.com/lexispot/lexispot-packages.git backup
cd backup

# Download all release assets
gh release download v1.0.0 --repo lexispot/lexispot-packages
```

### Source Files
Always keep your original SQLite files:
```
/Users/valerio/Documents/commands/dict-builder/packages/*.sqlite
```

These are your source of truth!

---

## Support Resources

- **GitHub CLI Docs**: https://cli.github.com/manual/
- **GitHub Releases**: https://docs.github.com/en/repositories/releasing-projects-on-github
- **Fastly Network Map**: https://www.fastly.com/network-map/

---

**Setup Date**: December 2025  
**Version**: v1.0.0  
**Status**: ✅ Active and Working
