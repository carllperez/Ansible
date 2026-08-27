# CORE BABA Automation Tutorial

This tutorial recreates the CORE BABA work from the original lab and keeps every Day1SirRob-derived configuration block as a standalone YAML file.

If you are new to VS Code, WSL, Ansible, YAML, or Semaphore, complete [START HERE](../START-HERE.md) before following this page.

## What This Guide Changes

The BABA configuration playbooks can change the switch hostname, Layer 3 addresses, trunk/LACP ports, DHCP pools, VLANs, and access/voice port assignments. `show-version-baba.yml` is read-only; the other BABA playbooks make changes. The camera reservation playbook remains blocked until its real client identifiers are known.

## Addressing

| Item | Template value |
|---|---|
| Hostname | `COREbaba-~~` |
| Management SVI | `10.~~.1.4/24` |
| Routed Gi0/1 | `10.~~.~~.4/24` |
| VLAN 10 SVI | `10.~~.10.4/24` |
| VLAN 50 SVI | `10.~~.50.4/24` |
| VLAN 100 SVI | `10.~~.100.4/24` |
| Ansible inventory group | `baba` |

Replace every `~~` with the assigned monitor number in the working copies, not in this reusable template repository.

For monitor 71, the replacements include:

| Template | Working value |
|---|---|
| `COREbaba-~~` | `COREbaba-71` |
| `10.~~.1.4` | `10.71.1.4` |
| `10.~~.10.4` | `10.71.10.4` |
| `VLAN ~~` | `VLAN 71` |

> **Never enter `~~` on a Cisco switch.** It is only a documentation placeholder. Replace it with the assigned number before pasting a bootstrap command or copying a YAML file to Semaphore.

## Files and Exact VS Code Locations

Use VS Code Remote-SSH on the physical PC to open `C:\Users\Administrator\ansible-lab` on the Windows Server VM. Create and save the working files there; do not recreate them with Nano in Ubuntu.

| Copy YAML from | Save through VS Code on the VM as | Purpose |
|---|---|---|
| [show-version.yml](show-version.yml) | `C:\Users\Administrator\ansible-lab\show-version-baba.yml` | Read-only SSH/Ansible test |
| [baba-base.yml](baba-base.yml) | `C:\Users\Administrator\ansible-lab\baba-base.yml` | Base and SVI configuration |
| [baba-ospf.yml](baba-ospf.yml) | `C:\Users\Administrator\ansible-lab\baba-ospf.yml` | Day 1 OSPF for Edge and CUCM routing |
| [baba-lacp.yml](baba-lacp.yml) | `C:\Users\Administrator\ansible-lab\baba-lacp.yml` | Fa0/10-12 trunk and LACP |
| [baba-dhcp.yml](baba-dhcp.yml) | `C:\Users\Administrator\ansible-lab\baba-dhcp.yml` | DHCP pools and exclusions |
| [baba-vlans.yml](baba-vlans.yml) | `C:\Users\Administrator\ansible-lab\baba-vlans.yml` | VLANs and access/voice ports |
| [baba-camera-dhcp.yml](baba-camera-dhcp.yml) | `C:\Users\Administrator\ansible-lab\baba-camera-dhcp.yml` | Camera reservations; do not run yet |

All BABA YAML files use `hosts: baba`. This is intentional. Do not change them to `hosts: cisco`, because the combined group also contains TAAS.

Sir Rob's original VTY `password pass` and `login` commands remain visible in the Day 1 source-reference section for comparison. The runnable `baba-base.yml` deliberately preserves the working automation access instead: VTY 0-4 and VTY 5-14 use `login local` and `transport input ssh`. This small difference is an Ansible/SSH prerequisite confirmed in the live lab; plain `login` prevented Semaphore from authenticating with the local `admin` account.

Day 1 source reference (comparison only; do not use this block to replace the working SSH lines):

```cisco
line vty 0 14
 password pass
 login
 exec-timeout 0 0
```

## Create and Save the BABA YAML through VS Code

**Do in: PHYSICAL PC → VS Code Remote-SSH window → Windows Server VM folder**

1. Open `C:\Users\Administrator\ansible-lab`.
2. Create the filenames from the table above.
3. Copy the complete matching repository YAML into each file.
4. Replace every `~~` with the assigned monitor number in the working copy.
5. Save with **Ctrl+S**. VS Code transfers the saved file over SSH to the VM.

