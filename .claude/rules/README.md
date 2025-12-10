# Claude Rules for AI Workloads Collection

This directory contains modular rules for working with the `agnosticd.ai_workloads` Ansible collection. Each file focuses on a specific aspect of the project.

## Rule Files

### [ansible.md](ansible.md)
Core Ansible conventions and best practices:
- FQCN usage
- YAML style guidelines
- Task naming conventions
- Variable naming patterns
- Idempotency requirements
- Error handling patterns

### [openshift.md](openshift.md)
OpenShift and Operator Lifecycle Manager (OLM) specific patterns:
- Operator installation workflow
- Subscription configuration
- Catalog snapshot support
- Resource readiness checks
- Version detection strategies
- Common troubleshooting

### [role-structure.md](role-structure.md)
Standard role structure and organization:
- Directory layout requirements
- File purposes and conventions
- Standard task organization
- Variable documentation format
- Adding new roles checklist

### [testing.md](testing.md)
Testing guidelines and workflows:
- Static analysis requirements
- Manual testing procedures
- Idempotency verification
- Catalog snapshot testing
- Debugging techniques
- CI/CD considerations

### [variables.md](variables.md)
Variable naming and documentation standards:
- Naming conventions
- Internal variable patterns
- Required standard variables
- Documentation format
- Secret handling
- Validation patterns

### [security.md](security.md)
Security best practices:
- Secrets management
- RBAC and permissions
- Network security
- Audit and logging
- Container image security
- Security scanning
- Compliance considerations

## Using These Rules

Claude Code automatically loads these rules when working in this repository. The modular structure allows for:

- **Focused context**: Each file contains related guidelines
- **Easy maintenance**: Update specific areas without affecting others
- **Clear organization**: Find relevant guidelines quickly
- **Reusability**: Reference specific rule files as needed

## Rule Organization

Rules are organized by concern:
- **ansible.md**: General Ansible practices applicable to any playbook
- **openshift.md**: OpenShift-specific patterns for this collection
- **role-structure.md**: Structural standards for this collection
- **testing.md**: Testing workflows and validation
- **variables.md**: Variable conventions and documentation
- **security.md**: Security practices for all roles

## Adding New Rules

When adding new rule files:

1. Create a descriptive filename (e.g., `documentation.md`, `ci-cd.md`)
2. Follow the same format as existing rules
3. Include practical examples
4. Reference from main CLAUDE.md if needed
5. Update this README with the new rule file

## Related Documentation

- [AGENTS.md](../../AGENTS.md) - Comprehensive developer guidelines
- [README.md](../../README.md) - Collection documentation
- [galaxy.yml](../../galaxy.yml) - Collection metadata
