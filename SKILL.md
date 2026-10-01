---
name: app-security-agent
description: |
  OWASP MASVS v2.1.0 security audit for Flutter mobile applications.
  Scans code against all 8 MASVS control groups (24 controls), produces
  actionable findings with severity ratings and fix guidance.

  Supports two modes:
  - Full project audit (standalone invocation)
  - Targeted file check (invoked by flutter-bug-fix STEP 6)

  Triggers: "security audit", "owasp mobile", "masvs", "app security", "mobile security"
---

# App Security Agent

OWASP MASVS v2.1.0 security audit pipeline for Flutter Clean Architecture projects.

## Dependencies

This skill references other Flutter skills:
- **flutter-best-practice**: Defensive coding rules and secure defaults (read-only reference)
- **flutter-clean-arch**: Architecture boundary validation for security checks

## OWASP MASVS v2.1.0 Control Map

| Group | Controls | Flutter Focus |
|---|---|---|
| MASVS-STORAGE | 1-2 | flutter_secure_storage, SharedPreferences usage, temp files, logs |
| MASVS-CRYPTO | 1-2 | Encryption algorithms, key management, hardcoded secrets |
| MASVS-AUTH | 1-3 | Supabase auth, local auth (biometric), session management |
| MASVS-NETWORK | 1-2 | HTTPS enforcement, certificate pinning, API security |
| MASVS-PLATFORM | 1-3 | Deep links, WebView config, UI security (tap jacking, screenshots) |
| MASVS-CODE | 1-4 | Min SDK, forced updates, dependency vulns, input validation |
| MASVS-RESILIENCE | 1-4 | Root/jailbreak detection, anti-tampering, obfuscation |
| MASVS-PRIVACY | 1-4 | Data minimization, tracking, transparency, user data control |

---

## Execution Flow

**MANDATORY: Follow steps in order. Do NOT skip steps.**

Use TodoWrite to track each step as a task. Only ONE step is in_progress at a time.

---

### STEP 1: SCOPE DETERMINATION

**Mode A — Full project audit** (standalone invocation):
1. Locate project root (look for `pubspec.yaml`)
2. Scan `pubspec.yaml` for dependencies (flutter_secure_storage, local_auth, http, dio, supabase_flutter, etc.)
3. Identify all feature modules in `lib/features/`
4. Catalog data/network/auth related files across the project
5. Set scope = all MASVS control groups

**Mode B — Targeted file check** (invoked by flutter-bug-fix):
1. Receive list of modified files as input
2. Determine which MASVS groups are relevant based on file content:
   - Storage operations → MASVS-STORAGE
   - Crypto/token handling → MASVS-CRYPTO
   - Auth/session code → MASVS-AUTH
   - Network/API calls → MASVS-NETWORK
   - Platform channels, WebViews, deep links → MASVS-PLATFORM
   - Dependencies, input handling → MASVS-CODE
   - Any new code → MASVS-CODE-4 (input validation) always applies
3. Set scope = only relevant control groups

**Output**: Scope defined with file inventory and applicable MASVS groups.

---

### STEP 2: MASVS-STORAGE — Secure Data Storage

Audit files for storage security violations.

**MASVS-STORAGE-1: App securely stores sensitive data**

Search patterns:
```dart
// CRITICAL: Sensitive data in insecure storage
SharedPreferences  // check what's being stored
getSharedPreferences
prefs.setString('token'     // FINDING: token in plaintext storage
prefs.setString('password'  // FINDING: password in plaintext storage
prefs.setString('secret'    // FINDING: secret in plaintext storage
prefs.setString('key'       // FINDING: potential key in plaintext storage
prefs.setString('session'   // FINDING: session in plaintext storage
prefs.setString('credit'    // FINDING: financial data in plaintext storage
prefs.setString('card'      // FINDING: card data in plaintext storage

// EXPECTED: Sensitive data in secure storage
FlutterSecureStorage
secureStorage.write
secureStorage.read
```

Checks:
- [ ] Tokens, credentials, PII stored with `FlutterSecureStorage` only
- [ ] `SharedPreferences` used ONLY for non-sensitive settings (theme, locale, onboarding flags)
- [ ] No sensitive data in `GetStorage` or `Hive` without encryption
- [ ] SQLite/Drift databases with sensitive data use SQLCipher or equivalent encryption

**MASVS-STORAGE-2: App prevents leakage of sensitive data**

Search patterns:
```dart
// CRITICAL: Data leakage vectors
print(         // check for sensitive data in prints
debugPrint(    // check for sensitive data
log(           // check for tokens/passwords in logs
Logger(        // check logger output
developer.log( // check developer logs
clipboard      // sensitive data to clipboard
Clipboard.setData  // check what's being copied
```

