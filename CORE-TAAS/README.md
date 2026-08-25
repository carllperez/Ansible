# CORE TAAS — Complete Day 1 Sir Rob Automation Tutorial

If you are new to VS Code, WSL, Ansible, YAML, or Semaphore, complete [START HERE](../START-HERE.md) before following this page.

This guide contains every command block in `DAY1-May5-SirRob.txt` that is explicitly assigned to CORE TAAS or shared by `TAAS/BABA`:

1. CORE TAAS basic Layer 3 configuration.
2. Fa0/10-12 switch-to-switch trunk configuration.
3. Fa0/10-12 LACP EtherChannel configuration.
4. The trunk, EtherChannel, and Port-Channel bandwidth verification commands included with those blocks.

The source labels DHCP, VLAN/access-port placement, and camera reservations for the CORE BABA leaf switch. Those commands are intentionally not copied to TAAS.

## What This Guide Changes

The TAAS configuration playbooks can change the switch hostname, SVI addresses, Fa0/10-12 trunking, and LACP EtherChannel. `show-version-taas.yml` is read-only; `taas-base.yml`, `taas-trunk.yml`, and `taas-lacp.yml` make changes.

## Monitor-Number Placeholder

`~~` represents the assigned monitor/student number. Keep `~~` in the reusable GitHub files. Replace it only in the working copies saved through VS Code Remote-SSH under `C:\Users\Administrator\ansible-lab` on the VM.

Example for monitor 71:

```text
COREtaas-~~  → COREtaas-71
10.~~.1.2   → 10.71.1.2
```

> **Never enter `~~` on a Cisco switch.** It is only a documentation placeholder. Replace it with the assigned number before pasting a bootstrap command or copying a YAML file to Semaphore.

## TAAS Addressing from Day 1

| Item | Template value |
|---|---|
| Hostname | `COREtaas-~~` |
| Management SVI | `10.~~.1.2 255.255.255.0` |
| VLAN 10 SVI | `10.~~.10.2 255.255.255.0` |
| VLAN 50 SVI | `10.~~.50.2 255.255.255.0` |
| VLAN 100 SVI | `10.~~.100.2 255.255.255.0` |
| Trunk/LACP ports | `FastEthernet0/10-12` |
| EtherChannel | `Port-channel1` |
| Ansible inventory group | `taas` |

## Complete File Set and VS Code Locations

VS Code on the physical PC saves these files over SSH into the Windows Server VM:

| Repository YAML | Save through VS Code on the VM as | Source block |
|---|---|---|
| [show-version.yml](show-version.yml) | `C:\Users\Administrator\ansible-lab\show-version-taas.yml` | Read-only Ansible connection test; not a Day 1 configuration block |
| [taas-base.yml](taas-base.yml) | `C:\Users\Administrator\ansible-lab\taas-base.yml` | CORE SWITCH SA TAAS — basic Layer 3 |
| [taas-trunk.yml](taas-trunk.yml) | `C:\Users\Administrator\ansible-lab\taas-trunk.yml` | TAAS/BABA trunk ports |
| [taas-lacp.yml](taas-lacp.yml) | `C:\Users\Administrator\ansible-lab\taas-lacp.yml` | @taas/BABA LACP EtherChannel |

Every TAAS YAML file uses:

```yaml
hosts: taas
```

Do not change it to `hosts: cisco`, because the combined group also contains BABA.

## Day 1 Source Command Reference

The following Cisco blocks show the complete TAAS scope being converted. Cisco abbreviations are retained here for comparison; the YAML uses normalized full interface and command names.

These blocks are a reference for comparing the playbooks with Sir Rob’s source. They are not one large paste block. Follow the numbered bootstrap and Semaphore sections below for the actual procedure.

### A. CORE TAAS Basic Layer 3

