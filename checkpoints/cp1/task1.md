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
