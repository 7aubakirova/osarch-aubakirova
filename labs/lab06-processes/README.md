# Lab 6 — Live processes and memory pressure

Pair: Samira Aubakirova   Driver first half: Samira
Machine: MacBook Air (Apple Silicon, UTM VM: Ubuntu 24.04 ARM64 & Windows 11 ARM64)
Date: 2026-11-22

## What I did

### Ubuntu Linux (Process Inspection & OOM Killer)

```bash
python3 -c "import time; data = ['x'*1024 for _ in range(50000)]; time.sleep(600)" &
[1] 9712

PID=$(pgrep -n python3)
echo "PID: $PID | PPID: $(ps -o ppid= -p $PID | tr -d ' ') | FDs $(ls /proc/$PID/fd | wc -l)"
PID: 9712 | PPID: 1341 | FDs 3

ps -o pid,ppid,rss,vsz,comm -p $PID
  PID  PPID   RSS   VSZ COMMAND
 9712  1341 11424 19980 python3
```

```bash
python3 -c "a = []; [a.append('x' * 10**7) for _ in range(1000)]"
Killed

sudo dmesg -T | grep -i -E "oom|killed process" | tail -n 10
[Sun Nov 22 17:38:52 2026] python3 invoked oom-killer: gfp_mask=0x140dca(GFP_HIGHUSER_MOVABLE|__GFP_ZERO|__GFP_COMP), order=0, oom_score_adj=0
[Sun Nov 22 17:38:52 2026] oom-kill:constraint=CONSTRAINT_NONE,nodemask=(null),cpuset=/,mems_allowed=0,global_oom,task_memcg=/user.slice/user-1000.slice/session-1.scope,task=python3,pid=9868,uid=1000
[Sun Nov 22 17:38:52 2026] Out of memory: Killed process 9868 (python3) total-vm:3476848kB, anon-rss:1390788kB, file-rss:1456kB, shmem-rss:0kB, UID:1000 pgtables:6832kB oom_score_adj:0
```

### Windows 11 (Process Explorer & Memory Exhaustion)

```powershell
# Inspected firefox.exe (PID 7060, PPID 6380 explorer.exe) in Task Manager & Process Explorer:
# Working Set: 145,268 K - 179,848 K | Private Bytes: 341,068 K - 341,180 K | Handles: 1,000+

$list = New-Object System.Collections.Generic.List[byte[]]; while ($true) { $list.Add((New-Object byte[] (100MB))); Start-Sleep -Milliseconds 200 }
New-Object : Exception calling ".ctor" with "1" argument(s): "Exception of type 'System.OutOfMemoryException' was thrown."
At line:1 char:88
+ ... byte[]]; while ($true) { $list.Add((New-Object byte[] (100MB))); Star ...
```

```powershell
# Checked Event Viewer (eventvwr.msc -> Windows Logs -> System -> Filter Event ID 2004):
# Source: Resource-Exhaustion-Detector | Level: Warning | Time: 11/22/2026 11:49:19 PM
# Message: Windows successfully diagnosed a low virtual memory condition. The following programs consumed the most virtual memory: powershell.exe ...
```

## Result

| Metric | Windows 11 (`firefox.exe`) | Ubuntu Linux (`python3`) |
| :--- | :--- | :--- |
| **PID** | `7060` | `9712` |
| **PPID (Parent)** | `6380` (`explorer.exe`) | `1341` (`-bash`) |
| **Physical Memory (Working Set / RSS)** | `145,268 K` – `179,848 K` (~142–176 MB) | `11,424 K` (~11.2 MB) |
| **Virtual / Committed Memory (Private Bytes / VSZ)** | `341,068 K` – `341,180 K` (~333 MB) | `19,980 K` (~19.5 MB) |
| **Open Descriptors (Handles / FDs)** | `1,000+` | `3` (`0`, `1`, `2` standard streams) |
| **OOM / Memory Pressure Behavior** | Throws `System.OutOfMemoryException` at the application/runtime level, compresses memory, and logs Event ID `2004` (`Resource-Exhaustion-Detector`) without immediately killing the process. | Kernel invokes the **OOM Killer**, selects the highest-scoring process (`oom_score_adj: 0`), and terminates it immediately via `SIGKILL` (`Killed process 9868`). |

## What did not work the first time

1. **Running `python` instead of `python3` in Ubuntu:**
   - **What I saw:** `Command 'python' not found, did you mean: command 'python3'` (`Exit 127`) when launching the background memory-allocation script.
   - **What it turned out to be:** Ubuntu 24.04 does not symlink `/usr/bin/python` to Python 3 by default.
   - **What I did about it:** Replaced `python` with `python3` and verified the background PID (`9712`) before inspecting `/proc`.

2. **Checking `/proc/<PID>/fd` after process termination:**
   - **What I saw:** `ls: cannot access '/proc/9788/fd': No such file or directory`.
   - **What it turned out to be:** The background `sleep` timer had already completed (`[1]+ Done`), causing the kernel to remove `/proc/9788`.
   - **What I did about it:** Re-launched the background `python3` process and queried `/proc/$PID/fd` immediately while the process was still active (`PID 9712`), confirming `3` open file descriptors.

## Evidence

- [evidence/task-manager.png](evidence/task-manager.png) — Proves Windows 11 Task Manager process metrics (`Working Set`, CPU, memory usage) for `firefox.exe`.
- [evidence/process-explorer.png](evidence/process-explorer.png) — Proves detailed Windows process hierarchy, `Private Bytes` (`341,180 K`), `Working Set` (`145,268 K`), PID `7060`, and open `Handles` for `firefox.exe`.
- [evidence/ubuntu-metrics.png](evidence/ubuntu-metrics.png) — Proves exact Ubuntu Linux process metrics via `ps` and `/proc/9712/fd` (`PID: 9712`, `PPID: 1341`, `RSS: 11424`, `VSZ: 19980`, `FDs: 3`).
- [evidence/htop.png](evidence/htop.png) — Proves Ubuntu Linux process tree hierarchy (`F5` tree view) and real-time memory/CPU bars in `htop`.
- [evidence/ubuntu-oom.png](evidence/ubuntu-oom.png) — Proves Linux kernel OOM Killer activation (`Killed` output and `dmesg` log: `Out of memory: Killed process 9868 (python3)`).
- [evidence/windows-oom.png](evidence/windows-oom.png) — Proves Windows 11 handling of memory exhaustion in PowerShell (`System.OutOfMemoryException` thrown while the shell process survives).
- [evidence/windows-event-2004.png](evidence/windows-event-2004.png) — Proves Windows system logging of low virtual memory pressure in Event Viewer (`Event ID 2004`, `Resource-Exhaustion-Detector`).
