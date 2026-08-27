# Semaphore Project Setup

Complete this after the YAML files exist in `/ansible` and the matching CLI inventory exists at `/ansible/inventory.ini`.

> Preserve the existing Semaphore project. If `Cisco Admin`, `Cisco Inventory`, `Cisco Playbooks`, or an existing task template is already present and working, verify or edit that entry instead of creating a duplicate. Do not recreate the Semaphore container, database, admin account, or project.

**All steps in this file are done in: PHYSICAL PC → WEB BROWSER → the existing Semaphore GUI → Cisco Automation project.**

## What the Four Semaphore Items Mean

| Semaphore item | Purpose in this lab |
|---|---|
| Key Store | Holds the Cisco login credential used by Ansible. |
| Inventory | Stores the BABA/TAAS/CUCM device list used by Semaphore tasks. The current working lab uses inline content in `Cisco Inventory`. |
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
- **Success must show:** the Cisco login key name and credential type only. The existing lab screenshot names this working entry `Cisco SSH Credentials`; keep a working existing name instead of creating a duplicate.
- **Hide:** the password, secret fields, private keys, and browser password-manager prompts.
- **Status:** Included below.

The Key Store list confirms that the Cisco login credential and the local repository key exist without revealing their secret values.

<img width="1918" height="918" alt="Semaphore Key Store listing Cisco SSH Credentials and the Local Repository Key without exposing passwords" src="https://github.com/user-attachments/assets/3b7e876f-d3e7-4dc2-9634-158927bb7e48" />

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

[cucm]
cucm ansible_host=10.~~.100.8

[cisco:children]
baba
taas
cucm

[cisco:vars]
ansible_connection=network_cli
ansible_network_os=ios
ansible_ssh_common_args='-o KexAlgorithms=+diffie-hellman-group14-sha1 -o HostKeyAlgorithms=+ssh-rsa -o Ciphers=+aes128-cbc -o MACs=+hmac-sha1'
```

Do not add `ansible_user` or `ansible_password` to this GUI content when `Cisco Admin` is selected; the Key Store supplies them.

The GUI inventory and `/ansible/inventory.ini` are separate copies in the current lab. A terminal command can succeed with the container file while a Semaphore task fails because the GUI inventory is stale. Keep the group names and addresses identical in both places.

If an already-working Semaphore installation uses a **file** inventory instead of inline content, preserve that working type and point it to `/ansible/inventory.ini`. Do not switch a working inventory type merely to match a screenshot.

The inventory must contain separate `baba`, `taas`, and `cucm` groups. Never point device-specific playbooks at the combined `cisco` group.

### Screenshot guide: Cisco Inventory

- **Capture:** the saved `Cisco Inventory` details page without exposing credentials.
- **Success must show:** separate `[baba]`, `[taas]`, and `[cucm]` groups, their monitor-specific addresses, and the selected `Cisco Admin` credential.
- **Hide:** passwords and any unrelated production inventory entries.
- **Status:** Included below.

The historical screenshots below predate the CUCM addition. The current inventory must also contain `[cucm]`, `cucm ansible_host=10.~~.100.8`, and `cucm` under `[cisco:children]`.

<img width="1917" height="961" alt="Semaphore Cisco Inventory editor showing the selected Cisco credential and separate BABA and TAAS entries" src="https://github.com/user-attachments/assets/cf38915d-80c9-49f4-9d27-8952d477e716" />

<img width="1920" height="984" alt="Expanded Semaphore inventory showing separate baba and taas groups plus the cisco children group and network CLI variables" src="https://github.com/user-attachments/assets/96e1d753-4353-4767-9dd0-11e9861a2296" />

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
- **Status:** Included below.

The repository list and edit form both confirm that Semaphore reads playbooks from the local `/ansible` directory.

<img width="1917" height="913" alt="Semaphore repository list showing Cisco Playbooks at the local path slash ansible" src="https://github.com/user-attachments/assets/2069f7ec-6c22-4e42-be88-a071fee0d6c0" />

<img width="1920" height="914" alt="Semaphore Edit Repository form showing Cisco Playbooks, local path slash ansible, and Local Repository Key" src="https://github.com/user-attachments/assets/ba621a86-abcb-4a53-835d-ff12d2225976" />

## 4. Verify or Create Task Templates

Reuse existing working templates. Add only the missing BABA/TAAS/CUCM templates, using the same repository and inventory for every template.

The Base Layer 3 templates are used only when the switch received the minimum management/SSH bootstrap. If Sir Rob's complete base block was already applied manually, run the matching Show Version template, verify the switch, and skip the Base Layer 3 template. A visible Semaphore button is not an instruction that it must be run.

Do not create or use a `Configure Cisco Interface` template from an old `interface.yml` test. It is not part of the final BABA/TAAS/CUCM template set below.

| Template name | Playbook | Safe to run? |
|---|---|---|
| BABA — Show Version | `show-version-baba.yml` | Yes; read-only |
| BABA — Base Layer 3 | `baba-base.yml` | Configuration |
| BABA — OSPF | `baba-ospf.yml` | Configuration; required before CUCM remote access |
| BABA — Trunk and LACP | `baba-lacp.yml` | Configuration |
| BABA — DHCP | `baba-dhcp.yml` | Configuration |
| BABA — VLANs and Ports | `baba-vlans.yml` | Configuration |
| BABA — Camera Reservations | `baba-camera-dhcp.yml` | **Do not run yet** |
| TAAS — Show Version | `show-version-taas.yml` | Yes; read-only |
| TAAS — Base Layer 3 | `taas-base.yml` | Configuration |
| TAAS — Trunk Ports | `taas-trunk.yml` | Configuration |
| TAAS — LACP EtherChannel | `taas-lacp.yml` | Configuration |
| CUCM — Show Version and Routing | `show-version-cucm.yml` | Yes; read-only; run first |
| CUCM — Base and Default Route | `cucm-base.yml` | Configuration; normally skip after full console bootstrap |
| CUCM — OSPF | `cucm-ospf.yml` | Configuration; initial OSPF must be console-bootstrapped first |
| CUCM — Analog Phones | `cucm-analog-phones.yml` | Configuration; review ports and dial plan |
| CUCM — Telephony Service Rebuild | `cucm-telephony-service.yml` | **Blocked/destructive** |
| CUCM — Video | `cucm-video.yml` | Configuration; requires ephones 1 and 2 |
| CUCM — Day 1 Inter-CUCM Calls | `cucm-inter-cucm-voip.yml` | Configuration; preserves the unrestricted Day 1 trusted list and every active source dial peer |
| CUCM — Check Ephone Status | `cucm-ephone-status.yml` | Yes; read-only phone-registration and number check |
| CUCM — Restart Ephones | `cucm-restart-ephones.yml` | Recreates Day 1 phone files and fast-restarts ephones 1 and 2 |
| CUCM — Discover and Assign Ephones | `cucm-auto-discover-ephones.yml` | Automatically maps BABA Fa0/5 to ephone 1 and Fa0/7 to ephone 2 through CDP |

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
CUCM files → hosts: cucm
```

