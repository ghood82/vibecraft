---
name: secrets-management
description: "Enterprise secrets management — Vault, AWS/GCP/Azure secret managers, SOPS, sealed secrets, rotation policies. Use for vault setup, secret rotation, secrets manager integration, key vault, SOPS encrypted files, sealed secrets, credential storage architecture. Do NOT use for simple .env file setup (use env-secrets)."
---

# Secrets Management

Enterprise-grade secrets lifecycle — storage, rotation, injection, and audit across cloud providers and orchestration platforms.

## Core Principle

Secrets are not config. They require encryption at rest, access-controlled delivery, audit trails, and automated rotation. If it can cause a breach when leaked, it belongs in a secrets manager — not a `.env` file or git repo.

## Provider Selection Guide

| Provider | Best For | Key Feature |
|----------|----------|-------------|
| **HashiCorp Vault** | Multi-cloud, on-prem, advanced policy | Dynamic secrets, leasing, PKI |
| **AWS Secrets Manager** | AWS-native workloads | Automatic RDS rotation, CloudFormation |
| **GCP Secret Manager** | GCP-native workloads | IAM-integrated, versioned, replication |
| **Azure Key Vault** | Azure-native workloads | HSM-backed, certificate management |
| **SOPS (Mozilla)** | Encrypted files in git | GitOps-friendly, multi-provider KMS |
| **Sealed Secrets** | Kubernetes-native | Encrypt for cluster, safe to commit |
| **External Secrets Operator** | K8s + any provider | Sync external secrets into K8s |

## HashiCorp Vault

### Architecture
```
App → Vault Agent (sidecar/init) → Vault Server → Storage Backend
                                         ↓
                                   Audit Log (mandatory)
```

### Secret Engines
- **KV v2**: Versioned key-value (most common)
- **Database**: Dynamic credentials with TTL (Postgres, MySQL, MongoDB)
- **AWS**: Dynamic IAM credentials
- **PKI**: Auto-issued TLS certificates
- **Transit**: Encryption-as-a-service (encrypt without exposing keys)

### Policy Pattern (Least Privilege)
```hcl
# Application read-only policy
path "secret/data/myapp/{{identity.entity.aliases.auth_kubernetes.metadata.namespace}}/*" {
  capabilities = ["read"]
}

# CI/CD deploy policy — scoped to environment
path "secret/data/myapp/production/*" {
  capabilities = ["read"]
  required_parameters = ["version"]
}
```

### Vault Agent Injection (Kubernetes)
```yaml
annotations:
  vault.hashicorp.com/agent-inject: "true"
  vault.hashicorp.com/role: "myapp"
  vault.hashicorp.com/agent-inject-secret-db: "secret/data/myapp/db"
  vault.hashicorp.com/agent-inject-template-db: |
    {{- with secret "secret/data/myapp/db" -}}
    postgresql://{{ .Data.data.username }}:{{ .Data.data.password }}@db:5432/myapp
    {{- end }}
```

## Cloud Provider Integration

### AWS Secrets Manager
```bash
# Store
aws secretsmanager create-secret --name myapp/prod/db \
  --secret-string '{"username":"admin","password":"rotated-value"}'

# Retrieve in app (SDK)
const client = new SecretsManagerClient({ region: "us-east-1" });
const secret = await client.send(new GetSecretValueCommand({ SecretId: "myapp/prod/db" }));
```

### GCP Secret Manager
```bash
echo -n "db-password" | gcloud secrets create myapp-db-password --data-file=-
gcloud secrets add-iam-policy-binding myapp-db-password \
  --member="serviceAccount:myapp@project.iam.gserviceaccount.com" \
  --role="roles/secretmanager.secretAccessor"
```

### Azure Key Vault
```bash
az keyvault secret set --vault-name myapp-vault --name db-password --value "secret"
az keyvault set-policy --name myapp-vault --spn $APP_ID --secret-permissions get list
```

## SOPS — Encrypted Files in Git

```bash
# Encrypt with AWS KMS
sops --encrypt --kms arn:aws:kms:us-east-1:ACCOUNT:key/KEY_ID secrets.yaml > secrets.enc.yaml

# Encrypt with age (local key)
sops --encrypt --age $(cat keys.txt | grep public | cut -d: -f2) secrets.yaml > secrets.enc.yaml

# Decrypt in CI
sops --decrypt secrets.enc.yaml > secrets.yaml
```

**`.sops.yaml` config** — per-directory encryption rules:
```yaml
creation_rules:
  - path_regex: production/.*\.yaml$
    kms: "arn:aws:kms:us-east-1:ACCOUNT:key/PROD_KEY"
  - path_regex: staging/.*\.yaml$
    kms: "arn:aws:kms:us-east-1:ACCOUNT:key/STAGING_KEY"
```

## Kubernetes Secrets

### Sealed Secrets (Bitnami)
```bash
# Encrypt — safe to commit to git
kubeseal --format yaml < secret.yaml > sealed-secret.yaml

# Only the cluster's controller can decrypt
kubectl apply -f sealed-secret.yaml
```

### External Secrets Operator
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: myapp-secrets
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: myapp-secrets
  data:
    - secretKey: DATABASE_URL
      remoteRef:
        key: myapp/prod/db
        property: connection_string
```

## Secret Rotation

### Rotation Policy
| Secret Type | Rotation Frequency | Method |
|------------|-------------------|--------|
| Database credentials | 30 days | Vault dynamic secrets or provider auto-rotation |
| API keys | 90 days | Dual-key rotation (create new → deploy → revoke old) |
| TLS certificates | Auto before expiry | Vault PKI or cert-manager |
| Service tokens | 24 hours | Short-lived, auto-refreshed |
| Encryption keys | Annually | Key versioning, re-encrypt on read |

### Dual-Key Rotation Pattern
```
1. Generate new key (v2) alongside existing key (v1)
2. Deploy app update that accepts both v1 and v2
3. Verify v2 is working in production
4. Revoke v1
5. Deploy app update that only accepts v2
```

## Environment-Specific Injection

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────┐
│ Development  │     │    Staging        │     │ Production   │
│ .env.local   │     │ Vault (dev path)  │     │ Vault (prod) │
│ or direnv    │     │ + ESO sync        │     │ + ESO sync   │
└─────────────┘     └──────────────────┘     └─────────────┘
```

- **Dev**: `.env.local` or `direnv` — convenience, no vault overhead
- **Staging**: Vault with relaxed policies, synced to K8s via ESO
- **Production**: Vault with strict policies, audit logging, auto-rotation

## Access Patterns (Least Privilege)

1. **Identity-based**: App authenticates as itself (K8s service account, IAM role)
2. **Short-lived**: Secrets have TTL, auto-expire, force refresh
3. **Scoped**: Each service reads only its own secrets path
4. **Audited**: Every secret access logged with identity, timestamp, path
5. **No human access to prod**: Break-glass procedure only, dual-approval

## Anti-Patterns

| Don't | Do Instead |
|-------|-----------|
| Hardcode secrets in source | Use secrets manager SDK or injected env vars |
| Share secrets via Slack/email | Use Vault one-time-secret or `vault kv put` |
| Same credentials across envs | Unique per environment, unique per service |
| Long-lived static credentials | Dynamic secrets with TTL or auto-rotation |
| Store secrets in ConfigMaps | Use K8s Secrets + ESO or Vault Agent |
| Decrypt in CI logs | Redirect to file, never `echo $SECRET` |

Read `references/secrets-patterns.md` for full implementation examples per provider, Terraform modules, and CI/CD secret injection patterns.
