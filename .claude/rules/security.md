# Security Guidelines

Security considerations when working with this collection:

## Secrets Management

### Never Hardcode Secrets

❌ **Bad:**
```yaml
- name: Create secret
  kubernetes.core.k8s:
    definition:
      apiVersion: v1
      kind: Secret
      metadata:
        name: api-secret
      stringData:
        token: "sk-abc123xyz789"  # NEVER DO THIS
```

✅ **Good:**
```yaml
- name: Create secret
  kubernetes.core.k8s:
    definition:
      apiVersion: v1
      kind: Secret
      metadata:
        name: api-secret
      stringData:
        token: "{{ ocp4_workload_<name>_api_token }}"
```

### Default Secret Values

Use placeholder values in `defaults/main.yml`:

```yaml
# API token for authentication
# IMPORTANT: Override this value, do not use default
ocp4_workload_<name>_api_token: "changeme"

# Azure client secret
# IMPORTANT: Override this value, do not use default
ocp4_workload_<name>_azure_client_secret: "changeme"
```

### Validate Secrets

Check that secrets have been properly configured:

```yaml
- name: Validate secret is configured
  ansible.builtin.assert:
    that:
      - ocp4_workload_<name>_api_token is defined
      - ocp4_workload_<name>_api_token != "changeme"
      - ocp4_workload_<name>_api_token | length > 0
    fail_msg: "API token must be configured before running"
```

## RBAC and Permissions

### Minimal Permissions

Create service accounts with only required permissions:

```yaml
- name: Create service account
  kubernetes.core.k8s:
    definition:
      apiVersion: v1
      kind: ServiceAccount
      metadata:
        name: <workload>-sa
        namespace: <namespace>

- name: Create minimal role
  kubernetes.core.k8s:
    definition:
      apiVersion: rbac.authorization.k8s.io/v1
      kind: Role
      metadata:
        name: <workload>-role
        namespace: <namespace>
      rules:
        - apiGroups: [""]
          resources: ["secrets"]
          verbs: ["get", "list"]  # Only what's needed
```

### Document Required Permissions

In role README, document what cluster permissions are needed:

```markdown
## Required Permissions

This role requires cluster-admin or the following permissions:

- Create/delete operators in namespace
- Create/delete subscriptions
- Create/delete custom resources
- Read ClusterServiceVersions
```

## Network Security

### Network Policies

Consider adding NetworkPolicies for workload isolation:

```yaml
- name: Create network policy
  kubernetes.core.k8s:
    definition:
      apiVersion: networking.k8s.io/v1
      kind: NetworkPolicy
      metadata:
        name: <workload>-netpol
        namespace: <namespace>
      spec:
        podSelector:
          matchLabels:
            app: <workload>
        policyTypes:
          - Ingress
        ingress:
          - from:
            - namespaceSelector:
                matchLabels:
                  name: openshift-ingress
```

### Document Network Requirements

Document required network connectivity:

```markdown
## Network Requirements

This workload requires:
- Outbound HTTPS (443) to operator registries
- Outbound HTTPS (443) to AI service endpoints
- Ingress from OpenShift router namespace
```

## File Security

### Sensitive Files

Add sensitive files to `.gitignore`:

```gitignore
# Secrets and credentials
*.secret
*.key
*.pem
credentials.yml
secrets.yml

# Local test files with real data
test-vars.yml
local-config.yml
```

### File Permissions

When creating files with secrets:

```yaml
- name: Create credentials file
  ansible.builtin.copy:
    content: "{{ secret_content }}"
    dest: /tmp/credentials
    mode: '0600'  # Owner read/write only
```

## Audit and Logging

### Avoid Logging Secrets

Use `no_log` for tasks with sensitive data:

```yaml
- name: Create secret
  kubernetes.core.k8s:
    definition: "{{ secret_definition }}"
  no_log: true  # Prevents secret from appearing in logs

- name: Set API token
  ansible.builtin.set_fact:
    api_token: "{{ provided_token }}"
  no_log: true
```

### Debug Output

Be careful with debug output:

```yaml
# Bad - might expose secrets
- debug:
    var: all_variables

# Good - specific non-sensitive info
- debug:
    msg: "Deploying operator in namespace {{ namespace }}"
```

## Container Images

### Image Pull Secrets

When using private registries:

```yaml
- name: Create image pull secret
  kubernetes.core.k8s:
    definition:
      apiVersion: v1
      kind: Secret
      metadata:
        name: registry-credentials
        namespace: <namespace>
      type: kubernetes.io/dockerconfigjson
      data:
        .dockerconfigjson: "{{ dockerconfig | b64encode }}"
  no_log: true
```

### Image Sources

- Prefer official Red Hat registries
- Document image sources in role variables
- Support air-gapped deployments with local registries

## Security Scanning

### Before Committing

```bash
# Check for secrets in code
git diff | grep -i "password\|secret\|token\|key"

# Scan with trufflehog or similar
trufflehog filesystem .

# Review files being committed
git diff --cached
```

## Compliance

### Document Security Considerations

In role README, include security section:

```markdown
## Security Considerations

- Secrets are never hardcoded in tasks
- All sensitive variables use 'changeme' defaults
- Service accounts follow principle of least privilege
- Network policies recommended for production
- Supports disconnected/air-gapped deployments
```

## Common Security Issues

### Issues to Avoid

1. ❌ Committing actual secrets to git
2. ❌ Using overly permissive RBAC (cluster-admin everywhere)
3. ❌ Logging sensitive information
4. ❌ Using HTTP instead of HTTPS
5. ❌ Not validating secret values
6. ❌ Hardcoding credentials in files

### Security Checklist

- [ ] No secrets in defaults/main.yml
- [ ] Secrets validated before use
- [ ] Sensitive tasks use `no_log: true`
- [ ] RBAC follows least privilege
- [ ] Image sources documented
- [ ] Network requirements documented
- [ ] Secrets properly templated
- [ ] `.gitignore` includes sensitive files
