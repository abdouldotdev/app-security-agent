# Security Audit Report — OWASP MASVS v2.1.0

## Audit Metadata

**Project**: {project_name}
**Date**: {date}
**Mode**: {Full Audit | Targeted Check | Single Group}
**Scope**: {all groups | list of audited groups}
**Files audited**: {count}

---

## Summary

| Severity | Count |
|---|---|
| CRITICAL | {n} |
| HIGH | {n} |
| MEDIUM | {n} |
| LOW | {n} |
| INFO | {n} |

**Overall MASVS compliance**: {percentage}%

### Compliance by Group

| MASVS Group | Controls Passing | Score |
|---|---|---|
| MASVS-STORAGE | {n}/{total} | {pass/fail} |
| MASVS-CRYPTO | {n}/{total} | {pass/fail} |
| MASVS-AUTH | {n}/{total} | {pass/fail} |
| MASVS-NETWORK | {n}/{total} | {pass/fail} |
| MASVS-PLATFORM | {n}/{total} | {pass/fail} |
| MASVS-CODE | {n}/{total} | {pass/fail} |
| MASVS-RESILIENCE | {n}/{total} | {pass/fail} |
| MASVS-PRIVACY | {n}/{total} | {pass/fail} |

---

## Top Priority Fixes

1. **{finding_title}** — {severity} — `{file}:{line}` — {one-line description}
2. **{finding_title}** — {severity} — `{file}:{line}` — {one-line description}
3. **{finding_title}** — {severity} — `{file}:{line}` — {one-line description}

---

## Findings

### MASVS-STORAGE

#### Finding S-{n}: {title}

- **Control**: MASVS-STORAGE-{1|2}
- **Severity**: {CRITICAL|HIGH|MEDIUM|LOW|INFO}
- **File**: `{file_path}:{line_number}`

**Issue**:
{Description of the vulnerability or weakness.}

**Problematic code**:
```dart
{exact code snippet}
```

**Recommended fix**:
```dart
{corrected code snippet}
```

**Rationale**: {Why this matters — risk and impact.}

---

{Repeat finding block for each finding in each MASVS group.}
{Only include MASVS group headers that have findings or were audited.}

---

### MASVS-CRYPTO

#### Finding C-{n}: {title}

{Same structure as above.}

---

### MASVS-AUTH

#### Finding A-{n}: {title}

{Same structure as above.}

---

### MASVS-NETWORK

#### Finding N-{n}: {title}

{Same structure as above.}

---

### MASVS-PLATFORM

#### Finding P-{n}: {title}

{Same structure as above.}

---

### MASVS-CODE

#### Finding CD-{n}: {title}

{Same structure as above.}

---

### MASVS-RESILIENCE

#### Finding R-{n}: {title}

{Same structure as above.}

---

### MASVS-PRIVACY

#### Finding PR-{n}: {title}

{Same structure as above.}

---

## Clean Controls

Controls that passed audit with no findings:

- {MASVS-CONTROL-ID}: {brief confirmation of why it passes}

---

## Recommendations

### Immediate Actions (CRITICAL/HIGH)

1. {action with file reference}
2. {action with file reference}

### Short-term Improvements (MEDIUM)

1. {action}
2. {action}

### Long-term Hardening (LOW/INFO)

1. {action}
2. {action}

---

## Human Validation Checklist

Manual checks that require device testing (cannot be verified via static analysis):

- [ ] {check 1 — e.g., Verify app data not visible in device backup}
- [ ] {check 2 — e.g., Test biometric auth failure fallback}
- [ ] {check 3 — e.g., Verify HTTPS in network traffic via proxy}
- [ ] {check 4 — e.g., Test behavior on rooted device}