```cisco
config t
hostname COREtaas-~~
enable secret pass
service password-encryption
no logging console
no ip domain-lookup

line console 0
 password pass
 login
 exec-timeout 0 0

line vty 0 14
 password pass
 login
 exec-timeout 0 0

interface Vlan1
 no shutdown
 ip address 10.~~.1.2 255.255.255.0
 description MGMTDATA

interface Vlan10
 no shutdown
 ip address 10.~~.10.2 255.255.255.0
 description WIRELESS

interface Vlan50
 no shutdown
 ip address 10.~~.50.2 255.255.255.0
 description IPCCTV

interface Vlan100
 no shutdown
 ip address 10.~~.100.2 255.255.255.0
 description VOICEVLAN
end
```

Converted to `taas-base.yml`.

### B. TAAS/BABA Trunk Ports

```cisco
config t
interface range fa0/10-12
! shutdown
 no shutdown
 switchport trunk encapsulation dot1q
 switchport mode trunk
 do show interfaces trunk
end
```

`! shutdown` is a Cisco comment in the source, not an active shutdown command. The YAML therefore applies `no shutdown` but does not shut the links.

Converted to `taas-trunk.yml`.

### C. TAAS/BABA LACP EtherChannel

```cisco
config t
interface range fa0/10-12
 channel-group 1 mode active
 channel-protocol lacp
 do show etherchannel summary
 do show interfaces port-channel 1 | include BW
```

Converted to `taas-lacp.yml`.

The YAML also uses `save_when: modified` after each configuration block so a successful change is saved. That is an automation safeguard; it does not add unrelated network configuration.

## 1. Preserve the Existing Semaphore Setup

Semaphore remains in the existing Docker container inside Ubuntu. Do not recreate its container, database, project, users, credentials, or port configuration.

**Run on: WINDOWS SERVER VM → POWERSHELL or VS Code terminal**

```powershell
wsl -d Ubuntu
```

**Then run on: VM → WSL UBUNTU**

```bash
sudo docker ps
sudo docker exec semaphore ls -la /ansible
```

If both work, leave Semaphore unchanged.

## 2. Back Up and Bootstrap TAAS for SSH

Sir Rob's TAAS Day 1 block does not contain SSH commands, but Ansible cannot reach a blank switch without management connectivity and SSH. Apply this prerequisite from the console before using Ansible.

**Run on: CORE TAAS → CISCO CLI**

```cisco
enable
show running-config
copy running-config startup-config

configure terminal
hostname COREtaas-71
enable secret pass
service password-encryption
no logging console
no ip domain-lookup

interface Vlan1
 no shutdown
 ip address 10.71.1.2 255.255.255.0
 description MGMTDATA
 exit

username admin privilege 15 secret pass
ip domain-name rivanit.com
crypto key generate rsa
```

When asked for the modulus, enter:

```text
1024
```

Continue:

```cisco
ip ssh version 2
line vty 0 14
 login local
 transport input ssh
 exec-timeout 0 0
 exit
end
write memory
```

The example above is for monitor 71. For a different monitor, replace every `71` before entering the commands.

The SSH block is an automation prerequisite, not presented as part of Sir Rob's TAAS base block.

Verify:

```cisco
show ip interface brief
show ip ssh
show running-config | include hostname
```

