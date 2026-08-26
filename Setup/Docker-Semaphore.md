# Existing Docker and Semaphore Setup Inside Ubuntu

This tutorial keeps the working Semaphore deployment exactly as it is.

> **Do not remove, recreate, migrate, or replace the `semaphore` container.** Do not run a new `docker run` command, change its database, or add new volumes just to follow this guide.

Semaphore remains inside Docker running in WSL Ubuntu. The tutorial only checks it, starts it if stopped, and copies playbooks into its existing `/ansible` directory.

## What You Are Checking

- **Docker** runs background environments called containers.
- **`semaphore`** is the name of the existing Semaphore container.
- **`/ansible`** is the playbook folder inside that container.
- **Port 3000** is the web connection used to open Semaphore.

This page is successful when the container is running, `/ansible` is visible, and the physical PC can open `http://208.8.8.200:3000`.

> If no `semaphore` container exists at all, stop. This guide preserves and verifies the original deployment; it does not create a different database or replacement container.

## Command Location Labels

- **VM → WSL UBUNTU** — run after entering Ubuntu with `wsl -d Ubuntu`.
- **EXISTING SEMAPHORE CONTAINER** — accessed through `sudo docker exec semaphore ...`.
- **PHYSICAL PC → WEB BROWSER** — used to open the existing Semaphore GUI.

## 1. Enter Ubuntu

**Run on: WINDOWS SERVER VM → POWERSHELL, or the VS Code terminal connected to the VM**

```powershell
wsl -d Ubuntu
```

The prompt should change to the Ubuntu user, such as:

```text
ubadmin@WIN-...:~$
```

## 2. Verify the Existing Docker Installation

**Run on: VM → WSL UBUNTU**

```bash
docker --version
which docker
sudo docker ps
```

The expected container name is `semaphore`.

Example of the important part of a passing result:

```text
NAMES       STATUS
semaphore   Up ...
```

`docker ps` lists only running containers. If `semaphore` is not shown, check stopped containers before concluding that it is missing.

If it exists but is stopped:

```bash
sudo docker ps -a
sudo docker start semaphore
sudo docker ps
```

If Docker itself is stopped and `sudo service docker start` works:

```bash
sudo service docker start
sudo docker ps
```

If WSL returns `docker: unrecognized service`, use the same working lab method:

```bash
sudo dockerd > /tmp/dockerd.log 2>&1 &
sudo docker ps
```

If it still fails:

```bash
cat /tmp/dockerd.log
```

### Screenshot guide: Existing Semaphore container is running

- **Capture:** the Ubuntu terminal immediately after `sudo docker ps`.
- **Success must show:** container name `semaphore`, an `Up` status, and the published port if shown.
- **Hide:** unrelated container names, environment values, registry credentials, or tokens.
- **Status:** Screenshot pending.

## 3. Verify the Existing `/ansible` Directory

**Run on: VM → WSL UBUNTU**

```bash
sudo docker exec semaphore ls -la /ansible
```

If `/ansible` is missing, stop. Do not create it blindly. First confirm that this is the same container and Docker environment used by the working Semaphore installation:

```bash
sudo docker inspect semaphore --format '{{.Name}} {{.Config.Image}}'
sudo docker exec semaphore pwd
```

A missing `/ansible` directory can mean the wrong Docker daemon or container is running. Resolve that before copying files or changing the container.

## 4. Verify the VS Code Folder from Ubuntu

VS Code Remote-SSH saves the source-of-truth files on the Windows Server VM at `C:\Users\Administrator\ansible-lab`. WSL accesses that same folder through `/mnt/c`.

**Run on: VM → WSL UBUNTU**

```bash
ls -la /mnt/c/Users/Administrator/ansible-lab
```

Expected mounted path:

```text
/mnt/c/Users/Administrator/ansible-lab
```

Do not create a second working copy under the Ubuntu home directory. Docker remains inside Ubuntu; only the YAML source is stored in the VS Code folder on the VM.

## 5. Verify Ansible and Paramiko Without Changing Semaphore

**Run on: VM → WSL UBUNTU**

```bash
ansible --version
python3 -c "import paramiko; print(paramiko.__version__)"
```

Check the existing container environment:

```bash
sudo docker exec semaphore ansible --version
sudo docker exec semaphore python3 -c "import paramiko; print(paramiko.__version__)"
```

Do not upgrade or downgrade the working container unless the existing tasks actually fail and the troubleshooting section specifically matches the error.

## 6. Verify or Restore Existing Semaphore GUI Access

**Run on: WINDOWS SERVER VM → POWERSHELL**

```powershell
Test-NetConnection localhost -Port 3000
Test-NetConnection 208.8.8.200 -Port 3000
netsh interface portproxy show all
```

If both connection tests succeed and the forwarding table already points to the current WSL address, do not change anything.

If `localhost:3000` succeeds but `208.8.8.200:3000` fails, get the current WSL address and store the first result:

```powershell
$currentWslAddress = ((wsl -d Ubuntu hostname -I).Trim() -split '\s+')[0]
$currentWslAddress
```

The value should look like `172.x.x.x`. WSL can receive a different address after WSL or the VM restarts, so never copy a previously documented address. In this lab, later addresses in the complete `hostname -I` output can include Docker's bridge address; the command above deliberately selects the first WSL address.

**Run on: WINDOWS SERVER VM → POWERSHELL (ADMIN)**

```powershell
netsh interface portproxy delete v4tov4 listenaddress=208.8.8.200 listenport=3000
netsh interface portproxy add v4tov4 listenaddress=208.8.8.200 listenport=3000 connectaddress=$currentWslAddress connectport=3000
netsh interface portproxy show all
```

Keep the existing firewall rule. The following block creates it only when it is missing:

```powershell
if (-not (Get-NetFirewallRule -DisplayName "Semaphore 3000" -ErrorAction SilentlyContinue)) {
    New-NetFirewallRule -DisplayName "Semaphore 3000" -Direction Inbound -Protocol TCP -LocalPort 3000 -Action Allow
}
```

Retest:

```powershell
Test-NetConnection 208.8.8.200 -Port 3000
```

**Open on: PHYSICAL PC → WEB BROWSER**

```text
http://208.8.8.200:3000
```

If the existing GUI works, leave its container, database, users, projects, credentials, and port configuration unchanged.

The WSL address can change after WSL or the VM restarts. If `localhost:3000` still works but the VM address stops working, repeat only the WSL-address and port-proxy checks above.

### Screenshot guide: Semaphore opens from the physical PC

- **Capture:** the browser with `http://208.8.8.200:3000` visible and the Semaphore project page open.
- **Success must show:** the expected Semaphore page loads through the VM address.
- **Hide:** passwords, saved-login prompts, tokens, and any task output containing credentials.
- **Status:** Screenshot pending.

## Passing Checkpoint

Do not continue to the VS Code or Semaphore project guides until all four checks pass:

```text
[ ] Ubuntu opens with wsl -d Ubuntu
[ ] sudo docker ps shows semaphore as Up
[ ] sudo docker exec semaphore ls -la /ansible succeeds
[ ] http://208.8.8.200:3000 opens from the physical PC
```

## 7. What This Tutorial Changes

The remaining guides only:

1. create or edit YAML in VS Code under `C:\Users\Administrator\ansible-lab` on the VM;
2. access the same files from WSL at `/mnt/c/Users/Administrator/ansible-lab`;
3. copy those files into the existing `semaphore:/ansible` directory;
4. add/update Semaphore inventory, repository, and task-template entries;
5. run the documented Cisco playbooks.

They do not rebuild Semaphore.
