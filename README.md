# Cisco Automation with Ansible and Semaphore

This repository is a complete, reproducible tutorial for the Cisco automation lab built with a physical PC, VS Code Remote-SSH, a Windows Server VMware VM, WSL2 Ubuntu, Docker, the existing Semaphore container, Ansible, CORE BABA, CORE TAAS, and the CUCM/CME router.

## New to Ansible, Linux, or Cisco Automation?

Begin with [Start Here — Beginner Orientation](START-HERE.md). It explains the terminology, the five places where commands are entered, the difference between read-only and configuration tasks, and the checkpoint that must pass before each next step.

The tutorials also include visible **Screenshot guide** segments beneath steps where an image would help. Completed examples are embedded in the matching setup guides; any remaining screenshot is clearly marked pending and identifies what to capture, what success must show, and what sensitive information must be hidden.

Do not start by pressing a configuration button in Semaphore. First identify the assigned monitor number, confirm the physical switches and cabling, back up the switches, and complete the read-only connection tests.

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
  ├── CORE TAAS 10.~~.1.2
  └── CUCM/CME 10.~~.100.8 (through BABA routing and OSPF)
```

## Start Here

Follow these guides in order:

1. [Beginner orientation](START-HERE.md)
2. [WSL2 and Ubuntu](Setup/WSL2-Ubuntu.md)
3. [Verify the existing Docker, Ansible, and Semaphore setup](Setup/Docker-Semaphore.md)
4. [VS Code Remote-SSH and Windows VM YAML workflow](Setup/VSCode-Remote-SSH.md)
5. [Semaphore project, inventory, repository, and templates](Setup/Semaphore-Project.md)
6. [CORE BABA tutorial](CORE-BABA/README.md)
7. [CORE TAAS tutorial](CORE-TAAS/README.md)
8. [CUCM / Cisco Unified CallManager Express tutorial](CUCM/README.md)
9. [Reusable multi-monitor deployment](Reusable-Multi-Monitor/README.md)
10. [Troubleshooting](Troubleshooting.md)

## Repository Contents

```text
Cisco-Ansible/
├── README.md
├── START-HERE.md
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
│   ├── baba-ospf.yml
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
├── CUCM/
│   ├── README.md
│   ├── show-version.yml
│   ├── cucm-base.yml
│   ├── cucm-ospf.yml
│   ├── cucm-analog-phones.yml
│   ├── cucm-telephony-service.yml
│   └── cucm-video.yml
└── Reusable-Multi-Monitor/
    ├── README.md
    ├── inventory.example.ini
    ├── CORE-BABA/ (six variable-based YAML files)
    └── CORE-TAAS/ (four variable-based YAML files)
```

## Safety and Scope

- BABA playbooks use `hosts: baba`; TAAS playbooks use `hosts: taas`; CUCM playbooks use `hosts: cucm`. This prevents switch-only or voice-router-only configuration from being sent to the wrong device.
- Configuration commands are derived from `DAY1-May5-SirRob.txt`. SSH commands are documented separately as the minimum automation prerequisite.
- Sir Rob's original VTY `password pass` and `login` commands remain visible in the source-reference sections. The runnable base playbooks intentionally use `login local` and `transport input ssh` on VTY 0-4 and 5-14 because the live lab proved that plain `login` prevented Semaphore from authenticating with the local `admin` account. This is clearly labeled as an Ansible/SSH prerequisite.
- `baba-camera-dhcp.yml` is deliberately marked **DO NOT RUN** until real camera client identifiers replace `001a.xxxx.yyyy`; an assertion also stops the play before configuration while the placeholders remain or the two values match.
- `interface.yml`, `baba.yml`, and other earlier test files are not part of the final Day1SirRob playbook set.
- The complete Day 1 topology requires the documented BABA-to-CUCM OSPF relationship before the PC, WSL, or Semaphore remote-access test. A successful local BABA VLAN 100 ping is not the final checkpoint.
- CUCM telephony reset, placeholder phone MAC addresses, trusted-list, remote dial peers, IVR, and SIP sections require explicit voice/network-owner review before use.
- Back up the device configuration and run the read-only `show-version.yml` test before any configuration playbook.
- The original Day 1 files retain `~~` for exact reusable documentation. Use `Reusable-Multi-Monitor/` when managing several monitor numbers with one set of playbooks.

## Choose One Base-Configuration Path

Do not treat the console bootstrap and the complete Sir Rob base block as two required configurations. Choose one path for each switch:

| Path | What you enter manually | Do you run `baba-base.yml` or `taas-base.yml`? |
|---|---|---|
| **A — Automation (recommended)** | Only the management-IP, local-user, and SSH bootstrap needed for Ansible connectivity. | **Yes.** Run Show Version first, then the matching base playbook. |
| **B — Complete manual base** | The complete monitor-corrected Sir Rob base configuration. | **No.** Skip the base playbook, run Show Version, verify the configuration, and continue with the next applicable playbook. |

Some commands in the documented bootstrap overlap harmlessly with the base playbook. That overlap makes the playbook idempotently confirm the intended state; it is not an instruction to paste Sir Rob's complete base block and then apply it again.

Whichever path is used, VTY 0-4 and 5-14 must use `login local` and `transport input ssh` before Semaphore can authenticate. Sir Rob's plain `login` source lines must not replace the working SSH prerequisite.

`Configure Cisco Interface`, `interface.yml`, `baba.yml`, and similarly named earlier tests are not part of either final path. Do not use them for a new BABA or TAAS deployment.

## Recommended Run Order

1. Bootstrap management IP and SSH on both Cisco devices from the Cisco console.
2. Build `inventory.ini`, replace every `~~`, and copy the same BABA/TAAS group definitions into the working Semaphore GUI inventory.
3. Run both `show-version.yml` playbooks.
4. If using Path A, run `taas-base.yml` and `baba-base.yml`, then repeat both read-only connection tests. If the complete base was applied manually under Path B, skip the matching base playbook.
5. Run `taas-trunk.yml`, then the BABA trunk/LACP playbook on the other side.
6. Run `taas-lacp.yml`, then verify the bundle on both switches.
7. Run BABA-only `baba-dhcp.yml` and `baba-vlans.yml`.
8. Do not run `baba-camera-dhcp.yml` until the real identifiers are known.
9. Run `baba-ospf.yml`, complete the matching CUCM console OSPF bootstrap, require a FULL BABA/CUCM neighbor, and pass the PC/WSL SSH test before any CUCM voice configuration.

## Configuration Source

The device configuration follows [DAY1-May5-SirRob.txt](https://github.com/carllperez/ccna2/blob/main/DAY1-May5-SirRob.txt). The source assigns base Layer 3 configuration to CORE TAAS and CORE BABA, trunk/LACP to both switches, and DHCP/VLAN/access-port/camera-reservation configuration to CORE BABA.