Then test from Ubuntu. Change `71` to the assigned monitor number.

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
ping -c 4 10.71.1.2
ssh admin@10.71.1.2
```

The bootstrap checkpoint passes only when the hostname and management address are correct, `show ip ssh` reports SSH enabled, and the manual Ubuntu SSH login succeeds. Stop here if any check fails.

<!-- SCREENSHOT: TAAS Vlan1 address and SSH enabled -->

## 3. Create the Working Files through VS Code

**Do in: PHYSICAL PC → VS Code Remote-SSH window → Windows Server VM folder**

1. Open `C:\Users\Administrator\ansible-lab`.
2. Create `show-version-taas.yml`, `taas-base.yml`, `taas-trunk.yml`, and `taas-lacp.yml`.
3. Copy the complete matching YAML from `CORE-TAAS/`.
4. Replace every `~~` with the assigned monitor number.
5. Confirm each YAML says `hosts: taas`.
6. Save with **Ctrl+S**. VS Code transfers the changes over SSH to the VM.

For the exact beginner copy procedure, see [How to Copy a YAML File into VS Code](../START-HERE.md#how-to-copy-a-yaml-file-into-vs-code).

Enter WSL from the VS Code terminal and list the same VM files:

```powershell
wsl -d Ubuntu
```

```bash
ls -lh /mnt/c/Users/Administrator/ansible-lab
```

Check for unreplaced monitor placeholders:

```bash
grep -n -- '~~' \
  /mnt/c/Users/Administrator/ansible-lab/show-version-taas.yml \
  /mnt/c/Users/Administrator/ansible-lab/taas-base.yml \
  /mnt/c/Users/Administrator/ansible-lab/taas-trunk.yml \
  /mnt/c/Users/Administrator/ansible-lab/taas-lacp.yml
```

The command should return no lines before deployment.

<!-- SCREENSHOT: TAAS YAML files open in the VS Code Remote-SSH VM folder -->

## 4. Add TAAS to the Existing Inventory

Create or edit the inventory through VS Code at `C:\Users\Administrator\ansible-lab\inventory.ini`.

The full inventory should include separate BABA and TAAS groups:

```ini
[baba]
baba ansible_host=10.~~.1.4

[taas]
taas ansible_host=10.~~.1.2

[cisco:children]
baba
taas

[cisco:vars]
ansible_user=admin
ansible_password=pass
ansible_connection=network_cli
ansible_network_os=ios
ansible_ssh_common_args='-o KexAlgorithms=+diffie-hellman-group14-sha1 -o HostKeyAlgorithms=+ssh-rsa -o Ciphers=+aes128-cbc -o MACs=+hmac-sha1'
```

Replace every `~~` and save the VS Code file with **Ctrl+S**.

Check the TAAS target:

```bash
grep -n -A2 '^\[taas\]' /mnt/c/Users/Administrator/ansible-lab/inventory.ini
```

## 5. Copy TAAS Files to the Existing Semaphore Container

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/inventory.ini semaphore:/ansible/inventory.ini
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/show-version-taas.yml semaphore:/ansible/show-version-taas.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/taas-base.yml semaphore:/ansible/taas-base.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/taas-trunk.yml semaphore:/ansible/taas-trunk.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/taas-lacp.yml semaphore:/ansible/taas-lacp.yml
```

Verify filenames and non-zero sizes:

```bash
sudo docker exec semaphore ls -lh \
  /ansible/inventory.ini \
  /ansible/show-version-taas.yml \
  /ansible/taas-base.yml \
  /ansible/taas-trunk.yml \
  /ansible/taas-lacp.yml
```

<!-- SCREENSHOT: All TAAS files in the existing /ansible directory -->

