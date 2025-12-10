# OpenShift & Operator Patterns

When working with OpenShift resources and operators, follow these patterns:

## Operator Installation Pattern

Standard sequence for operator deployment:

1. Create namespace (if needed)
2. Create OperatorGroup
3. Create Subscription with appropriate channel
4. Wait for ClusterServiceVersion (CSV) to reach `Succeeded`
5. Create Custom Resource instances

## Subscription Configuration

```yaml
- name: Create operator subscription
  kubernetes.core.k8s:
    state: present
    definition:
      apiVersion: operators.coreos.com/v1alpha1
      kind: Subscription
      metadata:
        name: <operator-name>
        namespace: <namespace>
      spec:
        channel: "{{ ocp4_workload_<role>_channel }}"
        name: <operator-name>
        source: "{{ catalog_source }}"
        sourceNamespace: openshift-marketplace
        installPlanApproval: "{{ 'Automatic' if auto_approval else 'Manual' }}"
        startingCSV: "{{ starting_csv }}"
```

## Catalog Snapshot Support

Always support catalog snapshot deployment:

```yaml
- name: Create catalog source for snapshot
  when: ocp4_workload_<role>_use_catalog_snapshot | bool
  kubernetes.core.k8s:
    state: present
    definition:
      apiVersion: operators.coreos.com/v1alpha1
      kind: CatalogSource
      metadata:
        name: "{{ ocp4_workload_<role>_catalogsource_name }}"
        namespace: openshift-marketplace
      spec:
        sourceType: grpc
        image: "{{ ocp4_workload_<role>_catalog_snapshot_image }}:{{ ocp4_workload_<role>_catalog_snapshot_image_tag }}"
        displayName: "Snapshot Catalog"
```

## Resource Readiness

- Always wait for resources to be ready before proceeding
- Use `kubernetes.core.k8s` with `wait: true` and `wait_condition`
- For operators, wait for CSV phase to be `Succeeded`
- For pods, wait for Ready condition
- **IMPORTANT**: All `wait_timeout`, `retries`, and `delay` values must be configurable variables in `defaults/main.yml`

Example with configurable timeouts:
```yaml
- name: Wait for ClusterPolicy ready
  kubernetes.core.k8s_info:
    api_version: nvidia.com/v1
    kind: ClusterPolicy
    namespace: "{{ namespace }}"
    wait: true
    wait_condition:
      type: Ready
      status: "True"
    wait_timeout: "{{ ocp4_workload_<role>_cluster_policy_wait_timeout }}"
```

## Namespace Management

- Create namespaces explicitly with proper labels
- Use `openshift-operators` for cluster-scoped operators
- Set appropriate namespace for Custom Resources

## Version Detection

When supporting multiple operator versions:

```yaml
- name: Get installed CSV
  kubernetes.core.k8s_info:
    api_version: operators.coreos.com/v1alpha1
    kind: ClusterServiceVersion
    namespace: <namespace>
  register: r_csvs

- name: Detect version
  ansible.builtin.set_fact:
    _version_3: "{{ csv_name.startswith('operator.3.') }}"
```

## Common Issues

- Always check operator events: `oc get events -n <namespace>`
- Verify CSV status: `oc get csv -n <namespace>`
- Check pod logs for operator issues
- Ensure sufficient cluster resources (CPU, memory)
