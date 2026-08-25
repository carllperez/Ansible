# VS Code Remote SSH

## Overview

VS Code runs on the physical PC and connects to the Windows Server VM through SSH.

The working workflow is:

```text
Physical PC VS Code
→ SSH to Windows Server VM
→ C:\Users\Administrator\ansible-lab
→ WSL terminal
→ docker cp
→ Semaphore /ansible
```

## 1. Enable OpenSSH on Windows Server

PowerShell as Administrator:

```powershell
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH.Server*'
```

If required:

```powershell
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0
```

Start SSH:

```powershell
Start-Service sshd
Set-Service -Name sshd -StartupType Automatic
```

Verify:

```powershell
Get-Service sshd
netstat -ano | findstr :22
```

<!-- SCREENSHOT: sshd running -->

## 2. Test from the Physical PC

```powershell
ssh Administrator@208.8.8.~~
```

<!-- SCREENSHOT: Successful PC to VM SSH -->

## 3. VS Code

Install the Microsoft **Remote - SSH** extension.

Add:

```text
ssh Administrator@208.8.8.~~
```

Connect with:

```text
Remote-SSH: Connect to Host
```

The status bar should show:

```text
SSH: 208.8.8.~~
```

<!-- SCREENSHOT: VS Code Remote SSH connected -->

## 4. Windows Working Copy

Create/use:

```text
C:\Users\Administrator\ansible-lab
```

From Ubuntu, the Windows path is:

```text
/mnt/c/Users/Administrator/ansible-lab
```

<!-- SCREENSHOT: ANSIBLE-LAB in VS Code Explorer -->

## 5. Enter Ubuntu from VS Code

Open the integrated terminal:

```powershell
wsl -d Ubuntu
```

## 6. Copy Playbooks to Semaphore

General command:

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/<PLAYBOOK>.yml semaphore:/ansible/<PLAYBOOK>.yml
```

CORE BABA examples:

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-lacp.yml semaphore:/ansible/baba-lacp.yml
```

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-dhcp.yml semaphore:/ansible/baba-dhcp.yml
```

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-vlans.yml semaphore:/ansible/baba-vlans.yml
```

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-camera-dhcp.yml semaphore:/ansible/baba-camera-dhcp.yml
```

Verify:

```bash
sudo docker exec semaphore ls -la /ansible
```

<!-- SCREENSHOT: docker cp from VS Code -->
<!-- SCREENSHOT: copied files in /ansible -->

## 7. Syntax Check

```bash
sudo docker exec -it semaphore bash
```

Then:

```bash
cd /ansible
ansible-playbook -i inventory.ini baba-lacp.yml --syntax-check
```

Exit:

```bash
exit
```

> Run the appropriate `docker cp` command again every time a playbook is changed in the Windows VS Code working folder.
