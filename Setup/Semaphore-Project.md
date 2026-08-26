# Semaphore Project Setup

Complete this after the YAML files exist in `/ansible` and the matching CLI inventory exists at `/ansible/inventory.ini`.

> Preserve the existing Semaphore project. If `Cisco Admin`, `Cisco Inventory`, `Cisco Playbooks`, or an existing task template is already present and working, verify or edit that entry instead of creating a duplicate. Do not recreate the Semaphore container, database, admin account, or project.

**All steps in this file are done in: PHYSICAL PC → WEB BROWSER → the existing Semaphore GUI → Cisco Automation project.**

## What the Four Semaphore Items Mean

| Semaphore item | Purpose in this lab |
|---|---|
| Key Store | Holds the Cisco login credential used by Ansible. |
| Inventory | Stores the BABA/TAAS device list used by Semaphore tasks. The current working lab uses inline content in `Cisco Inventory`. |
| Repository | Points to `/ansible`, where the playbook YAML files are stored. |
| Task Template | Combines one playbook with the inventory, repository, and credential so it can be run from the GUI. |

The names below are labels inside Semaphore. They do not create new folders or Cisco devices.

Before using the GUI, verify from WSL that the required YAML file exists inside the container. `/ansible/inventory.ini` remains the matching inventory for terminal syntax checks and manual Ansible tests. A Semaphore template cannot use YAML that was saved only in VS Code and never copied into the container.

## 1. Verify or Create the Cisco Login Credential

1. Open **Key Store**.
2. Reuse the working Cisco credential if it already exists; otherwise click **New Key**.
3. Choose a login/password credential.
4. Name it `Cisco Admin`.
5. Username: `admin`.
6. Password: the Cisco lab password (`pass` in the Day 1 lab).
7. Save.

`pass` is a lab credential retained from the Day 1 exercise. Do not reuse it on production equipment or expose a real production password in GitHub or `inventory.ini`.

### Screenshot guide: Cisco credential entry

- **Capture:** the Key Store list showing an entry named `Cisco Admin`.
- **Success must show:** the key name and credential type only.
- **Hide:** the password, secret fields, private keys, and browser password-manager prompts.
- **Status:** Screenshot pending.

## 2. Verify or Create the Inventory

1. Open **Inventory**.
2. Edit the working inventory if it already exists; otherwise click **New Inventory**.
3. Name: `Cisco Inventory`.
4. User credential: `Cisco Admin`.
5. Keep the current inventory type that displays an inline inventory editor. Depending on the Semaphore version, this can be labeled **Static**.
6. Replace the inline content with the complete inventory below.
7. Replace every `~~` with the assigned monitor number.
8. Save.

```ini
[baba]
baba ansible_host=10.~~.1.4

[taas]
taas ansible_host=10.~~.1.2

[cisco:children]
baba
taas

[cisco:vars]
ansible_connection=network_cli
ansible_network_os=ios
ansible_ssh_common_args='-o KexAlgorithms=+diffie-hellman-group14-sha1 -o HostKeyAlgorithms=+ssh-rsa -o Ciphers=+aes128-cbc -o MACs=+hmac-sha1'
```

Do not add `ansible_user` or `ansible_password` to this GUI content when `Cisco Admin` is selected; the Key Store supplies them.

The GUI inventory and `/ansible/inventory.ini` are separate copies in the current lab. A terminal command can succeed with the container file while a Semaphore task fails because the GUI inventory is stale. Keep the group names and addresses identical in both places.

If an already-working Semaphore installation uses a **file** inventory instead of inline content, preserve that working type and point it to `/ansible/inventory.ini`. Do not switch a working inventory type merely to match a screenshot.

The inventory must contain separate `baba` and `taas` groups. Never point BABA-only playbooks at the combined `cisco` group.

### Screenshot guide: Cisco Inventory

- **Capture:** the saved `Cisco Inventory` details page without exposing credentials.
- **Success must show:** separate `[baba]` and `[taas]` groups, both monitor-specific addresses, and the selected `Cisco Admin` credential.
- **Hide:** passwords and any unrelated production inventory entries.
- **Status:** Screenshot pending.

## 3. Verify or Create the Repository

1. Open **Repositories**.
2. Edit the working local repository if it already exists; otherwise click **New Repository**.
3. Name: `Cisco Playbooks`.
4. URL/path: `/ansible`.
5. Choose a `None` access key for the local folder.
6. Save.

