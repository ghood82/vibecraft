# Compliance Frameworks — Implementation Reference

## HIPAA Implementation Checklist

### Administrative Safeguards
- [ ] Designate a Security Officer and Privacy Officer
- [ ] Conduct annual risk assessment
- [ ] Maintain policies and procedures (review annually)
- [ ] Workforce training on PHI handling (onboarding + annual)
- [ ] Business Associate Agreement (BAA) register — track all vendors
- [ ] Incident response plan specific to PHI breaches
- [ ] Sanction policy for workforce violations

### Technical Safeguards
- [ ] Unique user identification (no shared accounts)
- [ ] Emergency access procedure (break-glass)
- [ ] Automatic logoff (session timeout ≤ 15 min for PHI apps)
- [ ] Encryption at rest — AES-256 for all PHI stores
- [ ] Encryption in transit — TLS 1.2+ (no exceptions)
- [ ] Audit controls — log all PHI access with immutable audit trail
- [ ] Integrity controls — checksums, version control for PHI records
- [ ] Multi-factor authentication for PHI access

### Physical Safeguards
- [ ] Facility access controls (badge, biometric)
- [ ] Workstation security policies (screen lock, clean desk)
- [ ] Device and media controls (encrypted drives, secure disposal)
- [ ] Cloud provider has BAA signed (AWS, GCP, Azure all offer BAAs)

### HIPAA Audit Log Schema
```typescript
interface HIPAAAuditEvent {
  eventId: string;           // UUID
  timestamp: string;         // ISO 8601
  actor: {
    userId: string;
    role: string;
    ipAddress: string;
    userAgent: string;
  };
  action: "CREATE" | "READ" | "UPDATE" | "DELETE" | "EXPORT" | "PRINT";
  resource: {
    type: "patient_record" | "lab_result" | "prescription" | "billing";
    id: string;
    patientId: string;       // hashed
  };
  justification: string;     // why the access was needed
  outcome: "success" | "failure" | "denied";
  phiAccessed: string[];     // which PHI fields were touched
}
```

### HIPAA-Compliant API Pattern
```typescript
// Middleware: Enforce minimum necessary
function phiAccessMiddleware(requiredFields: string[]) {
  return async (req: Request, res: Response, next: NextFunction) => {
    // 1. Verify user has PHI access role
    if (!req.user.roles.includes("phi_access")) {
      await auditLog.record({ ...req.auditContext, outcome: "denied" });
      return res.status(403).json({ error: "PHI access denied" });
    }

    // 2. Log the access with justification
    await auditLog.record({
      actor: req.user,
      action: "READ",
      resource: { type: "patient_record", id: req.params.id },
      justification: req.headers["x-access-justification"] || "not_provided",
      phiAccessed: requiredFields,
      outcome: "success",
    });

    // 3. Strip fields not in requiredFields from response
    res.locals.phiFilter = (record: any) => {
      return Object.fromEntries(
        Object.entries(record).filter(([k]) => requiredFields.includes(k) || !isPHIField(k))
      );
    };

    next();
  };
}
```

## SOC 2 Control Mapping

### Common Criteria → Implementation

| Control ID | Description | Implementation |
|-----------|-------------|----------------|
| CC1.1 | COSO Integrity & Ethics | Code of conduct, security policy docs |
| CC2.1 | Internal communication | Security awareness training program |
| CC3.1 | Risk assessment | Annual risk assessment + quarterly review |
| CC5.1 | Control activities | Automated policy enforcement (OPA, Sentinel) |
| CC6.1 | Logical access | RBAC, MFA, SSO via IdP (Okta, Auth0) |
| CC6.2 | System credentials | Password policy, API key rotation |
| CC6.3 | Role-based access | Least privilege, quarterly access reviews |
| CC6.6 | System boundaries | WAF, network segmentation, VPC config |
| CC6.7 | Data restrictions | Encryption at rest + in transit |
| CC6.8 | Prevent malicious software | EDR, dependency scanning, container scanning |
| CC7.1 | Monitoring | SIEM, CloudTrail, alerting pipeline |
| CC7.2 | Anomaly detection | Failed login alerts, unusual access patterns |
| CC7.3 | Change evaluation | PR reviews, CI/CD gates, staging validation |
| CC7.4 | Incident response | Runbook, PagerDuty, post-mortem process |
| CC8.1 | Change management | Git-based workflow, PR approvals, audit trail |
| CC9.1 | Risk mitigation | Risk register, control testing schedule |