Checks:
- [ ] No `print()` / `debugPrint()` / `log()` with tokens, passwords, PII, or full API responses
- [ ] Logs are disabled or sanitized in release builds
- [ ] No sensitive data copied to clipboard without warning
- [ ] Keyboard cache disabled for sensitive inputs (`enableSuggestions: false, autocorrect: false`)
- [ ] Screenshot protection on sensitive screens (if applicable)
- [ ] Backup rules configured to exclude sensitive files (Android `allowBackup: false` or backup rules)

**If Mode B and MASVS-STORAGE not in scope: SKIP to next applicable group.**

---

### STEP 3: MASVS-CRYPTO — Cryptography

**MASVS-CRYPTO-1: App employs current strong cryptography**

Search patterns:
```dart
// CRITICAL: Weak/deprecated crypto
MD5            // FINDING: MD5 is broken
SHA1           // WARNING: SHA1 deprecated for security
DES            // FINDING: DES is broken
RC4            // FINDING: RC4 is broken
ECB            // FINDING: ECB mode is insecure
encrypt(       // check algorithm used
decrypt(       // check algorithm used
AES            // verify mode (CBC/GCM) and IV handling

// CHECK: Hardcoded secrets
'-----BEGIN'   // hardcoded certificates/keys
apiKey =       // hardcoded API key
secretKey =    // hardcoded secret
const.*key.*=  // hardcoded key values
password.*=.*' // hardcoded password
```

Checks:
- [ ] No MD5, SHA1, DES, RC4, 3DES, or ECB mode used for security purposes
- [ ] AES uses GCM or CBC with HMAC (not ECB)
- [ ] RSA key size >= 2048 bits
- [ ] No hardcoded encryption keys, API keys, or secrets in source code
- [ ] Cryptographic operations use well-maintained libraries (pointycastle, cryptography)

**MASVS-CRYPTO-2: App performs key management according to industry best practices**

Checks:
- [ ] Encryption keys not stored alongside encrypted data
- [ ] Keys stored in platform keystore (iOS Keychain / Android Keystore) via FlutterSecureStorage
- [ ] No key derivation from easily guessable values (user ID, device ID alone)
- [ ] Key rotation mechanism exists for long-lived keys

---

### STEP 4: MASVS-AUTH — Authentication & Authorization

**MASVS-AUTH-1: Secure authentication and authorization protocols**

Search patterns:
```dart
// CHECK: Auth implementation
supabase.auth.signIn
supabase.auth.signUp
signInWithPassword
signInWithOAuth
signInWithOtp
getSession
currentSession
accessToken
refreshToken
// CHECK: Token handling
jwt            // JWT handling
bearer         // Bearer token usage
Authorization  // Auth headers
```

Checks:
- [ ] Authentication uses Supabase Auth (or equivalent proven auth provider), not custom implementation
- [ ] Tokens stored in `FlutterSecureStorage`, not `SharedPreferences`
- [ ] Session refresh logic exists (handle expired tokens gracefully)
- [ ] Logout clears ALL sensitive data (tokens, cached user data, secure storage)
- [ ] Auth state changes handled globally (redirect to login on 401/session expiry)

**MASVS-AUTH-2: Secure local authentication**

Search patterns:
```dart
// CHECK: Biometric / local auth
local_auth
LocalAuthentication
authenticate(
biometricOnly
canCheckBiometrics
```

Checks:
- [ ] If local auth used: `local_auth` package with proper error handling
- [ ] Biometric auth used as second factor, not sole authentication
- [ ] Fallback mechanism exists (PIN/password) if biometric fails
- [ ] `authenticateWithBiometrics` uses `biometricOnly: false` for proper fallback

**MASVS-AUTH-3: Additional authentication for sensitive operations**

Checks:
- [ ] Payment/financial operations require re-authentication or step-up auth
- [ ] Account deletion requires password confirmation
- [ ] Sensitive data export requires authentication
- [ ] Session timeout configured for sensitive operations

---

### STEP 5: MASVS-NETWORK — Network Communication

**MASVS-NETWORK-1: Secure network traffic**

Search patterns:
```dart
// CRITICAL: Insecure network
http://        // non-HTTPS URLs (except localhost/10.0.2.2)
allowBadCertificates
badCertificateCallback
SecurityContext(withTrustedRoots: false)

// CHECK: Network client config
Dio(
HttpClient(
http.Client(
Supabase.initialize(url:
```

Checks:
- [ ] ALL remote URLs use HTTPS (no plaintext HTTP except localhost for debug)
- [ ] No `badCertificateCallback` returning `true` in production code
- [ ] Network security config restricts cleartext traffic (Android `network_security_config.xml`)
- [ ] iOS App Transport Security (ATS) not globally disabled in `Info.plist`
- [ ] API base URLs in constants/env, not hardcoded across files