Exception: `cucm-telephony-service.yml` and `cucm-auto-discover-ephones.yml` intentionally start with `hosts: baba` to read the two phone MAC addresses from BABA CDP, then use `hosts: cucm` for the actual CUCM configuration.

### Screenshot guide: Separate BABA, TAAS, and CUCM templates

- **Capture:** the Task Templates list with the BABA, TAAS, and CUCM names visible.
- **Success must show:** separate device task names with their correct playbook, inventory, and repository columns. Camera and CUCM telephony-reset tasks must be visibly blocked or marked unsafe.
- **Hide:** credentials and unrelated project names.
- **Status:** Included below.

This working Task Templates list shows separate BABA and TAAS configuration tasks plus read-only show tasks. Older filenames visible in this historical screenshot may differ from the corrected repository; always use the filename table immediately above.

<img width="1920" height="915" alt="Semaphore Task Templates list showing separate BABA and TAAS tasks with successful status" src="https://github.com/user-attachments/assets/89d258b2-95fe-4913-bea6-d6297005877b" />

## 5. First Test

Run the two switch read-only templates first:

1. `BABA — Show Version`
2. `TAAS — Show Version`

Both must finish with `failed=0` and `unreachable=0` before any switch configuration template is run. After the CUCM console and OSPF checkpoints in `CUCM/README.md` pass, also run `CUCM — Show Version and Routing`; it must finish with the same zero-failure recap before any voice configuration.

If either value is non-zero, open the task output, copy the complete first error, and use `Troubleshooting.md`. Repeatedly clicking Run will not correct inventory, SSH, or file-path errors.

### Screenshot guide: Successful read-only task

- **Capture:** the output and end recap of a read-only show task in Semaphore.
- **Success must show:** the task name, target hostnames, returned Cisco data, and a recap with `failed=0` and `unreachable=0`.
- **Hide:** credentials, session tokens, and unrelated device configuration.
- **Status:** Included below.

This example uses the read-only **Show IP Interface Brief** task. For a new deployment, still run the BABA and TAAS **Show Version** templates first as instructed above. The first image shows returned interface data from both hosts; the second shows the final recap with no failures or unreachable devices.

<img width="996" height="727" alt="Successful Semaphore Show IP Interface Brief task displaying read-only Cisco interface data for BABA" src="https://github.com/user-attachments/assets/1e6d9b91-7579-4c25-8cb4-8aeba3fe8cab" />

<img width="991" height="600" alt="Semaphore read-only task recap showing BABA and TAAS with failed zero and unreachable zero" src="https://github.com/user-attachments/assets/12f841d3-3250-4431-8bc1-09b214d6a61b" />

## Passing Checkpoint

```text
[ ] Cisco Admin exists once
[ ] Cisco Inventory contains separate BABA, TAAS, and CUCM groups
[ ] GUI inventory addresses match /ansible/inventory.ini
[ ] Cisco Playbooks points to /ansible
[ ] BABA, TAAS, and CUCM read-only templates use the correct filenames
[ ] All applicable read-only jobs finish with failed=0 and unreachable=0
```
