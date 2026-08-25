# Cisco Automation with Ansible and Semaphore

This repository documents a Cisco automation lab using Ansible and Semaphore.

## Architecture

```text
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
       Cisco Devices
```

`~~` represents the assigned monitor/student number. Replace it only where required by the lab configuration.

## Documentation

### Environment Setup

1. [WSL2 and Ubuntu](Setup/WSL2-Ubuntu.md)
2. [Docker and Semaphore](Setup/Docker-Semaphore.md)
3. [VS Code Remote SSH](Setup/VSCode-Remote-SSH.md)

### Cisco Devices

- [CORE BABA](CORE-BABA/README.md)
- [CORE TAAS](CORE-TAAS/README.md)

### Troubleshooting

- [Troubleshooting Guide](Troubleshooting.md)

## Configuration Source

Cisco device configuration in this repository follows `DAY1-May5-SirRob.txt`.

Do not introduce unrelated VLANs, interfaces, addressing, or Cisco configuration into the device playbooks. SSH-related commands documented here are automation prerequisites where required.

<!-- SCREENSHOT: Overall lab topology -->
