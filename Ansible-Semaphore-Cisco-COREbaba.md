# Cisco COREbaba-~~ Automation with Ansible, Docker, and Semaphore

  ---------------------- --------------------------------------
  Device                 Cisco COREbaba-~~
  Management IP          `10.~~.1.4`
  Configuration Source   `DAY1-May5-SirRob.txt`
  Automation             Ansible
  GUI                    Semaphore
  Container Platform     Docker
  Linux Environment      Ubuntu on WSL2
  VM                     Windows Server on VMware Workstation
  VM IP used from PC     `208.8.8.~~`
  Semaphore Port         `3000`
  ---------------------- --------------------------------------

## Overview

This guide documents the lab setup used to automate Cisco `COREbaba-~~`
using Ansible through the Semaphore web GUI.

The Cisco configuration playbooks are based strictly on the Day 1 Sir
Rob configuration. The student placeholder `~~` is kept as `~~` so the user can replace it with their assigned monitor number.

The final working path is:

``` text
Physical PC
   |
   | SSH / HTTP
   v
Windows Server VM (208.8.8.~~)
   |
   v
WSL2 Ubuntu
   |
   v
Docker
   |
   +--- Semaphore :3000
   |
   +--- Ansible
          |
          | SSH
          v
     COREbaba-~~ (10.~~.1.4)
```

```{=html}
<!-- SCREENSHOT: Final working topology or lab setup -->
```
## Important CORE BABA Prerequisite

Before Ansible/Semaphore can manage CORE BABA, apply the required Day 1
base/VLAN configuration from the Sir Rob GitHub configuration.

SSH must also be added because Ansible requires remote CLI access.

For this lab, use:

``` cisco
configure terminal
ip domain-name rivanit.com
username admin privilege 15 secret pass
crypto key generate rsa modulus 1024
ip ssh version 2

line vty 0 4
 login local
 transport input ssh
end

write memory
```

Verify:

``` cisco
show ip ssh
show run | section line vty
show run | include domain-name
```

```{=html}
<!-- SCREENSHOT: COREbaba SSH configuration -->
```
```{=html}
<!-- SCREENSHOT: show ip ssh verification -->
```
> The SSH bootstrap is an automation prerequisite. The Day 1 device
> configuration itself must remain aligned with `DAY1-May5-SirRob.txt`.

------------------------------------------------------------------------

# 1. Windows Server VM and WSL2

The automation server runs on a Windows Server virtual machine in VMware
Workstation.

The VM used in this lab has:

``` text
Windows Server VM: 208.8.8.~~/24
VMware NAT gateway: 208.8.8.2
Physical PC VMnet8: 208.8.8.1
```

Check WSL:

``` powershell
wsl -l -v
```

Expected:

``` text
NAME      STATE      VERSION
Ubuntu    Running    2
```

Open Ubuntu:

``` powershell
wsl -d Ubuntu
```

```{=html}
<!-- SCREENSHOT: wsl -l -v showing Ubuntu version 2 -->
```
## Nested Virtualization Issue

During setup, WSL2 produced:

``` text
HCS_E_HYPERV_NOT_INSTALLED
WSL2 is unable to start since virtualization is not enabled on this machine
```

Because Windows Server itself is running as a VMware VM, nested
virtualization must be enabled.

Power off the VM completely, then go to:

``` text
VMware Workstation
VM Settings
Processors
Virtualization Engine
```

Enable:

``` text
Virtualize Intel VT-x/EPT or AMD-V/RVI
```

Boot the VM and verify WSL2 again:

``` powershell
wsl -l -v
wsl -d Ubuntu
```

```{=html}
<!-- SCREENSHOT: VMware nested virtualization setting -->
```
```{=html}
<!-- SCREENSHOT: WSL working after nested virtualization fix -->
```

------------------------------------------------------------------------

# 2. Docker in Ubuntu WSL

Docker is used to run Semaphore.

Check the Docker client:

``` bash
which docker
docker context ls
```

During the lab, Docker was installed but the daemon was not available
through the normal service command:

``` bash
sudo service docker start
```

Error:

``` text
docker: unrecognized service
```

The working method was to start the daemon manually:

``` bash
sudo dockerd > /tmp/dockerd.log 2>&1 &
```

Verify:

