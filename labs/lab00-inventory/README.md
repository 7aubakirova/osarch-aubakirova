# Lab 0 — Inventory your own machine

Pair:                     Driver first half:
Machine:
Date:

## What I did

Steps, with commands in code blocks. Not prose about the steps.

<'''
<system_profiler SPHardwareDataType>
'''
'''
<sysctl hw.logicalcpu>
'''
'''
<system_profiler SPMemoryDataType>
'''
'''
<diskutil info disk0>
'''
'''
<df -h />
'''
'''
<sysctl kern.hv_support>
'''>
<output, trimmed>
```

## Result 
| What | Value | Where I got it |
| :--- | :--- | :--- |
| CPU model| Apple M5 | `system_profiler SPHardwareDataType` |
| CPU core count| 10 (4 Super, 6 Efficiency) | `system_profiler SPHardwareDataType` |
| CPU thread count | 10 (Apple Silicon does not use Hyper-Threading, so threads = cores) | `sysctl hw.logicalcpu` |
| Total memory | 16 GB | `system_profiler SPMemoryDataType` |
| Memory modules & speed| Integrated unified memory (LPDDR5) | `system_profiler SPMemoryDataType` |
| Disk model & type| APPLE SSD AP0512Z (Solid State, Apple Fabric NVMe) | `diskutil info disk0` |
| Disk free space| [You still need to run a command for this]| Run `df -h /` to find out |
| Firmware type, version & date | Apple System Firmware (iBoot) v18000.120.36 | `system_profiler SPHardwareDataType` |
| Hardware virtualization| Present and enabled (Value: 1) | `sysctl kern.hv_support` |

What did not work the first time

At least one thing. What I saw, what it turned out to be, what I did about it.

Evidence

- [evidence/](evidence/) — one line per file on what it proves
