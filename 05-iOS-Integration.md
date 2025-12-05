# 05 - iOS Integration Guide

## How LexiSpot Connects to the CDN

This guide explains how your iOS app integrates with the GitHub CDN to download and manage language packages.

---

## Architecture Overview

```
User Device (iPhone/iPad)
         ↓
    LexiSpot App
         ↓
    [Fetches manifest.json from GitHub]
         ↓
    GitHub CDN (Fastly)
         ↓
    [Downloads .zst packages]
         ↓
    Local Storage (App Sandbox)
```

---

## Key Components

### 1. ManifestConfiguration

**Location**: `LexiSpot/Models/LanguageManifest.swift`

```swift
struct ManifestConfiguration {
    // Primary manifest URL
    static let manifestURL = URL(string: 
        "https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json"
    )!
    
    // Fallback URL (same as primary for now)
    static let fallbackManifestURL = URL(string: 
        "https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json"
    )!
    
    // Public key for signature verification (dev mode: not enforced)
    static let publicKeyBase64 = "MCowBQYDK2VwAyEAYourED25519PublicKeyHere="
    
    // Update checking interval
    static let refreshInterval: TimeInterval = 3600 // 1 hour
}
```

### 2. DownloadManager

**Location**: `LexiSpot/Services/DownloadManager.swift`

**Responsibilities**:
- Fetches manifest.json on app launch
- Manages package downloads
- Verifies SHA256 checksums
- Handles background downloads
- Sends notifications on completion

**Key Methods**:
```swift
// Fetch manifest
func fetchManifest() async throws

// Start download
func startDownload(for package: LanguagePackage) async throws

// Pause/Resume
func pauseDownload(for packageId: String) async
func resumeDownload(for packageId: String) async

// Delete
func deletePackage(_ languageCode: String) async throws
```

### 3. PackageInstaller

**Location**: `LexiSpot/Services/PackageInstaller.swift`

**Responsibilities**:
- Decompresses .zst files
- Validates SQLite database
- Installs to App Group storage
- Manages installed package registry

### 4. Translator

**Location**: `LexiSpot/Services/Translator.swift`

**Responsibilities**:
- Loads installed databases
- Performs offline lookups
- Manages in-memory indexes
- Returns translation results

---

## Data Flow

### App Launch Sequence

1. **Initialize DownloadManager**
   ```swift
   // In TranslatorApp.swift
   .task {
       do {
           try await downloadManager.initialize()
       } catch {
           alertCenter.post(.error("Initialization Failed", message: error.localizedDescription))
       }
   }
   ```

2. **Load Cached Manifest**
   - Reads from: `Application Support/manifest_cache.json`
   - Displays available packages immediately

3. **Fetch Fresh Manifest**
   - Downloads from GitHub
   - Verifies signature (dev mode: skipped)
   - Compares with cached version
   - Updates available packages list

4. **Check for Updates**
   - Compares installed package versions
   - Shows "Update Available" badge if newer version exists

### Download Sequence

1. **User Taps Download**
   ```swift
   // In LanguagesView.swift
   Button("Download") {
       Task {
           try await downloadManager.startDownload(for: package)
       }
   }
   ```

2. **Verify Storage Space**
   - Checks available disk space
   - Requires 2x package size (for download + extraction)

3. **Background Download**
   - Uses `URLSession.background`
   - Downloads continue even if app is closed
   - Progress updates via delegate

4. **Verify Checksum**
   ```swift
   let downloadedData = try Data(contentsOf: downloadLocation)
   let checksum = SHA256.hash(data: downloadedData)
       .compactMap { String(format: "%02x", $0) }
       .joined()
   
   guard checksum == package.checksum else {
       throw DownloadManagerError.checksumMismatch
   }
   ```

5. **Decompress**
   ```swift
   // Zstandard decompression in background queue
   let decompressed = try decompress(zstFile)
   ```

6. **Install**
   - Validates SQLite schema
   - Moves to: `Application Support/Languages/LANG.sqlite`
   - Updates installed packages registry

7. **Index for Spotlight** (optional)
   - Creates CoreSpotlight index
   - Enables system-wide search

8. **Notify User**
   - Local notification: "Download Complete"
   - Updates UI to show "Installed"

### Translation Sequence

1. **User Enters Query**
   ```swift
   // In SearchView.swift
   let results = await translator.translate(term: query)
   ```

