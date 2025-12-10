# Ansible Collection - agnosticd.ai_workloads

An Ansible collection that provides roles for deploying AI-related infrastructure and workloads to Red Hat OpenShift clusters. This collection simplifies the deployment of GPU operators, AI platforms, and supporting services required for AI/ML workloads on OpenShift.

## Table of Contents

- [Overview](#overview)
- [Requirements](#requirements)
- [Installation](#installation)
- [Roles](#roles)
- [Usage](#usage)
- [Variables and Configuration](#variables-and-configuration)
- [Examples](#examples)
- [Contributing](#contributing)
- [License](#license)

## Overview

The `agnosticd.ai_workloads` collection provides production-ready Ansible roles for deploying AI infrastructure on OpenShift. All roles support:

- **Idempotent operations**: Safe to run multiple times
- **Lifecycle management**: Both provision (`ACTION=provision`) and cleanup (`ACTION=destroy`)
- **Catalog snapshots**: Support for air-gapped and version-pinned deployments
- **Flexible configuration**: Sensible defaults with extensive customization options
- **OpenShift native**: Follows Operator Lifecycle Manager (OLM) best practices

## Requirements

### Control Node Requirements

- Ansible Core 2.15 or higher
- Python 3.9 or higher

### Ansible Collections

- `kubernetes.core` (>= 3.0.0)
- `community.general`
- `agnosticd.core` (for user info reporting)

### OpenShift Cluster Requirements

- Red Hat OpenShift Container Platform 4.14 or higher
- Cluster admin privileges for operator installation
- Access to OpenShift OperatorHub (or configured catalog snapshots)

### Authentication

- Valid `oc` or `kubectl` CLI configuration
- Active session with cluster admin permissions
- Or appropriate kubeconfig file configured

## Installation

### From Ansible Galaxy

```bash
ansible-galaxy collection install agnosticd.ai_workloads
```

### From Source

```bash
git clone <repository-url>
cd ai_workloads
ansible-galaxy collection build
ansible-galaxy collection install agnosticd-ai_workloads-*.tar.gz
```

## Roles

### ocp4_workload_nvidia_gpu_operator

Deploys the NVIDIA GPU Operator to enable GPU support on OpenShift worker nodes.

**Key Features:**
- Requires Node Feature Discovery (NFD) operator pre-installed
- Configures GPU ClusterPolicy for node management
- Optional DCGM (Data Center GPU Manager) Exporter dashboard
- Supports GPU monitoring and metrics collection

**Prerequisites:** Node Feature Discovery operator must be installed and configured.

**Typical Use Cases:**
- Enabling GPU acceleration for AI/ML workloads
- Training deep learning models
- Running GPU-accelerated inference workloads

### ocp4_workload_openshift_ai

Deploys Red Hat OpenShift AI (formerly Red Hat OpenShift Data Science) platform.

**Key Features:**
- Installs rhods-operator via OLM
- Configures DataScienceCluster with selectable components
- Supports both OpenShift AI 2.x and 3.x versions
- Automatic version detection and appropriate resource creation
- Configurable component management states (Managed, Removed)

**Available Components:**
- Dashboard: Web UI for data scientists
- Workbenches: Jupyter notebook environments
- Data Science Pipelines: Kubeflow Pipelines integration
- Model Mesh Serving: Multi-model serving infrastructure
- Model Registry: Centralized model versioning
- LLama Stack Operator: LLM deployment support
- AI Pipelines: Advanced pipeline workflows

**Typical Use Cases:**
- Setting up end-to-end ML platform
- Providing notebook environments for data science teams
- Deploying model serving infrastructure

### ocp4_workload_ols

Deploys OpenShift Lightspeed (OLS) - AI-powered assistance for OpenShift.

**Key Features:**
- Multiple AI platform support: Azure OpenAI, OpenAI, Watson, OpenShift AI
- Token-based authentication for Azure AI Cognitive Services
- API key authentication for other platforms
- Automatic secret management for credentials
- Flexible OLSConfig generation based on platform

**Authentication Methods:**
- Azure token-based with automatic token refresh
- Static API key for OpenAI, Watson
- Integration with OpenShift AI deployed models

**Typical Use Cases:**
- Providing AI assistance for OpenShift operations
- Integrating with organizational AI platforms
- Testing OLS with different LLM backends

### ocp4_workload_s3_model_copy_to_minio

Copies ML models from external S3-compatible storage to internal MinIO instance.

**Key Features:**
- Copies models from external S3 to cluster-internal MinIO
- Supports multiple model folders/directories
- Configurable source and destination credentials
- Useful for air-gapped or isolated clusters

**Typical Use Cases:**
- Pre-loading models into cluster-internal storage
- Migrating models between storage systems
- Supporting disconnected/air-gapped deployments

### ocp4_workload_toolhive

Deploys ToolHive, a platform for managing and executing AI agent tools.

**Key Features:**
- Installs ToolHive operator via Helm charts
- Manages custom resource definitions (CRDs)
- Configurable operator and runner images
- Registry API for tool discovery

**Typical Use Cases:**
- Building AI agent workflows
- Managing reusable AI tools and functions
- Orchestrating multi-tool AI applications

## Usage

### Basic Role Usage

All roles follow a consistent pattern with `ACTION` variable controlling behavior:

```yaml
---
- name: Deploy OpenShift AI
  hosts: localhost
  gather_facts: false
  vars:
    ACTION: provision  # or "destroy" to remove
  roles:
    - agnosticd.ai_workloads.ocp4_workload_openshift_ai
```

### Using in a Playbook

```yaml
---
- name: Setup AI Infrastructure on OpenShift
  hosts: localhost
  gather_facts: false
  vars:
    ACTION: provision

  tasks:
    - name: Deploy NVIDIA GPU Operator
      ansible.builtin.include_role:
        name: agnosticd.ai_workloads.ocp4_workload_nvidia_gpu_operator
      vars:
        ocp4_workload_nvidia_gpu_operator_channel: v24.9

    - name: Deploy OpenShift AI
      ansible.builtin.include_role:
        name: agnosticd.ai_workloads.ocp4_workload_openshift_ai
      vars:
        ocp4_workload_openshift_ai_channel: stable
        ocp4_workload_openshift_ai_dashboard: Managed
        ocp4_workload_openshift_ai_workbenches: Managed
```

### Command Line Execution

```bash
# Provision a workload
ansible-playbook playbook.yml -e ACTION=provision

# Remove a workload
ansible-playbook playbook.yml -e ACTION=destroy
```

## Variables and Configuration

### Common Variable Patterns

All roles follow these variable naming conventions:

#### Channel Selection
```yaml
ocp4_workload_<role_name>_channel: stable
```

#### Install Plan Approval
```yaml
ocp4_workload_<role_name>_automatic_install_plan_approval: true
ocp4_workload_<role_name>_starting_csv: ""  # Empty for latest
```

#### Catalog Snapshot Support
```yaml
ocp4_workload_<role_name>_use_catalog_snapshot: false
ocp4_workload_<role_name>_catalogsource_name: redhat-operators-snapshot-<name>
ocp4_workload_<role_name>_catalog_snapshot_image: quay.io/org/catalog
ocp4_workload_<role_name>_catalog_snapshot_image_tag: v4.19_2025_07_21
```

### Role-Specific Variables

Each role provides detailed variable documentation in its `defaults/main.yml` file. Refer to role-specific README files for comprehensive variable lists and descriptions.

## Examples

### Example 1: Full AI Stack with GPUs

```yaml
---
- name: Deploy Complete AI Stack
  hosts: localhost
  gather_facts: false
  vars:
    ACTION: provision

  tasks:
    # Assumes NFD operator already installed

    - name: Deploy NVIDIA GPU Operator
      ansible.builtin.include_role:
        name: agnosticd.ai_workloads.ocp4_workload_nvidia_gpu_operator
      vars:
        ocp4_workload_nvidia_gpu_operator_channel: v24.9
        ocp4_workload_nvidia_gpu_operator_create_dashboard: true

    - name: Deploy OpenShift AI with GPU support
      ansible.builtin.include_role:
        name: agnosticd.ai_workloads.ocp4_workload_openshift_ai
      vars:
        ocp4_workload_openshift_ai_channel: stable
        ocp4_workload_openshift_ai_deploy_dsc: true
        ocp4_workload_openshift_ai_dashboard: Managed
        ocp4_workload_openshift_ai_workbenches: Managed
        ocp4_workload_openshift_ai_datasciencepipelines: Managed
```

### Example 2: OpenShift Lightspeed with Azure OpenAI

```yaml
---
- name: Deploy OpenShift Lightspeed
  hosts: localhost
  gather_facts: false
  vars:
    ACTION: provision
    ocp4_workload_ai_platform: azure

  tasks:
    - name: Deploy OLS with Azure OpenAI
      ansible.builtin.include_role:
        name: agnosticd.ai_workloads.ocp4_workload_ols
      vars:
        ocp4_workload_ols_install_operator: true
        ocp4_workload_ols_get_token: true
        ocp4_workload_ols_azure_tenant_id: "{{ azure_tenant_id }}"
        ocp4_workload_ols_main_client_id: "{{ azure_client_id }}"
        ocp4_workload_ols_main_client_secret: "{{ azure_client_secret }}"
```

### Example 3: Using Catalog Snapshots

```yaml
---
- name: Deploy OpenShift AI with Catalog Snapshot
  hosts: localhost
  gather_facts: false
  vars:
    ACTION: provision

  tasks:
    - name: Deploy OpenShift AI
      ansible.builtin.include_role:
        name: agnosticd.ai_workloads.ocp4_workload_openshift_ai
      vars:
        ocp4_workload_openshift_ai_channel: stable
        ocp4_workload_openshift_ai_use_catalog_snapshot: true
        ocp4_workload_openshift_ai_catalogsource_name: rhoai-snapshot-2025-07
        ocp4_workload_openshift_ai_catalog_snapshot_image: quay.io/org/olm-snapshot
        ocp4_workload_openshift_ai_catalog_snapshot_image_tag: v4.19_2025_07_21
        ocp4_workload_openshift_ai_automatic_install_plan_approval: false
        ocp4_workload_openshift_ai_starting_csv: rhods-operator.2.25.0
```

### Example 4: Model Copy to Internal Storage

```yaml
---
- name: Copy Models to MinIO
  hosts: localhost
  gather_facts: false
  vars:
    ACTION: provision

  tasks:
    - name: Copy models from external S3 to MinIO
      ansible.builtin.include_role:
        name: agnosticd.ai_workloads.ocp4_workload_s3_model_copy_to_minio
      vars:
        ocp4_workload_s3_model_copy_to_minio_project: ic-shared-minio
        ocp4_workload_s3_model_copy_to_minio_bucket: models
        ocp4_workload_s3_model_copy_to_minio_root_user: "{{ minio_user }}"
        ocp4_workload_s3_model_copy_to_minio_root_password: "{{ minio_password }}"
        ocp4_workload_s3_model_copy_to_minio_s3_access_key: "{{ s3_access }}"
        ocp4_workload_s3_model_copy_to_minio_s3_secret_key: "{{ s3_secret }}"
        ocp4_workload_s3_model_copy_to_minio_s3_bucket: external-models
        ocp4_workload_s3_model_copy_to_minio_s3_folders:
          - llama-3.1-8b
          - granite-7b
          - mistral-7b
```

## Project Structure

```
ai_workloads/
├── README.md                          # This file
├── AGENTS.md                          # Developer guidelines and best practices
├── galaxy.yml                         # Collection metadata
├── meta/
│   └── runtime.yml                    # Runtime requirements
└── roles/
    ├── ocp4_workload_nvidia_gpu_operator/
    │   ├── defaults/
    │   │   └── main.yml              # Default variables
    │   ├── files/
    │   │   └── nvidia_gpu_clusterpolicy.json
    │   ├── meta/
    │   │   └── main.yml              # Role metadata
    │   └── tasks/
    │       ├── main.yml              # Entry point
    │       ├── workload.yml          # Provisioning tasks
    │       └── remove_workload.yml   # Cleanup tasks
    │
    ├── ocp4_workload_openshift_ai/
    │   ├── defaults/main.yml
    │   ├── meta/main.yml
    │   ├── tasks/
    │   │   ├── main.yml
    │   │   ├── workload.yml
    │   │   └── remove_workload.yml
    │   └── templates/
    │       ├── redhat-datasciencecluster.yaml.j2
    │       └── rhods-dashboard-route.yaml.j2
    │
    ├── ocp4_workload_ols/
    │   ├── defaults/main.yaml
    │   ├── meta/main.yml
    │   ├── tasks/
    │   │   ├── main.yaml
    │   │   ├── workload.yaml
    │   │   ├── remove_workload.yaml
    │   │   ├── install_ols_operator.yaml
    │   │   └── setup_azure_token.yaml
    │   └── templates/
    │       ├── create_secret.yaml.j2
    │       ├── create_secret_token.yaml.j2
    │       ├── create_azure_api_token.yaml.j2
    │       └── install_*_olsconfig.yaml.j2
    │
    ├── ocp4_workload_s3_model_copy_to_minio/
    │   ├── defaults/main.yml
    │   ├── meta/main.yml
    │   └── tasks/
    │       ├── main.yml
    │       ├── workload.yml
    │       └── remove_workload.yml
    │
    └── ocp4_workload_toolhive/
        ├── defaults/main.yml
        ├── meta/main.yml
        └── tasks/
            ├── main.yml
            ├── workload.yml
            └── remove_workload.yml
```

## Contributing

Contributions are welcome! When contributing to this collection:

1. Follow the guidelines in [AGENTS.md](AGENTS.md)
2. Ensure all roles follow the established patterns
3. Test both provision and destroy actions
4. Update documentation for any new variables or features
5. Ensure code passes static checks (ansible-lint, yamllint)

### Development Guidelines

- Use Fully Qualified Collection Names (FQCN) for all modules
- Prefix all role variables with `ocp4_workload_<role_name>_`
- Document all variables in `defaults/main.yml`
- Ensure idempotency of all tasks
- Support catalog snapshot deployment patterns
- Follow the standard role structure

See [AGENTS.md](AGENTS.md) for detailed Ansible best practices and patterns used in this collection.

## Testing

### Static Analysis

```bash
# Lint all roles
ansible-lint roles/

# YAML validation
yamllint .
```

### Manual Testing

```bash
# Test provision
ansible-playbook test-playbook.yml -e ACTION=provision

# Verify deployment
oc get operators
oc get pods -A

# Test cleanup
ansible-playbook test-playbook.yml -e ACTION=destroy
```

## Troubleshooting

### Common Issues

1. **Operator fails to install**
   - Verify cluster connectivity: `oc whoami`
   - Check OperatorHub availability: `oc get packagemanifests`
   - Review operator events: `oc get events -n <namespace>`

2. **Catalog snapshot not found**
   - Verify catalog snapshot image is accessible
   - Check CatalogSource status: `oc get catalogsource -n openshift-marketplace`
   - Ensure correct image registry credentials

3. **ClusterServiceVersion (CSV) not reaching Succeeded phase**
   - Check CSV status: `oc get csv -n <namespace>`
   - Review pod logs: `oc logs -n <namespace> <operator-pod>`
   - Verify required resources (CPU, memory) available

4. **Authentication failures (OLS)**
   - Verify secrets are correctly created: `oc get secrets -n openshift-lightspeed`
   - Check credential validity
   - Review OLS operator logs for authentication errors

## Support

For issues, questions, or contributions:

- Report bugs via GitHub Issues
- Check existing documentation in role README files
- Review [AGENTS.md](AGENTS.md) for development guidelines

## License

GPL-2.0-or-later

## Authors

- Wolfgang Kulhanek <wkulhane@redhat.com>

## References

- [Red Hat OpenShift AI Documentation](https://docs.redhat.com/en/documentation/red_hat_openshift_ai_self-managed)
- [NVIDIA GPU Operator Documentation](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/overview.html)
- [OpenShift Lightspeed Documentation](https://docs.redhat.com/en/documentation/openshift_lightspeed)
- [Operator Lifecycle Manager (OLM)](https://olm.operatorframework.io/)
- [AgnosticD](https://github.com/redhat-cop/agnosticd)