**MASVS-NETWORK-2: Identity pinning**

Checks:
- [ ] Certificate or public key pinning implemented for critical API endpoints
- [ ] Pinning failures trigger secure fallback (not silent bypass)
- [ ] Pinning configuration supports key rotation

---

### STEP 6: MASVS-PLATFORM — Platform Interaction

**MASVS-PLATFORM-1: Secure IPC mechanisms**

Search patterns:
```dart
// CHECK: Deep links / URL schemes
uni_links
go_router       // route guards for deep links
DeepLink
getInitialUri
uriLinkStream
intent-filter   // Android manifest
CFBundleURLSchemes  // iOS Info.plist
```

Checks:
- [ ] Deep link handlers validate and sanitize incoming URLs
- [ ] Deep links to sensitive screens require authentication check
- [ ] No sensitive data passed via URL parameters
- [ ] Android intent filters use `android:autoVerify="true"` for App Links
- [ ] iOS Universal Links properly configured (not just custom URL schemes)

**MASVS-PLATFORM-2: Secure WebViews**

Search patterns:
```dart
// CHECK: WebView usage
WebView(
WebViewController
InAppWebView
javascriptMode: JavascriptMode.unrestricted  // WARNING
javaScriptEnabled: true
WebViewPlatform
```

Checks:
- [ ] JavaScript disabled in WebViews unless explicitly required
- [ ] WebViews restrict navigation to trusted domains only
- [ ] No JavaScript bridges exposing sensitive native functionality
- [ ] File access disabled in WebViews (`allowFileAccess: false`)
- [ ] WebView content loaded over HTTPS only

**MASVS-PLATFORM-3: Secure UI**

Checks:
- [ ] Sensitive screens prevent screenshot/screen recording where appropriate
- [ ] No sensitive data in system notifications visible on lock screen
- [ ] Pasteboard cleared after timeout for sensitive copied data
- [ ] UI does not display sensitive data in plaintext by default (e.g., password fields masked)

---

### STEP 7: MASVS-CODE — Code Quality

**MASVS-CODE-1: Up-to-date platform version**

Checks:
- [ ] `minSdkVersion` >= 23 (Android 6.0) in `android/app/build.gradle`
- [ ] iOS minimum deployment target >= 13.0 in `ios/Podfile` and Xcode project
- [ ] Flutter SDK version in `pubspec.yaml` is current stable or n-1

**MASVS-CODE-2: App updates mechanism**

Checks:
- [ ] Force-update mechanism exists for critical security updates
- [ ] Version check against remote config or API on app start
- [ ] Graceful handling when update is required (block usage, show update prompt)

**MASVS-CODE-3: No known vulnerabilities in dependencies**

```bash
# Run dependency audit
flutter pub outdated
flutter pub deps
```

Checks:
- [ ] No dependencies with known CVEs (check pub.dev advisories)
- [ ] All dependencies on recent versions (check `flutter pub outdated`)
- [ ] No abandoned packages (check last publish date on pub.dev)
- [ ] Dependency overrides justified and documented in `pubspec.yaml`

**MASVS-CODE-4: Input validation and sanitization**

Search patterns:
```dart
// CHECK: Input handling
TextEditingController  // verify validation exists
TextFormField(         // check validator: param
validator:             // check validation logic
onSubmitted:           // check if validated before use
fromJson(              // check JSON parsing defensiveness
```

Checks:
- [ ] All user inputs validated before use (email, phone, amounts, free text)
- [ ] Server-side validation not solely relied upon (client-side defense in depth)
- [ ] JSON parsing uses defensive patterns (null checks, type checks, defaults)
- [ ] No string interpolation of user input into queries (SQL injection via Supabase RPC)
- [ ] File uploads validate type, size, and content before processing

---

### STEP 8: MASVS-RESILIENCE — Resilience Against Reverse Engineering

**Note**: MASVS-RESILIENCE is defense-in-depth. Applicability depends on the app's threat model. For financial/healthcare/enterprise apps, these controls are important. For simple utility apps, they may be less critical.

**MASVS-RESILIENCE-1: Platform integrity validation**

Checks:
- [ ] Root/jailbreak detection implemented (consider: flutter_jailbreak_detection, safe_device)
- [ ] App behavior on rooted/jailbroken device defined (warn, restrict, or block)
- [ ] Emulator detection for production builds (if threat model requires it)

**MASVS-RESILIENCE-2: Anti-tampering**

