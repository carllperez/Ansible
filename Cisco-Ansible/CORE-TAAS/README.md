# CORE TAAS Automation

## Status

CORE TAAS automation is the next device configuration phase.

This document is intentionally kept separate from CORE BABA so the environment setup and troubleshooting do not need to be repeated.

Cisco commands and playbooks added here must follow `DAY1-May5-SirRob.txt` strictly.

## Planned Workflow

```text
1. Apply/verify required CORE TAAS base configuration
2. Add SSH prerequisite for Ansible if required
3. Add CORE TAAS to the Ansible/Semaphore inventory
4. Test read-only connectivity
5. Convert each CORE TAAS Day 1 configuration block into YAML
6. Run through Semaphore
7. Verify on the Cisco CLI
```

<!-- SCREENSHOT: CORE TAAS console -->
<!-- SCREENSHOT: CORE TAAS Semaphore task -->

> Continue this file as CORE TAAS configuration is completed.
