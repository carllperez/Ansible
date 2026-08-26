# Reusable Multi-Monitor Deployment

This directory provides the safe variable-based version of the Day 1 CORE BABA and CORE TAAS playbooks. The original `CORE-BABA/` and `CORE-TAAS/` files keep the requested `~~` documentation placeholder; these reusable files use `{{ monitor }}` so one playbook can support monitor 71, 72, 73, and other assigned numbers.

> **Advanced, optional path:** Complete one device with the original BABA or TAAS tutorial first. Use this directory after you understand the inventory, read-only test, backups, and verification steps. It does not rebuild or change the existing Semaphore installation.

## Where the Target Monitor Is Set

The target is set in two places for safety:

```ini
[baba]
baba72 ansible_host=10.72.1.4 monitor=72
```

- `monitor=72` stores the monitor number used by `{{ monitor }}` inside the YAML.
- `baba72` is the inventory name of that one switch.
- `ansible_host=10.72.1.4` is the reachable management address.

At run time, both command options must name the same inventory device:

```text
--extra-vars target_device=baba72 --limit baba72
```

`target_device=baba72` tells the playbook which device was approved. `--limit baba72` prevents Ansible from expanding the run to another inventory host. Do not type a separate monitor number at run time, do not replace `{{ monitor }}`, and do not use `~~` in the reusable YAML.

## Safety Design

Every configuration playbook:

- refuses to select any host unless `target_device` is supplied explicitly;
- confirms the selected host belongs to the correct `baba` or `taas` inventory group;
- reads `monitor` from the target host's inventory entry;
- confirms the value is an integer from 1 through 254;
- confirms the management address matches the device type and monitor number;
- uses `serial: 1` so switches are changed one at a time;
- stops the play on the first failed host;
- never accepts a runtime monitor number that could disagree with inventory.
- preserves the working SSH access by configuring both VTY 0-4 and VTY 5-14 with `login local` and `transport input ssh` in the reusable base playbooks.

Select one inventory hostname with both `--extra-vars target_device=baba72` and `--limit baba72`. The explicit target chooses the host; the limit is an additional guardrail.

## 1. Prerequisites

The target switch must already have its management address and SSH bootstrap. For BABA 72, Ansible must be able to reach `10.72.1.4`; for TAAS 72, it must reach `10.72.1.2`.

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
ping -c 4 10.72.1.4
ssh admin@10.72.1.4
```

If the switch is factory-default or its management address is changing, use the Cisco console first. Do not use a full base playbook to change the address carrying the active Ansible session.

## 2. Create the Multi-Monitor Inventory in VS Code

**Do in: PHYSICAL PC → VS Code Remote-SSH window → Windows Server VM folder**

Create this folder structure:

```text
C:\Users\Administrator\ansible-lab\Reusable-Multi-Monitor\
├── inventory.ini
├── CORE-BABA\
└── CORE-TAAS\
```

Copy `Reusable-Multi-Monitor/inventory.example.ini` into the VS Code file `C:\Users\Administrator\ansible-lab\Reusable-Multi-Monitor\inventory.ini`. Keep only real devices. Each inventory hostname must be unique, and each `monitor` must agree with its management address:

```ini
[baba]
baba72 ansible_host=10.72.1.4 monitor=72

