# WSL2 and Ubuntu Setup

Ubuntu on WSL2 provides the Linux environment for Docker, Ansible, and Semaphore. In this lab WSL2 runs inside a Windows Server 2022 VMware VM, so VMware must expose nested virtualization to the VM.

This guide includes both the short installation path and the manual Windows Server 2022 path used in the original lab.

> **Existing working system:** Run the verification step first. If Ubuntu already reports `VERSION 2`, do not reinstall WSL or Ubuntu.

## What You Are Doing

- **WSL2** lets Windows run an Ubuntu Linux environment.
- **Nested virtualization** lets WSL2 run inside the Windows Server VMware VM.
- **Ubuntu** is where the existing Docker and Semaphore services run.

This page has two paths. If Ubuntu already shows `VERSION 2`, use only the verification and update steps. Use the installation steps only when Ubuntu is missing or still uses WSL1.

## Command Location Labels

- **PHYSICAL PC** — the computer in front of you.
- **WINDOWS SERVER VM — POWERSHELL (ADMIN)** — PowerShell inside the VM, opened as Administrator.
- **WINDOWS SERVER VM → WEB BROWSER** — a browser opened inside the VM for an official Microsoft download.
- **VM → WSL UBUNTU** — the Ubuntu terminal opened with `wsl -d Ubuntu`.

## 1. Verify Before Installing

**Run on: WINDOWS SERVER VM → POWERSHELL (ADMIN)**

```powershell
wsl -l -v
```

