# Start Here — Beginner Orientation

Use this page before running any command. It explains the lab in plain language and shows how the remaining guides fit together.

## What This Project Does

This project lets a person click a task in the Semaphore web page and have Ansible send approved Cisco configuration commands to CORE BABA or CORE TAAS over SSH.

```text
You click a Semaphore task
        ↓
Semaphore runs an Ansible YAML playbook
        ↓
Ansible reads the selected switch from Cisco Inventory in Semaphore
        ↓
Ansible connects to the switch over SSH
        ↓
Cisco IOS applies or displays the requested configuration
```

The first console connection is still required. A factory-default switch does not yet have the management address and SSH service that Ansible needs.

## Important Words

| Word | Plain-language meaning |
|---|---|
| Physical PC | The computer in front of you. VS Code and the web browser run here. |
| Windows Server VM | The virtual Windows Server machine at `208.8.8.200`. |
| WSL2 | A Linux environment running inside Windows Server. |
| Ubuntu | The Linux distribution used inside WSL2. |
| Docker | The program that runs Semaphore in an isolated container. |
| Container | The running Semaphore environment. Its name in this lab is `semaphore`. |
| Ansible | The automation program that connects to Cisco IOS and runs playbooks. |
| Inventory | The device list that tells Ansible which switches exist and how to reach them. The working Semaphore project stores it in the `Cisco Inventory` GUI entry; `/ansible/inventory.ini` is the matching CLI copy. |
| Playbook | A YAML file containing an ordered set of Ansible tasks. |
| YAML | The indentation-sensitive text format used by Ansible playbooks. |
| Semaphore | The web interface used to organize and run Ansible playbooks. |
| Repository | In Semaphore, the folder containing the playbooks. This lab uses `/ansible`. |
| Task template | A saved Semaphore button that joins a playbook, inventory, and credential. |
| Management IP | The address Ansible uses to reach a switch over SSH. |
| SVI | A Cisco VLAN interface, such as `interface Vlan1`, that can hold an IP address. |
| LACP | The protocol used to combine several physical links into one EtherChannel. |

## Five Different Places Commands Are Entered

Do not paste every command into the same window. Each code block has a location label.

| Documentation label | Where to type |
|---|---|
| **PHYSICAL PC → POWERSHELL** | PowerShell on the physical computer. |
| **WINDOWS SERVER VM → POWERSHELL (ADMIN)** | Administrator PowerShell inside the Windows Server VM. |
| **VS Code TERMINAL → VM → WSL UBUNTU** | VS Code terminal after entering `wsl -d Ubuntu`. |
| **EXISTING SEMAPHORE CONTAINER** | A command executed through `sudo docker exec semaphore ...`. |
| **CORE BABA/TAAS → CISCO CLI** | The Cisco console or an SSH session connected to that switch. |

Never type the visible prompt itself. For example, if the guide shows:

```text
PS C:\Users\Administrator>
```

type only the command after `>`.

Run one command block at a time. Read its result before continuing.

## How to Copy a YAML File into VS Code

GitHub holds the tutorial copy. VS Code holds the working copy that will eventually be sent to Semaphore.

1. In GitHub, open the required `.yml` file.
2. Use the file’s **Copy raw file** button, or open **Raw** and copy all of the text.
3. In the VS Code Remote-SSH window, open `C:\Users\Administrator\ansible-lab`.
4. Create the exact working filename listed in the BABA or TAAS tutorial.
5. Paste the complete YAML beginning with `---`.
6. Do not paste the Markdown lines that contain three backticks.
7. Keep the existing spaces. YAML indentation is part of the file’s meaning; do not replace spaces with tabs.
8. Replace `~~` only in an original Day 1 working copy. Do not replace `{{ monitor }}` in a reusable file.
9. Save with **Ctrl+S**.
10. Run the placeholder check and syntax check from the matching tutorial before copying the file into Semaphore.

Editing the GitHub page, saving in VS Code, and copying into the Semaphore container are three different actions. A change is not used by Semaphore until the saved VM file is copied to `semaphore:/ansible`.

Inventory is the exception in the current working lab: Semaphore tasks read the inline `Cisco Inventory` content saved in the GUI, while terminal syntax and manual tests read `/ansible/inventory.ini`. Keep those two inventories identical. Changing only one can make the GUI behave differently from the terminal.

## Understand the Two File Sets

### Original Day 1 files

The files under `CORE-BABA/` and `CORE-TAAS/` contain `~~` as a documentation placeholder. Copy them into the VS Code working folder and replace every `~~` with the assigned monitor number.

Example for monitor 71:

```text
COREbaba-~~  becomes COREbaba-71
10.~~.1.4   becomes 10.71.1.4
```

### Reusable multi-monitor files

The files under `Reusable-Multi-Monitor/` contain `{{ monitor }}`. Do not replace that text. The reusable inventory stores a monitor number for each switch, and the run command explicitly selects one inventory hostname.

New users should complete one original Day 1 deployment before using the reusable set.

## Rules When Several People Use the Lab

- Only one person may run a configuration task against a particular switch at a time.
- Say the physical switch name, monitor number, management address, and template name aloud or in the team change record before pressing **Run**.
- Do not edit or replace a YAML or inventory file while another task is running.
- Name reusable Semaphore templates with the device and monitor, such as `BABA 72 — Show Version`.
- Keep BABA and TAAS in separate inventory groups.
- Use individual Semaphore accounts when the working installation supports them; do not publish or casually share administrator credentials.
- Record who ran the task, when it ran, the target, the playbook, and whether the final recap succeeded.
- After a failed task, stop. Let the current operator collect the error before anyone else retries or changes a file.

