# CORE BABA Automation

## Overview

This section documents automation of `COREbaba-~~`.

`~~` is the assigned monitor/student number.

Cisco configuration is based strictly on `DAY1-May5-SirRob.txt`. SSH is documented separately as an automation prerequisite.

## Management

```text
Hostname: COREbaba-~~
Management IP: 10.~~.1.4
```

## 1. SSH Prerequisite

After applying the required Day 1 base/VLAN configuration, add SSH for Ansible:

```cisco
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

```cisco
show ip ssh
show run | section line vty
show run | include domain-name
```

<!-- SCREENSHOT: CORE BABA SSH -->

## 2. Semaphore Inventory

```ini
[cisco]
baba ansible_host=10.~~.1.4

[cisco:vars]
ansible_connection=network_cli
ansible_network_os=ios
ansible_ssh_common_args='-o KexAlgorithms=+diffie-hellman-group14-sha1 -o HostKeyAlgorithms=+ssh-rsa -o Ciphers=+aes128-cbc -o MACs=+hmac-sha1'
```

<!-- SCREENSHOT: Cisco Inventory -->

## 3. Show Version Test

`show-version.yml`:

```yaml
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

Semaphore:

```text
Name: Show Cisco Version
Repository: Cisco Playbooks
Playbook: show-version.yml
Inventory: Cisco Inventory
```

<!-- SCREENSHOT: Successful Show Cisco Version -->

## 4. Trunk and LACP

Source sequence includes:

```cisco
config t
interface range fa0/10-12
 shutdown
 no shutdown
 switchport trunk encapsulation dot1q
 switchport mode trunk
end
```

`baba-lacp.yml`:

```yaml
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

Copy:

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-lacp.yml semaphore:/ansible/baba-lacp.yml
```

Verify:

```cisco
show interfaces trunk
show etherchannel summary
show lacp neighbor
```

<!-- SCREENSHOT: LACP Semaphore task -->
<!-- SCREENSHOT: show etherchannel summary -->

## 5. DHCP

`baba-dhcp.yml`:

```yaml
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
        parents: [ip dhcp pool MGMTDATA]
        lines:
          - network 10.~~.1.0 255.255.255.0
          - default-router 10.~~.1.4
          - domain-name MGMTDATA.COM
          - dns-server 10.~~.1.10

    - name: Configure WIFIDATA DHCP pool
      ios_config:
        parents: [ip dhcp pool WIFIDATA]
        lines:
          - network 10.~~.10.0 255.255.255.0
          - default-router 10.~~.10.4
          - domain-name WIFIDATA.COM
          - dns-server 10.~~.1.10

    - name: Configure IPCCTV DHCP pool
      ios_config:
        parents: [ip dhcp pool IPCCTV]
        lines:
          - network 10.~~.50.0 255.255.255.0
          - default-router 10.~~.50.4
          - domain-name IPCCTV.COM
          - dns-server 10.~~.1.10

    - name: Configure VOICEVLAN DHCP pool
      ios_config:
        parents: [ip dhcp pool VOICEVLAN]
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

Copy:

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-dhcp.yml semaphore:/ansible/baba-dhcp.yml
```

<!-- SCREENSHOT: DHCP Semaphore task -->

## 6. VLANs and Access Ports

VLANs from the Day 1 configuration:

```text
10  WIFIVLAN
50  IPCameraVLAN
69  vlanNIrobert
70  EXTRAVLAN
~~  HRD-POLICY
100 VOICEVLAN
```

`baba-vlans.yml`:

```yaml
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

    - name: Create monitor-number VLAN
      ios_config:
        parents: [vlan ~~]
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

Copy:

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-vlans.yml semaphore:/ansible/baba-vlans.yml
```

Verify:

```cisco
show vlan brief
show interfaces status
```

<!-- SCREENSHOT: VLAN Semaphore task -->
<!-- SCREENSHOT: show vlan brief -->

## 7. Camera DHCP Reservations

The source contains placeholders for camera client identifiers:

```cisco
ip routing

ip dhcp pool CAMERA6
 host 10.~~.50.6 255.255.255.0
 client-identifier 001a.xxxx.yyyy

ip dhcp pool CAMERA8
 host 10.~~.50.8 255.255.255.0
 client-identifier 001a.xxxx.yyyy
```

Do not run the reservation playbook until the actual client identifiers are known.

## Verification

```cisco
show ip ssh
show ip interface brief
show interfaces trunk
show etherchannel summary
show lacp neighbor
show vlan brief
show ip dhcp pool
show ip dhcp binding
```

<!-- SCREENSHOT: Final CORE BABA verification -->