Semaphore supports local filesystem repositories, so `/ansible` is used directly.

If Semaphore asks for a Git URL but does not accept the local path, stop and confirm that the same working Semaphore version and existing project are being used. Do not replace the local repository with an unrelated public Git repository just to make the form accept a value.

### Screenshot guide: Local playbook repository

- **Capture:** the saved `Cisco Playbooks` repository details.
- **Success must show:** the local path `/ansible` and the expected access-key selection.
- **Hide:** unrelated repository URLs, tokens, and private access keys.
- **Status:** Screenshot pending.

## 4. Verify or Create Task Templates

Reuse existing working templates. Add only the missing BABA/TAAS templates, using the same repository and inventory for every template.

The Base Layer 3 templates are used only when the switch received the minimum management/SSH bootstrap. If Sir Rob's complete base block was already applied manually, run the matching Show Version template, verify the switch, and skip the Base Layer 3 template. A visible Semaphore button is not an instruction that it must be run.

Do not create or use a `Configure Cisco Interface` template from an old `interface.yml` test. It is not part of the final BABA/TAAS template set below.

| Template name | Playbook | Safe to run? |
|---|---|---|
| BABA — Show Version | `show-version-baba.yml` | Yes; read-only |
| BABA — Base Layer 3 | `baba-base.yml` | Configuration |
| BABA — Trunk and LACP | `baba-lacp.yml` | Configuration |
| BABA — DHCP | `baba-dhcp.yml` | Configuration |
| BABA — VLANs and Ports | `baba-vlans.yml` | Configuration |
| BABA — Camera Reservations | `baba-camera-dhcp.yml` | **Do not run yet** |
| TAAS — Show Version | `show-version-taas.yml` | Yes; read-only |
| TAAS — Base Layer 3 | `taas-base.yml` | Configuration |
| TAAS — Trunk Ports | `taas-trunk.yml` | Configuration |
| TAAS — LACP EtherChannel | `taas-lacp.yml` | Configuration |

For each template select:

```text
Repository: Cisco Playbooks
Inventory:  Cisco Inventory
```

For a first template, create only `BABA — Show Version`:

1. Open **Task Templates** or **Templates**.
2. Select **New Template**.
3. Enter the template name `BABA — Show Version`.
4. Select repository `Cisco Playbooks`.
5. Enter playbook filename `show-version-baba.yml`.
6. Select inventory `Cisco Inventory`.
7. Leave unrelated advanced fields unchanged.
8. Save the template.
9. Reopen it and verify the filename before running it.

Repeat the process only after the first read-only task succeeds. A template name is descriptive text; the **Playbook Filename** field is what decides which YAML actually runs.

Before clicking Run, open the YAML and confirm its target:

```text
BABA files → hosts: baba
TAAS files → hosts: taas
```

### Screenshot guide: Separate BABA and TAAS templates

- **Capture:** the Task Templates list with the BABA and TAAS names visible.
- **Success must show:** separate Show Version, base, trunk/LACP, and BABA-only task names; the camera task must be clearly marked **DO NOT RUN**.
- **Hide:** credentials and unrelated project names.
- **Status:** Screenshot pending.

## 5. First Test

Run these two read-only templates first:

1. `BABA — Show Version`
2. `TAAS — Show Version`

Both must finish with `failed=0` and `unreachable=0` before any configuration template is run.

If either value is non-zero, open the task output, copy the complete first error, and use `Troubleshooting.md`. Repeatedly clicking Run will not correct inventory, SSH, or file-path errors.

### Screenshot guide: Successful read-only task

- **Capture:** the end of a Show Version task in Semaphore.
- **Success must show:** the task name, target hostname, Cisco version output, and recap with `failed=0` and `unreachable=0`.
- **Hide:** credentials, session tokens, and unrelated device configuration.
- **Status:** Screenshot pending.

## Passing Checkpoint

```text
[ ] Cisco Admin exists once
[ ] Cisco Inventory contains separate BABA and TAAS groups
[ ] GUI inventory addresses match /ansible/inventory.ini
[ ] Cisco Playbooks points to /ansible
[ ] BABA and TAAS Show Version templates use the correct filenames
[ ] Both Show Version jobs finish with failed=0 and unreachable=0
```
