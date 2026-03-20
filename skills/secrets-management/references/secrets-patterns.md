# Secrets Patterns — Implementation Reference

## HashiCorp Vault — Full Setup

### Dev Server (Local Development)
```bash
vault server -dev -dev-root-token-id="dev-token"
export VAULT_ADDR="http://127.0.0.1:8200"
export VAULT_TOKEN="dev-token"

# Enable KV v2
vault secrets enable -path=secret kv-v2

# Store a secret
vault kv put secret/myapp/db username="admin" password="s3cur3p@ss"
vault kv get -format=json secret/myapp/db
```

### Production Vault (Terraform)
```hcl
resource "vault_mount" "kv" {
  path    = "secret"
  type    = "kv-v2"
  options = { version = "2" }
}

resource "vault_policy" "app_readonly" {
  name   = "myapp-readonly"
  policy = <<-EOT
    path "secret/data/myapp/*" {
      capabilities = ["read", "list"]
    }
    path "secret/metadata/myapp/*" {
      capabilities = ["list"]
    }
  EOT
}

resource "vault_kubernetes_auth_backend_role" "app" {
  backend                          = "kubernetes"
  role_name                        = "myapp"
  bound_service_account_names      = ["myapp"]
  bound_service_account_namespaces = ["production"]
  token_policies                   = ["myapp-readonly"]
  token_ttl                        = 3600
}
```

### Dynamic Database Credentials
```hcl
resource "vault_database_secret_backend" "postgres" {
  path = "database"

  postgresql {
    name              = "myapp-db"
    connection_url    = "postgresql://{{username}}:{{password}}@db.example.com:5432/myapp"
    username          = "vault_admin"
    password          = "admin_password"
    verify_connection = true
  }
}

resource "vault_database_secret_backend_role" "app" {
  backend     = "database"
  name        = "myapp-role"
  db_name     = "myapp-db"
  default_ttl = 3600
  max_ttl     = 86400

  creation_statements = [
    "CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}';",
    "GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO \"{{name}}\";",
  ]

  revocation_statements = [
    "DROP ROLE IF EXISTS \"{{name}}\";",
  ]
}
```

## AWS Secrets Manager — Patterns

### Automatic RDS Rotation (Terraform)
```hcl
resource "aws_secretsmanager_secret" "db" {
  name = "myapp/prod/db-credentials"
}

resource "aws_secretsmanager_secret_rotation" "db" {
  secret_id           = aws_secretsmanager_secret.db.id
  rotation_lambda_arn = aws_lambda_function.rotation.arn

  rotation_rules {
    automatically_after_days = 30
  }
}
```

### SDK Usage (Node.js)
```typescript
import { SecretsManagerClient, GetSecretValueCommand } from "@aws-sdk/client-secrets-manager";

const client = new SecretsManagerClient({ region: "us-east-1" });

// Cache secrets to reduce API calls
let cachedSecrets: Record<string, string> | null = null;
let cacheExpiry = 0;

async function getSecrets(secretId: string): Promise<Record<string, string>> {
  if (cachedSecrets && Date.now() < cacheExpiry) return cachedSecrets;

  const response = await client.send(
    new GetSecretValueCommand({ SecretId: secretId })
  );
  cachedSecrets = JSON.parse(response.SecretString!);
  cacheExpiry = Date.now() + 5 * 60 * 1000; // 5-minute cache
  return cachedSecrets!;
}
```

## GCP Secret Manager — Patterns

### Terraform Setup
```hcl
resource "google_secret_manager_secret" "db_password" {
  secret_id = "myapp-db-password"
  replication {
    auto {}
  }
}

resource "google_secret_manager_secret_version" "db_password" {
  secret      = google_secret_manager_secret.db_password.id
  secret_data = var.db_password
}

resource "google_secret_manager_secret_iam_member" "accessor" {
  secret_id = google_secret_manager_secret.db_password.id
  role      = "roles/secretmanager.secretAccessor"
  member    = "serviceAccount:${google_service_account.app.email}"
}
```