Checks:
- [ ] App signature verification implemented
- [ ] Code integrity checks present (hash verification of critical code)
- [ ] Tampering detection triggers appropriate response (not just logging)

**MASVS-RESILIENCE-3: Anti-static analysis**

Checks:
- [ ] Code obfuscation enabled for release builds (`--obfuscate --split-debug-info` in build command)
- [ ] Debug symbols stripped in release builds
- [ ] No sensitive strings in plaintext (use encrypted string storage or obfuscation)

**MASVS-RESILIENCE-4: Anti-dynamic analysis**

Checks:
- [ ] Debugger detection in release builds
- [ ] Frida/instrumentation detection (if threat model requires it)
- [ ] Release builds have `debuggable: false` (Android) and debug entitlements removed (iOS)

---

### STEP 9: MASVS-PRIVACY — User Privacy

**MASVS-PRIVACY-1: Data minimization**

Search patterns:
```dart
// CHECK: Permissions requested
INTERNET
CAMERA
READ_EXTERNAL_STORAGE
WRITE_EXTERNAL_STORAGE
ACCESS_FINE_LOCATION
ACCESS_COARSE_LOCATION
READ_CONTACTS
RECORD_AUDIO
// Check AndroidManifest.xml and Info.plist
```

Checks:
- [ ] Only permissions required by actual features are requested
- [ ] Permissions requested at point of use, not on app startup
- [ ] No excessive data collection (only collect what the feature needs)
- [ ] Temporary data cleaned up after use (cache, temp files)

**MASVS-PRIVACY-2: Prevent user identification**

Checks:
- [ ] No unique device identifiers sent to servers unless strictly necessary
- [ ] Analytics anonymized or use privacy-respecting identifiers
- [ ] No fingerprinting techniques used without consent

**MASVS-PRIVACY-3: Transparency about data collection**

Checks:
- [ ] Privacy policy accessible from within the app
- [ ] Data collection purposes disclosed to the user
- [ ] Third-party SDKs and their data practices documented

**MASVS-PRIVACY-4: User control over their data**

Checks:
- [ ] Account deletion functionality available
- [ ] User can export their data
- [ ] Users can opt out of non-essential data collection
- [ ] Consent mechanism for analytics/tracking

---

### STEP 10: FINDINGS COMPILATION & REPORT

1. Aggregate all findings from Steps 2-9
2. Classify each finding by severity:

| Severity | Criteria | Response |
|---|---|---|
| **CRITICAL** | Exploitable vulnerability, data breach risk | Must fix before release |
| **HIGH** | Security weakness with direct impact | Fix in current sprint |
| **MEDIUM** | Security improvement, defense-in-depth gap | Plan fix in next sprint |
| **LOW** | Best practice deviation, minor hardening | Track for improvement |
| **INFO** | Observation, recommendation | Optional improvement |

3. For each finding, provide:
   - MASVS control reference (e.g., MASVS-STORAGE-1)
   - File path and line number
   - Description of the issue
   - Code snippet showing the problem
   - Recommended fix with code example
   - Severity rating with justification

4. Generate the report using [assets/security-report-template.md](assets/security-report-template.md)

5. Provide summary statistics:
   - Total findings by severity
   - MASVS compliance score per group (% of controls passing)
   - Overall compliance percentage
   - Top 3 priority fixes

**Output**: Complete security audit report displayed to user.

**After the report is fully displayed, output the completion marker on its own line:**

```
--- SECURITY_AUDIT_COMPLETE ---
```

This marker guarantees ALL applicable MASVS groups were audited, all findings documented with severity and fix guidance. NEVER output this marker if the audit was incomplete.

---

## Error Handling

| Situation | Action |
|---|---|
| `pubspec.yaml` not found | Ask user for project root path |
| No Flutter project detected | Abort with message, this skill is Flutter-specific |
| Dependency audit fails | Note limitation, continue with static analysis |
| File access denied | Skip file, note in report as unauditable |
| Targeted mode with no security-relevant files | Return "No security-relevant changes detected" |
| flutter-best-practice not installed | Use built-in checks from this skill |

## Commands

### Default (no subcommand)
Full project audit: STEP 1 through STEP 10 with all MASVS groups.

### `check <file1> [file2] [...]`
Targeted file check (Mode B). Only audits MASVS groups relevant to the provided files.
Used by flutter-bug-fix STEP 6.
Example: `app-security-agent check lib/features/auth/data/datasources/auth_remote.dart`

### `group <MASVS-GROUP>`
Audit a single MASVS group across the entire project.
Example: `app-security-agent group MASVS-STORAGE`

### `quick`
Abbreviated audit: only CRITICAL and HIGH severity checks. Skips RESILIENCE and PRIVACY groups.
Faster for development-time checks.
