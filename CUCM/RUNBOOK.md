# Complete CUCM/CME Day 1 Runbook

This is the click-by-click and command-by-command runbook for configuring the Day 1 Cisco Unified CallManager Express router through the existing Ubuntu, Docker, Ansible, and Semaphore setup.

Use this runbook for a new CUCM router or to verify an existing one. Stop whenever a checkpoint fails. Do not continue and hope that a later playbook will repair an earlier network, SSH, routing, DHCP, or phone-registration problem.

## Before You Begin

`~~` means the assigned monitor number. The examples below use monitor 71:

```text
Template: CUCM-~~ and 10.~~.100.8
Example:  CUCM-71 and 10.71.100.8
```

Never enter `~~` into Cisco IOS. Replace it in the working copy first. The ephone MAC addresses are discovered automatically and are not typed into the files.

The five command locations used below are:

| Label | Where it means |
|---|---|
| **CORE BABA → CISCO CLI** | The console or SSH session connected to CORE BABA |
| **CUCM → CISCO CLI** | The console or SSH session connected to the CUCM/CME router |
| **WINDOWS SERVER VM → POWERSHELL** | PowerShell inside the Windows Server VM |
| **VS CODE TERMINAL → WSL UBUNTU** | The Ubuntu terminal opened through the VS Code Remote-SSH window |
| **SEMAPHORE GUI** | The existing Semaphore web interface hosted inside Ubuntu |

## Phase A — Establish the Network and SSH Path

### Step 1: Verify BABA VLAN 100 and the CUCM cable

The CUCM `FastEthernet0/0` cable must connect to CORE BABA `FastEthernet0/3`.

**Run on: CORE BABA → CISCO CLI**

```cisco
show running-config interface FastEthernet0/3
show ip interface brief | include Vlan100
show vlan brief
```

For monitor 71, success requires:

```cisco
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 100

interface Vlan100
 ip address 10.71.100.4 255.255.255.0
 no shutdown
```

`FastEthernet0/3` has no IP address. The gateway address belongs to `Vlan100`.

### Step 2: Back up CUCM

**Run on: CUCM → CISCO CLI**

```cisco
enable
copy running-config startup-config
copy running-config flash:CUCM-before-Day1.cfg
show flash:
```

Do not continue until the backup filename appears.

### Step 3: Apply the minimum console bootstrap

This step must be completed from the console because Ansible cannot reach a router that has no IP address or SSH service.

**Run on: CUCM → CISCO CLI**

```cisco
configure terminal
hostname CUCM-71
enable secret pass
service password-encryption
no logging console
no ip domain-lookup

interface FastEthernet0/0
 ip address 10.71.100.8 255.255.255.0
 no shutdown
 duplex auto
 speed auto
 exit

username admin privilege 15 secret pass
ip domain-name rivanit.com
crypto key generate rsa
```

When IOS asks for the RSA modulus, enter:

```text
1024
```

If keys already exist, do not regenerate them. Continue with:

```cisco
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

ip routing
ip route 0.0.0.0 0.0.0.0 10.71.100.4
end
write memory
```

Replace `pass` with the approved lab credential. Do not commit the real credential to GitHub.

### Step 4: Configure both sides of Day 1 OSPF

**Run on: CORE BABA → CISCO CLI**

```cisco
configure terminal
ip routing
router ospf 1
 router-id 10.71.71.4
 network 10.71.0.0 0.0.255.255 area 0
 exit
interface GigabitEthernet0/1
 ip ospf network point-to-point
 exit
end
write memory
```

**Run on: CUCM → CISCO CLI**

```cisco
configure terminal
ip routing
router ospf 1
 router-id 10.71.100.8
 network 10.71.100.0 0.0.0.255 area 0
 exit
end
write memory
```

### Step 5: Pass the mandatory routing checkpoint

**Run on: CORE BABA → CISCO CLI**

```cisco
show ip ospf neighbor
show ip route ospf
ping 10.71.100.8
ping 10.71.100.8 source 10.71.1.4
```

**Run on: CUCM → CISCO CLI**

