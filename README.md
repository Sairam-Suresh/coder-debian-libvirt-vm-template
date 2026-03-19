---
display_name: Debian Libvirt VMs
description: Provision persistent Debian VM workspaces on libvirt with Coder, Tailscale, and optional desktop/devcontainer support.
icon: ../../../site/static/icon/debian.svg
maintainer_github: sairam-suresh
verified: false
tags: [vm, debian, libvirt, coder, tailscale]
---

# Debian VM Workspaces on Libvirt

Provision full Debian virtual machines as Coder workspaces using libvirt. This template is designed for teams that want VM-level isolation, persistent user data, and flexible developer onboarding with optional GUI and devcontainer support.

> Note: Should not be used for Production.

## Highlights

- Full VM workspaces on KVM/libvirt (q35, host-passthrough CPU).
- Persistent user data disk mounted to /home and /root.
- Cloud-init driven bootstrap with Coder agent, Tailscale, and tooling install.
- Optional desktop stack (XFCE + KasmVNC + Chrome).
- Optional devcontainer support with rootless Podman.
- Optional repository bootstrap via Coder git-clone module.
- Workspace apps and integrations include code-server, JetBrains Gateway, git commit signing, Coder Copilot, and Antigravity MCP (GitHub MCP server).

## What This Template Creates

Core resources are defined in [main.tf](main.tf):

- One libvirt domain per workspace.
- An OS qcow2 disk backed by a Debian cloud image.
- A separate persistent userdata qcow2 disk.
- A cloud-init ISO built from [cloud-init.yaml.tftpl](cloud-init.yaml.tftpl).
- Coder agent metadata, apps, and workspace modules.

System networking for cloud-init is configured in [network_config.cfg](network_config.cfg).

## Workspace Parameters

Users can configure the workspace at creation time:

- vm_vcpu: vCPU count (1-16)
- vm_memory: memory in GiB (4-20)
- vm_disk_size: persistent userdata disk size in GiB
- install_de: enable desktop environment (XFCE/KasmVNC/Chrome)
- enable_git_clone: clone a repository on startup
- repo_url or manual_folder_name: workspace folder source
- enable_devcontainer: install rootless Podman + devcontainer tooling
- tooling: language/runtime list installed via Mise (Python, Rust, Node.js, Dart, Flutter, .NET)

## Security and Secrets

The Tailscale auth key is not hardcoded in Terraform.

- Variable is declared in [main.tf](main.tf) as sensitive: tailscale_auth_key
- Example values file: [secrets.auto.tfvars.example](secrets.auto.tfvars.example)
- Real secrets file expected at runtime: secrets.auto.tfvars (gitignored by [.gitignore](.gitignore))

You can provide this variable using either:

- secrets.auto.tfvars
- TF_VAR_tailscale_auth_key
- terraform apply -var "tailscale_auth_key=..."

## Prerequisites

1. A Coder deployment with template admin rights.
2. A Linux host with libvirt/KVM available to the Coder provisioner.
3. A Debian cloud image available at the backing store path used in [main.tf](main.tf).
4. Tailscale auth key with permissions appropriate for your tailnet policy.
5. Access to required Coder external auth integration (GitHub), if using MCP integration.

## Quick Start

1. Clone this repository.
2. Create a real secrets file from the example:

```bash
cp secrets.auto.tfvars.example secrets.auto.tfvars
```

3. Edit secrets.auto.tfvars and set tailscale_auth_key.
4. Validate the template:

```bash
terraform init
terraform validate
```

5. Push to Coder:

```bash
coder templates push <template-name> --yes
```

## CI Publish Workflow

Automated template testing and publishing is defined in [.github/workflows/publish-template.yaml](.github/workflows/publish-template.yaml).

Current workflow behavior:

- Authenticates GitHub Actions runner to Tailscale.
- Creates secrets.auto.tfvars from GitHub secret TAILSCALE_AUTH_KEY.
- Validates Terraform.
- Pushes a new Coder template version.
- Creates a test workspace, runs commands, then deletes it.
- Promotes the version on success.

Required GitHub secrets:

- CODER_SESSION_TOKEN
- TAILSCALE_CI_AUTHKEY
- TAILSCALE_AUTH_KEY

## Notes for Operators

- The template rebuilds the OS disk when cloud-init template inputs change.
- Persistent developer data remains on the userdata volume across rebuilds.
- If you customize cloud-init behavior, verify which changes require a full disk replacement vs a normal restart.

## File Map

- [main.tf](main.tf): Terraform resources, Coder modules, variables, and workspace params.
- [cloud-init.yaml.tftpl](cloud-init.yaml.tftpl): VM bootstrap, package install, systemd units, and runtime setup.
- [network_config.cfg](network_config.cfg): cloud-init network configuration.
- [.github/workflows/publish-template.yaml](.github/workflows/publish-template.yaml): CI test and publish pipeline.
- [secrets.auto.tfvars.example](secrets.auto.tfvars.example): local/CI example for secret injection.