2. **Lazy Load Databases**
   - Opens SQLite connections on first use
   - Keeps in memory for session

3. **Execute FTS5 Query**
   ```sql
   SELECT term, translations, pos, examples, frequency
   FROM entries_fts
   WHERE entries_fts MATCH ?
   ORDER BY frequency DESC
   LIMIT 20
   ```

4. **Parse Results**
   - JSON decode translations
   - Build `TranslationEntry` objects

5. **Display to User**
   - Shows in SearchView
   - Highlights matched terms
   - Shows part of speech, examples

---

## Storage Locations

### App Group Storage

**Container**: `group.translator`

```
Application Support/
├── manifest_cache.json           # Cached manifest
├── installed_packages.json       # Registry of installed packages
├── download_state.json           # Download progress/state
└── Languages/                    # Installed packages
    ├── en.sqlite
    ├── fr.sqlite
    ├── es.sqlite
    └── ...
```

### Temporary Downloads

```
tmp/
└── Downloads/
    └── PACKAGE.sqlite.zst (temporary until verified)
```

---

## Network Handling

### Manifest Fetching

```swift
// In DownloadManager.swift
func fetchManifest() async throws {
    let (data, response) = try await session.data(from: ManifestConfiguration.manifestURL)
    
    guard let httpResponse = response as? HTTPURLResponse,
          httpResponse.statusCode == 200 else {
        throw DownloadManagerError.manifestFetchFailed
    }
    
    let manifest = try JSONDecoder().decode(LanguagePackageManifest.self, from: data)
    
    // Verify signature (dev mode: skipped)
    guard await verifyManifestSignature(data: data, signature: manifest.signature) else {
        throw DownloadManagerError.signatureInvalid
    }
    
    self.manifest = manifest
}
```

### Background Downloads

```swift
// Background session configuration
let config = URLSessionConfiguration.background(withIdentifier: "com.lexispot.downloads")
config.isDiscretionary = false
config.sessionSendsLaunchEvents = true
config.allowsCellularAccess = true

backgroundSession = URLSession(configuration: config, delegate: self, delegateQueue: nil)
```

### Network Monitoring

```swift
// Monitor network changes
let networkMonitor = NWPathMonitor()
networkMonitor.pathUpdateHandler = { path in
    if path.status == .satisfied {
        // Network available - resume downloads
        self.resumeAllPausedDownloads()
    } else {
        // Network lost - pause downloads
        self.pauseAllDownloads()
    }
}
```

---

## Error Handling

### Error Types

```swift
enum DownloadManagerError: Error {
    case noNetwork
    case downloadFailed(String)
    case checksumMismatch
    case signatureInvalid
    case installationFailed(String)
    case manifestFetchFailed
    case lowStorage
    case cancelled
}
```

### Error Recovery

**Network Errors**:
- Auto-pause downloads
- Resume when network returns
- Show notification to user

**CDN Failures**:
- Retry with exponential backoff
- Try fallback URL
- Alert user if persistent

**Checksum Mismatch**:
- Delete downloaded file
- Retry download
- Alert user after 3 failures

**Low Storage**:
- Calculate required space
- Show alert with cleanup options
- Don't start download

---

## Update Mechanism

### Automatic Updates

```swift
// Check for updates every hour
func checkForPackageUpdates() async {
    let installed = await Translator.shared.getInstalledPackages()
    
    for installedPkg in installed {
        if let available = availablePackages.first(where: { 
            $0.languageCode == installedPkg.languageCode 
        }),
        available.version != installedPkg.version {
            // New version available
            updateDownloadState(
                for: available, 
                state: .updateAvailable(newVersion: available.version)
            )
        }
    }
}
```

### User-Initiated Updates

```swift
// User taps "Update" button
Button("Update") {
    Task {
        try await downloadManager.startDownload(for: newVersion)
        // Old version replaced after successful install
    }
}
```

---

## Performance Optimizations

### Lazy Loading

- Databases loaded only when needed
- Connections kept open during session
- Closed when app backgrounds

### Caching

- Manifest cached locally
- Only fetch when expired or forced
- Translation results not cached (small memory footprint)

### Background Processing

- Downloads continue in background
- Decompression in background queue
- Spotlight indexing in background

### Memory Management

