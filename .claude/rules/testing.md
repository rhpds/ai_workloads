# Testing Guidelines

Testing standards for roles in this collection:

## Static Analysis

Always run before committing:

```bash
# Lint all roles
ansible-lint roles/

# YAML validation
yamllint .

# Check specific role
ansible-lint roles/ocp4_workload_<name>/
```

## Manual Testing Workflow

### 1. Test Provisioning

```bash
# Create test playbook
cat > test-<role>.yml <<EOF
---
- name: Test <role>
  hosts: localhost
  gather_facts: false
  vars:
    ACTION: provision
  roles:
    - ocp4_workload_<name>
EOF

# Run provision
ansible-playbook test-<role>.yml
```

### 2. Verify Deployment

```bash
# Check operators
oc get operators

# Check subscriptions
oc get subscription -A

# Check CSVs
oc get csv -A

# Check pods
oc get pods -n <namespace>

# Check custom resources
oc get <custom-resource> -A
```

### 3. Test Idempotency

```bash
# Run provision again - should succeed with no changes
ansible-playbook test-<role>.yml
```

### 4. Test Cleanup

```bash
# Run destroy
ansible-playbook test-<role>.yml -e ACTION=destroy

# Verify cleanup
oc get subscription -n <namespace>
oc get csv -n <namespace>
oc get <custom-resource> -A
```

## Testing Checklist

- [ ] Role runs successfully with default variables
- [ ] Role runs successfully with custom variables
- [ ] Idempotency test passes (run twice, no changes second time)
- [ ] Cleanup (destroy) removes all created resources
- [ ] Catalog snapshot mode works
- [ ] Manual InstallPlan approval works (if supported)
- [ ] All variables are documented
- [ ] Static checks pass (ansible-lint, yamllint)

## Testing with Catalog Snapshots

```bash
ansible-playbook test-<role>.yml \
  -e ocp4_workload_<name>_use_catalog_snapshot=true \
  -e ocp4_workload_<name>_catalogsource_name=test-snapshot \
  -e ocp4_workload_<name>_catalog_snapshot_image=quay.io/org/snapshot \
  -e ocp4_workload_<name>_catalog_snapshot_image_tag=v4.19_2025_07_21
```

## Testing Different Channels

```bash
# Test stable channel
ansible-playbook test-<role>.yml -e ocp4_workload_<name>_channel=stable

# Test fast channel
ansible-playbook test-<role>.yml -e ocp4_workload_<name>_channel=fast

# Test specific version
ansible-playbook test-<role>.yml -e ocp4_workload_<name>_channel=v24.9
```

## Debugging Failed Tests

```bash
# Check operator logs
oc logs -n <namespace> -l app=<operator>

# Check events
oc get events -n <namespace> --sort-by='.lastTimestamp'

# Describe CSV
oc describe csv -n <namespace> <csv-name>

# Check InstallPlan
oc get installplan -n <namespace>

# Increase Ansible verbosity
ansible-playbook test-<role>.yml -vvv
```

## CI/CD Considerations

When roles are integrated into CI/CD:

- Use catalog snapshots for reproducibility
- Pin specific CSVs with `starting_csv`
- Set appropriate timeouts for wait conditions
- Capture logs on failure
- Clean up resources after tests
