# VS Code Remote-SSH YAML Workflow

VS Code runs on the physical PC. Its Remote-SSH session connects to the Windows Server VM and saves the YAML directly into:

~~~text
C:\Users\Administrator\ansible-lab
~~~

WSL Ubuntu sees that same VM folder as:

~~~text
/mnt/c/Users/Administrator/ansible-lab
~~~

The Windows VM folder is the source of truth. Do not recreate the same files with Nano under the Ubuntu home directory.

## What This Means for a Beginner

- VS Code is installed and opened on the physical PC.
- **Remote-SSH** makes that VS Code window edit files located on the Windows Server VM.
- **Source of truth** means this is the one authoritative editable copy.
- WSL does not have a second copy; `/mnt/c/...` is Ubuntu's view of the same Windows folder.
- Semaphore does use a separate runtime copy inside the container, so every saved change must be copied again with `docker cp`.

You do not need the VS Code WSL extension for this workflow. The VS Code window remains connected to Windows Server over SSH; only the integrated terminal enters Ubuntu when Docker commands are needed.

## Actual File Flow

~~~text
PHYSICAL PC — VS Code
  → Remote-SSH to 208.8.8.200
  → edit/save YAML on the Windows Server VM
  → C:\Users\Administrator\ansible-lab
  → WSL view: /mnt/c/Users/Administrator/ansible-lab
  → docker cp
  → existing semaphore:/ansible
~~~

## 1. Connect VS Code to the VM

Install Microsoft's **Remote - SSH** extension on the physical PC.

**Run on: PHYSICAL PC → POWERSHELL or CMD**

~~~powershell
ssh Administrator@208.8.8.200
~~~

In VS Code:

~~~text
Remote-SSH: Connect to Host
ssh Administrator@208.8.8.200
~~~

The lower-left status area should show SSH: 208.8.8.200.

If the status area does not show that connection, stop. Files created in an ordinary local VS Code window will be saved on the wrong computer.

### Screenshot guide: VS Code is connected to the VM

- **Capture:** the complete VS Code window after Remote-SSH connects.
- **Success must show:** `SSH: 208.8.8.200` in the lower-left status area.
- **Hide:** unrelated open files, saved credentials, and personal account information.
- **Status:** Included below.

The complete VS Code view shows the remote VM folder, the YAML files, and `SSH: 208.8.8.200` in the lower-left corner.

<img width="1018" height="768" alt="VS Code connected by Remote SSH to 208.8.8.200 with the ansible-lab YAML files visible" src="https://github.com/user-attachments/assets/04996a4a-0c81-4d45-b696-dd00fbaca8fd" />

Zoomed view of the Remote-SSH connection indicator:

<img width="122" height="26" alt="Zoomed VS Code status showing SSH connection to 208.8.8.200" src="https://github.com/user-attachments/assets/ce9c1761-60f9-4c7e-a8bf-0edec3309ead" />

## 2. Open the YAML Folder on the VM

**Do in: PHYSICAL PC → VS Code Remote-SSH window**

1. Select **File → Open Folder**.
2. Open C:\Users\Administrator\ansible-lab.
3. Trust the folder if VS Code asks.
4. Create and edit all YAML and inventory files in this remote folder.
5. Save with **Ctrl+S**. Remote-SSH writes the saved content to the VM.

When VS Code asks whether you trust the folder, confirm only after checking that the displayed path is exactly the VM folder above.

Create the folder first if it does not exist:

**Run in: VS Code TERMINAL → WINDOWS SERVER VM — POWERSHELL**

~~~powershell
New-Item -ItemType Directory -Force C:\Users\Administrator\ansible-lab
~~~

## 3. Exact VS Code Filenames

Save the Day 1 working files under these VM paths:

~~~text
C:\Users\Administrator\ansible-lab\inventory.ini
C:\Users\Administrator\ansible-lab\show-version-baba.yml
C:\Users\Administrator\ansible-lab\baba-base.yml
C:\Users\Administrator\ansible-lab\baba-lacp.yml
C:\Users\Administrator\ansible-lab\baba-dhcp.yml
C:\Users\Administrator\ansible-lab\baba-vlans.yml
C:\Users\Administrator\ansible-lab\baba-camera-dhcp.yml
C:\Users\Administrator\ansible-lab\show-version-taas.yml
C:\Users\Administrator\ansible-lab\taas-base.yml
C:\Users\Administrator\ansible-lab\taas-trunk.yml
C:\Users\Administrator\ansible-lab\taas-lacp.yml
~~~

The repository contains two files named show-version.yml. Save the BABA file as show-version-baba.yml and the TAAS file as show-version-taas.yml in the VS Code VM folder.

`interface.yml`, `baba.yml`, and a Semaphore task named `Configure Cisco Interface` are earlier tests, not required working files. Do not copy or run them as part of a new BABA or TAAS deployment. Use the documented base playbook only when the switch received the minimum connectivity/SSH bootstrap; skip it if the complete base was applied manually.

`inventory.ini` in this folder is the editable file used for terminal syntax checks and manual Ansible tests. The current working Semaphore tasks use a separate inline entry named `Cisco Inventory` in the GUI. Whenever a switch group or IP address changes, update both copies so the GUI and terminal select the same devices. The Key Store supplies the GUI credential; do not paste a real production password into GitHub.

For the safe multi-monitor files, preserve this subfolder structure:

~~~text
C:\Users\Administrator\ansible-lab\Reusable-Multi-Monitor\inventory.ini
C:\Users\Administrator\ansible-lab\Reusable-Multi-Monitor\CORE-BABA\*.yml
C:\Users\Administrator\ansible-lab\Reusable-Multi-Monitor\CORE-TAAS\*.yml
~~~

