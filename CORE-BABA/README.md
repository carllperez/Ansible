# CORE BABA Automation

`~~` is the assigned monitor/student number.

## Files

```text
CORE-BABA/
├── README.md
├── show-version.yml
├── baba-base.yml
├── baba-lacp.yml
├── baba-dhcp.yml
├── baba-vlans.yml
└── baba-camera-dhcp.yml
```

| File | Purpose |
|---|---|
| `show-version.yml` | Read-only connectivity/device verification |
| `baba-base.yml` | Day 1 CORE BABA base Layer 3/interface configuration |
| `baba-lacp.yml` | Fa0/10-12 trunking and LACP |
| `baba-dhcp.yml` | DHCP excluded addresses and pools |
| `baba-vlans.yml` | VLAN creation and port placement |
| `baba-camera-dhcp.yml` | Camera DHCP reservations; do not run until actual client IDs are known |

> The earlier `interface.yml` was only a connectivity/configuration test and is intentionally excluded from the final Day1SirRob playbook set.

## SSH Prerequisite
**Run on: CISCO CLI**

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

<!-- SCREENSHOT: CORE BABA SSH -->
<!-- SCREENSHOT: Show Version -->
<!-- SCREENSHOT: LACP -->
<!-- SCREENSHOT: DHCP -->
<!-- SCREENSHOT: VLANs -->