```cisco
show ip ospf neighbor
show ip route ospf
show ip route 0.0.0.0
ping 10.71.100.4
ping 10.71.1.4
```

Success requires a `FULL` OSPF neighbor and successful pings. Do not begin Semaphore if the PC-to-CUCM route is still missing.

### Step 6: Test Windows and WSL connectivity

**Run on: WINDOWS SERVER VM → POWERSHELL**

```powershell
ping 10.71.100.8
Test-NetConnection 10.71.100.8 -Port 22
```

**Run on: VS CODE TERMINAL → WSL UBUNTU**

```bash
ping -c 4 10.71.100.8
ssh \
  -o KexAlgorithms=+diffie-hellman-group14-sha1 \
  -o HostKeyAlgorithms=+ssh-rsa \
  -o Ciphers=+aes128-cbc \
  -o MACs=+hmac-sha1 \
  admin@10.71.100.8
```

The prompt must become `CUCM-71#`. If it says `COREbaba-71#`, the connection went to `10.71.100.4` instead of CUCM.

## Phase B — Prepare Ansible and Semaphore

### Step 7: Add CUCM to both inventory copies

The working lab has a VS Code inventory file and a Semaphore GUI inventory. Both must contain the same CUCM alias and address.

If the existing working inventory uses one `[cisco]` group, keep it and add only the CUCM line:

```ini
[cisco]
baba ansible_host=10.71.1.4
taas ansible_host=10.71.1.2
cucm ansible_host=10.71.100.8

[cisco:vars]
ansible_user=admin
ansible_password=pass
ansible_connection=network_cli
ansible_network_os=ios
ansible_ssh_common_args='-o KexAlgorithms=+diffie-hellman-group14-sha1 -o HostKeyAlgorithms=+ssh-rsa -o Ciphers=+aes128-cbc -o MACs=+hmac-sha1'
```

Do not rebuild or replace a working inventory merely to use a different group layout. The host alias `cucm` is what matches `hosts: cucm` in every CUCM playbook.

Edit these two locations:

1. **VS Code Remote-SSH:** `C:\Users\Administrator\ansible-lab\inventory.ini`
2. **Semaphore GUI:** **Inventory → Cisco Inventory → Edit**

### Step 8: Save all CUCM YAML files through VS Code

Save the repository files in `C:\Users\Administrator\ansible-lab` using these working filenames:

| Repository file | Working filename |
|---|---|
| `show-version.yml` | `show-version-cucm.yml` |
| `cucm-base.yml` | `cucm-base.yml` |
| `cucm-ospf.yml` | `cucm-ospf.yml` |
| `cucm-analog-phones.yml` | `cucm-analog-phones.yml` |
| `cucm-telephony-service.yml` | `cucm-telephony-service.yml` |
| `cucm-video.yml` | `cucm-video.yml` |
| `cucm-inter-cucm-voip.yml` | `cucm-inter-cucm-voip.yml` |
| `cucm-ephone-status.yml` | `cucm-ephone-status.yml` |
| `cucm-restart-ephones.yml` | `cucm-restart-ephones.yml` |
| `cucm-auto-discover-ephones.yml` | `cucm-auto-discover-ephones.yml` |

Replace every `~~` with the assigned monitor number in the working files. Do not replace values in the GitHub templates.

Do not type either phone MAC address into a YAML file. The telephony rebuild and recovery playbooks automatically read the `SEP` device on BABA Fa0/5 for ephone 1 and the device on Fa0/7 for ephone 2 through CDP.

### Step 9: Check that no monitor placeholder remains

**Run on: VS CODE TERMINAL → WSL UBUNTU**

```bash
grep -n -- '~~' /mnt/c/Users/Administrator/ansible-lab/*.yml
```

The command must return no lines. Apart from the approved lab password and the replacement of `~~`, no per-phone value must be entered.

### Step 10: Copy the files into the existing Semaphore container