``` bash
sudo docker ps
```

If Docker fails to start, inspect:

``` bash
cat /tmp/dockerd.log
```

```{=html}
<!-- SCREENSHOT: Docker daemon running -->
```
```{=html}
<!-- SCREENSHOT: docker ps showing Semaphore -->
```
> In this WSL environment, `dockerd` may need to be started again after
> a restart.

------------------------------------------------------------------------

# 3. Semaphore Container

Verify that the Semaphore container exists:

``` bash
sudo docker ps -a
```

Start it if required:

``` bash
sudo docker start semaphore
```

Verify:

``` bash
sudo docker ps
```

The container should show a status similar to:

``` text
semaphoreui/semaphore:latest   Up ...
```

Enter the container:

``` bash
sudo docker exec -it semaphore bash
```

The Ansible working directory used by Semaphore is:

``` text
/ansible
```

Check:

``` bash
cd /ansible
ls -la
```

```{=html}
<!-- SCREENSHOT: /ansible directory inside Semaphore -->
```

------------------------------------------------------------------------

# 4. Ansible Project Files

The working Ansible files were initially kept in Ubuntu:

``` text
~/ansible-lab
```

Files included:

``` text
Dockerfile
baba-config.yml
baba.yml
interface.yml
inventory.ini
show-version.yml
baba-lacp.yml
baba-dhcp.yml
baba-vlans.yml
```

For easier editing, a Windows-accessible copy was created:

``` text
C:\Users\Administrator\ansible-lab
```

From Ubuntu:

``` bash
sudo cp -r . /mnt/c/Users/Administrator/ansible-lab
```

The Windows folder became the main editing location in VS Code.

```{=html}
<!-- SCREENSHOT: VS Code showing ANSIBLE-LAB files -->
```

------------------------------------------------------------------------

# 5. Physical PC to Windows Server SSH

OpenSSH Server was enabled on the Windows Server VM so VS Code on the
physical PC could connect remotely.

On Windows Server PowerShell as Administrator:

``` powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH.Server*'
```

If required:

``` powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

Start and enable SSH:

``` powershell
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
```

Verify:

``` powershell
Get-Service sshd
netstat -ano | findstr :22
```

From the physical PC:

``` powershell
ssh Administrator@208.8.8.~~
```

```{=html}
<!-- SCREENSHOT: Successful PC to VM SSH -->
```
## Ping Worked Differently from SSH

At one stage SSH worked while ICMP ping did not. Ping was allowed
separately through Windows Firewall:

``` powershell
New-NetFirewallRule -DisplayName "Allow ICMPv4 Ping" `
-Direction Inbound `
-Protocol ICMPv4 `
-IcmpType 8 `
-Action Allow
```

------------------------------------------------------------------------

# 6. VS Code Remote SSH

On the physical PC, install the Microsoft `Remote - SSH` extension in VS
Code.

Add the VM:

``` text
ssh Administrator@208.8.8.~~
```

Connect using:

``` text
Remote-SSH: Connect to Host
```

The VS Code status bar should show:

``` text
SSH: 208.8.8.~~
```

Open:

``` text
C:\Users\Administrator\ansible-lab
```

```{=html}
<!-- SCREENSHOT: VS Code connected to SSH 208.8.8.~~ -->
```
```{=html}
<!-- SCREENSHOT: ANSIBLE-LAB Explorer -->
```
To access Ubuntu from the VS Code terminal:

``` powershell
wsl -d Ubuntu
```

Then:

``` bash
cd ~/ansible-lab
```

The VS Code WSL extension was also tested, but nested WSL/Remote-SSH
integration produced virtualization-related errors. The reliable
workflow used Remote SSH to Windows Server and opened WSL from the
integrated terminal.

------------------------------------------------------------------------

# 7. Exposing Semaphore GUI to the Physical PC

Semaphore worked locally on the VM but initially did not load from the
physical PC.

Test from Windows Server:

``` powershell
Test-NetConnection localhost -Port 3000
Test-NetConnection 208.8.8.~~ -Port 3000
```

The lab initially showed:

``` text
localhost:3000       = True
208.8.8.~~:3000     = False
```

Allow port 3000:

``` powershell
New-NetFirewallRule -DisplayName "Semaphore 3000" `
-Direction Inbound `
-Protocol TCP `
-LocalPort 3000 `
-Action Allow
```

Get the WSL address:

``` powershell
wsl -d Ubuntu hostname -I
```

The working WSL address at the time was:

``` text
172.18.107.91
```

`172.17.0.1` was the Docker bridge and was not used.

Create a Windows port proxy:

``` powershell
netsh interface portproxy add v4tov4 `
listenaddress=208.8.8.~~ `
listenport=3000 `
connectaddress=172.18.107.91 `
connectport=3000
```

Verify:

``` powershell
netsh interface portproxy show all
```

Expected:

``` text
208.8.8.~~  3000  172.18.107.91  3000
```

Then the Semaphore GUI became reachable from the physical PC using the
Windows Server VM address on port `3000`.

```{=html}
<!-- SCREENSHOT: portproxy output -->
```
```{=html}
<!-- SCREENSHOT: Semaphore GUI opened from physical PC -->
```
> The WSL IP can change after restarting WSL or the VM. If Semaphore
> becomes unreachable, run `wsl -d Ubuntu hostname -I` again and update
> the port proxy.

------------------------------------------------------------------------

# 8. Semaphore Project Setup

A Semaphore project was created for Cisco automation.

## Key Store

Create a key:

``` text
Name: Cisco SSH Credentials
Type: Login with password
Username: admin
Password: pass
```

```{=html}
<!-- SCREENSHOT: Semaphore Cisco SSH Credentials -->
```
## Inventory

Create an Ansible Inventory:

``` text
Name: Cisco Inventory
User Credentials: Cisco SSH Credentials
Type: Static
```

Inventory:

``` ini
[cisco]
baba ansible_host=10.~~.1.4

