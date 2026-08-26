# Troubleshooting

Every command below includes its run location. Do not paste PowerShell commands into Ubuntu or Cisco commands into Windows.

## How to Use This Page

1. Stop the configuration run. Do not keep pressing **Run** when the same task fails.
2. Save the complete first error and the final Ansible recap; the first error is usually more useful than the last line.
3. Identify where the failure occurs: physical PC, Windows Server VM, Ubuntu, Semaphore container, or Cisco switch.
4. Run only the read-only checks in the matching section below.
5. Change one thing at a time, then repeat the read-only `Show Version` task before any configuration task.

Never publish inventory passwords, private keys, full device configurations, or Semaphore credentials in screenshots, GitHub issues, or chat messages.

## Quick Triage

| Symptom | Start with |
|---|---|
| Ubuntu will not open | WSL2 virtualization section |
| `docker ps` fails | Docker daemon section |
| Semaphore page will not load | Semaphore container and port 3000 sections |
| Manual Cisco SSH fails | Cisco SSH and legacy algorithm sections |
| `syntax-check` fails | Missing file, empty file, placeholder, and YAML line named in the error |
| `Could not match supplied host pattern` | Semaphore's selected GUI inventory and the YAML `hosts:` value |
| Ansible reports `unreachable` | Inventory address, network reachability, credentials, and SSH |
| Ansible reports `failed` after connecting | First failed task and Cisco IOS message in the output |
| EtherChannel stays down | LACP section; inspect both switches |

Do not run a configuration playbook merely to test connectivity. Use the matching `show-version` playbook.

## Screenshot guide: Recording a failure for help

- **Capture:** the task name, target hostname, first failed task, complete first error, and final Ansible recap.
- **Also record in text:** the command location and the last checkpoint that passed; screenshots are not searchable enough by themselves.
- **Hide:** passwords, private keys, camera identifiers, tokens, cookies, and unrelated device configuration.
- **Do not hide:** the relevant filename, task name, hostname, management IP, error type, and recap values needed to diagnose the failure.
- **Status:** Screenshot pending.

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
$currentWslAddress = ((wsl -d Ubuntu hostname -I).Trim() -split '\s+')[0]
$currentWslAddress
netsh interface portproxy delete v4tov4 listenaddress=208.8.8.200 listenport=3000
netsh interface portproxy add v4tov4 listenaddress=208.8.8.200 listenport=3000 connectaddress=$currentWslAddress connectport=3000
netsh interface portproxy show all
```

The first address returned by `hostname -I` is the WSL address in this lab. Do not copy an old `172.x.x.x` example: WSL can receive a new address after a restart.

Check the firewall rule and create it only if it does not already exist:

```powershell
if (-not (Get-NetFirewallRule -DisplayName "Semaphore 3000" -ErrorAction SilentlyContinue)) {
    New-NetFirewallRule -DisplayName "Semaphore 3000" -Direction Inbound -Protocol TCP -LocalPort 3000 -Action Allow
}
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

For the working local `admin` account, both ranges must contain these lines:

```cisco
line vty 0 4
 login local
 transport input ssh

line vty 5 14
 login local
 transport input ssh
```

If the running configuration shows plain `login`, use the switch console—not the failing SSH session—to repair it:

```cisco
enable
configure terminal
line vty 0 4
 login local
 transport input ssh
 exit
line vty 5 14
 login local
 transport input ssh
 exit
end
write memory
show running-config | section line vty
```

This SSH prerequisite was confirmed in the live lab: BABA already used `login local`, while TAAS failed until both VTY ranges were changed from plain `login` to `login local`.

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

## `Could Not Match Supplied Host Pattern`

Example warning:

```text
Could not match supplied host pattern, ignoring: taas
```

This means the selected inventory does not contain the group named by the playbook. Ansible usually skips the play, so no switch configuration is applied. Do not solve it by changing a TAAS playbook from `hosts: taas` to `hosts: cisco`; that could target BABA too.

First inspect the playbook and CLI inventory copies.

**Run on: VS Code TERMINAL → VM → WSL UBUNTU**

```bash
sudo docker exec semaphore sed -n '1,40p' /ansible/taas-trunk.yml
sudo docker exec semaphore ansible-inventory -i /ansible/inventory.ini --graph
sudo docker exec semaphore grep -n -A2 '^\[taas\]' /ansible/inventory.ini
```

The playbook must say `hosts: taas`. The inventory graph must contain both `@baba` and `@taas`, with the correct host under each group.

Then inspect the inventory actually selected by the Semaphore template.

**Do in: PHYSICAL PC → WEB BROWSER → SEMAPHORE GUI**

1. Open the failed TAAS task template and confirm **Inventory** is `Cisco Inventory`.
2. Open **Inventory → Cisco Inventory**.
3. Confirm the inline content contains a `[taas]` section with `taas ansible_host=10.~~.1.2` after replacing `~~` with the real monitor number.
4. Confirm it also has a separate `[baba]` section; do not put both devices only under `[cisco]`.
5. Save the inventory, rerun `TAAS — Show Version`, and require `failed=0` and `unreachable=0` before retrying a configuration template.

The GUI inventory and `/ansible/inventory.ini` are separate in the current lab. A successful `ansible-inventory` terminal check does not prove that the Semaphore GUI entry is current.

### Screenshot guide: Corrected TAAS inventory target

- **Capture:** `Cisco Inventory` in Semaphore and the bottom of the successful `TAAS — Show Version` result.
- **Success must show:** a `[taas]` group containing the TAAS management address and a recap with `failed=0`, `unreachable=0`.
- **Hide:** credentials, tokens, passwords, and unrelated device entries.
- **Status:** Screenshot pending.

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

## Unsure Whether to Run the Base Playbook

Ask what was configured from the Cisco console:

- If only the management IP, local user, RSA keys, SSH, and VTY `login local` were configured, run Show Version and then the matching base playbook.
- If Sir Rob's complete BABA or TAAS base block was already applied with the correct monitor values, run Show Version, verify the configuration, and **skip the matching base playbook**.

The base playbook is an automated way to apply the complete base, not a mandatory second copy of a complete manual configuration. Although most tasks are idempotent, the playbook can change values that differ from its intended state.

If Semaphore still shows `Configure Cisco Interface`, inspect its playbook field. A template pointing to `interface.yml`, `baba.yml`, or another old test is outside the final documented workflow. Mark it **OLD — DO NOT RUN** or remove it through your normal change-control process; do not use it for a new TAAS or BABA switch.

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
