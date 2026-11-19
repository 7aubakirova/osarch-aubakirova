# Task 1a: The computer in front of you

| Figure | This computer | Command it came from |
|---|---|---|
| CPU model | Apple M5 | `sysctl -n machdep.cpu.brand_string` |
| Cores / logical processors | 10 physical / 10 logical | `sysctl hw.physicalcpu hw.logicalcpu` |
| Hardware virtualization: supported? enabled in firmware? | Supported: yes (`kern.hv_support: 1`). Firmware: no setting on a Mac, the hypervisor framework is available by default | `sysctl kern.hv_support` |
| Memory: total / available now | 16 GB total (17179869184 bytes) / 295 MB unused (15G used, including compressed memory and cache) | `sysctl -n hw.memsize`; `top -l 1 \| grep PhysMem` |
| System disk: size, free space, SSD or HDD | Physical disk 500.3 GB; APFS container 494.3 GB, 333.0 GB free; SSD (Solid State: Yes, Internal) | `diskutil list`; `diskutil info / \| grep -E "Solid State\|Container Total Space\|Container Free Space\|Device Location"` |
| Partition style of the system disk | GPT (`GUID_partition_scheme`) | `diskutil list` |
| OS and version | macOS 26.5.1 (build 25F80) | `sw_vers` |

# Task 1b: A machine you have never seen

## 1. Both guests?
Guest sizes (from my UTM settings): Ubuntu U vCPU / 2 GB RAM / 5.94 GB disk, Windows W vCPU / 4 GB RAM / 20.18 GB disk.
Assumptions: the host Windows 11 idles at ~3.5 GB RAM (assumption); the 31 GB free is already net of the host.

- CPU: U + W vCPU needed vs 4 threads at 1.1 GHz. Technically possible (CPU can be overcommitted), but slow and nothing is left for the host.
- Memory: 2 + 4 = 6 GB needed vs 8 - 3.5 = ~4.5 GB left. Does not fit; the host would swap onto slow eMMC.
- Disk: 5.94 + 20.18 = 26.12 GB vs 31 GB free. Fits on paper with ~5 GB margin, but the disks grow as they are used, so it is risky.

Answer: no, not both at the same time, because of memory. Also: S mode blocks installing a hypervisor, and my guests are ARM64 while this CPU is AMD x86-64, so they would have to be rebuilt as x86 images.

## 2. Upgrade first
Nothing can be upgraded: RAM and eMMC are soldered, and so is the CPU.
First change: leave S mode (needed to install a hypervisor), then put the VM disks on an external USB SSD.
Then run one guest at a time (Ubuntu Server only, 2 GB), or use a lab machine for both.