## 4. Edit Monitor Values in VS Code

For the original Day 1 working copies, replace every ~~ with the assigned monitor number before deployment.

Example for BABA 72:

~~~text
COREbaba-~~  → COREbaba-72
10.~~.1.4   → 10.72.1.4
~~~

Do not replace {{ monitor }} in Reusable-Multi-Monitor. Those files read the value from inventory.

## 5. Verify the VM Files from WSL

Open the VS Code integrated terminal. It initially runs on the Windows Server VM.

**Run in: PHYSICAL PC → VS Code TERMINAL → WINDOWS SERVER VM**

~~~powershell
wsl -d Ubuntu
~~~

The prompt should change from a Windows PowerShell prompt such as:

```text
PS C:\Users\Administrator>
```

to an Ubuntu prompt similar to:

```text
ubadmin@WIN-...:~$
```

Do not type `PS C:\Users\Administrator>` or `ubadmin@WIN-...:~$`; those are prompt examples, not commands.

**Then run on: VS Code TERMINAL → VM → WSL UBUNTU**

~~~bash
ls -lh /mnt/c/Users/Administrator/ansible-lab
grep -R -n -- '~~' /mnt/c/Users/Administrator/ansible-lab
~~~

For the original Day 1 working files, the grep command should return no unreplaced monitor placeholders. It is normal for explanatory Markdown or untouched GitHub templates to contain ~~; do not run those files as working playbooks.

### Screenshot guide: WSL sees the files saved by VS Code

- **Capture:** the VS Code terminal after `ls -lh /mnt/c/Users/Administrator/ansible-lab`.
- **Success must show:** the expected YAML filenames and sizes greater than zero.
- **Hide:** unrelated filenames or folders belonging to other projects.
- **Status:** Included below.

The WSL terminal lists the same YAML files saved in the Windows VM folder. The older test files visible in this historical screenshot are not part of the required deployment workflow; follow the file list in this guide.

<img width="711" height="336" alt="WSL terminal listing non-empty YAML files in the Windows Server ansible-lab folder" src="https://github.com/user-attachments/assets/019319ad-49b4-442a-89d1-a999ff18c6bb" />

## 6. Copy the Day 1 Files into Existing Semaphore

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

~~~bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/inventory.ini semaphore:/ansible/inventory.ini
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/show-version-baba.yml semaphore:/ansible/show-version-baba.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-base.yml semaphore:/ansible/baba-base.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-lacp.yml semaphore:/ansible/baba-lacp.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-dhcp.yml semaphore:/ansible/baba-dhcp.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-vlans.yml semaphore:/ansible/baba-vlans.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/baba-camera-dhcp.yml semaphore:/ansible/baba-camera-dhcp.yml

sudo docker cp /mnt/c/Users/Administrator/ansible-lab/show-version-taas.yml semaphore:/ansible/show-version-taas.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/taas-base.yml semaphore:/ansible/taas-base.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/taas-trunk.yml semaphore:/ansible/taas-trunk.yml
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/taas-lacp.yml semaphore:/ansible/taas-lacp.yml
~~~

Verify the runtime copies:

~~~bash
sudo docker exec semaphore ls -lh /ansible
~~~

Copying `inventory.ini` into the container does not overwrite Semaphore's inline GUI inventory. After an inventory change, also open **Semaphore → Inventory → Cisco Inventory**, update the matching `[baba]` and `[taas]` entries, and save it. Follow [Semaphore Project Setup](Semaphore-Project.md#2-verify-or-create-the-inventory).

## 7. Copy the Reusable Multi-Monitor Files

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

~~~bash
sudo docker exec -u 0 semaphore mkdir -p /ansible/reusable/CORE-BABA
sudo docker exec -u 0 semaphore mkdir -p /ansible/reusable/CORE-TAAS

sudo docker cp /mnt/c/Users/Administrator/ansible-lab/Reusable-Multi-Monitor/inventory.ini semaphore:/ansible/reusable/inventory.ini
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/Reusable-Multi-Monitor/CORE-BABA/. semaphore:/ansible/reusable/CORE-BABA/
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/Reusable-Multi-Monitor/CORE-TAAS/. semaphore:/ansible/reusable/CORE-TAAS/
~~~

## 8. Syntax Check

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

~~~bash
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
~~~

Syntax checking does not configure the Cisco devices.

### Screenshot guide: Successful syntax check

- **Capture:** one complete `ansible-playbook ... --syntax-check` command and its result.
- **Success must show:** the correct playbook name with no YAML or syntax error.
- **Hide:** inventory credentials or unrelated terminal history.
- **Status:** Screenshot pending.

## 9. Correct Update Cycle

After changing a playbook:

1. edit it in the VS Code Remote-SSH folder;
2. save it to C:\Users\Administrator\ansible-lab on the VM;
3. enter WSL from the VS Code terminal;
4. verify the same file under /mnt/c/Users/Administrator/ansible-lab;
5. run the matching docker cp command again;
6. verify the file size in semaphore:/ansible;
7. syntax-check it;
8. run the intended Semaphore template only after confirming its target.

There is no Nano editing step and no second source-of-truth copy under the Ubuntu home directory.

## Passing Checkpoint

Before opening Semaphore, confirm:

```text
[ ] VS Code shows SSH: 208.8.8.200
[ ] The open folder is C:\Users\Administrator\ansible-lab
[ ] Ctrl+S saves without an error
[ ] WSL lists the same files under /mnt/c/Users/Administrator/ansible-lab
[ ] The required files exist inside semaphore:/ansible with non-zero sizes
[ ] Semaphore's Cisco Inventory has the same BABA/TAAS groups and IPs as inventory.ini
[ ] Every syntax check succeeds
```