[cisco:vars]
ansible_connection=network_cli
ansible_network_os=ios
ansible_ssh_common_args='-o KexAlgorithms=+diffie-hellman-group14-sha1 -o HostKeyAlgorithms=+ssh-rsa -o Ciphers=+aes128-cbc -o MACs=+hmac-sha1'
```

```{=html}
<!-- SCREENSHOT: Semaphore Cisco Inventory -->
```
## Repository

The Semaphore repository uses the local path:

``` text
Name: Cisco Playbooks
Path: /ansible
Type: abs. path
```

Semaphore required an Access Key even for the local repository, so a
local/empty key was created when required by the installed Semaphore
version.

```{=html}
<!-- SCREENSHOT: Cisco Playbooks repository -->
```

------------------------------------------------------------------------

# 9. VS Code Copy/Paste Workflow

The playbooks are edited from VS Code on the physical PC while connected to the Windows Server VM through Remote SSH.

The Windows working folder is:

```text
C:\Users\Administrator\ansible-lab
```

Open the VS Code integrated terminal and enter Ubuntu:

```powershell
wsl -d Ubuntu
```

The prompt should change to the Ubuntu/WSL shell.

## Copy a Playbook from VS Code into Semaphore

Use the following pattern after saving a `.yml` file in VS Code:

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/<PLAYBOOK>.yml semaphore:/ansible/<PLAYBOOK>.yml
```

For the CORE BABA playbooks used in this guide:

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-lacp.yml semaphore:/ansible/baba-lacp.yml
```

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-dhcp.yml semaphore:/ansible/baba-dhcp.yml
```

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-vlans.yml semaphore:/ansible/baba-vlans.yml
```

For the camera DHCP playbook, if completed:

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-camera-dhcp.yml semaphore:/ansible/baba-camera-dhcp.yml
```

Verify an individual file:

```bash
sudo docker exec semaphore ls -l /ansible/baba-lacp.yml
```

Verify all playbooks:

```bash
sudo docker exec semaphore ls -la /ansible
```

Enter the Semaphore container when a syntax check is required:

```bash
sudo docker exec -it semaphore bash
```

Then:

```bash
cd /ansible
ansible-playbook -i inventory.ini baba-lacp.yml --syntax-check
```

Exit the container:

```bash
exit
```

<!-- SCREENSHOT: VS Code terminal copying baba-lacp.yml into Semaphore -->
<!-- SCREENSHOT: /ansible directory showing copied YAML playbooks -->
<!-- SCREENSHOT: Successful Ansible syntax check -->

