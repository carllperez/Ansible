# VS Code Terminal and Ubuntu YAML Workflow

VS Code runs on the physical PC and connects to the Windows Server VM. The integrated terminal then enters WSL Ubuntu. All YAML files are created inside Ubuntu under `~/ansible-lab`.

## Workflow

```text
PHYSICAL PC: VS Code
  → Remote SSH to Windows Server VM
  → VS Code integrated terminal
  → wsl -d Ubuntu
  → create/edit ~/ansible-lab/*.yml inside Ubuntu
  → docker cp ~/ansible-lab/<file> semaphore:/ansible/<file>
```

`~/ansible-lab` is the source of truth. Do not maintain a second working copy under `C:\Users\Administrator`.

## 1. Connect VS Code to the Windows Server VM

Install Microsoft's **Remote - SSH** extension on the physical PC.

**Run on: PHYSICAL PC → POWERSHELL or CMD**

```powershell
ssh Administrator@208.8.8.~~
```

In VS Code run:

```text
Remote-SSH: Connect to Host
ssh Administrator@208.8.8.~~
```

The status bar should show `SSH: 208.8.8.~~`.

<!-- SCREENSHOT: VS Code status bar showing SSH connection to the VM -->

## 2. Enter Ubuntu from the VS Code Terminal

Open **Terminal → New Terminal**.

**Run in: PHYSICAL PC → VS Code integrated terminal connected to the VM**

```powershell
wsl -d Ubuntu
```

Confirm the current environment:

```bash
whoami
uname -a
pwd
```

## 3. Create the Ubuntu Working Folder

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
mkdir -p ~/ansible-lab
cd ~/ansible-lab
pwd
```

All files in this tutorial are pasted here.

## 4. Where to Paste a YAML File

Example for `baba-lacp.yml`:

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
cd ~/ansible-lab
nano baba-lacp.yml
```

Then:

1. Copy all YAML from `CORE-BABA/baba-lacp.yml` in this package/repository.
2. Paste it into the open Nano editor in the Ubuntu terminal.
3. Replace every `~~` with the assigned monitor number.
4. Save with **Ctrl+O**, press **Enter**.
5. Exit with **Ctrl+X**.
6. Verify the file:

```bash
sed -n '1,240p' ~/ansible-lab/baba-lacp.yml
```

Repeat the same procedure using these exact Ubuntu filenames:

```text
~/ansible-lab/show-version-baba.yml
~/ansible-lab/baba-base.yml
~/ansible-lab/baba-lacp.yml
~/ansible-lab/baba-dhcp.yml
~/ansible-lab/baba-vlans.yml
~/ansible-lab/baba-camera-dhcp.yml
~/ansible-lab/show-version-taas.yml
~/ansible-lab/taas-base.yml
~/ansible-lab/taas-trunk.yml
~/ansible-lab/taas-lacp.yml
~/ansible-lab/inventory.ini
```

The repository contains two files named `show-version.yml`. Paste the BABA one as `show-version-baba.yml` and the TAAS one as `show-version-taas.yml`.

<!-- SCREENSHOT: Ubuntu terminal listing every file in ~/ansible-lab -->

## 5. Verify No Monitor Placeholders Remain

The reusable repository keeps `~~`, but the Ubuntu working copies must use the assigned number before execution.

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
grep -R -n -- '~~' ~/ansible-lab
```

If the command prints a YAML or inventory line, edit that file again before copying it to Semaphore.

## 6. Copy Ubuntu Files into the Existing Semaphore Container

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
sudo docker cp ~/ansible-lab/inventory.ini semaphore:/ansible/inventory.ini
sudo docker cp ~/ansible-lab/show-version-baba.yml semaphore:/ansible/show-version-baba.yml
sudo docker cp ~/ansible-lab/baba-base.yml semaphore:/ansible/baba-base.yml
sudo docker cp ~/ansible-lab/baba-lacp.yml semaphore:/ansible/baba-lacp.yml
sudo docker cp ~/ansible-lab/baba-dhcp.yml semaphore:/ansible/baba-dhcp.yml
sudo docker cp ~/ansible-lab/baba-vlans.yml semaphore:/ansible/baba-vlans.yml
sudo docker cp ~/ansible-lab/baba-camera-dhcp.yml semaphore:/ansible/baba-camera-dhcp.yml

sudo docker cp ~/ansible-lab/show-version-taas.yml semaphore:/ansible/show-version-taas.yml
sudo docker cp ~/ansible-lab/taas-base.yml semaphore:/ansible/taas-base.yml
sudo docker cp ~/ansible-lab/taas-trunk.yml semaphore:/ansible/taas-trunk.yml
sudo docker cp ~/ansible-lab/taas-lacp.yml semaphore:/ansible/taas-lacp.yml
```

Verify file names and non-zero sizes:

```bash
sudo docker exec semaphore ls -lh /ansible
```

<!-- SCREENSHOT: Existing Semaphore /ansible directory showing all copied files -->

## 7. Syntax Check

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/show-version-baba.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/baba-base.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/baba-lacp.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/baba-dhcp.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/baba-vlans.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/baba-camera-dhcp.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/show-version-taas.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/taas-base.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/taas-trunk.yml --syntax-check
sudo docker exec semaphore ansible-playbook -i /ansible/inventory.ini /ansible/taas-lacp.yml --syntax-check
```

Syntax checking does not change the Cisco devices.

## 8. Update Cycle

After changing a playbook:

1. edit and save it under `~/ansible-lab` in Ubuntu;
2. confirm `~~` is gone from the working copy;
3. run its `sudo docker cp ~/ansible-lab/...` command again;
4. confirm its size in `/ansible`;
5. syntax-check it;
6. run the existing Semaphore template only after confirming the correct `hosts:` group.