For the exact beginner copy procedure, see [How to Copy a YAML File into VS Code](../START-HERE.md#how-to-copy-a-yaml-file-into-vs-code).

Open the VS Code terminal and enter Ubuntu only for verification and Docker commands:

```powershell
wsl -d Ubuntu
```

Before copying files to Semaphore, check that no monitor placeholders remain:

```bash
grep -n -- '~~' \
  /mnt/c/Users/Administrator/ansible-lab/show-version-baba.yml \
  /mnt/c/Users/Administrator/ansible-lab/baba-base.yml \
  /mnt/c/Users/Administrator/ansible-lab/baba-ospf.yml \
  /mnt/c/Users/Administrator/ansible-lab/baba-lacp.yml \
  /mnt/c/Users/Administrator/ansible-lab/baba-dhcp.yml \
  /mnt/c/Users/Administrator/ansible-lab/baba-vlans.yml \
  /mnt/c/Users/Administrator/ansible-lab/baba-camera-dhcp.yml
```

The command should return no lines. The camera client identifiers are different placeholders and must remain blocked until the real values are known.

### Screenshot guide: BABA working files in VS Code

- **Capture:** the VS Code Explorer with `C:\Users\Administrator\ansible-lab` open.
- **Success must show:** all seven BABA working YAML filenames and the Remote-SSH connection indicator.
- **Hide:** passwords, camera identifiers, and unrelated files.
- **Status:** Screenshot pending.

## Choose the BABA Base Path Before Continuing

- **Automation path (recommended):** perform only the connectivity/SSH bootstrap below, run `BABA — Show Version`, and then run `BABA — Base Layer 3`.
- **Complete manual path:** if you already pasted Sir Rob's entire BABA base block with every `~~` replaced, correct both VTY ranges to `login local` plus `transport input ssh`, run Show Version, and **skip `BABA — Base Layer 3`**.

Do not paste the complete base block and then run the base playbook just because both appear in the tutorial. The bootstrap below deliberately overlaps a few base settings so Ansible can connect and safely confirm them; it is not the complete BABA base workflow.

Do not use an old `Configure Cisco Interface`, `interface.yml`, or `baba.yml` task for this step. Those test files are outside the final Day 1 workflow.

## 1. Back Up and Bootstrap BABA

Ansible cannot configure a completely blank switch until the switch has a reachable management IP and SSH. Perform this minimum bootstrap from the console.

If the switch already contains configuration, display it and save it before making changes. If your organization requires an external backup, also copy the displayed configuration to its approved storage location.

**Run on: CORE BABA → CISCO CLI**

```cisco
enable
show running-config
copy running-config startup-config

configure terminal
hostname COREbaba-71
enable secret pass
service password-encryption
no logging console
no ip domain-lookup

interface Vlan1
 no shutdown
 ip address 10.71.1.4 255.255.255.0
 description MGMTDATA
 exit

username admin privilege 15 secret pass
ip domain-name rivanit.com
crypto key generate rsa modulus 1024
ip ssh version 2

line vty 0 4
 login local
 transport input ssh
 exec-timeout 0 0
 exit

line vty 5 14
 login local
 transport input ssh
 exec-timeout 0 0
 exit

end
write memory
```

The example above is for monitor 71. For a different monitor, replace every `71` before entering the commands.

If the IOS image does not accept `crypto key generate rsa modulus 1024`, enter `crypto key generate rsa` and type `1024` when prompted.

Verify:

```cisco
show ip interface brief
show ip ssh
show running-config | include hostname
```

Then test from Ubuntu. Change `71` to the assigned monitor number.

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
ping -c 4 10.71.1.4
ssh admin@10.71.1.4
```

The bootstrap checkpoint passes only when the hostname and management address are correct, `show ip ssh` reports SSH enabled, and the manual Ubuntu SSH login succeeds. Stop here if any check fails; Ansible will not repair a switch it cannot reach.

### Screenshot guide: BABA management and SSH bootstrap

- **Capture:** the Cisco CLI results for `show ip interface brief` and `show ip ssh`.
- **Success must show:** the correct monitor-specific Vlan1 address and SSH version 2 enabled.
- **Hide:** password commands, secrets, RSA key material, and unrelated running configuration.
- **Status:** Screenshot pending.

## 2. Add BABA to the Inventory

**Do in: PHYSICAL PC → VS Code Remote-SSH window**

Create `C:\Users\Administrator\ansible-lab\inventory.ini` and copy the complete repository `inventory.example.ini` into it.

```text
C:\Users\Administrator\ansible-lab\inventory.ini
```

Replace every `~~`. The relevant entry must resolve to:

```ini
[baba]
baba ansible_host=10.~~.1.4
```

The full inventory also contains TAAS and shared Cisco connection variables. Keep the credentials in Semaphore's Key Store for a non-lab deployment; `pass` is retained here because it is the Day 1 lab credential.

This file is the inventory copy used by terminal syntax checks and manual Ansible commands. The current working Semaphore tasks use the separate inline `Cisco Inventory` saved in the GUI. Open **Semaphore → Inventory → Cisco Inventory** and make sure it contains the same `[baba]` and `[taas]` groups and the same monitor-specific IP addresses. Updating only the file does not update the GUI. Follow [Semaphore Project Setup — Verify or Create the Inventory](../Setup/Semaphore-Project.md#2-verify-or-create-the-inventory) for the exact GUI content.

## 3. Copy BABA Files to Semaphore

Open the VS Code integrated terminal, enter Ubuntu, and remain inside Ubuntu for all commands.

**Run in: PHYSICAL PC → VS Code terminal connected to VM**

```powershell
wsl -d Ubuntu
```

**Then run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/inventory.ini semaphore:/ansible/inventory.ini
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/show-version-baba.yml semaphore:/ansible/show-version-baba.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-base.yml semaphore:/ansible/baba-base.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-ospf.yml semaphore:/ansible/baba-ospf.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-lacp.yml semaphore:/ansible/baba-lacp.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-dhcp.yml semaphore:/ansible/baba-dhcp.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-vlans.yml semaphore:/ansible/baba-vlans.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-camera-dhcp.yml semaphore:/ansible/baba-camera-dhcp.yml
sudo docker exec semaphore ls -lh /ansible
```

The file sizes must be greater than zero.

### Screenshot guide: BABA files inside the Semaphore container

- **Capture:** the final `sudo docker exec semaphore ls -lh /ansible` output.
- **Success must show:** `inventory.ini`, all seven BABA YAML files, and sizes greater than zero.
- **Hide:** unrelated container files or credentials.
- **Status:** Screenshot pending.

## 4. Syntax Check

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/show-version-baba.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/baba-base.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/baba-ospf.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/baba-lacp.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/baba-dhcp.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/baba-vlans.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/baba-camera-dhcp.yml --syntax-check
```

Syntax-check the camera file, but do not run it.

## 5. Create Semaphore Buttons

Use the instructions in `Setup/Semaphore-Project.md` to create these templates:

```text
BABA — Show Version          → show-version-baba.yml
BABA — Base Layer 3          → baba-base.yml
BABA — OSPF                  → baba-ospf.yml
BABA — Trunk and LACP        → baba-lacp.yml
BABA — DHCP                  → baba-dhcp.yml
BABA — VLANs and Ports       → baba-vlans.yml
BABA — Camera Reservations   → baba-camera-dhcp.yml (DO NOT RUN)
```

Each uses:

```text
Repository: Cisco Playbooks
Inventory:  Cisco Inventory
```

### Screenshot guide: BABA task templates

- **Capture:** the BABA Task Templates list in Semaphore.
- **Success must show:** the correct BABA template names; camera reservations must be clearly marked **DO NOT RUN**.
- **Hide:** credentials and unrelated projects.
- **Status:** Screenshot pending.

## 6. Run and Verify in Day 1 Order

### 6.1 Read-Only Connection Test

Run `BABA — Show Version`.

Expected recap:

```text
failed=0
unreachable=0
```

Do not continue until this succeeds.

### Screenshot guide: BABA read-only connection test

- **Capture:** the bottom of the `BABA — Show Version` Semaphore task output.
- **Success must show:** the BABA hostname, Cisco version output, `failed=0`, and `unreachable=0`.
- **Hide:** credentials, tokens, and unrelated task output.
- **Status:** Screenshot pending.

### 6.2 Base Layer 3

Run `BABA — Base Layer 3` **only for the automation path after the minimum bootstrap**. If Sir Rob's complete BABA base was already applied manually, skip this template and continue with verification/trunking.

The runnable base playbook keeps local SSH authentication on both VTY ranges. Run `BABA — Show Version` again immediately after the base playbook and stop if it reports `failed` or `unreachable`.

**Verify on: CORE BABA → CISCO CLI**

```cisco
show ip interface brief
show running-config | include hostname
show running-config | section line vty
```

Both `line vty 0 4` and `line vty 5 14` must show `login local` and `transport input ssh`.

Expected addresses include:

```text
GigabitEthernet0/1  10.~~.~~.4
Vlan1               10.~~.1.4
Vlan10              10.~~.10.4
Vlan50              10.~~.50.4
Vlan100             10.~~.100.4
```

An SVI can remain down until its VLAN exists and has an active member/trunk; that does not necessarily mean the IP command failed.

### 6.3 Trunk and LACP

These three links depend on configuration at both ends. Use this order:

1. Run `TAAS — Trunk Ports` and confirm it succeeds.
2. Run `BABA — Trunk and LACP`.
3. Run `TAAS — LACP EtherChannel` immediately afterward.
4. Verify the bundle on both switches.

It is normal for the bundle to remain down between steps while only one side is configured.

**Verify on: CORE BABA → CISCO CLI**

```cisco
show interfaces trunk
show etherchannel summary
show interfaces port-channel 1 | include BW
show lacp neighbor
```

The Day1SirRob source has `! shutdown` as a comment, so this corrected playbook does not deliberately shut Fa0/10-12. It applies `no shutdown`, Dot1Q trunking, and active LACP.

### Screenshot guide: BABA EtherChannel verification

- **Capture:** `show etherchannel summary` and `show lacp neighbor` after both switches are configured.
- **Success must show:** Port-channel1 and Fa0/10-12 bundled, with the expected TAAS LACP neighbor.
- **Hide:** unrelated interface descriptions or infrastructure details.
- **Status:** Screenshot pending.

### 6.4 DHCP

Run `BABA — DHCP`.

**Verify on: CORE BABA → CISCO CLI**

```cisco
show running-config | section ip dhcp
show ip dhcp pool
show ip dhcp binding
```

This playbook belongs only on BABA. It creates the MGMTDATA, WIFIDATA, IPCCTV, and VOICEVLAN pools and option 150 for the CUCM address.

### 6.5 VLANs and Port Placement

Run `BABA — VLANs and Ports`.

**Verify on: CORE BABA → CISCO CLI**

```cisco
show vlan brief
show interfaces status
show interfaces trunk
```

Sir Rob's example contains `vlan 71` for the student-specific HRD-POLICY VLAN. This reusable package intentionally changes that one student value to `vlan ~~`; replace `~~` with the assigned monitor number before copying the playbook to Semaphore.

### 6.6 OSPF for Edge and CUCM Routing

Run `BABA — OSPF` before attempting remote PC, WSL, or Semaphore access to CUCM. This is the CORE BABA block from Sir Rob's Day 1 OSPF section.

The matching CUCM OSPF block must be applied from the CUCM console during its initial bootstrap because Semaphore cannot reach CUCM before the routed path exists. Follow [CUCM / Cisco Unified CallManager Express Tutorial](../CUCM/README.md).

**Verify on: CORE BABA → CISCO CLI**

```cisco
show running-config | section router ospf
show ip ospf neighbor
show ip route ospf
ping 10.~~.100.8 source 10.~~.1.4
```

Do not continue to CUCM automation unless the BABA/CUCM OSPF neighbor is FULL and the source-address ping succeeds.

### 6.7 Camera Reservations — Stop Here

Do not run `BABA — Camera Reservations` while these placeholders remain:

```text
client-identifier 001a.xxxx.yyyy
```

Replace them with the actual Camera 6 and Camera 8 client identifiers, have the values reviewed, copy the edited YAML into Semaphore again, syntax-check it, and only then run it.

Edit only these two values in the working YAML:

```yaml
camera6_client_identifier: "REAL-CAMERA-6-ID"
camera8_client_identifier: "REAL-CAMERA-8-ID"
```

The playbook now stops before making any change if either value still contains `xxxx` or if both values are identical. The example names above are explanatory text, not real identifiers; replace them with the approved Cisco-format values.

After the reservations are applied, the playbook runs Sir Rob's `show ip dhcp binding` verification and displays the result in the Semaphore task output.

## 7. Final BABA Verification

**Run on: CORE BABA → CISCO CLI**

```cisco
show running-config | include hostname
show ip interface brief
show interfaces trunk
show etherchannel summary
show lacp neighbor
show vlan brief
show ip dhcp pool
show ip dhcp binding
show ip ospf neighbor
show ip route ospf
```

The BABA walkthrough is complete when:

- the final Semaphore task recap has `failed=0` and `unreachable=0`;
- the expected hostname and monitor-specific addresses are present;
- Port-channel1 and Fa0/10-12 are bundled on both switches;
- the expected VLANs, ports, and DHCP pools appear;
- the expected OSPF neighbors and routes appear before CUCM automation begins;
- `BABA — Camera Reservations` has not been run unless both real identifiers were reviewed;
- the final configuration has been saved according to your organization’s change procedure.

### Screenshot guide: Final BABA verification

- **Capture:** the final Cisco verification output, split into readable images if necessary.
- **Success must show:** hostname, monitor-specific SVIs, Port-channel1, expected VLANs, and DHCP pools.
- **Hide:** secrets, camera identifiers, DHCP client details that should not be public, and unrelated configuration.
- **Status:** Screenshot pending.