### SOC 2 Evidence Automation (Terraform)
```hcl
# CC6.1 — MFA Enforcement
resource "aws_iam_account_password_policy" "strict" {
  minimum_password_length        = 14
  require_lowercase_characters   = true
  require_uppercase_characters   = true
  require_numbers                = true
  require_symbols                = true
  max_password_age               = 90
  password_reuse_prevention      = 24
}

# CC7.1 — Audit Logging
resource "aws_cloudtrail" "main" {
  name                       = "soc2-audit-trail"
  s3_bucket_name             = aws_s3_bucket.audit_logs.id
  include_global_service_events = true
  is_multi_region_trail      = true
  enable_log_file_validation = true

  event_selector {
    read_write_type           = "All"
    include_management_events = true
  }
}

# CC6.7 — Encryption at Rest
resource "aws_s3_bucket_server_side_encryption_configuration" "audit" {
  bucket = aws_s3_bucket.audit_logs.id
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm     = "aws:kms"
      kms_master_key_id = aws_kms_key.audit.arn
    }
  }
}
```

### Quarterly Access Review Script
```typescript
async function quarterlyAccessReview() {
  const users = await idp.listUsers();
  const report: AccessReviewItem[] = [];

  for (const user of users) {
    const lastLogin = await idp.getLastLogin(user.id);
    const roles = await idp.getUserRoles(user.id);
    const isActive = user.status === "active";
    const daysSinceLogin = daysBetween(lastLogin, new Date());

    report.push({
      user: user.email,
      roles,
      lastLogin,
      status: user.status,
      flags: [
        daysSinceLogin > 90 && "STALE_ACCOUNT",
        roles.includes("admin") && daysSinceLogin > 30 && "INACTIVE_ADMIN",
        !isActive && roles.length > 0 && "DISABLED_WITH_ROLES",
      ].filter(Boolean),
      recommendation: daysSinceLogin > 90 ? "REVOKE" : "RETAIN",
    });
  }

  return report;
}
```

## GDPR Implementation Patterns

### Data Subject Request Handler
```typescript
// Handles all GDPR DSR types
class DataSubjectRequestHandler {
  async handleRequest(type: DSRType, userId: string): Promise<DSRResponse> {
    await this.verifyIdentity(userId);
    await this.logRequest(type, userId);

    switch (type) {
      case "ACCESS":
        return this.exportAllData(userId);
      case "ERASURE":
        return this.deleteUserData(userId);
      case "RECTIFICATION":
        return this.provideEditAccess(userId);
      case "PORTABILITY":
        return this.exportPortableFormat(userId);
      case "RESTRICTION":
        return this.restrictProcessing(userId);
      case "OBJECTION":
        return this.processObjection(userId);
    }
  }

  private async exportAllData(userId: string): Promise<DSRResponse> {
    const data = {
      profile: await db.users.findById(userId),
      orders: await db.orders.findByUser(userId),
      activity: await db.activityLog.findByUser(userId),
      consents: await db.consents.findByUser(userId),
      communications: await db.emails.findByUser(userId),
    };
    // Remove internal fields (IDs, internal notes)
    return { format: "json", data: sanitizeForExport(data) };
  }

  private async deleteUserData(userId: string): Promise<DSRResponse> {
    // Soft delete first (30-day recovery window)
    await db.users.softDelete(userId);
    // Anonymize analytics (retain aggregates)
    await db.analytics.anonymize(userId);
    // Retain legally required data (invoices, tax records)
    await db.orders.retainForCompliance(userId, { years: 7 });
    // Schedule hard delete
    await scheduler.schedule("hard-delete-user", { userId }, { delay: "30d" });

    return { status: "scheduled", hardDeleteDate: addDays(new Date(), 30) };
  }
}
```

