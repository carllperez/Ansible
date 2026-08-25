# Docker and Semaphore Setup

## 1. Install Docker

Inside Ubuntu:

```bash
sudo apt update
sudo apt install docker.io -y
```

Verify:

```bash
docker --version
which docker
```

Expected Docker path:

```text
/usr/bin/docker
```

Try:

```bash
sudo service docker start
sudo docker ps
```

## 2. Docker Daemon in WSL

In this lab:

```bash
sudo service docker start
```

returned:

```text
docker: unrecognized service
```

The working method was:

```bash
sudo dockerd > /tmp/dockerd.log 2>&1 &
```

Wait a few seconds:

```bash
sudo docker ps
```

If it fails:

```bash
cat /tmp/dockerd.log
```

<!-- SCREENSHOT: Docker daemon running -->

> `dockerd` may need to be started again after restarting WSL or the Windows Server VM.

## 3. Install Ansible

```bash
sudo apt install ansible -y
```

Verify:

```bash
ansible --version
```

## 4. Create the Ansible Workspace

```bash
mkdir -p ~/ansible-lab
cd ~/ansible-lab
```

Verify:

```bash
pwd
```

## 5. Pull Semaphore

```bash
sudo docker pull semaphoreui/semaphore:latest
```

Verify:

```bash
sudo docker images
```

<!-- SCREENSHOT: Semaphore image -->

## 6. Semaphore Container

For a basic fresh deployment:

```bash
sudo docker run -d \
  --name semaphore \
  -p 3000:3000 \
  semaphoreui/semaphore:latest
```

Verify:

```bash
sudo docker ps
```

If an existing container is stopped:

```bash
sudo docker start semaphore
```

Check all containers:

```bash
sudo docker ps -a
```

Logs:

```bash
sudo docker logs semaphore
```

<!-- SCREENSHOT: Semaphore container running -->

> If an existing Semaphore deployment uses environment variables, volumes, or database containers, preserve that deployment instead of replacing it with the simple example above.

## 7. Create `/ansible`

```bash
sudo docker exec -u 0 semaphore mkdir -p /ansible
```

Copy the current workspace:

```bash
sudo docker cp "$HOME/ansible-lab/." semaphore:/ansible/
```

Verify:

```bash
sudo docker exec semaphore ls -la /ansible
```

<!-- SCREENSHOT: /ansible directory -->

## 8. Expose Semaphore to the Physical PC

On Windows Server PowerShell:

```powershell
Test-NetConnection localhost -Port 3000
Test-NetConnection 208.8.8.~~ -Port 3000
```

Allow port 3000:

```powershell
New-NetFirewallRule -DisplayName "Semaphore 3000" -Direction Inbound -Protocol TCP -LocalPort 3000 -Action Allow
```

Get the current WSL IP:

```powershell
wsl -d Ubuntu hostname -I
```

Use the WSL address, not the Docker bridge address.

Create the port proxy:

```powershell
netsh interface portproxy add v4tov4 listenaddress=208.8.8.~~ listenport=3000 connectaddress=<WSL-IP> connectport=3000
```

Verify:

```powershell
netsh interface portproxy show all
```

<!-- SCREENSHOT: portproxy configuration -->
<!-- SCREENSHOT: Semaphore GUI from physical PC -->

> The WSL IP can change after a restart. Update the port proxy if the GUI stops loading.
