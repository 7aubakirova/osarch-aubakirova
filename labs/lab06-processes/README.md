# Lab 6 — Live processes and memory pressure

Pair: Samira Aubakirova 
Driver first half: Samira Aubakirova
Machine: MacBook Air (Apple Silicon, UTM VMs: Ubuntu 26.04.1 LTS ARM64 & Windows 11 ARM64)
Date: 26.11.2026

## What I did

### 1. Inspecting live processes across three tools (Task Manager, Process Explorer, `htop`)

On Windows 11 (`WIN-BRC62OO7064`), I inspected the main `firefox.exe` process (**PID `7060`**) simultaneously in **Task Manager** (Details tab) and **Sysinternals Process Explorer** (lower pane set to `Handles`):

```text
Task Manager (Details):
  Memory (active private working set): 195,860 K
  Commit size: 301,900 K
  Handles: 1,... (column truncated in screenshot; >1,000 open kernel handles)

Process Explorer:
  Private Bytes: 340,884 K
  Working Set:   204,836 K
  Lower pane (Handles): ALPC Port, Desktop, Directory, Event, File
```

On the Ubuntu guest (`lab01-ubuntu`), I launched a background Python workload allocating ~50 MB and inspected its process tree (`F5`), PID, PPID, memory counters (`VIRT`, `RES`, `SHR`), and open file descriptors in **`htop`** and `/proc`:

```bash
python3 -c "import time; data = ['x'*1024 for _ in range(50000)]; time.sleep(3600)" &
[1] 1559

htop
# Pressed F5 (Tree view):
# Parent: -bash (PID 1475) -> Child: python3 (PID 1559, VIRT: 18272, RES: 9768, SHR: 6004, MEM%: 0.6)
```

### 2. Controlled memory pressure test and swap observation (`memory-pressure.txt`)

To observe gradual memory consumption and swap usage before OOM termination, I recorded `free -h` **before**, **during** (4 seconds into a paced allocation loop adding 50 MB every 0.5 s), and **after** the kernel OOM Killer terminated the process:

```bash
mkdir -p ~/lab06_fix
echo "=== BEFORE ===" > ~/lab06_fix/memory-pressure.txt
free -h >> ~/lab06_fix/memory-pressure.txt

python3 -c "import time; a = []; [ (a.append('x' * 50 * 1024 * 1024), time.sleep(0.5)) for _ in range(100) ]" &
[1] 2050
sleep 4
echo "=== DURING ===" >> ~/lab06_fix/memory-pressure.txt
free -h >> ~/lab06_fix/memory-pressure.txt

# Waited ~15 s until process 2050 was killed by the OOM killer:
echo "=== AFTER ===" >> ~/lab06_fix/memory-pressure.txt
free -h >> ~/lab06_fix/memory-pressure.txt
cat ~/lab06_fix/memory-pressure.txt
```

```text
=== BEFORE ===
               total        used        free      shared  buff/cache   available
Mem:           1.6Gi       325Mi       1.1Gi       5.2Mi       217Mi       1.3Gi
Swap:          2.0Gi        12Mi       2.0Gi
=== DURING ===
               total        used        free      shared  buff/cache   available
Mem:           1.6Gi       713Mi       785Mi       5.2Mi       217Mi       914Mi
Swap:          2.0Gi        12Mi       2.0Gi
=== AFTER ===
               total        used        free      shared  buff/cache   available
Mem:           1.6Gi       292Mi       1.3Gi       5.1Mi       106Mi       1.3Gi
Swap:          2.0Gi        17Mi       2.0Gi
```

### 3. Extracting kernel OOM logs (`kernel-log-extract.txt`) and Windows Event ID 2004

```bash
sudo dmesg -T | grep -i -E "oom|killed process" | tail -n 15 > ~/lab06_fix/kernel-log-extract.txt
cat ~/lab06_fix/kernel-log-extract.txt
```

```text
[Thu Nov 26 16:37:29 2026] python3 invoked oom-killer: gfp_mask=0x440dc0(GFP_KERNEL_ACCOUNT|__GFP_ZERO|__GFP_COMP), order=0, oom_score_adj=0
[Thu Nov 26 16:37:29 2026] oom-kill:constraint=CONSTRAINT_NONE,nodemask=(null),cpuset=/,mems_allowed=0,global_oom,task_memcg=/user.slice/user-1000.slice/session-4.scope,task=python3,pid=2050,uid=1000
[Thu Nov 26 16:37:29 2026] Out of memory: Killed process 2050 (python3) total-vm:3501928kB, anon-rss:1385428kB, file-rss:1520kB, shmem-rss:0kB, UID:1000 pgtables:6848kB oom_score_adj:0
```

On Windows 11, exhausting virtual memory via PowerShell triggered a managed `System.OutOfMemoryException` and logged a system-level low-memory warning in Event Viewer (`System` log, Source: `Resource-Exhaustion-Detector`, **Event ID `2004`**).

## Result

### Three-tool comparison (`Task Manager` vs `Process Explorer` vs `htop`)