> Repeat the `docker cp` command every time a playbook is changed in the VS Code Windows working folder so Semaphore receives the updated copy.

---

# 10. Copying Playbooks into Semaphore

VS Code edits the files in:

``` text
C:\Users\Administrator\ansible-lab
```

Semaphore reads from:

``` text
/ansible
```

Therefore, after editing a playbook, copy it into the container.

Example:

``` bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-lacp.yml \
semaphore:/ansible/baba-lacp.yml
```

Verify:

``` bash
sudo docker exec semaphore ls -l /ansible/baba-lacp.yml
```

For all files:

``` bash
sudo docker exec semaphore ls -la /ansible
```

```{=html}
<!-- SCREENSHOT: Playbook copied into /ansible -->
```

------------------------------------------------------------------------

# 11. SSH Compatibility with Older Cisco IOS

Manual SSH to CORE BABA required legacy algorithms:

``` bash
ssh \
-o KexAlgorithms=+diffie-hellman-group14-sha1 \
-o HostKeyAlgorithms=+ssh-rsa \
-o Ciphers=+aes128-cbc \
-o MACs=+hmac-sha1 \
admin@10.~~.1.4
```

Successful connection should reach:

``` text
COREbaba-~~#
```

```{=html}
<!-- SCREENSHOT: Successful SSH to COREbaba-~~ -->
```
## Paramiko Error

Semaphore initially failed with:

``` text
Incompatible ssh peer (no acceptable kex algorithm)
```

Check versions inside the Semaphore container:

``` bash
ansible --version
python3 -c "import paramiko; print(paramiko.__version__)"
ssh -V
```

The environment initially used:

``` text
Ansible Core: 2.20.1
Python: 3.12.14
Paramiko: 4.0.0
OpenSSH: 9.9p2
```

The working fix was to use Paramiko 3.x:

``` bash
/opt/semaphore/apps/ansible/13.5.0/venv/bin/pip install "paramiko<4"
```

Verify:

``` bash
/opt/semaphore/apps/ansible/13.5.0/venv/bin/python \
-c "import paramiko; print(paramiko.__version__)"
```

Working version:

``` text
3.5.1
```

Restart Semaphore:

``` bash
exit
sudo docker restart semaphore
sudo docker ps
```

```{=html}
<!-- SCREENSHOT: Paramiko 3.5.1 -->
```
```{=html}
<!-- SCREENSHOT: Semaphore successful task after Paramiko fix -->
```

------------------------------------------------------------------------

# 12. Connectivity Testing

From inside Semaphore:

``` bash
ping -c 4 10.~~.1.4
```

Then test SSH:

``` bash
ssh \
-o KexAlgorithms=+diffie-hellman-group14-sha1 \
-o HostKeyAlgorithms=+ssh-rsa \
-o Ciphers=+aes128-cbc \
-o MACs=+hmac-sha1 \
admin@10.~~.1.4
```

If ping works but SSH fails, verify the Cisco SSH configuration:

``` cisco
show ip ssh
show run | section line vty
show run | include username
show run | include domain-name
show ip interface brief
```

------------------------------------------------------------------------

# 13. Show Version Playbook

`show-version.yml` was created as a safe read-only test:

``` yaml
---
- name: Show Cisco Version
  hosts: cisco
  gather_facts: no

  tasks:
    - name: Run show version
      ios_command:
        commands:
          - show version
      register: output

    - name: Display output
      debug:
        var: output.stdout_lines
```

Create the Semaphore template:

``` text
Name: Show Cisco Version
Repository: Cisco Playbooks
Playbook: show-version.yml
Inventory: Cisco Inventory
```

Run it from Semaphore.

Successful recap:

``` text
failed=0
unreachable=0
```

```{=html}
<!-- SCREENSHOT: Successful Show Cisco Version task -->
```

------------------------------------------------------------------------

# 14. CORE BABA Trunk and LACP

The trunk/LACP playbook follows the Day 1 Sir Rob CORE BABA section for
`Fa0/10-12`.

The source sequence includes:

``` cisco
config t
interface range fa0/10-12
 shutdown
 no shutdown
 switchport trunk encapsulation dot1q
 switchport mode trunk
end
```

LACP uses the same interface range and Port-Channel group defined by the
Day 1 configuration.

