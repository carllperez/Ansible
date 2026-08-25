# CORE BABA Automation Tutorial

This tutorial recreates the CORE BABA work from the original lab and keeps every Day1SirRob-derived configuration block as a standalone YAML file.

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

## Files and Exact VS Code Locations

Use VS Code Remote-SSH on the physical PC to open `C:\Users\Administrator\ansible-lab` on the Windows Server VM. Create and save the working files there; do not recreate them with Nano in Ubuntu.

| Copy YAML from | Save through VS Code on the VM as | Purpose |
|---|---|---|
| `CORE-BABA/show-version.yml` | `C:\Users\Administrator\ansible-lab\show-version-baba.yml` | Read-only SSH/Ansible test |
| `CORE-BABA/baba-base.yml` | `C:\Users\Administrator\ansible-lab\baba-base.yml` | Base and SVI configuration |
| `CORE-BABA/baba-lacp.yml` | `C:\Users\Administrator\ansible-lab\baba-lacp.yml` | Fa0/10-12 trunk and LACP |
| `CORE-BABA/baba-dhcp.yml` | `C:\Users\Administrator\ansible-lab\baba-dhcp.yml` | DHCP pools and exclusions |
| `CORE-BABA/baba-vlans.yml` | `C:\Users\Administrator\ansible-lab\baba-vlans.yml` | VLANs and access/voice ports |
| `CORE-BABA/baba-camera-dhcp.yml` | `C:\Users\Administrator\ansible-lab\baba-camera-dhcp.yml` | Camera reservations; do not run yet |

All BABA YAML files use `hosts: baba`. This is intentional. Do not change them to `hosts: cisco`, because the combined group also contains TAAS.

The Day 1 YAML keeps Sir Rob's VTY `password pass` and `login` commands. SSH setup is documented separately as the prerequisite that makes Ansible access possible; the playbook does not remove an existing `transport input ssh` command.

## Create and Save the BABA YAML through VS Code

**Do in: PHYSICAL PC → VS Code Remote-SSH window → Windows Server VM folder**

1. Open `C:\Users\Administrator\ansible-lab`.
2. Create the filenames from the table above.
3. Copy the complete matching repository YAML into each file.
4. Replace every `~~` with the assigned monitor number in the working copy.
5. Save with **Ctrl+S**. VS Code transfers the saved file over SSH to the VM.

Open the VS Code terminal and enter Ubuntu only for verification and Docker commands:

```powershell
wsl -d Ubuntu
```

Before copying files to Semaphore, check that no monitor placeholders remain:

```bash
grep -n -- '~~' \
  /mnt/c/Users/Administrator/ansible-lab/show-version-baba.yml \
  /mnt/c/Users/Administrator/ansible-lab/baba-base.yml \
  /mnt/c/Users/Administrator/ansible-lab/baba-lacp.yml \
  /mnt/c/Users/Administrator/ansible-lab/baba-dhcp.yml \
  /mnt/c/Users/Administrator/ansible-lab/baba-vlans.yml \
  /mnt/c/Users/Administrator/ansible-lab/baba-camera-dhcp.yml
```

The command should return no lines. The camera client identifiers are different placeholders and must remain blocked until the real values are known.

<!-- SCREENSHOT: BABA YAML files open in the VS Code Remote-SSH VM folder -->

## 1. Back Up and Bootstrap BABA

Ansible cannot configure a completely blank switch until the switch has a reachable management IP and SSH. Perform this minimum bootstrap from the console.

**Run on: CORE BABA → CISCO CLI**

```cisco
enable
configure terminal
hostname COREbaba-~~
enable secret pass
service password-encryption
no logging console
no ip domain-lookup

interface Vlan1
 no shutdown
 ip address 10.~~.1.4 255.255.255.0
 description MGMTDATA
 exit

username admin privilege 15 secret pass
ip domain-name rivanit.com
crypto key generate rsa modulus 1024
ip ssh version 2

line vty 0 14
 login local
 transport input ssh
 exec-timeout 0 0
 exit

end
write memory
```

If the IOS image does not accept `crypto key generate rsa modulus 1024`, enter `crypto key generate rsa` and type `1024` when prompted.

Verify:

```cisco
show ip interface brief
show ip ssh
show running-config | include hostname
```

<!-- SCREENSHOT: CORE BABA Vlan1 and SSH verification -->

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
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-lacp.yml semaphore:/ansible/baba-lacp.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-dhcp.yml semaphore:/ansible/baba-dhcp.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-vlans.yml semaphore:/ansible/baba-vlans.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-camera-dhcp.yml semaphore:/ansible/baba-camera-dhcp.yml
sudo docker exec semaphore ls -lh /ansible
```

The file sizes must be greater than zero.

<!-- SCREENSHOT: All BABA YAML files in Semaphore /ansible -->

## 4. Syntax Check

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/show-version-baba.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/baba-base.yml --syntax-check
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

## 6. Run and Verify in Day 1 Order

### 6.1 Read-Only Connection Test

Run `BABA — Show Version`.

Expected recap:

```text
failed=0
unreachable=0
```

Do not continue until this succeeds.

### 6.2 Base Layer 3

Run `BABA — Base Layer 3`.

Because the source changes VTY authentication from the bootstrap's `login local` to `password pass` plus `login`, run `BABA — Show Version` again immediately after the base playbook. Stop if it reports `failed` or `unreachable`.

**Verify on: CORE BABA → CISCO CLI**

```cisco
show ip interface brief
show running-config | include hostname
```

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

Run `BABA — Trunk and LACP`, then run the matching TAAS LACP playbook so both ends agree.

**Verify on: CORE BABA → CISCO CLI**

```cisco
show interfaces trunk
show etherchannel summary
show interfaces port-channel 1 | include BW
show lacp neighbor
```

The Day1SirRob source has `! shutdown` as a comment, so this corrected playbook does not deliberately shut Fa0/10-12. It applies `no shutdown`, Dot1Q trunking, and active LACP.

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

### 6.6 Camera Reservations — Stop Here

Do not run `BABA — Camera Reservations` while these placeholders remain:

```text
client-identifier 001a.xxxx.yyyy
```

Replace them with the actual Camera 6 and Camera 8 client identifiers, have the values reviewed, copy the edited YAML into Semaphore again, syntax-check it, and only then run it.

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
```

<!-- SCREENSHOT: Successful BABA Semaphore templates -->

<!-- SCREENSHOT: CORE BABA final verification commands -->