The VM folder and `/ansible` container folder are shared working locations. One person’s copied file can affect the next person’s task, so verify the filename and target immediately before every run.

## How to Use the Screenshot Segments

Some steps include a **Screenshot guide** directly below them. The actual images are still pending; the guide tells the team exactly what a useful screenshot should contain.

When screenshots are added later:

- capture only the application or terminal area needed for the step;
- keep the command and its successful result visible in the same image when possible;
- use the monitor number shown in the surrounding example, or clearly label a different one;
- hide passwords, private keys, camera client identifiers, browser sessions, and unrelated device configuration;
- do not hide the hostname, relevant management IP, filename, task name, or success result needed to teach the step;
- add a short caption explaining what the learner should notice;
- never use a screenshot as the only instruction—keep the written steps and commands.

An error screenshot must show the first failed task and the final recap. Before sharing it, remove credentials and unrelated sensitive configuration.

## Before You Begin

Confirm these items with the instructor or network owner:

- the assigned monitor number;
- which physical switch is CORE BABA and which is CORE TAAS;
- that Fa0/10, Fa0/11, and Fa0/12 connect between the intended switches;
- the Cisco console method and login details;
- permission to change the switches;
- the correct management addresses;
- the real camera client identifiers, if camera reservations will be used.

Also confirm that the Windows Server VM can reach the Cisco management network. Do not guess interface cabling, IP addresses, or camera identifiers.

## Choose the Correct Starting Path

### Path A — The existing lab already works

Use this path for the current lab. Verify each layer and leave it unchanged when it passes:

1. `wsl -l -v` shows Ubuntu version 2.
2. `sudo docker ps` shows the `semaphore` container as running.
3. `sudo docker exec semaphore ls -la /ansible` shows the playbook folder.
4. `http://208.8.8.200:3000` opens Semaphore from the physical PC.
5. VS Code connects to `208.8.8.200` over Remote-SSH.

Do not reinstall a passing component.

### Path B — Recreating the lab from the beginning

Follow the setup guides in order:

1. [WSL2 and Ubuntu](Setup/WSL2-Ubuntu.md)
2. [Existing Docker and Semaphore verification](Setup/Docker-Semaphore.md)
3. [VS Code Remote-SSH workflow](Setup/VSCode-Remote-SSH.md)
4. [Semaphore project setup](Setup/Semaphore-Project.md)

The Docker/Semaphore guide documents the existing lab rather than inventing a different Semaphore database or container. If no Semaphore container exists at all, obtain the original deployment details or a reviewed deployment standard before creating one.

## Beginner Deployment Order

Stop at the first failed checkpoint.

| Stage | Action | Pass condition |
|---:|---|---|
| 1 | Back up the current switch configuration. | The running configuration has been saved or copied for recovery. |
| 2 | Bootstrap management IP and SSH from the Cisco console. | `show ip ssh` is enabled, the management IP is correct, and both VTY ranges show `login local` plus `transport input ssh`. |
| 3 | Test SSH from Ubuntu. | A manual SSH login reaches the intended switch. |
| 4 | Create and inspect both inventory copies. | `Cisco Inventory` in the GUI and `/ansible/inventory.ini` both contain separate BABA and TAAS groups with the same IPs. |
| 5 | Copy files into `semaphore:/ansible`. | Every required file exists and has a non-zero size. |
| 6 | Run syntax checks. | Every command reports the playbook name with no syntax error. |
| 7 | Run Show Version. | `failed=0` and `unreachable=0`. |
| 8 | Run the base playbook. | The job succeeds and Show Version still works afterward. |
| 9 | Configure and verify trunks/LACP on both ends. | EtherChannel members show bundled rather than `D`/down. |
| 10 | Run BABA-only DHCP and VLAN playbooks. | The expected pools, VLANs, and ports appear. |
| 11 | Camera reservations. | Run only after two real, reviewed client identifiers are available. |

## Read-Only Versus Configuration Tasks

### Read-only

- Show Version playbooks
- Cisco `show ...` commands
- inventory and file listings
- Ansible syntax checks

These inspect state without intentionally changing switch configuration.

### Changes the switch

- Base Layer 3
- Trunk or LACP
- DHCP
- VLAN and port placement
- Camera reservations

Back up first and verify the target before running any configuration task.

## The Working File Flow

```text
VS Code on the physical PC
  → Remote-SSH to Windows Server 208.8.8.200
  → save under C:\Users\Administrator\ansible-lab
  → WSL sees the same file under /mnt/c/Users/Administrator/ansible-lab
  → docker cp copies YAML into semaphore:/ansible
  → Semaphore runs the container YAML with its saved GUI inventory
```

Saving a YAML file in VS Code does not automatically update the copy inside the container. Run the matching `docker cp` command after every YAML edit. When device addresses or groups change, also update both the VS Code/container `inventory.ini` copy and the inline `Cisco Inventory` in Semaphore.

## When Something Does Not Match

Stop and record:

1. the command location;
2. the exact command entered;
3. the complete error;
4. the target hostname and management address;
5. the last checkpoint that passed.

Then use [Troubleshooting](Troubleshooting.md). Do not solve a connectivity error by repeatedly running a configuration playbook.
