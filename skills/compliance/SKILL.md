---
name: compliance
description: "Regulatory compliance — HIPAA, SOC 2, GDPR, PCI DSS. Code-level enforcement, audit logging, data privacy, compliance-as-code. Use for HIPAA PHI handling, SOC 2 controls, GDPR data subject rights, PCI cardholder data, compliance audit, regulatory requirements, data privacy implementation. Do NOT use for general security scanning (use security)."
---

# Compliance

Implement, audit, and enforce regulatory compliance at the code and infrastructure level. Every framework gets concrete controls — not just checkboxes but actual implementation patterns.

## Framework Quick Reference

| Framework | Scope | Key Concern | Penalty |
|-----------|-------|-------------|---------|
| **HIPAA** | Healthcare / PHI | Protected health information | $100–$50K per violation |
| **SOC 2** | SaaS / service providers | Trust service criteria | Loss of enterprise deals |
| **GDPR** | EU personal data | Data subject rights | 4% global annual revenue |
| **PCI DSS** | Payment card data | Cardholder data protection | Fines + loss of processing |

## HIPAA — Healthcare Data

### Protected Health Information (PHI) Rules
PHI = any individually identifiable health information. The 18 HIPAA identifiers include names, dates, SSNs, medical record numbers, device IDs, photos, biometrics.

### Minimum Necessary Standard
Only access/expose the minimum PHI required for the specific function. Every query, API response, and log entry must justify what PHI it includes.

### Technical Safeguards
```
Encryption at rest:      AES-256 for databases, object storage, backups
Encryption in transit:   TLS 1.2+ for all connections (no exceptions)
Access controls:         RBAC with role-per-function, MFA for PHI access
Audit logging:           Every PHI access logged (who, what, when, why)
Automatic log-off:       Session timeout for PHI-accessing applications
Integrity controls:      Checksums on PHI records, tamper-evident logging
```

### BAA Requirements
Every vendor/subprocessor touching PHI must have a signed Business Associate Agreement **before** any data flows. Track BAAs in a register with renewal dates.

### Breach Notification
- **Individual notice**: Within 60 days of discovery
- **HHS notice**: Within 60 days (if 500+ individuals) or annual log (if fewer)
- **Media notice**: If 500+ individuals in a single state

### Code-Level Enforcement
```typescript
// NEVER log PHI
logger.info("Patient lookup", { patientId: hash(id), action: "view" }); // ✓
logger.info("Patient lookup", { name: patient.name, ssn: patient.ssn }); // ✗ VIOLATION

// Field-level encryption for PHI at rest
const encrypted = await encrypt(patient.diagnosis, phiEncryptionKey);
await db.insert(records).values({ id, diagnosis: encrypted, ...metadata });

// PHI access audit trail
await auditLog.record({
  actor: currentUser.id,
  action: "VIEW_RECORD",
  resource: `patient/${patientId}`,
  justification: "treatment",
  timestamp: new Date().toISOString(),
});
```

## SOC 2 — Trust Service Criteria

### The Five Pillars
| Criteria | Focus | Key Controls |
|----------|-------|-------------|
| **Security** | Protection against unauthorized access | Firewalls, IDS, MFA, encryption |
| **Availability** | System uptime and performance | SLAs, DR plan, monitoring, backups |
| **Processing Integrity** | Accurate and complete processing | Input validation, reconciliation, QA |
| **Confidentiality** | Protection of confidential info | Encryption, access controls, DLP |
| **Privacy** | Personal information handling | Consent, retention, data subject rights |

### Evidence Collection Pattern
```
Control: "Access to production requires MFA"
Evidence:
  - Screenshot: IdP MFA policy configuration
  - Log sample: Auth events showing MFA challenge
  - Policy doc: "Production Access Policy v2.3"
  - Test: Attempt prod access without MFA → denied
```

### Continuous Monitoring
```yaml
# Automated SOC 2 evidence collection
controls:
  CC6.1-access-control:
    check: "All prod IAM roles require MFA"
    tool: aws-config-rule
    frequency: daily
    alert_on_drift: true

  CC7.2-monitoring:
    check: "Security alerts triaged within 24h"
    tool: pagerduty-sla-report
    frequency: weekly
    evidence_bucket: s3://soc2-evidence/cc7.2/
```

### Control Mapping to Code
```typescript
// CC6.1 — Logical access controls
const authMiddleware = requireAuth({ mfa: true, roles: ["admin", "engineer"] });

// CC6.6 — System boundary protection
const rateLimiter = rateLimit({ windowMs: 60_000, max: 100 });

// CC7.1 — Detect anomalies
if (loginAttempts > 5) {
  await alertSecurityTeam({ event: "brute_force", ip, userId });
  await lockAccount(userId, { duration: "30m" });
}

// CC8.1 — Change management
// Enforce: PR required, 1+ approval, CI passing, no direct push to main
```

## GDPR — EU Data Privacy

