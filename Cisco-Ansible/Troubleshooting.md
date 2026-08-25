# Troubleshooting

This file contains environment-wide issues encountered while building the Cisco Ansible/Semaphore lab.

## WSL2: `HCS_E_HYPERV_NOT_INSTALLED`

Cause: nested virtualization was not exposed to the Windows Server VMware VM.

Fix:

```text
VMware Workstation
→ VM Settings
→ Processors
→ Enable "Virtualize Intel VT-x/EPT or AMD-V/RVI"
```

Then:

```powershell
wsl -l -v
wsl -d Ubuntu
```

<!-- SCREENSHOT: WSL virtualization error -->

## Docker: Cannot Connect to Daemon

Error:

```text
Cannot connect to the Docker daemon at unix:///var/run/docker.sock
```

Check:

```bash
which docker
docker context ls
```

Working fix:

```bash
sudo dockerd > /tmp/dockerd.log 2>&1 &
sudo docker ps
```

Logs:

```bash
cat /tmp/dockerd.log
```

## Docker: `docker: unrecognized service`

The WSL environment did not expose Docker as a traditional init service.

Use:

```bash
sudo dockerd > /tmp/dockerd.log 2>&1 &
```

instead of repeatedly reinstalling Docker.

## PC Can SSH to VM but Cannot Ping It

SSH and ICMP are separate.

Allow ICMP on Windows Server:

```powershell
New-NetFirewallRule -DisplayName "Allow ICMPv4 Ping" -Direction Inbound -Protocol ICMPv4 -IcmpType 8 -Action Allow
```

## Semaphore GUI Works on localhost but Not VM IP

Check:

```powershell
Test-NetConnection localhost -Port 3000
Test-NetConnection 208.8.8.~~ -Port 3000
```

Get WSL IP:

```powershell
wsl -d Ubuntu hostname -I
```

Forward:

```powershell
netsh interface portproxy add v4tov4 listenaddress=208.8.8.~~ listenport=3000 connectaddress=<WSL-IP> connectport=3000
```

Verify:

```powershell
netsh interface portproxy show all
```

> WSL IP addresses can change after restart.

## Cisco SSH: No Acceptable KEX Algorithm

Error:

```text
Incompatible ssh peer (no acceptable kex algorithm)
```

Inventory compatibility options:

```ini
ansible_ssh_common_args='-o KexAlgorithms=+diffie-hellman-group14-sha1 -o HostKeyAlgorithms=+ssh-rsa -o Ciphers=+aes128-cbc -o MACs=+hmac-sha1'
```

Manual SSH test:

```bash
ssh \
-o KexAlgorithms=+diffie-hellman-group14-sha1 \
-o HostKeyAlgorithms=+ssh-rsa \
-o Ciphers=+aes128-cbc \
-o MACs=+hmac-sha1 \
admin@10.~~.1.4
```

## Paramiko Compatibility

Check:

```bash
python3 -c "import paramiko; print(paramiko.__version__)"
```

The working lab used Paramiko:

```text
3.5.1
```

The environment initially used Paramiko 4.0.0.

The working fix in the Semaphore Ansible virtual environment was:

```bash
/opt/semaphore/apps/ansible/13.5.0/venv/bin/pip install "paramiko<4"
```

Verify:

```bash
/opt/semaphore/apps/ansible/13.5.0/venv/bin/python -c "import paramiko; print(paramiko.__version__)"
```

Restart Semaphore:

```bash
sudo docker restart semaphore
```

<!-- SCREENSHOT: Paramiko version -->
<!-- SCREENSHOT: Successful task after Paramiko fix -->

## Semaphore `/ansible` Missing

```bash
sudo docker exec -u 0 semaphore mkdir -p /ansible
sudo docker cp "$HOME/ansible-lab/." semaphore:/ansible/
sudo docker exec semaphore ls -la /ansible
```

## Updated VS Code File Does Not Appear in Semaphore

The Windows folder and `/ansible` container directory are separate copies.

After every edit:

```bash
sudo docker cp /mnt/c/Users/Administrator/ansible-lab/<PLAYBOOK>.yml semaphore:/ansible/<PLAYBOOK>.yml
```

Verify:

```bash
sudo docker exec semaphore ls -la /ansible
```

## LACP Shows `D`

Example:

```text
Po1(SD)
Fa0/10(D)
Fa0/11(D)
Fa0/12(D)
```

`D` indicates down.

Check:

```cisco
show interfaces status
show interfaces trunk
show etherchannel summary
show lacp neighbor
```

Verify physical links and matching LACP configuration on the neighboring switch.

## Useful Environment Checks

Windows Server:

```powershell
wsl -l -v
Get-Service sshd
Test-NetConnection localhost -Port 3000
Test-NetConnection 208.8.8.~~ -Port 3000
netsh interface portproxy show all
```

Ubuntu/Docker:

```bash
sudo docker ps
sudo docker ps -a
sudo docker exec semaphore ls -la /ansible
```

Cisco:

```cisco
show ip ssh
show ip interface brief
show interfaces trunk
show etherchannel summary
show vlan brief
```