`baba-lacp.yml`:

``` yaml
---
- name: Configure CORE BABA Trunk and LACP
  hosts: cisco
  gather_facts: no

  tasks:
    - name: Configure Fa0/10-12 as trunks
      ios_config:
        parents:
          - interface range FastEthernet0/10 - 12
        lines:
          - shutdown
          - no shutdown
          - switchport trunk encapsulation dot1q
          - switchport mode trunk

    - name: Configure LACP EtherChannel
      ios_config:
        parents:
          - interface range FastEthernet0/10 - 12
        lines:
          - channel-protocol lacp
          - channel-group 1 mode active

    - name: Save configuration
      ios_config:
        save_when: modified
```

Syntax check:

``` bash
ansible-playbook -i inventory.ini baba-lacp.yml --syntax-check
```

Semaphore template:

``` text
Name: Configure Cisco LACP
Repository: Cisco Playbooks
Playbook: baba-lacp.yml
Inventory: Cisco Inventory
```

Verify on CORE BABA:

``` cisco
show interfaces trunk
show etherchannel summary
show lacp neighbor
```

During testing, `show etherchannel summary` displayed:

``` text
Po1(SD)
Fa0/10(D)
Fa0/11(D)
Fa0/12(D)
```

`D` means the links were down at that time. This indicates the LACP
configuration existed but the member links/partner state still needed to
be checked.

```{=html}
<!-- SCREENSHOT: Semaphore Configure Cisco LACP template -->
```
```{=html}
<!-- SCREENSHOT: show etherchannel summary -->
```
```{=html}
<!-- SCREENSHOT: show interfaces trunk -->
```

------------------------------------------------------------------------

# 15. CORE BABA DHCP

The DHCP playbook follows the Day 1 Sir Rob addressing convention with
`~~` kept as `~~` so it can be replaced with the assigned monitor number.

`baba-dhcp.yml`:

``` yaml
---
- name: Configure CORE BABA DHCP
  hosts: cisco
  gather_facts: no

  tasks:
    - name: Configure DHCP excluded addresses
      ios_config:
        lines:
          - ip dhcp excluded-address 10.~~.1.1 10.~~.1.100
          - ip dhcp excluded-address 10.~~.10.1 10.~~.10.100
          - ip dhcp excluded-address 10.~~.50.1 10.~~.50.100
          - ip dhcp excluded-address 10.~~.100.1 10.~~.100.100

    - name: Configure MGMTDATA DHCP pool
      ios_config:
        parents:
          - ip dhcp pool MGMTDATA
        lines:
          - network 10.~~.1.0 255.255.255.0
          - default-router 10.~~.1.4
          - domain-name MGMTDATA.COM
          - dns-server 10.~~.1.10

    - name: Configure WIFIDATA DHCP pool
      ios_config:
        parents:
          - ip dhcp pool WIFIDATA
        lines:
          - network 10.~~.10.0 255.255.255.0
          - default-router 10.~~.10.4
          - domain-name WIFIDATA.COM
          - dns-server 10.~~.1.10

    - name: Configure IPCCTV DHCP pool
      ios_config:
        parents:
          - ip dhcp pool IPCCTV
        lines:
          - network 10.~~.50.0 255.255.255.0
          - default-router 10.~~.50.4
          - domain-name IPCCTV.COM
          - dns-server 10.~~.1.10

    - name: Configure VOICEVLAN DHCP pool
      ios_config:
        parents:
          - ip dhcp pool VOICEVLAN
        lines:
          - network 10.~~.100.0 255.255.255.0
          - default-router 10.~~.100.4
          - domain-name VOICEVLAN.COM
          - dns-server 10.~~.1.10
          - option 150 ip 10.~~.100.8

    - name: Save configuration
      ios_config:
        save_when: modified
```

Semaphore template:

``` text
Name: Configure Cisco DHCP
Repository: Cisco Playbooks
Playbook: baba-dhcp.yml
Inventory: Cisco Inventory
```

```{=html}
<!-- SCREENSHOT: Configure Cisco DHCP Semaphore task -->
```
```{=html}
<!-- SCREENSHOT: Cisco DHCP verification -->
```

------------------------------------------------------------------------

# 16. CORE BABA VLANs and Access Ports