### Consent Management Database Schema
```sql
CREATE TABLE consent_records (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id),
  purpose VARCHAR(50) NOT NULL,          -- 'marketing', 'analytics', 'personalization'
  granted BOOLEAN NOT NULL,
  consent_text_version VARCHAR(20),       -- version of the text user agreed to
  method VARCHAR(30),                     -- 'checkbox', 'settings', 'api'
  ip_address INET,
  user_agent TEXT,
  granted_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  revoked_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_consent_user ON consent_records(user_id);
CREATE INDEX idx_consent_purpose ON consent_records(user_id, purpose);
```

## PCI DSS Implementation Patterns

### Tokenization Flow (Stripe)
```typescript
// Client-side: Collect card → get token (card never hits your server)
const { token } = await stripe.createToken(cardElement);

// Server-side: Use token to create charge
const charge = await stripe.charges.create({
  amount: 2000,
  currency: "usd",
  source: token.id,        // tok_xxx — not the actual card
  description: "Order #123",
});

// Store only: last4, brand, expiry (for display)
await db.paymentMethods.insert({
  userId,
  stripeToken: token.id,
  last4: token.card.last4,
  brand: token.card.brand,
  expMonth: token.card.exp_month,
  expYear: token.card.exp_year,
  // NEVER store: full number, CVV, magnetic stripe
});
```

### PCI Network Segmentation (Terraform)
```hcl
# Separate VPC for payment processing
resource "aws_vpc" "payment" {
  cidr_block = "10.1.0.0/16"
  tags = { Name = "payment-vpc", "pci-scope" = "true" }
}

# Strict security group — only payment service ports
resource "aws_security_group" "payment_api" {
  vpc_id = aws_vpc.payment.id

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = [aws_vpc.main.cidr_block]  # Only from app VPC
  }

  egress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Stripe API
  }
}
```

## Compliance-as-Code — OPA Policies

### Prevent Unencrypted Storage
```rego
package terraform.aws

deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_s3_bucket"
  not has_encryption(resource)
  msg := sprintf("S3 bucket '%s' must have server-side encryption enabled [PCI 3.4, SOC2 CC6.7]", [resource.name])
}

deny[msg] {
  resource := input.resource_changes[_]
  resource.type == "aws_rds_instance"
  resource.change.after.storage_encrypted != true
  msg := sprintf("RDS instance '%s' must have storage encryption enabled [HIPAA §164.312(a)(2)(iv)]", [resource.name])
}

has_encryption(resource) {
  resource.change.after.server_side_encryption_configuration[_].rule[_].apply_server_side_encryption_by_default[_].sse_algorithm
}
```

### Enforce Logging
```rego
package kubernetes

deny[msg] {
  input.kind == "Deployment"
  container := input.spec.template.spec.containers[_]
  not has_audit_sidecar(input.spec.template.spec)
  msg := sprintf("Deployment '%s' must include audit logging sidecar [SOC2 CC7.1]", [input.metadata.name])
}

has_audit_sidecar(spec) {
  container := spec.containers[_]
  container.name == "audit-logger"
}
```

## Audit Evidence Templates

### Monthly Evidence Collection
```yaml
month: "2026-03"
framework: "SOC 2 Type II"
evidence:
  - control: CC6.1
    title: "Access Control Verification"
    evidence_type: screenshot
    collected_by: "automated"
    file: "evidence/2026-03/cc6.1-iam-mfa-policy.png"
    notes: "All IAM users have MFA enforced via organization SCP"

  - control: CC7.2
    title: "Security Alert Response SLA"
    evidence_type: report
    collected_by: "automated"
    file: "evidence/2026-03/cc7.2-alert-response-times.csv"
    notes: "100% of critical alerts triaged within 4h SLA"

  - control: CC8.1
    title: "Change Management Log"
    evidence_type: export
    collected_by: "automated"
    file: "evidence/2026-03/cc8.1-github-pr-log.csv"
    notes: "All changes via PR with required reviews"
```
