# Variable Conventions

Standards for variables in this collection:

## Naming Convention

All role variables MUST follow this pattern:
```
ocp4_workload_<role_name>_<variable_name>
```

Examples:
- `ocp4_workload_openshift_ai_channel`
- `ocp4_workload_nvidia_gpu_operator_namespace`
- `ocp4_workload_ols_api_token`

## Internal Variables

Variables used internally within a role (not meant for user configuration) should start with underscore:

```yaml
_ocp4_workload_openshift_ai_installed_csv: ""
_ocp4_workload_openshift_ai_version_3: false
```

## Registered Variables

**REQUIRED**: All variables created with `register` MUST be prefixed with `r_`:

```yaml
# Good - clear that these are registered results
- name: Get CSV status
  kubernetes.core.k8s_info:
    api_version: operators.coreos.com/v1alpha1
    kind: ClusterServiceVersion
  register: r_csv_list

- name: Check pod status
  kubernetes.core.k8s_info:
    kind: Pod
    namespace: "{{ namespace }}"
  register: r_pods

# Bad - no prefix makes it unclear these are registered
- name: Get CSV status
  kubernetes.core.k8s_info:
    api_version: operators.coreos.com/v1alpha1
    kind: ClusterServiceVersion
  register: csv_list  # Missing r_ prefix
```

Benefits:
- Immediately identifies variables from task results
- Improves code readability
- Prevents confusion with role variables
- Makes debugging easier

## Retry and Timeout Variables

**REQUIRED**: All `retries`, `delay`, and `wait_timeout` values must be configurable variables:

### Naming Convention

Use descriptive names that identify the operation being retried:

```yaml
# Format: ocp4_workload_<role>_<operation>_retries
# Format: ocp4_workload_<role>_<operation>_delay
# Format: ocp4_workload_<role>_<operation>_wait_timeout

# Examples:
ocp4_workload_openshift_ai_csv_wait_retries: 30
ocp4_workload_openshift_ai_csv_wait_delay: 10

ocp4_workload_nvidia_gpu_operator_cluster_policy_wait_timeout: 600
ocp4_workload_nvidia_gpu_operator_cluster_policy_wait_retries: 60
ocp4_workload_nvidia_gpu_operator_cluster_policy_wait_delay: 10

ocp4_workload_ols_operator_ready_retries: 30
ocp4_workload_ols_operator_ready_delay: 10
```

### Documentation Format

Always document what the timeout/retry is for:

```yaml
# CSV readiness wait configuration
# Number of times to check if CSV has reached Succeeded phase
ocp4_workload_<role>_csv_wait_retries: 30
# Seconds to wait between each retry attempt
ocp4_workload_<role>_csv_wait_delay: 10

# ClusterPolicy readiness timeout
# Maximum seconds to wait for ClusterPolicy to become ready
ocp4_workload_<role>_cluster_policy_wait_timeout: 600
```

### Benefits

- Users can adjust for cluster performance (slow/fast)
- Different environments need different timeouts (CI vs production)
- Faster testing with reduced wait times
- No code changes needed to tune performance

## Required Variables

Every role must define these standard variables:

```yaml
# Operator channel selection
ocp4_workload_<name>_channel: stable

# InstallPlan approval mode
ocp4_workload_<name>_automatic_install_plan_approval: true

# Starting CSV (empty for latest)
ocp4_workload_<name>_starting_csv: ""

# Catalog snapshot support
ocp4_workload_<name>_use_catalog_snapshot: false
ocp4_workload_<name>_catalogsource_name: snapshot-<name>
ocp4_workload_<name>_catalog_snapshot_image: quay.io/org/catalog
ocp4_workload_<name>_catalog_snapshot_image_tag: v4.19_2025_01_01
```

## Documentation Format

Document every variable in `defaults/main.yml`:

```yaml
---
# Channel for the operator subscription
# Valid values: stable, fast, or version-specific like v24.9
# Default: stable
ocp4_workload_<name>_channel: stable

# Enable automatic InstallPlan approval
# Set to false when you want to manually approve operator updates
# or when using a specific starting_csv
# Default: true
ocp4_workload_<name>_automatic_install_plan_approval: true

# Starting ClusterServiceVersion for the operator
# Recommended to leave empty to get latest in the channel
# Set to specific version like "operator.2.25.0" for pinned deployments
# Default: "" (empty string for latest)
ocp4_workload_<name>_starting_csv: ""
```

## Boolean Variables

- Use boolean types, not strings
- Check booleans with `| bool` filter
- Default to sensible values

```yaml
# Good
ocp4_workload_<name>_enable_feature: true

# Usage
when: ocp4_workload_<name>_enable_feature | bool
```

## Secret/Sensitive Variables

For variables containing secrets:

```yaml
# API token for authentication
# Required when ocp4_workload_<name>_auth_mode is 'token'
# IMPORTANT: Do not commit actual tokens to version control
ocp4_workload_<name>_api_token: "changeme"

# Azure tenant ID for authentication
# Required when using Azure token authentication
ocp4_workload_<name>_azure_tenant_id: "changeme"
```

## List Variables

```yaml
# List of model folders to copy from S3
# Each item should be a folder name in the source S3 bucket
ocp4_workload_<name>_s3_folders:
  - parasol-instruct
  - granite-7b-lab
  - merlinite-7b-lab
```

## Dictionary Variables

```yaml
# Model registry configuration
ocp4_workload_<name>_modelregistry_config:
  managementState: "{{ ocp4_workload_<name>_modelregistry }}"
  registriesNamespace: "{{ ocp4_workload_<name>_modelregistry_namespace }}"
```

## Variable Validation

Add validation for critical variables:

```yaml
- name: Validate required variables
  ansible.builtin.assert:
    that:
      - ocp4_workload_<name>_api_token is defined
      - ocp4_workload_<name>_api_token != "changeme"
      - ocp4_workload_<name>_api_token | length > 0
    fail_msg: "ocp4_workload_<name>_api_token must be set"
```

## Variable Precedence

Remember Ansible's variable precedence:
1. Command line `-e` variables (highest)
2. Role vars (avoid using)
3. Task vars
4. Block vars
5. Role defaults (lowest)

Use `defaults/main.yml` for all user-configurable variables.

## Examples from Existing Roles

### OpenShift AI
```yaml
ocp4_workload_openshift_ai_channel: stable
ocp4_workload_openshift_ai_deploy_dsc: true
ocp4_workload_openshift_ai_dashboard: Managed
ocp4_workload_openshift_ai_workbenches: Managed
```

### NVIDIA GPU Operator
```yaml
ocp4_workload_nvidia_gpu_operator_namespace: nvidia-gpu-operator
ocp4_workload_nvidia_gpu_operator_channel: v24.9
ocp4_workload_nvidia_gpu_operator_create_dashboard: true
ocp4_workload_nvidia_gpu_operator_wait_for_cluster_policy: true
```

### OLS
```yaml
ocp4_workload_ols_install_operator: true
ocp4_workload_ols_get_token: false
ocp4_workload_ols_api_token: "changeme"
ocp4_workload_ai_platform: azure  # Note: no role prefix for this one
```
