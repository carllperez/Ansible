# Troubleshooting

Every command below includes its run location. Do not paste PowerShell commands into Ubuntu or Cisco commands into Windows.

## WSL2: `HCS_E_HYPERV_NOT_INSTALLED`

Cause: nested virtualization is not exposed to the Windows Server VMware VM.

**Do on: PHYSICAL PC → VMware Workstation, with the VM powered off**

```text
VM Settings
→ Processors
→ Enable “Virtualize Intel VT-x/EPT or AMD-V/RVI”
```

**Then run on: WINDOWS SERVER VM — POWERSHELL (ADMIN)**

```powershell
wsl -l -v
wsl -d Ubuntu
```

## Docker: Cannot Connect to the Daemon

Error:

```text
Cannot connect to the Docker daemon at unix:///var/run/docker.sock
```

**Run on: VM → WSL UBUNTU**

```bash
which docker
docker context ls
sudo service docker start
sudo docker ps
```

If `sudo service docker start` returns `docker: unrecognized service`:

```bash
sudo dockerd > /tmp/dockerd.log 2>&1 &
sudo docker ps
```

If it still fails:

```bash
cat /tmp/dockerd.log
```

Do not repeatedly reinstall Docker when the client exists but only the daemon is stopped.

## Semaphore Container Is Stopped or Missing

**Run on: VM → WSL UBUNTU**

```bash
sudo docker ps -a
sudo docker start semaphore
sudo docker logs semaphore
```

Starting the existing container does not recreate it. If the container is truly missing, stop and recover the original deployment information instead of inventing a new database, account, or replacement container from this tutorial.

## Semaphore GUI Works on `localhost` but Not the VM IP

**Run on: WINDOWS SERVER VM — POWERSHELL (ADMIN)**

```powershell
Test-NetConnection localhost -Port 3000
Test-NetConnection 208.8.8.200 -Port 3000
wsl -d Ubuntu hostname -I
netsh interface portproxy show all
```

If the WSL IP changed, replace the old proxy:

```powershell
netsh interface portproxy delete v4tov4 listenaddress=208.8.8.200 listenport=3000
netsh interface portproxy add v4tov4 listenaddress=208.8.8.200 listenport=3000 connectaddress=<NEW-WSL-IP> connectport=3000
New-NetFirewallRule -DisplayName "Semaphore 3000" -Direction Inbound -Protocol TCP -LocalPort 3000 -Action Allow
```

Then open on the physical PC:

```text
http://208.8.8.200:3000
```

## PC Can SSH to the VM but Cannot Ping It

SSH and ICMP use different Windows firewall rules.

**Run on: WINDOWS SERVER VM — POWERSHELL (ADMIN)**

```powershell
New-NetFirewallRule -DisplayName "Allow ICMPv4 Ping" -Direction Inbound -Protocol ICMPv4 -IcmpType 8 -Action Allow
```

## Cisco SSH Says `Connection refused`

The switch is reachable but SSH is not enabled/bound to the VTY lines.

**Run on: TARGET SWITCH → CISCO CLI**

```cisco
show ip ssh
show running-config | section line vty
show running-config | include username|domain-name
```

Apply the SSH bootstrap from the matching CORE BABA or CORE TAAS README.

## No Acceptable KEX, Host Key, Cipher, or MAC

Older Cisco IOS SSH algorithms can be rejected by modern OpenSSH/Paramiko.

**Manual test from: VM → WSL UBUNTU**

```bash
ssh \
  -o KexAlgorithms=+diffie-hellman-group14-sha1 \
  -o HostKeyAlgorithms=+ssh-rsa \
  -o Ciphers=+aes128-cbc \
  -o MACs=+hmac-sha1 \
  admin@10.~~.1.4
```

The documented inventory contains equivalent compatibility options:

```ini
ansible_ssh_common_args='-o KexAlgorithms=+diffie-hellman-group14-sha1 -o HostKeyAlgorithms=+ssh-rsa -o Ciphers=+aes128-cbc -o MACs=+hmac-sha1'
```

Use these only for the older lab equipment; prefer modern SSH algorithms on production devices.

## Paramiko Compatibility

The working lab needed Paramiko 3.5.1 after Paramiko 4.0.0 caused legacy-device negotiation failures.

**Check on: VM → WSL UBUNTU**

```bash
python3 -c "import paramiko; print(paramiko.__version__)"
```

**Check inside: SEMAPHORE CONTAINER**

```bash
sudo docker exec semaphore python3 -c "import paramiko; print(paramiko.__version__)"
```

Historical recovery note: the earlier Semaphore image used this virtual-environment path. **Do not run these commands when the existing tasks work.** Use them only when the installed version is confirmed as the cause and the exact path exists:

```bash
sudo docker exec -u 0 semaphore /opt/semaphore/apps/ansible/13.5.0/venv/bin/pip install "paramiko<4"
sudo docker exec semaphore /opt/semaphore/apps/ansible/13.5.0/venv/bin/python -c "import paramiko; print(paramiko.__version__)"
sudo docker restart semaphore
```

The internal path can change with Semaphore releases. If it does not exist, inspect the installed Ansible environment instead of creating a guessed path.

## `ios_config` or `ios_command` Is Not Found

**Run on: VM → WSL UBUNTU**

```bash
sudo docker exec semaphore ansible-galaxy collection list | grep cisco.ios
sudo docker exec semaphore ansible-doc ios_config
sudo docker exec semaphore ansible-doc ios_command
```

Because this tutorial preserves the working Semaphore environment, do not install or upgrade collections automatically. Record the output and compare it with the working playbooks/container before changing the environment.

## `/ansible` or a YAML File Is Missing

**Run on: VM → WSL UBUNTU**

```bash
sudo docker exec -u 0 semaphore mkdir -p /ansible
sudo docker exec semaphore ls -la /ansible
```

Copy the missing file again from the VS Code Remote-SSH folder mounted in WSL:

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/<PLAYBOOK>.yml semaphore:/ansible/<PLAYBOOK>.yml
sudo docker exec semaphore ls -lh /ansible/<PLAYBOOK>.yml
```

If the copy reports `0B`, check `C:\Users\Administrator\ansible-lab` in the VS Code Remote-SSH window and confirm the file was saved to the VM before retrying.

## Updated VS Code YAML Does Not Appear in Semaphore

`C:\Users\Administrator\ansible-lab` on the Windows Server VM is the source of truth. WSL sees it at `/mnt/c/Users/Administrator/ansible-lab`, while container `/ansible` is the runtime copy used by Semaphore.

**Run after every save on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/<PLAYBOOK>.yml semaphore:/ansible/<PLAYBOOK>.yml
sudo docker exec semaphore ls -lh /ansible/<PLAYBOOK>.yml
```

## VS Code File Does Not Appear in WSL

Confirm that VS Code is in the Remote-SSH window for the correct Windows Server VM and that the file was saved.

**Run in: VS Code TERMINAL → WINDOWS SERVER VM — POWERSHELL**

```powershell
Get-ChildItem C:\Users\Administrator\ansible-lab
```

**Then run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
ls -lh /mnt/c/Users/Administrator/ansible-lab
```

Both commands must show the same saved filenames. If they do not, VS Code is open on a different machine/folder or the file has not been saved to the VM.

## Playbook Accidentally Targets Both Switches

Stop before running if a device-specific YAML says:

```yaml
hosts: cisco
```

Correct targets are:

```yaml
hosts: baba
```

for BABA files, and:

```yaml
hosts: taas
```

for TAAS files.

This prevents BABA DHCP/VLAN configuration from being sent to TAAS.

## Placeholder `~~` Was Not Replaced

**Check on: VM → WSL UBUNTU**

```bash
sudo docker exec semaphore grep -R -n -- '~~' /ansible
```

The reusable repository intentionally contains `~~`. The working `/ansible` copies must not contain it when a task is actually run.

## LACP Shows `D` or Does Not Bundle

Example:

```text
Po1(SD)
Fa0/10(D)
Fa0/11(D)
Fa0/12(D)
```

**Run on: BOTH CORE BABA and CORE TAAS → CISCO CLI**

```cisco
show interfaces status
show interfaces trunk
show etherchannel summary
show lacp neighbor
show interfaces port-channel 1 | include BW
```

Verify:

- all three physical cables connect the intended ports;
- Fa0/10-12 are up on both sides;
- both sides use Dot1Q trunk mode;
- both sides use `channel-group 1 mode active` and LACP;
- no incompatible prior EtherChannel configuration remains.

## Camera Reservation Playbook

If `001a.xxxx.yyyy` is still present, do not run the playbook.

**Check on: VM → WSL UBUNTU**

```bash
sudo docker exec semaphore grep -n '001a.xxxx.yyyy' /ansible/baba-camera-dhcp.yml
```

Replace both placeholders with the correct, distinct identifiers before deployment.

## Useful Checks by Location

**WINDOWS SERVER VM — POWERSHELL (ADMIN)**

```powershell
wsl -l -v
Get-Service sshd
Test-NetConnection localhost -Port 3000
Test-NetConnection 208.8.8.200 -Port 3000
netsh interface portproxy show all
```

**VM → WSL UBUNTU**

```bash
sudo docker ps
sudo docker ps -a
sudo docker logs semaphore
sudo docker exec semaphore ls -lh /ansible
```

**CISCO CLI**

```cisco
show ip ssh
show ip interface brief
show interfaces trunk
show etherchannel summary
show vlan brief
show ip dhcp binding
```