| Feature / Metric | Windows Task Manager (`firefox.exe`, PID `7060`) | Sysinternals Process Explorer (`firefox.exe`, PID `7060`) | Linux `htop` (`python3`, PID `1559`) |
| :--- | :--- | :--- | :--- |
| **Process hierarchy** | Flat list in *Details* tab (no parent-child tree). | Full parent-child tree (main `firefox.exe` PID `7060` with child renderer processes). | Tree view enabled with `F5` (`-bash` PID `1475` $\rightarrow$ `python3` PID `1559`). |
| **Physical RAM metric** | **Memory (active private working set):** `195,860 K` | **Working Set:** `204,836 K` | **RES:** `9768 K` (~9.5 MB), **SHR:** `6004 K` (~5.9 MB) |
| **Virtual / committed memory** | **Commit size:** `301,900 K` | **Private Bytes:** `340,884 K` | **VIRT:** `18272 K` (~17.8 MB) |
| **Open handles / FDs** | **Handles** column (truncated to `1,...`; >1,000 handles). | Lower pane (`Ctrl+L`) lists every open kernel handle by type (`ALPC Port`, `File`, `Event`, `Desktop`, `Directory`). | File descriptors visible in `/proc/1559/fd` (standard `0`, `1`, `2`). |
| **OOM / memory-pressure behavior** | Reports high commit charge; Windows logs **Event ID `2004`** while the shell catches `System.OutOfMemoryException`. | Highlights committed virtual memory (`Private Bytes`) and system commit charge (`77.97%`). | Kernel invokes the **OOM Killer**, which selects the process with the highest `oom_score` (`oom_score_adj: 0` is only an adjustment to it) and sends `SIGKILL` (`Killed process 2050`). |

### Where the tools disagree and why

- **Physical memory (`195,860 K` in Task Manager vs `204,836 K` in Process Explorer):** Task Manager's *Details* tab shows the **active private working set**, which counts only physical pages private to `firefox.exe` (`PID 7060`) and excludes shared DLL pages. Process Explorer's **Working Set** is the total physical RAM mapped into the process, including shared system DLLs, which accounts for the `~8,976 K` difference. `htop` separates these explicitly: **RES** (`9768 K`) is total resident memory and **SHR** (`6004 K`) is the shared part of it.
- **Committed / virtual memory (`301,900 K` in Task Manager vs `340,884 K` in Process Explorer):** **Commit size** and **Private Bytes** both measure private virtual memory backed by RAM or the pagefile. They differ slightly because the two snapshots were taken a few seconds apart while Firefox was allocating and freeing heap blocks, and because the tools may account for some private reservations differently.
- **Handles visibility:** Task Manager shows only a numeric count (**Handles** column, truncated to `1,...` at default width), whereas Process Explorer's lower pane shows the exact kernel objects (`ALPC Port`, `File`, `Event`, ...) held open by `PID 7060`.

## What did not work the first time

1. **Fast one-liner allocation script:**
   - **What I saw:** `[a.append('x' * 10**7) for _ in range(1000)]` was `Killed` almost instantly, before `free -h` could be run during the test, so the swap transition went unobserved.
   - **What it turned out to be:** With no delay between allocations, Python exhausted the 1.6 GiB guest RAM in under a second.
   - **What I did about it:** Added `time.sleep(0.5)` between 50 MB allocations (`PID 2050`) and captured `free -h` before (`325Mi` used, `12Mi` swap), during (`713Mi` used), and after (`292Mi` used, swap up to `17Mi`).
2. **Missing redirection when saving `dmesg` output:**
   - **What I saw:** `cat ~/lab06_fix/kernel-log-extract.txt` returned `No such file or directory`.
   - **What it turned out to be:** I had run `sudo dmesg -T | grep -i -E "oom|killed process"` without `> ~/lab06_fix/kernel-log-extract.txt`, so the output only went to stdout.
   - **What I did about it:** Re-ran the command with `| tail -n 15 > ~/lab06_fix/kernel-log-extract.txt` and copied both `.txt` files to the host via `scp`.
3. **Forgetting the `F5` tree view in the first `htop` capture:**
   - **What I saw:** The first `htop` screenshot showed a flat PID-sorted list, not parent-child relationships.
   - **What it turned out to be:** `htop` defaults to the sorted list unless `F5` (`Tree`) is toggled.
   - **What I did about it:** Launched `python3` (`PID 1559`), toggled `F5` to confirm its parent (`-bash`, `PID 1475`), and captured a new `evidence/htop.png`.

## Evidence

- [evidence/task-manager.png](evidence/task-manager.png) — Windows 11 Task Manager (*Details*) showing `firefox.exe` (`PID 7060`): `Memory (active private working set)` = `195,860 K`, `Commit size` = `301,900 K`.
- [evidence/process-explorer.png](evidence/process-explorer.png) — Process Explorer showing `firefox.exe` (`PID 7060`): `Private Bytes` = `340,884 K`, `Working Set` = `204,836 K`, process tree, and open kernel handles in the lower pane.
- [evidence/htop.png](evidence/htop.png) — Ubuntu `htop` in tree view (`F5`): `-bash` (`PID 1475`) and child `python3` (`PID 1559`, `VIRT 18272`, `RES 9768`, `SHR 6004`).
- [evidence/memory-pressure.txt](evidence/memory-pressure.txt) — raw `free -h` output before (`325Mi` used, `12Mi` swap), during (`713Mi` used), and after (`292Mi` used, `17Mi` swap) the paced memory-pressure test (`PID 2050`).
- [evidence/kernel-log-extract.txt](evidence/kernel-log-extract.txt) — raw `dmesg -T` extract showing `python3 invoked oom-killer` and `Out of memory: Killed process 2050 (python3)`.
- [evidence/ubuntu-oom.png](evidence/ubuntu-oom.png) — terminal screenshot confirming the Linux OOM Killer run and `dmesg` output.
- [evidence/windows-oom.png](evidence/windows-oom.png) — PowerShell screenshot showing `System.OutOfMemoryException` under virtual memory exhaustion on Windows 11.
- [evidence/windows-event-2004.png](evidence/windows-event-2004.png) — Event Viewer screenshot showing Event ID `2004` (`Resource-Exhaustion-Detector`) during low virtual memory.
