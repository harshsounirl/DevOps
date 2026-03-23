# DevOps
Development and Operations Automation

---

## Overview

This repository contains automation workflows for infrastructure provisioning and software deployment using **Ansible** and **Proxmox**, with automated builds via API and platform-specific package management.

---

## Technology Stack

| Component        | Tool/Platform              |
|------------------|----------------------------|
| Hypervisor       | Proxmox VE                 |
| Automation       | Ansible                    |
| Windows Packages | Chocolatey (choco)         |
| RHEL 8/9 Packages| DNF                        |
| API Integration  | Proxmox REST API           |

---

## Proxmox + Ansible Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        CONTROL NODE                             │
│                    (Ansible Controller)                         │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                     PROXMOX VE HOST                             │
│              REST API  (https://<host>:8006/api2/json)          │
└────────┬───────────────┬───────────────────┬────────────────────┘
         │               │                   │
         ▼               ▼                   ▼
    ┌─────────┐    ┌─────────┐         ┌─────────┐
    │  VM 1   │    │  VM 2   │   ...   │  VM N   │
    │ Windows │    │ RHEL 8/9│         │ RHEL 8/9│
    └─────────┘    └─────────┘         └─────────┘
```

---

## Build Flow via Proxmox API

```mermaid
flowchart TD
    A([Start]) --> B[Ansible Playbook Triggered]
    B --> C[Authenticate to Proxmox API\nPOST /api2/json/access/ticket]
    C --> D{Auth\nSuccessful?}
    D -- No --> E[Abort & Alert]
    D -- Yes --> F[Clone VM Template\nPOST /api2/json/nodes/node/qemu/vmid/clone]
    F --> G[Configure VM Resources\nPUT /api2/json/nodes/node/qemu/vmid/config\ncores, memory, disk, network]
    G --> H[Start VM\nPOST /api2/json/nodes/node/qemu/vmid/status/start]
    H --> I[Wait for VM Agent / SSH Ready\nGET /api2/json/nodes/node/qemu/vmid/agent/ping]
    I --> J{OS\nType?}
    J -- Windows --> K[Run Windows\nSoftware Install Playbook]
    J -- RHEL 8/9 --> L[Run RHEL\nSoftware Install Playbook]
    K --> M[Post-Install Validation\nWindows]
    L --> N[Post-Install Validation\nRHEL]
    M --> O[Register VM in CMDB/Inventory]
    N --> O
    O --> P([End])
```

---

## Windows Software Installation via Chocolatey

```mermaid
flowchart TD
    A([Start Windows Playbook]) --> B[Ansible connects via WinRM]
    B --> C{Chocolatey\nInstalled?}
    C -- No --> D[Bootstrap Chocolatey\nSet-ExecutionPolicy & install script]
    C -- Yes --> E[Update Chocolatey\nchoco upgrade chocolatey -y]
    D --> E
    E --> F[Install Core Packages\nchoco install -y git curl wget 7zip]
    F --> G[Install Dev Tools\nchoco install -y vscode python nodejs]
    G --> H[Install Security Tools\nchoco install -y sysinternals openssl]
    H --> I[Apply Windows Updates\nwin_updates module]
    I --> J[Verify Installations\nwin_command: choco list --local-only]
    J --> K{All Packages\nInstalled?}
    K -- No --> L[Log Failures\n& Retry]
    L --> F
    K -- Yes --> M[Reboot if Required\nwin_reboot module]
    M --> N([Windows Setup Complete])
```

### Example Ansible Task — Windows (Chocolatey)

```yaml
- name: Install packages via Chocolatey
  chocolatey.chocolatey.win_chocolatey:
    name:
      - git
      - curl
      - wget
      - 7zip
      - vscode
      - python
      - nodejs
      - sysinternals
      - openssl
    state: present
  vars:
    ansible_connection: winrm
    ansible_winrm_transport: ntlm
```

---

## RHEL 8/9 Software Installation via DNF

```mermaid
flowchart TD
    A([Start RHEL Playbook]) --> B[Ansible connects via SSH]
    B --> C[Register with Red Hat\nsubscription-manager register]
    C --> D{Subscription\nActive?}
    D -- No --> E[Attach Subscription Pool\nsubscription-manager attach --auto]
    D -- Yes --> F[Enable Required Repos\ndnf config-manager --enable rhel-8/9-appstream]
    E --> F
    F --> G[Update All Packages\ndnf update -y]
    G --> H[Install Base Packages\ndnf install -y git curl wget tar unzip]
    H --> I[Install Dev Tools\ndnf groupinstall -y 'Development Tools']
    I --> J[Install Security & Monitoring\ndnf install -y openssl aide firewalld]
    J --> K[Enable & Start Services\nsystemctl enable/start firewalld]
    K --> L[Apply SELinux Policies\nsemanage & restorecon]
    L --> M[Verify Installations\ndnf list installed]
    M --> N{All Packages\nInstalled?}
    N -- No --> O[Log Failures\n& Retry]
    O --> H
    N -- Yes --> P[Reboot if Kernel Updated\nansible reboot module]
    P --> Q([RHEL Setup Complete])
```

### Example Ansible Task — RHEL 8/9 (DNF)

```yaml
- name: Install packages via DNF
  ansible.builtin.dnf:
    name:
      - git
      - curl
      - wget
      - tar
      - unzip
      - openssl
      - aide
      - firewalld
    state: present
    update_cache: true

- name: Install Development Tools group
  ansible.builtin.dnf:
    name: "@Development Tools"
    state: present

- name: Enable and start firewalld
  ansible.builtin.systemd:
    name: firewalld
    enabled: true
    state: started
```

---

## End-to-End Automation Flow

```mermaid
flowchart LR
    A[Git Push /\nManual Trigger] --> B[Ansible Controller]
    B --> C[Proxmox API\nVM Provisioning]
    C --> D{Detect OS}
    D -->|Windows| E[WinRM\nConnection]
    D -->|RHEL 8/9| F[SSH\nConnection]
    E --> G[Chocolatey\nPackage Install]
    F --> H[DNF\nPackage Install]
    G --> I[Windows\nValidation]
    H --> J[RHEL\nValidation]
    I --> K[Inventory\nUpdate]
    J --> K
    K --> L[Notification\nSlack/Email]
```

---

## Directory Structure

```
DevOps/
├── inventories/
│   ├── proxmox/          # Dynamic inventory from Proxmox API
│   ├── windows/          # Windows hosts
│   └── rhel/             # RHEL 8/9 hosts
├── playbooks/
│   ├── provision_vm.yml  # Proxmox VM creation via API
│   ├── windows_setup.yml # Chocolatey software install
│   └── rhel_setup.yml    # DNF software install
├── roles/
│   ├── proxmox_api/      # Proxmox API interaction role
│   ├── chocolatey/       # Windows package management role
│   └── dnf_packages/     # RHEL package management role
├── vars/
│   ├── windows_packages.yml
│   └── rhel_packages.yml
└── README.md
```

---

## Prerequisites

- Proxmox VE 7+ with API token configured
- Ansible 2.14+ on control node
- WinRM enabled on Windows targets
- SSH key-based auth on RHEL targets
- Red Hat subscription for RHEL package repos
