# Task 2: Plan and justify

## 1. Allocation

Host: MacBook Air, Apple M5, 10 cores / 10 threads, 16 GB RAM, 333 GB free on the APFS container (494.3 GB total). Hypervisor: UTM. Both guests are ARM64 because the host is ARM64.

| | Ubuntu Server (ARM64) | Windows 11 (ARM64) |
|---|---|---|
| vCPU | 1 | 4 |
| Memory | 2 GB (2048 MiB) | 4 GB (4096 MiB) |
| Disk | 25 GB, dynamically allocated | 64 GB, dynamically allocated |
| Network adapter | 1 (virtio-net-pci) | 1 (virtio-net-pci) |
| Network mode | Shared Network (NAT) | Shared Network (NAT) |

## 2. Arithmetic 

Assumptions about what the host itself needs: about 6 GB of RAM, about 4 threads of CPU, and at least 50 GB of free disk for APFS and swap.

- CPU: 10 threads - (1 + 4) = 5 left for the host. vCPUs are scheduled, not reserved, so the host can use idle capacity. OK.
- Memory: 16 GB - (2 + 4) = 10 GB left, the host needs about 6 GB, so the margin is about 4 GB (this also covers UTM/QEMU overhead). In Task 1, `top` showed "15G used, 295M unused", but much of that is cache and compressed memory, so I did not give the guests more.
- Disk: worst case 25 + 64 = 89 GB, so 333 - 89 = 244 GB left, well above the 50 GB the host needs. The disks are dynamic, so real usage starts much lower.

Answer: yes, this computer can run both guests at the same time as asked.
If memory pressure became a problem, I would reduce Ubuntu to 1 GB first. I would not reduce Windows, because 4 GB is the Windows 11 minimum.

## 3. Justification

- Network mode, Shared Network (NAT): each guest gets its own address behind the host (typically 192.168.64.x). The host can reach Ubuntu by SSH, the guests can reach each other, and everything reaches the internet through the host. Bridged would give each guest its own MAC address, and a campus network with MAC filtering can silently drop it. Host-only would allow SSH but give no internet access, which breaks the requirement.
- Ubuntu 1 vCPU: Server has no GUI and mostly waits for SSH and light commands, so one core is enough. This leaves more for the host and for Windows.
- Ubuntu 2 GB RAM: more than the Server needs, so it stays responsive during updates. It is the first thing to reduce if the host is under pressure.
- Windows 4 vCPU: Windows is the heavier guest and the host has idle threads. The minimum for Windows 11 is 2 cores, and I went above it.
- Windows 4 GB RAM and 64 GB disk: these are exactly the Windows 11 minimums. I did not go lower because below them the installer and