The VLAN playbook follows the Day 1 Sir Rob VLAN/port assignment
section.

VLANs:

``` text
10  WIFIVLAN
50  IPCameraVLAN
69  vlanNIrobert
70  EXTRAVLAN
71  HRD-POLICY
100 VOICEVLAN
```

`baba-vlans.yml`:

``` yaml
---
- name: Configure CORE BABA VLANs and Access Ports
  hosts: cisco
  gather_facts: no

  tasks:
    - name: Create VLAN 10
      ios_config:
        parents: [vlan 10]
        lines: [name WIFIVLAN]

    - name: Create VLAN 50
      ios_config:
        parents: [vlan 50]
        lines: [name IPCameraVLAN]

    - name: Create VLAN 69
      ios_config:
        parents: [vlan 69]
        lines: [name vlanNIrobert]

    - name: Create VLAN 70
      ios_config:
        parents: [vlan 70]
        lines: [name EXTRAVLAN]

    - name: Create VLAN 71
      ios_config:
        parents: [vlan 71]
        lines: [name HRD-POLICY]

    - name: Create VLAN 100
      ios_config:
        parents: [vlan 100]
        lines: [name VOICEVLAN]

    - name: Configure Fa0/2
      ios_config:
        parents: [interface FastEthernet0/2]
        lines:
          - switchport mode access
          - switchport access vlan 10

    - name: Configure Fa0/4
      ios_config:
        parents: [interface FastEthernet0/4]
        lines:
          - switchport mode access
          - switchport access vlan 10

    - name: Configure Fa0/6
      ios_config:
        parents: [interface FastEthernet0/6]
        lines:
          - switchport mode access
          - switchport access vlan 50

    - name: Configure Fa0/8
      ios_config:
        parents: [interface FastEthernet0/8]
        lines:
          - switchport mode access
          - switchport access vlan 50

    - name: Configure Fa0/3
      ios_config:
        parents: [interface FastEthernet0/3]
        lines:
          - switchport mode access
          - switchport access vlan 100

    - name: Configure Fa0/5 for voice
      ios_config:
        parents: [interface FastEthernet0/5]
        lines:
          - switchport mode access
          - switchport voice vlan 100
          - mls qos trust device cisco-phone
          - switchport access vlan 1

    - name: Configure Fa0/7 for voice
      ios_config:
        parents: [interface FastEthernet0/7]
        lines:
          - switchport mode access
          - switchport voice vlan 100
          - mls qos trust device cisco-phone
          - switchport access vlan 1

    - name: Save configuration
      ios_config:
        save_when: modified
```

Verify:

``` cisco
show vlan brief
show interfaces status
```

```{=html}
<!-- SCREENSHOT: VLAN task in Semaphore -->
```
```{=html}
<!-- SCREENSHOT: show vlan brief -->
```

------------------------------------------------------------------------

# 17. Camera DHCP Reservation Block

The Day 1 Sir Rob configuration also contains camera DHCP reservations.

For BABA 71:

``` cisco
ip routing

ip dhcp pool CAMERA6
 host 10.~~.50.6 255.255.255.0
 client-identifier 001a.xxxx.yyyy

ip dhcp pool CAMERA8
 host 10.~~.50.8 255.255.255.0
 client-identifier 001a.xxxx.yyyy
```

The `client-identifier` values in the source are placeholders. Do not
run the reservation playbook until the actual identifiers are known.

```{=html}
<!-- SCREENSHOT: Camera DHCP reservation verification if completed -->
```

------------------------------------------------------------------------

# 18. Troubleshooting Summary

## Docker CLI Exists but Daemon Is Unreachable

Error:

``` text
Cannot connect to the Docker daemon at unix:///var/run/docker.sock
```

Check:

``` bash
which docker
docker context ls
```

Working fix in this WSL environment:

``` bash
sudo dockerd > /tmp/dockerd.log 2>&1 &
sudo docker ps
```

## `docker: unrecognized service`

The WSL environment did not expose Docker as a traditional init service.

Instead of repeatedly reinstalling Docker, start `dockerd` directly and
inspect `/tmp/dockerd.log` if necessary.

## Semaphore Cannot Reach Cisco Port 22

If ping works but SSH fails:

``` bash
ping -c 4 10.~~.1.4
```

