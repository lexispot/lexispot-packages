# GitHub CDN Language Package Guide

## 1. Requirements

-   macOS with Homebrew
-   Python (Homebrew)
-   Virtual environment
-   Wiktionary dump: enwiktionary-latest-pages-articles.xml.bz2

## 2. Folder Structure

dict-builder/ build_wiktionary_packages.py batches/ output/ venv/

## 3. Virtual Environment

cd dict-builder /opt/homebrew/bin/python3 -m venv venv source
venv/bin/activate

## 4. Install Dependencies

pip install zstandard lxml tqdm rich

## 5. Download Wiktionary Dump

https://dumps.wikimedia.org/enwiktionary/latest/enwiktionary-latest-pages-articles.xml.bz2
Place into dict-builder/

## 6. Configure Batches

Create batches/batch1.json: { "languages": \["fr", "es", "de"\] }

## 7. Run Builder

python build_wiktionary_packages.py --dump
enwiktionary-latest-pages-articles.xml.bz2 --batch batches/batch1.json
--out output

Outputs: - `<code>`{=html}.zst - `<code>`{=html}.sha256 -
`<code>`{=html}.json

## 8. Validate Results

shasum -a 256 output/fr.zst cat output/fr.sha256

## 9. GitHub as CDN

Use GitHub Releases to host packages: - Create repository:
dict-packages - Create a Release per version - Upload .zst, .sha256,
.json, index.json

Avoid GitHub LFS: - It has bandwidth caps, breaks distribution at scale.

## 10. index.json

Example: { "version": "1.0.0", "packages": \[ { "code": "fr", "name":
"French", "sha256": "...", "size": 123456, "url":
"https://github.com/`<user>`{=html}/`<repo>`{=html}/releases/download/v1.0.0/fr.zst"
} \] }

## 11. App Integration

The iOS app: - Downloads index.json - Displays available languages -
Downloads packages via background tasks - Verifies SHA-256 -
Decompresses into Application Support - Handles deletion and updates

## 12. Maintenance

To update languages: - Rebuild affected packages - Increment version in
index.json - Publish new GitHub Release - Keep older Releases tagged for
rollback

## 13. Checklist

✓ venv active\
✓ dependencies installed\
✓ batches ready\
✓ packages built\
✓ hashes verified\
✓ index.json updated\
✓ Release published\
✓ App connected to new URLs
