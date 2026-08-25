# Existing Docker and Semaphore Setup Inside Ubuntu

This tutorial keeps the working Semaphore deployment exactly as it is.

> **Do not remove, recreate, migrate, or replace the `semaphore` container.** Do not run a new `docker run` command, change its database, or add new volumes just to follow this guide.

Semaphore remains inside Docker running in WSL Ubuntu. The tutorial only checks it, starts it if stopped, and copies playbooks into its existing `/ansible` directory.

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

<!-- SCREENSHOT: Existing semaphore container showing Up inside Ubuntu -->

## 3. Verify the Existing `/ansible` Directory

**Run on: VM → WSL UBUNTU**

```bash
sudo docker exec semaphore ls -la /ansible
```

If `/ansible` is missing, create only that directory; this does not recreate Semaphore:

```bash
sudo docker exec -u 0 semaphore mkdir -p /ansible
sudo docker exec semaphore ls -la /ansible
```

## 4. Keep the Ubuntu Working Folder

The source-of-truth playbooks must live inside Ubuntu:

**Run on: VM → WSL UBUNTU**

```bash
mkdir -p ~/ansible-lab
cd ~/ansible-lab
pwd
```

Expected path for the original lab user:

```text
/home/ubadmin/ansible-lab
```

`~/ansible-lab` also works if a different Ubuntu username is used.

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

## 6. Verify Existing Semaphore GUI Access

**Run on: WINDOWS SERVER VM → POWERSHELL**

```powershell
Test-NetConnection localhost -Port 3000
Test-NetConnection 208.8.8.~~ -Port 3000
netsh interface portproxy show all
```

**Open on: PHYSICAL PC → WEB BROWSER**

```text
http://208.8.8.~~:3000
```

If the existing GUI works, leave its container, database, users, projects, credentials, and port configuration unchanged.

## 7. What This Tutorial Changes

The remaining guides only:

1. create or edit YAML under `~/ansible-lab` in Ubuntu;
2. copy those files into the existing `semaphore:/ansible` directory;
3. add/update Semaphore inventory, repository, and task-template entries;
4. run the documented Cisco playbooks.

They do not rebuild Semaphore.