- Use `@MainActor` for UI updates
- Background queues for heavy work
- Close database connections when not needed

---

## Security Features

### HTTPS Only

```swift
// All URLs use HTTPS
static let manifestURL = URL(string: "https://github.com/...")!
```

### Checksum Verification

```swift
// SHA256 for every package
let checksum = SHA256.hash(data: fileData)
    .compactMap { String(format: "%02x", $0) }
    .joined()

guard checksum == expectedChecksum else {
    throw DownloadManagerError.checksumMismatch
}
```

### Signature Verification (Optional)

```swift
// ED25519 signature verification
func verifyManifestSignature(data: Data, signature: String) async -> Bool {
    guard let signatureData = Data(base64Encoded: signature),
          let publicKeyData = Data(base64Encoded: ManifestConfiguration.publicKeyBase64) else {
        return false
    }
    
    let publicKey = try Curve25519.Signing.PublicKey(rawRepresentation: publicKeyData)
    return publicKey.isValidSignature(signatureData, for: data)
}
```

**Note**: In development, signature verification is skipped if it fails. For production, implement proper signing.

---

## UI Integration

### Languages View

Displays:
- Available packages from manifest
- Download status (not downloaded, downloading, installed)
- Progress bars
- Update badges
- Storage requirements

### Search View

- Real-time translation lookup
- Searches all installed languages
- Shows results grouped by language

### Downloads View

- Active downloads with progress
- Pause/Resume controls
- Cancel option
- Speed and ETA display

### Settings View

- Manage installed packages
- Delete packages to free space
- Check for updates
- View storage usage

---

## Testing

### Test Manifest Fetching

```swift
// In unit tests
func testManifestFetching() async throws {
    let manager = DownloadManager.shared
    try await manager.fetchManifest()
    
    XCTAssertNotNil(manager.manifest)
    XCTAssertGreaterThan(manager.availablePackages.count, 0)
}
```

### Test Download

```swift
func testPackageDownload() async throws {
    let package = // ... get test package
    try await downloadManager.startDownload(for: package)
    
    // Wait for completion
    // Verify file exists
    // Verify checksum
}
```

### Test Translation

```swift
func testTranslation() async throws {
    let results = await translator.translate(term: "hello")
    XCTAssertGreaterThan(results.count, 0)
}
```

---

## Debugging

### Enable Logging

Add print statements:
```swift
print("Fetching manifest from: \(ManifestConfiguration.manifestURL)")
print("Download progress: \(progress)")
print("Installed packages: \(installedPackages.count)")
```

### Check Network Requests

- Use Xcode Network Debugger
- Charles Proxy for detailed inspection

### Verify Storage

```swift
// Print storage paths
if let containerURL = AppGroupConfiguration.sharedContainerURL {
    print("Container: \(containerURL)")
    print("Packages: \(containerURL.appendingPathComponent("Languages"))")
}
```

### Test URLs Manually

```bash
# Test manifest
curl -v https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/manifest.json

# Test package
curl -v https://github.com/lexispot/lexispot-packages/releases/download/v1.0.0/en.sqlite.zst
```

---

## Configuration Changes

### Change CDN URL

Edit `ManifestConfiguration.swift`:
```swift
static let manifestURL = URL(string: "https://NEW_CDN_URL/manifest.json")!
```

### Change App Group

1. Update in Xcode: Signing & Capabilities
2. Update in code:
   ```swift
   struct AppGroupConfiguration {
       static let groupIdentifier = "group.NEW_GROUP_ID"
   }
   ```

### Change Update Interval

```swift
static let refreshInterval: TimeInterval = 3600 // seconds
```

---

## Production Checklist

Before App Store submission:

- [ ] Implement proper ED25519 signing
- [ ] Update `publicKeyBase64` with real key
- [ ] Test all downloads thoroughly
- [ ] Test offline mode
- [ ] Test update mechanism
- [ ] Test on various iOS versions
- [ ] Test on various device sizes
- [ ] Verify error messages are user-friendly
- [ ] Check that all URLs are HTTPS
- [ ] Verify no hardcoded test data
- [ ] Test with slow/unstable network
- [ ] Test with low storage

---

**Related Guides**:
- `01-Initial-Setup.md` - CDN setup
- `02-Adding-Languages.md` - Adding languages
- `04-Troubleshooting.md` - Fixing issues