[taas]
taas72 ansible_host=10.72.1.2 monitor=72
```

Do not put TAAS under `[baba]` or BABA under `[taas]`.

### Screenshot guide: Reusable inventory target

- **Capture:** VS Code showing only the relevant inventory group and one example entry, such as `baba72`.
- **Success must show:** the inventory hostname, matching `ansible_host`, and `monitor=72` on the same line.
- **Hide:** passwords, camera identifiers, and devices outside the tutorial example.
- **Status:** Screenshot pending.

## 3. Save the Reusable YAML Files through VS Code

Copy each repository YAML to the matching path in the VS Code Remote-SSH VM folder:

| Repository file | VS Code path on Windows Server VM |
|---|---|
| [CORE-BABA/show-version.yml](CORE-BABA/show-version.yml) | `C:\Users\Administrator\ansible-lab\Reusable-Multi-Monitor\CORE-BABA\show-version.yml` |
| [CORE-BABA/baba-base.yml](CORE-BABA/baba-base.yml) | `C:\Users\Administrator\ansible-lab\Reusable-Multi-Monitor\CORE-BABA\baba-base.yml` |
| [CORE-BABA/baba-lacp.yml](CORE-BABA/baba-lacp.yml) | `C:\Users\Administrator\ansible-lab\Reusable-Multi-Monitor\CORE-BABA\baba-lacp.yml` |
| [CORE-BABA/baba-dhcp.yml](CORE-BABA/baba-dhcp.yml) | `C:\Users\Administrator\ansible-lab\Reusable-Multi-Monitor\CORE-BABA\baba-dhcp.yml` |
| [CORE-BABA/baba-vlans.yml](CORE-BABA/baba-vlans.yml) | `C:\Users\Administrator\ansible-lab\Reusable-Multi-Monitor\CORE-BABA\baba-vlans.yml` |
| [CORE-BABA/baba-camera-dhcp.yml](CORE-BABA/baba-camera-dhcp.yml) | `C:\Users\Administrator\ansible-lab\Reusable-Multi-Monitor\CORE-BABA\baba-camera-dhcp.yml` |
| [CORE-TAAS/show-version.yml](CORE-TAAS/show-version.yml) | `C:\Users\Administrator\ansible-lab\Reusable-Multi-Monitor\CORE-TAAS\show-version.yml` |
| [CORE-TAAS/taas-base.yml](CORE-TAAS/taas-base.yml) | `C:\Users\Administrator\ansible-lab\Reusable-Multi-Monitor\CORE-TAAS\taas-base.yml` |
| [CORE-TAAS/taas-trunk.yml](CORE-TAAS/taas-trunk.yml) | `C:\Users\Administrator\ansible-lab\Reusable-Multi-Monitor\CORE-TAAS\taas-trunk.yml` |
| [CORE-TAAS/taas-lacp.yml](CORE-TAAS/taas-lacp.yml) | `C:\Users\Administrator\ansible-lab\Reusable-Multi-Monitor\CORE-TAAS\taas-lacp.yml` |

Do not replace `{{ monitor }}`. Ansible resolves it separately for every inventory host.

## 4. Copy to the Existing Semaphore Container

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
sudo docker exec -u 0 semaphore mkdir -p /ansible/reusable/CORE-BABA
sudo docker exec -u 0 semaphore mkdir -p /ansible/reusable/CORE-TAAS

sudo docker cp /mnt/c/Users/Administrator/ansible-lab/Reusable-Multi-Monitor/inventory.ini semaphore:/ansible/reusable/inventory.ini
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/Reusable-Multi-Monitor/CORE-BABA/. semaphore:/ansible/reusable/CORE-BABA/
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/Reusable-Multi-Monitor/CORE-TAAS/. semaphore:/ansible/reusable/CORE-TAAS/
```

This does not rebuild or migrate Semaphore.

`/ansible/reusable/inventory.ini` is the inventory used by the reusable CLI commands below. The current Day 1 `Cisco Inventory` in Semaphore is an inline GUI inventory and does not automatically read this file. For reusable GUI templates, create a separate inline entry such as `Reusable Cisco Inventory`, select the existing `Cisco Admin` credential, and paste the same reusable inventory content into it. Do not silently replace the working Day 1 inventory. If the installed Semaphore version already supports and uses a file-type inventory, it may instead point the separate reusable entry to `/ansible/reusable/inventory.ini`.

