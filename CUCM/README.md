# CUCM / Cisco Unified CallManager Express Tutorial

This guide adds the Day 1 Sir Rob CUCM router to the existing BABA, TAAS, Ansible, and Semaphore lab. In this repository, **CUCM** means the Cisco IOS router running Cisco Unified CallManager Express (CME); it is not a separate CUCM server appliance.

If you are configuring the entire CUCM/CME workflow, use the numbered [Complete CUCM/CME Day 1 Runbook](RUNBOOK.md). This README provides the detailed explanations and troubleshooting behind those steps.

Complete [START HERE](../START-HERE.md), the [CORE BABA tutorial](../CORE-BABA/README.md), and the BABA VLAN 100 checks before continuing.

## Critical Routing Lesson from the Live Lab

The PC and CUCM are not in the same subnet:

```text
PC / automation network: 10.~~.1.0/24
CUCM voice network:       10.~~.100.0/24
```

The physical path is:

```text
PHYSICAL PC / WINDOWS VM / WSL / SEMAPHORE
                   |
                   v
       CORE BABA Vlan1 10.~~.1.4
                   |
          BABA Layer 3 routing
                   |
      CORE BABA Vlan100 10.~~.100.4
                   |
       BABA Fa0/3 access VLAN 100
                   |
        CUCM Fa0/0 10.~~.100.8
```

Putting BABA `Fa0/3` in VLAN 100 provides the Layer 2 connection, but it is not the complete remote-management path. The complete Day 1 topology also requires:

1. `ip routing` on CORE BABA and CUCM.
2. OSPF process 1 on CORE BABA and CUCM.
3. An OSPF neighbor relationship across VLAN 100.
4. CUCM's default route through `10.~~.100.4`.
5. A correct PC/Windows route or gateway toward CORE BABA.

In the live monitor-71 lab, CUCM's interface and SSH configuration were correct, and BABA could reach CUCM locally, but the PC could not ping or SSH to CUCM until the missing Day 1 OSPF configuration was restored. **Do not begin Semaphore testing until the OSPF checkpoint in this guide passes.**

## Monitor Placeholder and Addressing

`~~` is the assigned monitor/student number. Keep it in the GitHub template and replace it only in the working copy saved through VS Code.

| Item | Template | Monitor 71 example |
|---|---|---|
| CUCM hostname | `CUCM-~~` | `CUCM-71` |
| CUCM Fa0/0 | `10.~~.100.8/24` | `10.71.100.8/24` |
| BABA Vlan100 gateway | `10.~~.100.4/24` | `10.71.100.4/24` |
| BABA OSPF router ID | `10.~~.~~.4` | `10.71.71.4` |
| CUCM OSPF router ID | `10.~~.100.8` | `10.71.100.8` |
| OSPF process/area | process `1`, area `0` | process `1`, area `0` |
| Ansible inventory group | `cucm` | `cucm` |

> **Never paste `~~` into Cisco IOS.** Replace every occurrence with the assigned monitor number first.

## What Comes Directly from Day 1 Sir Rob