### SDK Usage (Node.js)
```typescript
import { SecretManagerServiceClient } from "@google-cloud/secret-manager";

const client = new SecretManagerServiceClient();

async function getSecret(name: string): Promise<string> {
  const [version] = await client.accessSecretVersion({
    name: `projects/my-project/secrets/${name}/versions/latest`,
  });
  return version.payload!.data!.toString();
}
```

## Azure Key Vault — Patterns

### Terraform Setup
```hcl
resource "azurerm_key_vault" "main" {
  name                = "myapp-vault"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  tenant_id           = data.azurerm_client_config.current.tenant_id
  sku_name            = "standard"

  purge_protection_enabled = true
  soft_delete_retention_days = 90
}

resource "azurerm_key_vault_access_policy" "app" {
  key_vault_id = azurerm_key_vault.main.id
  tenant_id    = data.azurerm_client_config.current.tenant_id
  object_id    = azurerm_user_assigned_identity.app.principal_id

  secret_permissions = ["Get", "List"]
}
```

## SOPS — Complete Workflow

### Setup with age (Recommended for Teams)
```bash
# Generate key
age-keygen -o keys.txt
# keys.txt → store in password manager, distribute to team

# .sops.yaml — repo root
cat > .sops.yaml << 'EOF'
creation_rules:
  - path_regex: secrets/production/.*
    age: "age1abc...prod_key"
    encrypted_regex: "^(password|token|secret|key|connection_string)$"
  - path_regex: secrets/staging/.*
    age: "age1def...staging_key"
    encrypted_regex: "^(password|token|secret|key|connection_string)$"
  - path_regex: secrets/.*
    age: "age1ghi...dev_key"
EOF

# Encrypt
sops --encrypt secrets/production/db.yaml > secrets/production/db.enc.yaml

# Edit in-place (decrypts → opens editor → re-encrypts)
sops secrets/production/db.enc.yaml

# Decrypt in CI
export SOPS_AGE_KEY=$(cat /run/secrets/age-key)
sops --decrypt secrets/production/db.enc.yaml
```

## External Secrets Operator — K8s Integration

### ClusterSecretStore (AWS)
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: aws-secrets-manager
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa
            namespace: external-secrets
```

### ExternalSecret (Per-App)
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: myapp-secrets
  namespace: production
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: ClusterSecretStore
  target:
    name: myapp-secrets
    creationPolicy: Owner
  data:
    - secretKey: DATABASE_URL
      remoteRef:
        key: myapp/prod/db
        property: connection_string
    - secretKey: API_KEY
      remoteRef:
        key: myapp/prod/api
        property: key
```

## CI/CD Secret Injection

### GitHub Actions
```yaml
jobs:
  deploy:
    steps:
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: ${{ secrets.AWS_ROLE_ARN }}
          aws-region: us-east-1

      - name: Get secrets
        uses: aws-actions/aws-secretsmanager-get-secrets@v2
        with:
          secret-ids: |
            DB,myapp/prod/db
          parse-json-secrets: true

      - name: Deploy
        env:
          DATABASE_URL: ${{ env.DB_CONNECTION_STRING }}
        run: npm run deploy
```

### GitLab CI
```yaml
deploy:
  id_tokens:
    VAULT_ID_TOKEN:
      aud: https://vault.example.com
  secrets:
    DATABASE_URL:
      vault: myapp/prod/db/connection_string@secret
      token: $VAULT_ID_TOKEN
```

## Secret Rotation — Automation Script

```typescript
// Dual-key rotation for API keys
async function rotateApiKey(service: string) {
  // 1. Generate new key
  const newKey = await provider.createKey(service);

  // 2. Store both keys (old + new)
  await vault.put(`${service}/api-key`, {
    current: newKey,
    previous: await vault.get(`${service}/api-key`).then(s => s.current),
  });

  // 3. Deploy app (accepts both keys during transition)
  await deploy({ acceptKeys: ["current", "previous"] });

  // 4. Verify new key works
  await healthCheck(service, newKey);

  // 5. Revoke old key
  const oldKey = await vault.get(`${service}/api-key`).then(s => s.previous);
  await provider.revokeKey(service, oldKey);

  // 6. Update vault (remove previous)
  await vault.put(`${service}/api-key`, { current: newKey, previous: null });

  await auditLog.record({ action: "KEY_ROTATED", service, timestamp: new Date() });
}
```