## 5. Syntax and Read-Only Tests

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
sudo docker exec semaphore ansible-playbook -i /ansible/reusable/inventory.ini /ansible/reusable/CORE-BABA/show-version.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/reusable/inventory.ini /ansible/reusable/CORE-BABA/show-version.yml --extra-vars target_device=baba72 --limit baba72
```

The read-only test must finish with `failed=0` and `unreachable=0`.

### Screenshot guide: Reusable read-only result

- **Capture:** the complete command or Semaphore task name plus the final recap.
- **Success must show:** only the selected inventory hostname and recap values `failed=0`, `unreachable=0`.
- **Hide:** credentials and unrelated inventory hosts.
- **Status:** Screenshot pending.

Before any configuration run, read the command from left to right and confirm that the inventory line, `target_device`, and `--limit` all identify the same physical switch.

## 6. Semaphore Templates

Reuse the existing project and repository. Create separate templates per device, select the separate `Reusable Cisco Inventory`, and set the template's Ansible command arguments to the explicit target and matching limit:

```text
--extra-vars target_device=baba72 --limit baba72
```

Example BABA 72 templates:

| Template | Playbook | Required target/limit |
|---|---|---|
| BABA 72 — Show Version | `reusable/CORE-BABA/show-version.yml` | `target_device=baba72`, limit `baba72` |
| BABA 72 — Base | `reusable/CORE-BABA/baba-base.yml` | `target_device=baba72`, limit `baba72` |
| BABA 72 — Trunk/LACP | `reusable/CORE-BABA/baba-lacp.yml` | `target_device=baba72`, limit `baba72` |
| BABA 72 — DHCP | `reusable/CORE-BABA/baba-dhcp.yml` | `target_device=baba72`, limit `baba72` |
| BABA 72 — VLANs | `reusable/CORE-BABA/baba-vlans.yml` | `target_device=baba72`, limit `baba72` |

For CLI runs, use `/ansible/reusable/inventory.ini`. For GUI runs in the current inline-inventory setup, use the separate `Reusable Cisco Inventory` containing identical device entries. If the working Semaphore version does not expose an arguments, extra-variables, or limit field, use the documented CLI commands for this reusable set; do not remove the explicit-target safeguard.

### Screenshot guide: Reusable Semaphore target safeguards

- **Capture:** the reusable task template with its playbook and arguments fields visible.
- **Success must show:** the same device name in `target_device=baba72` and `--limit baba72`.
- **Hide:** credentials, tokens, and unrelated devices.
- **Status:** Screenshot pending.

## 7. Safe CLI Example for BABA 72

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
sudo docker exec semaphore ansible-playbook -i /ansible/reusable/inventory.ini /ansible/reusable/CORE-BABA/show-version.yml --extra-vars target_device=baba72 --limit baba72
sudo docker exec semaphore ansible-playbook -i /ansible/reusable/inventory.ini /ansible/reusable/CORE-BABA/baba-base.yml --extra-vars target_device=baba72 --limit baba72 --check
sudo docker exec semaphore ansible-playbook -i /ansible/reusable/inventory.ini /ansible/reusable/CORE-BABA/baba-base.yml --extra-vars target_device=baba72 --limit baba72
```

Review check-mode output before the real configuration run. Network modules and IOS versions can differ in check-mode behavior, so it is an additional safeguard rather than a substitute for backups and verification.

For a beginner-safe first reusable run, stop after `show-version.yml`. Continue to a configuration playbook only after another operator has confirmed the inventory address, physical switch, monitor number, backup, and matching target/limit values.

## Reusable Deployment Checklist

- The target switch already has a reachable management IP and SSH.
- The inventory entry is in the correct `[baba]` or `[taas]` group.
- The inventory hostname, address, and `monitor` value agree.
- The selected GUI reusable inventory matches `/ansible/reusable/inventory.ini` when running from Semaphore.
- `target_device` and `--limit` contain the same one inventory hostname.
- The read-only show-version run succeeds with no failed or unreachable hosts.
- The switch configuration is backed up before a configuration playbook runs.
- BABA and TAAS trunk/LACP changes follow the paired order in their main tutorials.

## Camera Reservations

The reusable camera playbook will stop unless both real client identifiers exist, differ from one another, and no longer contain the original placeholder. Do not create a camera Semaphore template until those values are known and reviewed.