The source is [DAY1-May5-SirRob.txt](https://github.com/carllperez/ccna2/blob/main/DAY1-May5-SirRob.txt). The CUCM scope includes:

- CUCM base configuration and `FastEthernet0/0` addressing.
- Analog POTS dial peers.
- CME telephony service, directory numbers, and two example ephones.
- Video calling.
- Incoming-call trusted list.
- Outgoing VoIP dial peers to other monitor networks.
- CUCM OSPF process 1.
- CUCM Layer 3 routing and a default route through CORE BABA.
- Later IVR and SIP sections.

SSH is a prerequisite for Ansible. Sir Rob's source shows plain `login` under the VTY lines, while the working lab requires `login local` and `transport input ssh` on both VTY ranges. The runnable files preserve the working SSH prerequisite and do not replace it with plain `login`.

## Safety Boundaries

- Back up the router before every configuration section.
- Perform the initial interface, SSH, OSPF, and default-route bootstrap from the Cisco console. Semaphore cannot repair CUCM while the PC has no routed path to it.
- `cucm-telephony-service.yml` begins with `no telephony-service`, exactly as the source does. That can remove an existing CME configuration, so the playbook is blocked unless an explicit maintenance variable is supplied.
- Sir Rob's ephone MAC addresses `cafe.face.baba` and `fafa.caca.baba` are placeholders. The automated playbooks obtain the real MAC on BABA Fa0/5 for ephone 1 and the real MAC on Fa0/7 for ephone 2 through CDP. The destructive telephony playbook still requires `confirm_telephony_reset=true` before it changes CUCM.
- The source trusted-list command permits `0.0.0.0/0`. The standalone Day 1 inter-CUCM playbook preserves that exact command without narrowing it. Run it only in the intended isolated Day 1 lab.
- IVR, SIP pools, SIP-phone MAC addresses, extensions, and later remote-country destinations must be reviewed by the voice/network owner before use.
- Run only one CUCM configuration task at a time.

## Files and Exact VS Code Locations

Use the physical PC's VS Code Remote-SSH window to open `C:\Users\Administrator\ansible-lab` on the Windows Server VM.

| Repository file | Save through VS Code as | Purpose |
|---|---|---|
| [show-version.yml](show-version.yml) | `C:\Users\Administrator\ansible-lab\show-version-cucm.yml` | Read-only identity, interface, SSH, and OSPF test |
| [cucm-base.yml](cucm-base.yml) | `C:\Users\Administrator\ansible-lab\cucm-base.yml` | Base, SSH, Fa0/0, IP routing, and default route |
| [cucm-ospf.yml](cucm-ospf.yml) | `C:\Users\Administrator\ansible-lab\cucm-ospf.yml` | CUCM side of Day 1 OSPF |
| [cucm-analog-phones.yml](cucm-analog-phones.yml) | `C:\Users\Administrator\ansible-lab\cucm-analog-phones.yml` | Four Day 1 analog POTS dial peers |
| [cucm-telephony-service.yml](cucm-telephony-service.yml) | `C:\Users\Administrator\ansible-lab\cucm-telephony-service.yml` | Destructive CME rebuild; blocked by default |
| [cucm-video.yml](cucm-video.yml) | `C:\Users\Administrator\ansible-lab\cucm-video.yml` | Video and H.323 slow-start settings |
| [cucm-inter-cucm-voip.yml](cucm-inter-cucm-voip.yml) | `C:\Users\Administrator\ansible-lab\cucm-inter-cucm-voip.yml` | Exact Day 1 incoming trusted list and active outgoing CUCM dial peers |
| [cucm-ephone-status.yml](cucm-ephone-status.yml) | `C:\Users\Administrator\ansible-lab\cucm-ephone-status.yml` | Read-only ephone registration, MAC, button, and number check |
| [cucm-restart-ephones.yml](cucm-restart-ephones.yml) | `C:\Users\Administrator\ansible-lab\cucm-restart-ephones.yml` | Recreate Day 1 phone files and restart ephones 1 and 2 |
| [cucm-auto-discover-ephones.yml](cucm-auto-discover-ephones.yml) | `C:\Users\Administrator\ansible-lab\cucm-auto-discover-ephones.yml` | Discover phone MACs from BABA Fa0/5 and Fa0/7, assign them, and reload both ephones |

CUCM-only plays use:

```yaml
hosts: cucm
```

Do not change that to `hosts: cisco`, because the combined group also contains BABA and TAAS. The telephony rebuild and automatic ephone-discovery files intentionally contain a first play with `hosts: baba` to read CDP from Fa0/5 and Fa0/7, followed by a second play with `hosts: cucm` to apply the discovered addresses.

## 1. Verify the Physical BABA-to-CUCM Connection

The lab cable must connect CUCM `FastEthernet0/0` to CORE BABA `FastEthernet0/3`.

**Run on: CORE BABA → CISCO CLI**

```cisco
show running-config interface FastEthernet0/3
show ip interface brief | include Vlan100
show vlan brief
```

The relevant BABA configuration for monitor 71 is:

```cisco
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 100

interface Vlan100
 ip address 10.71.100.4 255.255.255.0
 no shutdown
```

`Fa0/3` does not receive an IP address. The VLAN 100 gateway address belongs to `interface Vlan100`.

### Screenshot guide: CUCM cable and BABA VLAN 100

- **Capture:** `show running-config interface FastEthernet0/3`, the Vlan100 line from `show ip interface brief`, and VLAN 100 from `show vlan brief`.
- **Success must show:** Fa0/3 assigned to VLAN 100 and Vlan100 `10.~~.100.4` up/up.
- **Hide:** unrelated configuration and credentials.
- **Status:** Screenshot pending.

## 2. Back Up CUCM

**Run on: CUCM → SECURECRT/SERIAL CONSOLE → CISCO CLI**

```cisco
enable
show running-config
copy running-config startup-config
```

If organizational policy requires an external backup, copy the displayed configuration to the approved location before continuing.

## 3. Bootstrap CUCM Base, SSH, Routing, and OSPF

This example is for monitor 71. Replace every `71` for another monitor.

**Run on: CUCM → SECURECRT/SERIAL CONSOLE → CISCO CLI**

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

When prompted for the RSA modulus, enter:

```text
1024
```

Continue:

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

router ospf 1
 router-id 10.71.100.8
 network 10.71.100.0 0.0.0.255 area 0
 exit

end
write memory
```

If RSA keys already exist, do not repeatedly regenerate them. Verify with `show ip ssh`.

## 4. Configure the Matching OSPF Side on CORE BABA

Sir Rob's OSPF block configures both devices. CUCM cannot form an adjacency by itself.

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

The `GigabitEthernet0/1` point-to-point command belongs to BABA's routed Edge link. Do not put it on BABA `Fa0/3` or CUCM `Fa0/0`.

## 5. Mandatory OSPF and Reachability Checkpoint

Do not proceed to inventory or Semaphore until every test below passes.

**Run on: CORE BABA → CISCO CLI**

```cisco
show ip ospf neighbor
show ip route ospf
ping 10.71.100.8
ping 10.71.100.8 source 10.71.1.4
```

**Run on: CUCM → CISCO CLI**

```cisco
show ip interface brief
show ip ssh
show ip ospf neighbor
show ip route ospf
show ip route 0.0.0.0
ping 10.71.100.4
ping 10.71.1.4
```

Pass conditions:

- CUCM `FastEthernet0/0` is `10.71.100.8` and up/up.
- BABA and CUCM appear as OSPF neighbors in the `FULL` state.
- CUCM has OSPF-learned BABA networks.
- CUCM's gateway of last resort is `10.71.100.4`.
- The source-address ping from BABA `10.71.1.4` succeeds.

### Screenshot guide: OSPF fixed and operational

- **Capture:** `show ip ospf neighbor` on both BABA and CUCM plus the successful BABA source-address ping.
- **Success must show:** a FULL neighbor state and `!!!!!`/100 percent success.
- **Hide:** passwords, RSA key material, and unrelated neighbors.
- **Status:** Screenshot pending.

## 6. Test from Windows and WSL Before Ansible

**Run on: WINDOWS SERVER VM → POWERSHELL**

```powershell
ping 10.71.100.8
Test-NetConnection 10.71.100.8 -Port 22
```

If the Windows host does not already use BABA as the gateway for the lab network, Sir Rob's source includes a Windows route toward BABA. Use the narrowest route approved for the lab rather than adding unrelated corporate networks.

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
ping -c 4 10.71.100.8
ssh \
  -o KexAlgorithms=+diffie-hellman-group14-sha1 \
  -o HostKeyAlgorithms=+ssh-rsa \
  -o Ciphers=+aes128-cbc \
  -o MACs=+hmac-sha1 \
  admin@10.71.100.8
```

The prompt must change to:

```text
CUCM-71#
```

If the prompt says `COREbaba-71#`, you connected to `10.71.100.4`, not CUCM.

## 7. Add CUCM to Both Inventories

The current lab has two inventory copies:

1. `C:\Users\Administrator\ansible-lab\inventory.ini`, copied to `/ansible/inventory.ini` for command-line testing.
2. **Semaphore GUI → Inventory → Cisco Inventory**, used by Semaphore task templates.

Both must contain the same monitor-specific values.

**Edit in: PHYSICAL PC → VS Code Remote-SSH → WINDOWS SERVER VM**

```ini
[baba]
baba ansible_host=10.71.1.4

[taas]
taas ansible_host=10.71.1.2

[cucm]
cucm ansible_host=10.71.100.8

[cisco:children]
baba
taas
cucm

[cisco:vars]
ansible_user=admin
ansible_password=pass
ansible_connection=network_cli
ansible_network_os=ios
ansible_ssh_common_args='-o KexAlgorithms=+diffie-hellman-group14-sha1 -o HostKeyAlgorithms=+ssh-rsa -o Ciphers=+aes128-cbc -o MACs=+hmac-sha1'
```

Do not add CUCM only to the file and forget the Semaphore GUI inventory.

## 8. Copy CUCM YAML through VS Code

**Do in: PHYSICAL PC → VS Code Remote-SSH window → `C:\Users\Administrator\ansible-lab`**

1. Create each filename from the file table above.
2. Copy the complete matching GitHub YAML.
3. Replace every `~~` with the monitor number.
4. Save with **Ctrl+S**.

Check for unreplaced monitor placeholders:

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
grep -n -- '~~' \
  /mnt/c/Users/Administrator/ansible-lab/show-version-cucm.yml \
  /mnt/c/Users/Administrator/ansible-lab/cucm-base.yml \
  /mnt/c/Users/Administrator/ansible-lab/cucm-ospf.yml \
  /mnt/c/Users/Administrator/ansible-lab/cucm-analog-phones.yml \
  /mnt/c/Users/Administrator/ansible-lab/cucm-telephony-service.yml \
  /mnt/c/Users/Administrator/ansible-lab/cucm-video.yml \
  /mnt/c/Users/Administrator/ansible-lab/cucm-inter-cucm-voip.yml \
  /mnt/c/Users/Administrator/ansible-lab/cucm-ephone-status.yml \
  /mnt/c/Users/Administrator/ansible-lab/cucm-restart-ephones.yml \
  /mnt/c/Users/Administrator/ansible-lab/cucm-auto-discover-ephones.yml
```

The command should return no lines before a working copy is used.

No phone MAC address needs to be typed into the YAML. The telephony playbook discovers both phones from their fixed BABA ports. Do not set `confirm_telephony_reset=true` merely to bypass the safety check.

## 9. Copy CUCM Files into the Existing Semaphore Container

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

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

## 10. Syntax Check and Read-Only Test

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

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

Then run only the read-only file:

```bash
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/show-version-cucm.yml
```

The recap must show `failed=0` and `unreachable=0`.

## 11. Create Semaphore CUCM Buttons

Create task templates using **Semaphore GUI → Task Templates**:

| Template name | Playbook | Safety |
|---|---|---|
| `CUCM — Show Version and Routing` | `show-version-cucm.yml` | Read-only; run first |
| `CUCM — Base and Default Route` | `cucm-base.yml` | Configuration; normally skip after full console bootstrap |
| `CUCM — OSPF` | `cucm-ospf.yml` | Configuration; OSPF must already be reachable for Semaphore to run it |
| `CUCM — Analog Phones` | `cucm-analog-phones.yml` | Configuration; verify FXS ports and dial plan |
| `CUCM — Telephony Service Rebuild` | `cucm-telephony-service.yml` | Blocked/destructive; do not run casually |
| `CUCM — Video` | `cucm-video.yml` | Configuration; requires ephones 1 and 2 |
| `CUCM — Day 1 Inter-CUCM Calls` | `cucm-inter-cucm-voip.yml` | Configuration; exact unrestricted Day 1 trusted list and active dial peers |
| `CUCM — Check Ephone Status` | `cucm-ephone-status.yml` | Read-only; use when the phones show no number |
| `CUCM — Restart Ephones` | `cucm-restart-ephones.yml` | Recreates phone files and fast-restarts ephones 1 and 2 |
| `CUCM — Discover and Assign Ephones` | `cucm-auto-discover-ephones.yml` | Reads CDP on BABA Fa0/5/Fa0/7, assigns both MACs, regenerates files, and restarts both phones |

For every template, select the existing project, repository `/ansible`, **Cisco Inventory**, and the Cisco login credential already used by the working BABA/TAAS tasks.

To create the three phone-recovery buttons:

1. Open **Semaphore GUI → Task Templates**.
2. Select **New Template**.
3. Create the read-only button with:

   ```text
   Name:              CUCM — Check Ephone Status
   Playbook Filename: cucm-ephone-status.yml
   Repository:        Cisco Playbooks
   Inventory:         Cisco Inventory
   Type:              Task
   ```

4. Save it.
5. Select **New Template** again.
6. Create the restart button with:

   ```text
   Name:              CUCM — Restart Ephones
   Playbook Filename: cucm-restart-ephones.yml
   Repository:        Cisco Playbooks
   Inventory:         Cisco Inventory
   Type:              Task
   ```

7. Save it.
8. Select **New Template** again.
9. Create the automatic assignment button with:

   ```text
   Name:              CUCM — Discover and Assign Ephones
   Playbook Filename: cucm-auto-discover-ephones.yml
   Repository:        Cisco Playbooks
   Inventory:         Cisco Inventory
   Type:              Task
   ```

10. Save it. Do not attach the destructive telephony-reset approval environment to any of these three recovery buttons.

## 12. Recommended CUCM Run Order

1. Back up CUCM.
2. Verify BABA Fa0/3 and Vlan100.
3. Console-bootstrap CUCM base, Fa0/0, SSH, `ip routing`, default route, and CUCM OSPF.
4. Configure/verify the matching BABA OSPF block.
5. Require a FULL OSPF neighbor and successful BABA source-address ping.
6. Require Windows and WSL ping/SSH success to `10.~~.100.8`.
7. Add CUCM to both inventory copies.
8. Copy and syntax-check the YAML files.
9. Run `CUCM — Show Version and Routing`.
10. Skip the base and OSPF playbooks if their complete configuration was already applied manually; use them for reviewed reconciliation only.
11. Review the voice-port and dial-plan assignments before running the analog-phone playbook.
12. Run the approved telephony-service rebuild. It automatically maps BABA Fa0/5 to ephone 1 and Fa0/7 to ephone 2 through CDP.
13. Run `CUCM — Check Ephone Status`; if the MAC mapping is wrong or the phones were replaced, run `CUCM — Discover and Assign Ephones`.
14. Run `CUCM — Restart Ephones` only when the assignments are correct but the phones need to reload their files.
15. Run `CUCM — Day 1 Inter-CUCM Calls` only in the intended isolated Day 1 lab. It deliberately preserves `ipv4 0.0.0.0 0.0.0.0`, all active peers from the source, and the local monitor peer.
16. Do not run the IVR or SIP sections until their placeholders and security impact have been reviewed.

## Both Ephones Show No Number: Step-by-Step Recovery

Do not repeat the destructive telephony-service rebuild first. A missing number can mean the phones did not register, the MAC addresses do not match, the directory-number buttons are missing, or the phones have not downloaded newly generated configuration files.

### Step 1: Run the read-only phone-status button

**Do in: SEMAPHORE GUI → TASK TEMPLATES**

Run:

```text
CUCM — Check Ephone Status
```

In the task log, inspect the output of `show ephone registered`.

- If both phones appear as `REGISTERED`, continue to Step 3.
- If no phones appear, continue to Step 2. Restarting cannot repair missing DHCP, VLAN, TFTP, or MAC information.

### Step 2: Check VLAN 100, DHCP option 150, and both MAC addresses

**Run on: CORE BABA → CISCO CLI**

```cisco
show interfaces FastEthernet0/5 switchport
show interfaces FastEthernet0/7 switchport
show ip dhcp binding
show running-config | section ip dhcp pool VOICEVLAN
```

Success requires:

- the phone ports to use voice VLAN 100;
- both phones to receive `10.~~.100.x` addresses;
- DHCP option 150 to point to `10.~~.100.8`.

**Run on: CUCM → CISCO CLI**

```cisco
show ephone attempted-registrations
show running-config | section ephone 1
show running-config | section ephone 2
```

Compare each configured `mac-address` with the label or settings screen of its physical phone. The Day 1 phone type must show `type 8945`.

If `show ephone attempted-registrations` lists real phones but `show ephone` lists different configured MAC addresses with `UNREGISTERED` and `IP:0.0.0.0`, the MAC mismatch is the cause. Do not rebuild the complete telephony service. Correct only the two ephone MAC assignments.

Run `CUCM — Discover and Assign Ephones`. It reads CDP on BABA Fa0/5 and Fa0/7, converts both `SEP` device identifiers to Cisco MAC format, assigns them to ephones 1 and 2, recreates the phone files, restarts both phones, and saves the result. It stops before changing CUCM unless exactly one different phone is discovered on each port.

### Step 3: Confirm the Day 1 numbers and buttons

**Run on: CUCM → CISCO CLI**

```cisco
show running-config | section ephone-dn
show running-config | section ephone 1
show running-config | section ephone 2
```

For monitor 71, the first displayed line should map as follows:

```text
ephone 1 → button 1:8 → ephone-dn 8 → number 7188
ephone 2 → button 1:4 → ephone-dn 4 → number 7144
```

If the numbers, buttons, phone type, and MAC addresses are correct, continue.

### Step 4: Regenerate the files and restart both phones

**Do in: SEMAPHORE GUI → TASK TEMPLATES**

Run:

```text
CUCM — Restart Ephones
```

The button sends the same Day 1 commands:

```cisco
telephony-service
 create cnf-files
ephone 1
 restart
ephone 2
 restart
```

Both phones temporarily reboot and download their regenerated files. Do not unplug them during this process.

### Step 5: Verify the result

After both phones finish booting, run `CUCM — Check Ephone Status` again or run directly on CUCM:

```cisco
show ephone registered
show ephone
```

Success requires both MAC addresses to show `REGISTERED` with their assigned button and directory number. Then test locally:

```text
Phone 1 calls Phone 2: dial 7144
Phone 2 calls Phone 1: dial 7188
```

## Troubleshooting: BABA Can Reach CUCM but the PC Cannot

This was the exact live-lab failure. Do not assume that a successful BABA-to-CUCM ping proves the full PC-to-CUCM route.

Check in this order:

**CORE BABA → CISCO CLI**

```cisco
show ip ospf neighbor
show ip route ospf
ping 10.~~.100.8 source 10.~~.1.4
```

**CUCM → CISCO CLI**

```cisco
show ip ospf neighbor
show ip route ospf
show ip route 0.0.0.0
```

If there is no OSPF neighbor, compare both Day 1 OSPF blocks. The process number is locally significant, but this lab uses process 1 on both; the shared VLAN 100 network must be in area 0 on both devices.

If OSPF is FULL and the BABA source-address ping succeeds, check the Windows gateway/route and then WSL. OSPF exchanges routes between Cisco devices; it does not automatically rewrite the PC's own route table.

## Work Still Requiring Voice-Team Values

The following later Day 1 sections are intentionally documented as source scope but are not yet runnable general-purpose buttons:

- ephone 1 and 2 MAC-address assignments;
- IVR/TCL application filenames and parameters;
- SIP phone MAC addresses, usernames, passwords, and remote SIP target.

Add these only after the actual phone inventory, dial plan, approved peer list, IOS feature support, and maintenance window are known.