If the result already shows Ubuntu on version 2, skip to [Update Ubuntu](#8-update-ubuntu):

```text
NAME      STATE      VERSION
Ubuntu    Stopped    2
```

If Ubuntu appears as version 1, do not download it again. Continue through the prerequisite and conversion sections.

If no distribution is installed, continue through the complete guide.

### Screenshot guide: Confirm Ubuntu is WSL2

- **Capture:** Administrator PowerShell after `wsl -l -v` finishes.
- **Success must show:** an Ubuntu row and `VERSION 2`.
- **Hide:** unrelated distribution names if they reveal another project; no password should appear.
- **Status:** Screenshot pending.

## 2. Enable Nested Virtualization

**Do on: PHYSICAL PC → VMware Workstation, while the Windows Server VM is powered off**

```text
VM Settings
→ Processors
→ Virtualization Engine
→ Enable “Virtualize Intel VT-x/EPT or AMD-V/RVI”
```

The CPU performance-counter and IOMMU options are not required for this lab. Start the Windows Server VM after enabling the virtualization-engine option.

Verify that Windows Server detects the hypervisor:

**Run on: WINDOWS SERVER VM → POWERSHELL (ADMIN)**

```powershell
systeminfo | findstr /i "Hyper-V"
```

The original lab eventually reported that a hypervisor had been detected. If WSL reports `HCS_E_HYPERV_NOT_INSTALLED`, power off the VM and correct this VMware setting before reinstalling anything.

### Screenshot guide: VMware nested virtualization

- **Capture:** the powered-off VM’s **Processors → Virtualization Engine** settings.
- **Success must show:** **Virtualize Intel VT-x/EPT or AMD-V/RVI** enabled.
- **Hide:** unrelated VM names or infrastructure details not needed for this lab.
- **Status:** Screenshot pending.

## 3. Enable the Windows Features

**Run on: WINDOWS SERVER VM → POWERSHELL (ADMIN)**

```powershell
Get-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
Get-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform
```

Enable a feature only if it is disabled:

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
Restart-Computer
```

On the original Windows Server 2022 build, the automatic installer initially reported `0x800f080c` and did not handle `VirtualMachinePlatform` correctly. If that happens, install the pending Windows Server cumulative updates, restart the VM, and repeat the feature checks. Do not repeatedly reinstall Ubuntu.

The original inbox WSL executable was version `10.0.20348.1`. You can check it with:

```powershell
(Get-Item C:\Windows\System32\wsl.exe).VersionInfo | Select FileVersion, ProductVersion
```

## 4. Install the WSL2 Kernel When Required

Check WSL status:

```powershell
wsl --status
```

If Windows reports that the WSL2 kernel is missing, use Microsoft's official kernel package:

```text
https://aka.ms/wsl2kernel
```

**Do on: WINDOWS SERVER VM → WEB BROWSER**

Download and run `wsl_update_x64.msi`, then return to Administrator PowerShell. The original lab reported kernel version `5.10.16` after this installation.

## 5. Try the Short Ubuntu Installation Path

**Run on: WINDOWS SERVER VM → POWERSHELL (ADMIN)**

```powershell
wsl --list --online
wsl --install -d Ubuntu-20.04
```

If Ubuntu installs, launch it and create the requested Linux username and password. The original lab used `ubadmin`.

If the install command fails while downloading or installing Ubuntu, use the manual procedure below.

## 6. Manual Ubuntu 20.04 Installation Used in the Lab

**Run on: WINDOWS SERVER VM → POWERSHELL (ADMIN)**

```powershell
New-Item -ItemType Directory -Path C:\Users\Administrator\WSL -Force
Set-Location C:\Users\Administrator\WSL
Invoke-WebRequest -Uri https://aka.ms/wslubuntu2004 -OutFile Ubuntu.appx -UseBasicParsing
Get-Item .\Ubuntu.appx | Select Name, Length
Add-AppxPackage .\Ubuntu.appx
```

The original download was `937,972,031` bytes. `Invoke-WebRequest` can be slow. For a new download, this faster alternative can be used instead of `Invoke-WebRequest`:

```powershell
curl.exe -L "https://aka.ms/wslubuntu2004" -o Ubuntu.appx
```

Do not run both download commands, and do not cancel a nearly complete download just to switch methods.

Launch the installed distribution:

```powershell
ubuntu2004
```

If that command is unavailable, try:

```powershell
ubuntu
```

Create the requested Linux username and password. A manually installed distribution can initially appear as WSL 1; convert it in the next section before installing Docker.

## 7. Convert and Verify Ubuntu as WSL2

**Run on: WINDOWS SERVER VM → POWERSHELL (ADMIN)**

```powershell
wsl -l -v
wsl --shutdown
wsl --set-version Ubuntu 2
wsl -l -v
```

Expected final result:

```text
NAME      STATE      VERSION
Ubuntu    Stopped    2
```

`wsl --set-default-version 2` controls future installations; `wsl --set-version Ubuntu 2` converts the existing Ubuntu installation.

In the original lab, the old `wsl.exe` displayed its help page instead of performing the conversion. The successful fix was to install the pending Windows Server cumulative update, restart the VM, confirm nested virtualization and both Windows features, and then retry `wsl --set-version Ubuntu 2`—without reinstalling Ubuntu.

Open Ubuntu:

```powershell
wsl -d Ubuntu
```

## 8. Update Ubuntu

**Run on: VM → WSL UBUNTU**

```bash
sudo apt update
sudo apt upgrade -y
```

At this point Ubuntu is ready for the existing Docker, Ansible, and Semaphore verification guide.

## Passing Checkpoint

Do not continue to Docker until all of these are true:

- `wsl -l -v` lists Ubuntu with `VERSION 2`;
- `wsl -d Ubuntu` opens an Ubuntu prompt without a virtualization error;
- `sudo apt update` completes successfully;
- an existing working Ubuntu installation was preserved instead of reinstalled.

Continue with [Docker and the existing Semaphore container](Docker-Semaphore.md).

### Screenshot guide: WSL passing checkpoint

- **Capture:** `wsl -l -v` showing version 2 beside an open Ubuntu prompt.
- **Success must show:** Ubuntu opens without a virtualization error.
- **Hide:** anything typed as a Linux password; passwords do not echo and must never be staged for a screenshot.
- **Status:** Screenshot pending.
