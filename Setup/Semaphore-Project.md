# Semaphore Project Setup

Complete this after the YAML files and `inventory.ini` exist in `/ansible`.

> Preserve the existing Semaphore project. If `Cisco Admin`, `Cisco Inventory`, `Cisco Playbooks`, or an existing task template is already present and working, verify or edit that entry instead of creating a duplicate. Do not recreate the Semaphore container, database, admin account, or project.

**All steps in this file are done in: PHYSICAL PC → WEB BROWSER → the existing Semaphore GUI → Cisco Automation project.**

## 1. Verify or Create the Cisco Login Credential

1. Open **Key Store**.
2. Reuse the working Cisco credential if it already exists; otherwise click **New Key**.
3. Choose a login/password credential.
4. Name it `Cisco Admin`.
5. Username: `admin`.
6. Password: the Cisco lab password (`pass` in the Day 1 lab).
7. Save.

<!-- SCREENSHOT: Semaphore Key Store entry named Cisco Admin -->

## 2. Verify or Create the Inventory

1. Open **Inventory**.
2. Edit the working inventory if it already exists; otherwise click **New Inventory**.
3. Name: `Cisco Inventory`.
4. User credential: `Cisco Admin`.
5. Type: file.
6. Inventory path: `/ansible/inventory.ini`.
7. Save.

The inventory must contain separate `baba` and `taas` groups. Never point BABA-only playbooks at the combined `cisco` group.

<!-- SCREENSHOT: Cisco Inventory using /ansible/inventory.ini -->

## 3. Verify or Create the Repository

1. Open **Repositories**.
2. Edit the working local repository if it already exists; otherwise click **New Repository**.
3. Name: `Cisco Playbooks`.
4. URL/path: `/ansible`.
5. Choose a `None` access key for the local folder.
6. Save.

Semaphore supports local filesystem repositories, so `/ansible` is used directly.

<!-- SCREENSHOT: Local repository named Cisco Playbooks with /ansible path -->

## 4. Verify or Create Task Templates

Reuse existing working templates. Add only the missing BABA/TAAS templates, using the same repository and inventory for every template.

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

Before clicking Run, open the YAML and confirm its target:

```text
BABA files → hosts: baba
TAAS files → hosts: taas
```

<!-- SCREENSHOT: Semaphore Task Templates showing separate BABA and TAAS buttons -->

## 5. First Test

Run these two read-only templates first:

1. `BABA — Show Version`
2. `TAAS — Show Version`

Both must finish with `failed=0` and `unreachable=0` before any configuration template is run.
