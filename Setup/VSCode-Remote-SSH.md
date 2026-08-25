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

## Actual File Flow

~~~text
PHYSICAL PC — VS Code
  → Remote-SSH to 208.8.8.~~
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
ssh Administrator@208.8.8.~~
~~~

In VS Code:

~~~text
Remote-SSH: Connect to Host
ssh Administrator@208.8.8.~~
~~~

The lower-left status area should show SSH: 208.8.8.~~.

<!-- SCREENSHOT: VS Code showing the Remote-SSH connection to the Windows Server VM -->

## 2. Open the YAML Folder on the VM

**Do in: PHYSICAL PC → VS Code Remote-SSH window**

1. Select **File → Open Folder**.
2. Open C:\Users\Administrator\ansible-lab.
3. Trust the folder if VS Code asks.
4. Create and edit all YAML and inventory files in this remote folder.
5. Save with **Ctrl+S**. Remote-SSH writes the saved content to the VM.

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

**Then run on: VS Code TERMINAL → VM → WSL UBUNTU**

~~~bash
ls -lh /mnt/c/Users/Administrator/ansible-lab
grep -R -n -- '~~' /mnt/c/Users/Administrator/ansible-lab
~~~

For the original Day 1 working files, the grep command should return no unreplaced monitor placeholders. It is normal for explanatory Markdown or untouched GitHub templates to contain ~~; do not run those files as working playbooks.

<!-- SCREENSHOT: WSL listing the YAML files from the Windows VM mounted path -->

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
