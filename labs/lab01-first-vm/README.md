# Lab 1 — Your first virtual machine

Pair: Aubakirova 
Driver first half: Aubakirova
Machine: Host (MacBook Air/macOS), Guest (Ubuntu Server 22.04 LTS)
Date: 18.10.2026

## Resource Justification
I allocated 2 vCPUs, 4 GB of RAM, and 25 GB of disk space to the guest VM. My host system has 16 GB of RAM and 8 CPU cores. Since my host OS takes around 6 GB of RAM when idle, giving 4 GB to this VM leaves sufficient RAM for smooth host operation and allows running a second VM simultaneously in future labs.

## What I did
1. Created a new Virtual Machine and installed Ubuntu Server 22.04 LTS (console-only).
2. Took a VM snapshot named clean-install before modifying any configurations.
3. Verified the guest OS version using the command line:
   ```bash
   lsb_release -a > guest-release.txt
4. Intentionally broke the system by removing ⁠/etc/hostname⁠:
   ```bash
   sudo rm /etc/hostname
   ```
5. Verified the system was broken, then performed a rollback to the ⁠clean-install⁠ snapshot and confirmed system recovery.
## Result
The VM was successfully created and configured with custom resources justified against the host hardware. The snapshot ⁠clean-install⁠ was created, the deliberate system breakage was reproduced, and rolling back restored the VM to fully working condition.
## What did not work the first time
Initially, the VM failed to boot because the ISO image was not properly attached. I had to go to storage settings and select the Ubuntu ISO manually.
## Evidence
 - [evidence/vm-settings.png](evidence/vm-settings.png) — proves configured CPU, memory, and disk settings.
- [evidence/guest-release.txt](evidence/guest-release.txt) — output of lsb_release -a proving Ubuntu Server installation.
- [evidence/broken.png](evidence/broken.png) — screenshot showing the guest in its broken state.
- [evidence/snapshot-list.png](evidence/snapshot-list.png) — snapshot management list showing successful rollback to clean-install.
