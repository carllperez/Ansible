# WSL2 and Ubuntu Setup

## Overview

Ubuntu on WSL2 provides the Linux environment used for Ansible, Docker, and Semaphore.

This lab runs WSL2 inside a Windows Server VMware virtual machine, so nested virtualization is required.

## 1. Install WSL

Open **PowerShell as Administrator**:

```powershell
wsl --version
```

If WSL is not installed:

```powershell
wsl --install
```

If Windows Server requires manual feature enablement:

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

Restart:

```powershell
Restart-Computer
```

<!-- SCREENSHOT: WSL installation -->

## 2. Set WSL2 as Default

```powershell
wsl --set-default-version 2
```

## 3. Install Ubuntu

```powershell
wsl --install -d Ubuntu
```

Verify:

```powershell
wsl -l -v
```

Expected:

```text
NAME      STATE      VERSION
Ubuntu    Stopped    2
```

If Ubuntu is version 1:

```powershell
wsl --set-version Ubuntu 2
```

Open Ubuntu:

```powershell
wsl -d Ubuntu
```

The Ubuntu username used in the lab was:

```text
ubadmin
```

<!-- SCREENSHOT: Ubuntu installed as WSL2 -->

## 4. Nested Virtualization

If WSL2 returns:

```text
HCS_E_HYPERV_NOT_INSTALLED
WSL2 is unable to start since virtualization is not enabled on this machine
```

Shut down the Windows Server VM completely.

In VMware Workstation:

```text
VM
→ Settings
→ Processors
→ Virtualization Engine
```

Enable:

```text
Virtualize Intel VT-x/EPT or AMD-V/RVI
```

Start the VM and verify:

```powershell
wsl -l -v
wsl -d Ubuntu
```

<!-- SCREENSHOT: VMware nested virtualization -->
<!-- SCREENSHOT: Working WSL2 -->

## 5. Update Ubuntu

```bash
sudo apt update
```

Optional:

```bash
sudo apt upgrade -y
```

<!-- SCREENSHOT: sudo apt update completed -->
