# Ansible Role: ansible-role-awx

This Ansible role installs and configures AWX (Ansible Tower) on a Kubernetes cluster using K3s and the AWX Operator. It also sets up cert-manager for managing SSL certificates with Let's Encrypt.

## Requirements

- Ansible 2.9 or higher
- Kubernetes cluster (K3s recommended)
- Helm 3.x
- Access to the internet for downloading images and dependencies

## Role Variables

The following variables can be defined in `defaults/main.yml` or overridden in your playbook:

- `awx_operator_tag`: The version of the AWX Operator to install (default: `2.19.0`).
- `awx_service_type`: The type of service to expose AWX (default: `nodeport`).
- `awx_nodeport_port`: The port to use for the NodePort service (default: `32000`).
- `letsencrypt_email`: Your email address for Let's Encrypt notifications.

## Dependencies

This role has no dependencies on other roles.

## Example Playbook

```yaml
- hosts: localhost
  roles:
    - ansible-role-awx
```

## License

MIT

## Author Information

This role was created in 2023 by [Your Name].