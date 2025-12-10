# Ansible Best Practices

When working with Ansible code in this project, follow these conventions:

## Module Usage

- Always use Fully Qualified Collection Names (FQCN)
  - `kubernetes.core.k8s` not `k8s`
  - `ansible.builtin.include_tasks` not `include_tasks`
  - `ansible.builtin.set_fact` not `set_fact`

## YAML Style

- Use `.yml` extension (except for OLS role which uses `.yaml`)
- Start files with `---`
- Use 2-space indentation
- Use `key: value` format, never `key=value`

## Task Naming

- Every task MUST have a descriptive `name` field
- Start task names with present tense verbs
- Be specific about what the task does
- Example: "Deploy OpenShift AI operator subscription"

## Variable Naming

- Prefix all role variables: `ocp4_workload_<role_name>_<variable>`
- Use snake_case for all variable names
- Document all variables in `defaults/main.yml` with comments
- Mark internal variables with leading underscore: `_ocp4_workload_<role>_internal_var`
- **REQUIRED**: Prefix all registered variables with `r_` for readability and identification
  - Example: `register: r_csv_list`, `register: r_operator_status`, `register: r_pod_info`
  - This makes it immediately clear which variables come from task results

## Idempotency

- Use declarative modules like `kubernetes.core.k8s` with `state: present`
- Avoid `shell` and `command` modules when possible
- When using shell/command, add `changed_when` and `failed_when`
- Use `wait` and `wait_condition` for resource readiness

## Retry and Delay Configuration

**REQUIRED**: All `retries`, `delay`, and `wait_timeout` values MUST be variables defined in `defaults/main.yml`:

❌ **Bad - Hardcoded values:**
```yaml
- name: Wait for operator CSV
  kubernetes.core.k8s_info:
    api_version: operators.coreos.com/v1alpha1
    kind: ClusterServiceVersion
  register: r_csv_list
  until: r_csv_list.resources | length > 0
  retries: 30
  delay: 10
```

✅ **Good - Configurable values:**
```yaml
- name: Wait for operator CSV
  kubernetes.core.k8s_info:
    api_version: operators.coreos.com/v1alpha1
    kind: ClusterServiceVersion
  register: r_csv_list
  until: r_csv_list.resources | length > 0
  retries: "{{ ocp4_workload_<role>_csv_wait_retries }}"
  delay: "{{ ocp4_workload_<role>_csv_wait_delay }}"
```

**In defaults/main.yml:**
```yaml
# Wait configuration for CSV readiness
# Retries: number of attempts to check CSV status
ocp4_workload_<role>_csv_wait_retries: 30
# Delay: seconds between retry attempts
ocp4_workload_<role>_csv_wait_delay: 10
```

Benefits:
- Allows users to adjust timeouts for slow/fast clusters
- Supports different environments (CI, production, development)
- Makes testing faster by reducing wait times
- Provides flexibility without code changes

## Error Handling

- Use `failed_when` to define actual failure conditions
- Use `until` and `retries` for eventual consistency
- Add debug tasks with `verbosity: 1` or `verbosity: 2`
- In cleanup tasks, use `ignore_errors: true` judiciously

## Common Patterns

### Wait for Operator Ready
```yaml
- name: Wait for operator CSV to be succeeded
  kubernetes.core.k8s_info:
    api_version: operators.coreos.com/v1alpha1
    kind: ClusterServiceVersion
    namespace: "{{ namespace }}"
  register: r_csv_list
  until:
    - r_csv_list.resources | length > 0
    - r_csv_list.resources | selectattr('metadata.name', 'match', '^' + operator_name) | list | length > 0
  retries: "{{ ocp4_workload_<role>_csv_wait_retries }}"
  delay: "{{ ocp4_workload_<role>_csv_wait_delay }}"
```

### Conditional Resource Creation
```yaml
- name: Create resource
  when: ocp4_workload_<role>_enable_feature | bool
  kubernetes.core.k8s:
    state: present
    definition: "{{ resource_definition }}"
```

## Testing

- Test both `ACTION=provision` and `ACTION=destroy`
- Verify idempotency by running twice
- Check operator readiness before creating CRs
- Validate all required variables are documented