Check CORE BABA:

``` cisco
show ip ssh
show run | section line vty
show ip interface brief
```

Ensure the SSH prerequisite using `rivanit.com` is present.

## Legacy Cisco SSH KEX Failure

Error:

``` text
Incompatible ssh peer (no acceptable kex algorithm)
```

Use the legacy SSH options in the Ansible inventory and Paramiko 3.5.1
as documented above.

## Semaphore Task Times Out

First verify manual SSH from the same Semaphore container.

Do not immediately modify the playbook if manual connectivity itself is
failing.

## `/ansible` Does Not Exist in Semaphore

Create/copy the working project:

``` bash
sudo docker exec -u 0 semaphore mkdir -p /ansible
sudo docker cp "$HOME/ansible-lab/." semaphore:/ansible/
```

Verify:

``` bash
sudo docker exec semaphore ls -la /ansible
```

## Semaphore Repository Requires Access Key

The installed Semaphore version required an Access Key selection even
for an absolute local repository. Create a local/empty key and assign it
to `Cisco Playbooks`.

## Semaphore GUI Works Locally but Not from Physical PC

Check:

``` powershell
Test-NetConnection localhost -Port 3000
Test-NetConnection 208.8.8.~~ -Port 3000
```

Then use the Windows firewall rule and `netsh interface portproxy`
configuration documented earlier.

## VS Code WSL Integration Fails

The lab encountered:

``` text
HCS_E_HYPERV_NOT_INSTALLED
```

Nested virtualization was enabled in VMware. For day-to-day editing, the
reliable workflow remained:

``` text
Physical PC VS Code
→ Remote SSH to 208.8.8.~~
→ edit C:\Users\Administrator\ansible-lab
→ WSL terminal
→ docker cp
→ Semaphore
```

------------------------------------------------------------------------

# 19. Final Working Workflow

1.  Start the Windows Server VM.
2.  Verify WSL2.
3.  Enter Ubuntu.
4.  Start `dockerd` if required.
5.  Verify/start the `semaphore` container.
6.  Confirm the WSL IP and port proxy if the GUI is unreachable.
7.  Connect VS Code on the physical PC to `208.8.8.~~` using Remote
    SSH.
8.  Edit playbooks in `C:\Users\Administrator\ansible-lab`.
9.  Copy updated playbooks to `/ansible` in the Semaphore container.
10. Syntax-check new playbooks.
11. Create/select the corresponding Semaphore Task Template.
12. Run the task.
13. Verify directly on COREbaba-~~.

```{=html}
<!-- SCREENSHOT: Final Semaphore templates page -->
```
```{=html}
<!-- SCREENSHOT: Final successful task history -->
```

------------------------------------------------------------------------

# 20. Useful Verification Commands

Cisco:

``` cisco
show ip ssh
show ip interface brief
show interfaces trunk
show etherchannel summary
show lacp neighbor
show vlan brief
show ip dhcp pool
show ip dhcp binding
show running-config
```

Ubuntu/Docker:

``` bash
sudo docker ps
sudo docker ps -a
sudo docker exec -it semaphore bash
sudo docker exec semaphore ls -la /ansible
```

Ansible:

``` bash
ansible --version
ansible-playbook -i inventory.ini show-version.yml
ansible-playbook -i inventory.ini baba-lacp.yml --syntax-check
```

Windows Server:

``` powershell
wsl -l -v
Get-Service sshd
Test-NetConnection localhost -Port 3000
Test-NetConnection 208.8.8.~~ -Port 3000
netsh interface portproxy show all
```

------------------------------------------------------------------------

# Notes

-   `~~` from the Day 1 Sir Rob configuration is the monitor-number placeholder and must be replaced with the assigned number.
-   CORE BABA management IP is `10.~~.1.4`.
-   SSH uses `rivanit.com` as the required domain name for this
    automation lab.
-   Keep Day 1 Cisco configuration aligned with the source
    configuration; do not introduce unrelated VLANs or interface
    settings.
-   Screenshot comments are intentionally left throughout this guide so
    final lab screenshots can be inserted before publishing.
-   The WSL IP may change after reboot.
-   Starting Docker manually may be required in this WSL setup.
-   Before pushing a configuration-changing playbook, verify the target
    interfaces and device connectivity.
