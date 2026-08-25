# WSL2 and Ubuntu Setup

Ubuntu on WSL2 provides the Linux environment for Docker, Ansible, and Semaphore. In this lab WSL2 runs inside a Windows Server VMware VM, so nested virtualization must be enabled.

## Command Location Labels

- **PHYSICAL PC** — the computer in front of you.
- **WINDOWS SERVER VM — POWERSHELL (ADMIN)** — PowerShell inside the VM, opened as Administrator.
- **VM → WSL UBUNTU** — the Ubuntu terminal opened with `wsl -d Ubuntu`.

## 1. Enable Nested Virtualization

**Do on: PHYSICAL PC → VMware Workstation, while the Windows Server VM is powered off**

```text
VM Settings
→ Processors
→ Virtualization Engine
→ Enable “Virtualize Intel VT-x/EPT or AMD-V/RVI”
```

Start the Windows Server VM.

## 2. Install WSL

**Run on: WINDOWS SERVER VM — POWERSHELL (ADMIN)**

```powershell
wsl --version
wsl --install
```

If Windows Server requires manual feature enablement:

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
Restart-Computer
```

After the VM restarts:

```powershell
wsl --set-default-version 2
```

## 3. Install Ubuntu

**Run on: WINDOWS SERVER VM — POWERSHELL (ADMIN)**

```powershell
wsl --install -d Ubuntu
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

Open Ubuntu and create the requested Linux username/password:

```powershell
wsl -d Ubuntu
```

The original lab used the Ubuntu username `ubadmin`.

<!-- SCREENSHOT: wsl -l -v showing Ubuntu version 2 -->

## 4. Update Ubuntu

**Run on: VM → WSL UBUNTU**

```bash
sudo apt update
sudo apt upgrade -y
```

If WSL reports `HCS_E_HYPERV_NOT_INSTALLED`, return to step 1 and enable nested virtualization before reinstalling anything.