### Data Subject Rights (implement all)
1. **Right of access** — Export all personal data on request (30 days)
2. **Right to rectification** — Allow users to correct their data
3. **Right to erasure** — Delete on request (with lawful exceptions)
4. **Right to portability** — Machine-readable export (JSON/CSV)
5. **Right to restrict processing** — Pause processing, retain data
6. **Right to object** — Opt out of profiling and marketing

### Consent Management
```typescript
// Track consent with full audit trail
interface ConsentRecord {
  userId: string;
  purpose: "marketing" | "analytics" | "personalization";
  granted: boolean;
  timestamp: string;       // ISO 8601
  method: "checkbox" | "settings_page" | "api";
  version: string;         // consent text version
  ip: string;              // for proof of consent
}
```

### Data Retention
```yaml
retention_policy:
  user_accounts:
    active: "until deletion request"
    after_deletion: "30 days (soft delete) → hard delete"
  payment_records:
    retention: "7 years (tax law)"
    anonymize_after: "7 years"
  access_logs:
    retention: "90 days"
    purpose: "security monitoring"
  marketing_data:
    retention: "until consent withdrawn"
```

### DPA Requirements
Every processor/sub-processor needs a Data Processing Agreement covering: processing purposes, data categories, retention, security measures, sub-processor list, breach notification (72 hours).

## PCI DSS — Payment Card Data

### Cardholder Data Environment (CDE)
**Never** store full card numbers, CVV, or magnetic stripe data. Use tokenization (Stripe, Braintree) to keep card data off your servers entirely.

### Network Segmentation
```
Internet → WAF → Load Balancer → App Servers (no card data)
                                        ↓
                                  Payment API (tokenized)
                                        ↓
                                  Stripe/Processor (has card data)
```

### Key Requirements
- Firewall between CDE and other networks
- No default vendor passwords
- Encrypt cardholder data in transit (TLS 1.2+)
- Restrict access on need-to-know basis
- Quarterly vulnerability scans (ASV)
- Annual penetration testing

## Code-Level Enforcement

### What to Log
```
✓ Authentication events (login, logout, MFA, failed attempts)
✓ Authorization decisions (access granted/denied)
✓ Data access (who viewed what record, when)
✓ Configuration changes (settings, permissions, policies)
✓ System events (startup, shutdown, errors)
```

### What NOT to Log
```
✗ Passwords or password hashes
✗ Full credit card numbers (mask: ****1234)
✗ Social Security Numbers
✗ PHI (diagnosis, treatment, conditions)
✗ Authentication tokens or session IDs
✗ Encryption keys or secrets
```

### Data Masking Utility
```typescript
const mask = {
  ssn: (v: string) => `***-**-${v.slice(-4)}`,
  card: (v: string) => `****-****-****-${v.slice(-4)}`,
  email: (v: string) => `${v[0]}***@${v.split("@")[1]}`,
  phone: (v: string) => `***-***-${v.slice(-4)}`,
  name: (v: string) => `${v[0]}${"*".repeat(v.length - 1)}`,
};
```

### Field-Level Encryption
```typescript
import { createCipheriv, createDecipheriv, randomBytes } from "crypto";

function encryptField(plaintext: string, key: Buffer): string {
  const iv = randomBytes(16);
  const cipher = createCipheriv("aes-256-gcm", key, iv);
  const encrypted = Buffer.concat([cipher.update(plaintext, "utf8"), cipher.final()]);
  const tag = cipher.getAuthTag();
  return `${iv.toString("hex")}:${tag.toString("hex")}:${encrypted.toString("hex")}`;
}
```

## Compliance-as-Code

### Open Policy Agent (OPA)
```rego
# Deny deployments without encryption
deny[msg] {
  input.resource == "aws_s3_bucket"
  not input.config.server_side_encryption_configuration
  msg := sprintf("S3 bucket %s must have encryption enabled", [input.name])
}
```

### HashiCorp Sentinel
```python
# Enforce tagging policy
main = rule {
  all tfplan.resources.aws_instance as _, instances {
    all instances as _, r {
      r.applied.tags contains "compliance-scope"
    }
  }
}
```

### Conftest (OPA for CI)
```bash
# Test Kubernetes manifests against policies
conftest test deployment.yaml --policy policy/
# Test Terraform plans
terraform plan -out=plan.tfplan
terraform show -json plan.tfplan | conftest test -
```

## Audit Checklist

Before any compliance audit:
1. Verify all controls have evidence (automated where possible)
2. Run a gap analysis against the framework's requirements
3. Ensure incident response plan is current and tested
4. Confirm all vendor agreements (BAAs, DPAs) are signed and current
5. Validate data retention policies are enforced (not just documented)
6. Test access controls — attempt access with revoked credentials
7. Review audit logs for completeness and tamper protection

Read `references/compliance-frameworks.md` for detailed per-framework checklists, Terraform compliance modules, and audit evidence templates.
