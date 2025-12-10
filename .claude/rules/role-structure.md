# Role Structure Standards

All roles in this collection follow a standardized structure:

## Directory Layout

```
roles/ocp4_workload_<name>/
├── defaults/
│   └── main.yml          # Default variables with documentation
├── files/                # Static files (optional)
├── meta/
│   └── main.yml          # Role metadata
├── tasks/
│   ├── main.yml          # Entry point (DO NOT MODIFY)
│   ├── workload.yml      # Provisioning logic
│   └── remove_workload.yml  # Cleanup logic
└── templates/            # Jinja2 templates (optional)
```

## tasks/main.yml

This file is standardized across all roles - **DO NOT MODIFY**:

```yaml
---
# Do not modify this file
- name: Running workload provision tasks
  when: ACTION == "provision"
  ansible.builtin.include_tasks: workload.yml

- name: Running workload removal tasks
  when: ACTION == "destroy"
  ansible.builtin.include_tasks: remove_workload.yml
```

## tasks/workload.yml

Contains all provisioning logic:
- Create resources in dependency order
- Wait for readiness at each step
- Add debug output for important state
- Follow idempotent patterns

## tasks/remove_workload.yml

Contains cleanup logic:
- Remove resources in reverse order
- Use `state: absent`
- Use `ignore_errors: true` carefully for cleanup
- Consider what should be preserved

## defaults/main.yml

Document all variables:

```yaml
---
# Channel for operator subscription
# Valid values: stable, fast, v24.9, etc.
ocp4_workload_<name>_channel: stable

# Enable automatic InstallPlan approval
# Set to false when specifying a starting_csv
ocp4_workload_<name>_automatic_install_plan_approval: true

# Starting ClusterServiceVersion
# Leave empty for latest in channel
ocp4_workload_<name>_starting_csv: ""

# Catalog snapshot support
ocp4_workload_<name>_use_catalog_snapshot: false
ocp4_workload_<name>_catalogsource_name: snapshot-<name>
ocp4_workload_<name>_catalog_snapshot_image: quay.io/org/catalog
ocp4_workload_<name>_catalog_snapshot_image_tag: v4.19_2025_07_21
```

## meta/main.yml

Standard metadata:

```yaml
---
galaxy_info:
  author: Wolfgang Kulhanek
  description: Deploy <workload> on OpenShift
  company: Red Hat
  license: GPL-2.0-or-later
  min_ansible_version: "2.15"
  platforms:
    - name: GenericLinux
      versions:
        - any
  galaxy_tags:
    - openshift
    - ai
    - workload

dependencies: []
```

## Adding New Roles

When creating a new role:

1. Copy structure from existing role
2. Use standard `tasks/main.yml` unchanged
3. Document all variables in `defaults/main.yml`
4. Support catalog snapshots
5. Implement both provision and destroy
6. Test idempotency
7. Add role to main README.md
8. Follow naming convention: `ocp4_workload_<name>`
