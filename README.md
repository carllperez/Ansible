# Cisco Automation with Ansible and Semaphore

This repository is a complete, reproducible tutorial for the Cisco automation lab built with a physical PC, VS Code Remote-SSH, a Windows Server VMware VM, WSL2 Ubuntu, Docker, the existing Semaphore container, Ansible, CORE BABA, and CORE TAAS.

VS Code runs on the physical PC and saves the working YAML over SSH into `C:\Users\Administrator\ansible-lab` on the Windows Server VM. WSL sees the same folder at `/mnt/c/Users/Administrator/ansible-lab`. Semaphore stays in its existing Docker container inside Ubuntu and is not rebuilt or migrated.

`~~` is the monitor/student-number placeholder. Before using any inventory or playbook, replace every `~~` with the assigned number. For monitor 71, `10.~~.1.4` becomes `10.71.1.4` and `vlan ~~` becomes `vlan 71`.

## Architecture

```text
PHYSICAL PC
  ├── VS Code Remote-SSH
  ├── edit/save YAML over SSH
  └── Browser to Semaphore
          |
          v
WINDOWS SERVER VM (208.8.8.200)
  └── C:\Users\Administrator\ansible-lab (source-of-truth YAML)
          |
          v
WSL2 UBUNTU
  ├── /mnt/c/Users/Administrator/ansible-lab (same VM folder)
  ├── Docker
  └── existing Semaphore container :3000
          |
          v
ANSIBLE OVER SSH
  ├── CORE BABA 10.~~.1.4
  └── CORE TAAS 10.~~.1.2
```

## Start Here

Follow these guides in order:

1. [WSL2 and Ubuntu](Setup/WSL2-Ubuntu.md)
2. [Verify the existing Docker, Ansible, and Semaphore setup](Setup/Docker-Semaphore.md)
3. [VS Code Remote-SSH and Windows VM YAML workflow](Setup/VSCode-Remote-SSH.md)
4. [Semaphore project, inventory, repository, and templates](Setup/Semaphore-Project.md)
5. [CORE BABA tutorial](CORE-BABA/README.md)
6. [CORE TAAS tutorial](CORE-TAAS/README.md)
7. [Reusable multi-monitor deployment](Reusable-Multi-Monitor/README.md)
8. [Troubleshooting](Troubleshooting.md)

## Repository Contents

```text
Cisco-Ansible/
├── README.md
├── Troubleshooting.md
├── inventory.example.ini
├── Setup/
│   ├── WSL2-Ubuntu.md
│   ├── Docker-Semaphore.md
│   ├── VSCode-Remote-SSH.md
│   └── Semaphore-Project.md
├── CORE-BABA/
│   ├── README.md
│   ├── show-version.yml
│   ├── baba-base.yml
│   ├── baba-lacp.yml
│   ├── baba-dhcp.yml
│   ├── baba-vlans.yml
│   └── baba-camera-dhcp.yml
├── CORE-TAAS/
│   ├── README.md
│   ├── show-version.yml
│   ├── taas-base.yml
│   ├── taas-trunk.yml
│   └── taas-lacp.yml
└── Reusable-Multi-Monitor/
    ├── README.md
    ├── inventory.example.ini
    ├── CORE-BABA/ (six variable-based YAML files)
    └── CORE-TAAS/ (four variable-based YAML files)
```

## Safety and Scope

- BABA playbooks use `hosts: baba`; TAAS playbooks use `hosts: taas`. This prevents BABA-only DHCP and access-port configuration from being sent to TAAS.
- Configuration commands are derived from `DAY1-May5-SirRob.txt`. SSH commands are documented separately as the minimum automation prerequisite.
- SSH is documented as a prerequisite. Both Day 1 base playbooks keep Sir Rob's VTY `password pass` and `login` commands; they do not add unrelated device configuration.
- `baba-camera-dhcp.yml` is deliberately marked **DO NOT RUN** until real camera client identifiers replace `001a.xxxx.yyyy`.
- `interface.yml`, `baba.yml`, and other earlier test files are not part of the final Day1SirRob playbook set.
- Back up the device configuration and run the read-only `show-version.yml` test before any configuration playbook.
- The original Day 1 files retain `~~` for exact reusable documentation. Use `Reusable-Multi-Monitor/` when managing several monitor numbers with one set of playbooks.

## Recommended Run Order

1. Bootstrap management IP and SSH on both Cisco devices from the Cisco console.
2. Build `inventory.ini` and replace every `~~`.
3. Run both `show-version.yml` playbooks.
4. Run `taas-base.yml` and `baba-base.yml`, then repeat both read-only connection tests.
5. Run `taas-trunk.yml`, then the BABA trunk/LACP playbook on the other side.
6. Run `taas-lacp.yml`, then verify the bundle on both switches.
7. Run BABA-only `baba-dhcp.yml` and `baba-vlans.yml`.
8. Do not run `baba-camera-dhcp.yml` until the real identifiers are known.

## Configuration Source

The device configuration follows [DAY1-May5-SirRob.txt](https://github.com/carllperez/ccna2/blob/main/DAY1-May5-SirRob.txt). The source assigns base Layer 3 configuration to CORE TAAS and CORE BABA, trunk/LACP to both switches, and DHCP/VLAN/access-port/camera-reservation configuration to CORE BABA.