**Run on: VS CODE TERMINAL → WSL UBUNTU**

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/inventory.ini semaphore:/ansible/inventory.ini
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/show-version-cucm.yml semaphore:/ansible/show-version-cucm.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/cucm-base.yml semaphore:/ansible/cucm-base.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/cucm-ospf.yml semaphore:/ansible/cucm-ospf.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/cucm-analog-phones.yml semaphore:/ansible/cucm-analog-phones.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/cucm-telephony-service.yml semaphore:/ansible/cucm-telephony-service.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/cucm-video.yml semaphore:/ansible/cucm-video.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/cucm-inter-cucm-voip.yml semaphore:/ansible/cucm-inter-cucm-voip.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/cucm-ephone-status.yml semaphore:/ansible/cucm-ephone-status.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/cucm-restart-ephones.yml semaphore:/ansible/cucm-restart-ephones.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/cucm-auto-discover-ephones.yml semaphore:/ansible/cucm-auto-discover-ephones.yml
sudo docker exec semaphore ls -lh /ansible
```

Every file must appear with a non-zero size.

### Step 11: Syntax-check every playbook

**Run on: VS CODE TERMINAL → WSL UBUNTU**

```bash
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/show-version-cucm.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/cucm-base.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/cucm-ospf.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/cucm-analog-phones.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/cucm-telephony-service.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/cucm-video.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/cucm-inter-cucm-voip.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/cucm-ephone-status.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/cucm-restart-ephones.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/cucm-auto-discover-ephones.yml --syntax-check
```

Do not create configuration buttons until all syntax checks pass.

### Step 12: Create the Semaphore buttons

**Do in: SEMAPHORE GUI → TASK TEMPLATES**

For every template use:

```text
Repository: Cisco Playbooks
Inventory:  Cisco Inventory
Type:       Task
```

Create these templates:

| Button name | Playbook filename | Purpose |
|---|---|---|
| `CUCM — Show Version and Routing` | `show-version-cucm.yml` | Read-only network test |
| `CUCM — Base and Default Route` | `cucm-base.yml` | Base, SSH, Fa0/0, and default route |
| `CUCM — OSPF` | `cucm-ospf.yml` | Day 1 CUCM OSPF |
| `CUCM — Analog Phones` | `cucm-analog-phones.yml` | Day 1 POTS dial peers 1–4 |
| `CUCM — Telephony Service Rebuild` | `cucm-telephony-service.yml` | Destructive Day 1 CME rebuild |
| `CUCM — Check Ephone Status` | `cucm-ephone-status.yml` | Read-only phone status |
| `CUCM — Restart Ephones` | `cucm-restart-ephones.yml` | Regenerate files and restart both phones |
| `CUCM — Discover and Assign Ephones` | `cucm-auto-discover-ephones.yml` | Automatically map BABA Fa0/5 to ephone 1 and Fa0/7 to ephone 2 |
| `CUCM — Video` | `cucm-video.yml` | Day 1 video and H.323 slow start |
| `CUCM — Day 1 Inter-CUCM Calls` | `cucm-inter-cucm-voip.yml` | Day 1 trusted list and outgoing peers |

### Step 13: Create the one-time telephony reset approval

**Do in: SEMAPHORE GUI → VARIABLE GROUPS or ENVIRONMENT**

Create:

```text
Name: CUCM Telephony Reset Approval
```

Enter this under **Extra Variables**:

```json
{
  "confirm_telephony_reset": true
}
```

Attach it only to `CUCM — Telephony Service Rebuild`. Do not attach it to the other buttons. Detach it after the successful rebuild.

## Phase C — Run the CUCM Buttons in Order

### Step 14: Run the read-only network test

Run:

```text
CUCM — Show Version and Routing
```

Pass condition:

```text
failed=0
unreachable=0
```

The output must show the expected hostname, `10.~~.100.8`, SSH version 2, and OSPF information.

### Step 15: Decide whether to run Base and OSPF

If Steps 3–5 already applied and verified the complete base and OSPF configuration, skip these two buttons. Their presence in Semaphore does not mean they must be rerun.

For a reviewed reconciliation, run one at a time:

```text
CUCM — Base and Default Route
CUCM — OSPF
```

After each task, rerun `CUCM — Show Version and Routing`. Stop if SSH or OSPF fails.

### Step 16: Run the analog-phone dial peers

Run:

```text
CUCM — Analog Phones
```

For monitor 71, verify on CUCM:

```cisco
show running-config | section dial-peer voice
```

The four POTS peers must contain:

```text
7100 → port 0/0/0
7101 → port 0/0/1
7102 → port 0/0/2
7103 → port 0/0/3
```

### Step 17: Run the Day 1 telephony-service rebuild once

This button starts with `no telephony-service`. Back up CUCM immediately before running it. Confirm that one powered Cisco phone is connected to BABA Fa0/5 and another is connected to Fa0/7. The playbook discovers their MAC addresses automatically and stops before changing CUCM if either port has no unique `SEP` phone.

Run:

```text
CUCM — Telephony Service Rebuild
```

Wait for:

```text
failed=0
unreachable=0
```

Do not interrupt the task. After success, detach `CUCM Telephony Reset Approval` from the template.

### Step 18: Check whether both phones registered

Run:

```text
CUCM — Check Ephone Status
```

Success requires both phones to appear under `show ephone registered` with real IP addresses.

For monitor 71, the expected first buttons are:

```text
ephone 1 → 7188
ephone 2 → 7144
```

If the log says `No ephone in specified type/condition`, inspect `show ephone attempted-registrations`. If the attempted MAC addresses differ from the configured ephone MAC addresses, run `CUCM — Discover and Assign Ephones`. It corrects only the ephone assignments and reloads the phones; it does not run `no telephony-service`.

### Step 19: Reload both phones when their configuration is correct

Run:

```text
CUCM — Restart Ephones
```

This regenerates the Day 1 phone configuration files and performs a fast restart of ephones 1 and 2. Both phones temporarily reboot. After they finish, run `CUCM — Check Ephone Status` again.

### Step 20: Enable Day 1 video

Run:

```text
CUCM — Video
```

Verify on CUCM:

```cisco
show running-config | section voice service voip
show running-config | section ephone 1
show running-config | section ephone 2
```

Both ephones must show `video`; the H.323 section must show `call start slow`.

### Step 21: Configure Day 1 incoming and outgoing inter-CUCM calls

Run only in the intended isolated Day 1 lab:

```text
CUCM — Day 1 Inter-CUCM Calls
```

The playbook deliberately preserves the source exactly:

- `ipv4 0.0.0.0 0.0.0.0` remains unrestricted.
- Dial peer 12 remains commented out and is not sent to IOS.
- Every other active source peer from 11 through 92 is configured.
- The local monitor peer is not removed.

Verify on CUCM:

```cisco
show running-config | section voice service voip
show running-config | section dial-peer voice
show dial-peer voice summary
write memory
```

Inter-CUCM calling also requires the remote CUCM and complete routed network to be operational.

## Phase D — Final Calling Tests

### Step 22: Test the two local IP phones

For monitor 71:

```text
From ephone 1, dial 7144.
From ephone 2, dial 7188.
```

Confirm two-way audio. A ringing phone without two-way audio is not a complete pass.

### Step 23: Test analog phones when connected

For monitor 71, test the approved FXS ports by dialing `7100`, `7101`, `7102`, and `7103` as applicable.

### Step 24: Test an approved remote CUCM

Dial one known remote four-digit extension that matches an active Day 1 destination pattern. The remote CUCM must have a reciprocal dial peer and a working route back to `10.71.100.8`.

### Step 25: Save and collect final evidence

**Run on: CUCM → CISCO CLI**

```cisco
write memory
show ip ospf neighbor
show ephone registered
show dial-peer voice summary
show running-config | section telephony-service
```

Save the Semaphore task logs showing `failed=0` and `unreachable=0`. Hide credentials and password hashes in screenshots.

## Later Day 1 Sections That Still Need Real Inputs

The raw Day 1 source continues with IVR/TCL and SIP phone sections. They are not part of the runnable button sequence above because the source contains environment-specific application files, service names, phone MAC placeholders, usernames, passwords, and remote targets.

Do not paste those placeholder values into IOS. Add those sections only after the required files and actual phone/peer values are available, while preserving the Day 1 Cisco commands themselves.