## 6. Syntax-Check Every TAAS Playbook

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/show-version-taas.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/taas-base.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/taas-trunk.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/taas-lacp.yml --syntax-check
```

Syntax checks do not configure the switch.

## 7. Verify or Add Semaphore Task Templates

In the existing Semaphore project, reuse the current repository and inventory. Add only the missing TAAS templates.

| Template | Playbook | Source order |
|---|---|---:|
| TAAS — Show Version | `show-version-taas.yml` | Pre-check |
| TAAS — Base Layer 3 | `taas-base.yml` | 1 |
| TAAS — Trunk Ports | `taas-trunk.yml` | 2 |
| TAAS — LACP EtherChannel | `taas-lacp.yml` | 3 |

Each template uses the existing:

```text
Repository: Cisco Playbooks
Inventory:  Cisco Inventory
```

<!-- SCREENSHOT: Four separate TAAS buttons in Semaphore -->

## 8. Run the TAAS Playbooks in Day 1 Order

### 8.1 Show Version — Read Only

Run `TAAS — Show Version` first.

Required recap:

```text
failed=0
unreachable=0
```

Do not run configuration templates until the read-only test succeeds.

### 8.2 Base Layer 3

Run `TAAS — Base Layer 3`.

Because the source changes VTY authentication from the bootstrap's `login local` to `password pass` plus `login`, run `TAAS — Show Version` again immediately after the base playbook. Stop if it reports `failed` or `unreachable`.

This applies:

- hostname and enable secret;
- password encryption;
- logging/domain-lookup settings;
- console and VTY settings;
- VLAN 1, 10, 50, and 100 SVIs with `.2` addresses and descriptions.

**Verify on: CORE TAAS → CISCO CLI**

```cisco
show running-config | include hostname
show running-config | section line console
show running-config | section line vty
show ip interface brief
```

Expected addresses:

```text
Vlan1    10.~~.1.2
Vlan10   10.~~.10.2
Vlan50   10.~~.50.2
Vlan100  10.~~.100.2
```

An SVI can remain protocol-down until the associated VLAN exists and has an active member or trunk. This tutorial does not invent a TAAS VLAN-creation block that is absent from the Day 1 source.

### 8.3 Trunk Ports

Run `TAAS — Trunk Ports`.

This applies the source commands to Fa0/10-12:

```text
no shutdown
switchport trunk encapsulation dot1q
switchport mode trunk
```

It then runs the source verification:

```cisco
show interfaces trunk
```

The trunk and LACP configuration spans both switches. Use this complete order:

1. Run `TAAS — Trunk Ports`.
2. Run `BABA — Trunk and LACP` on the other switch.
3. Run `TAAS — LACP EtherChannel` immediately afterward.
4. Verify the bundle on both switches.

It is normal for the links or bundle to remain down while only one side is configured.

### 8.4 LACP EtherChannel

Run `TAAS — LACP EtherChannel`.

This applies:

```text
channel-group 1 mode active
channel-protocol lacp
```

It then runs both source verification commands:

```cisco
show etherchannel summary
show interfaces port-channel 1 | include BW
```

**Additional verification on: CORE TAAS → CISCO CLI**

```cisco
show lacp neighbor
show interfaces status
```

## 9. Final TAAS Verification

**Run on: CORE TAAS → CISCO CLI**

```cisco
show running-config | include hostname
show ip interface brief
show interfaces trunk
show etherchannel summary
show interfaces port-channel 1 | include BW
show lacp neighbor
```

Expected EtherChannel indicators normally include:

```text
Po1(SU)
Fa0/10(P)
Fa0/11(P)
Fa0/12(P)
```

Exact flags vary by IOS. `D` indicates a down interface or bundle and must be investigated.

The TAAS walkthrough is complete when:

- the final Semaphore task recap has `failed=0` and `unreachable=0`;
- the expected hostname and monitor-specific SVI addresses are present;
- trunking is active on the intended interfaces;
- Port-channel1 and Fa0/10-12 are bundled on both switches;
- the final configuration has been saved according to your organization’s change procedure.

<!-- SCREENSHOT: Successful TAAS base task -->

<!-- SCREENSHOT: Successful TAAS trunk task and show interfaces trunk output -->

<!-- SCREENSHOT: Successful TAAS LACP task and bundled Po1 members -->

## Commands Intentionally Excluded from TAAS

The following Day 1 sections are not labeled for TAAS and therefore remain under CORE BABA or their own device documentation:

- CORE BABA routed Gi0/1 and `.4` SVI configuration;
- DHCP excluded addresses and DHCP pools;
- VLAN creation and Fa0/2-Fa0/8 access/voice placement;
- camera DHCP reservations;
- CUCM configuration;
- EDGE configuration, routing, NAT, and other later-device tasks.

This separation prevents BABA-only or router-only commands from being pushed to CORE TAAS.
