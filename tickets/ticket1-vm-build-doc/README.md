# Ticket 1 — VM build document

## 1. What was asked
> "Hi, we have someone starting on Monday and he needs a test computer set up. Nothing special, just so he can try things without breaking anything. Can you write down how to do it so I can do the next one myself? Thanks"  
> — Aigerim T. (Office Manager)

## 2. What was actually needed
The request is written by a non-technical manager and leaves out almost every engineering detail required to provision a working, safe machine:

1. **Host Architecture and Hypervisor Choice:**
   - *What the request omitted:* Aigerim did not specify what operating system or hardware host the computer runs on.
   - *Where it matters:* Our host machine is Apple Silicon (MacBook Air M5, ARM64 architecture). If she attempts to install a legacy x86 hypervisor (such as VirtualBox) or download an x86/Intel ISO, it will fail to boot or run under heavy emulation at unusable speeds. UTM using native ARM64 virtualization (Apple Hypervisor / QEMU backend) is required.
2. **Operating System and Toolchain Requirements:**
   - *What the request omitted:* "A test computer... just so he can try things" does not identify the target operating system, distribution, or developer tools.
   - *What was needed:* Modern software development typically requires a standard Linux runtime (Ubuntu 24.04 LTS ARM64) with shell access, package management (`apt`), and standard development runtimes.
3. **Resource Sizing and Capacity Planning:**
   - *What the request omitted:* No CPU, RAM, or disk allocations were provided.
   - *Calculations from physical host:*
     - Physical host specs: 16 GB unified RAM, 256 GB NVMe SSD.
     - macOS host reserve: ~8 GB reserved to ensure zero host memory compression or swap thrashing while running browsers and productivity apps.
     - Linux VM allocation: 2 vCPUs, 2048 MB (2 GB) RAM, and a 25 GB dynamic (sparse) virtual disk.
     - Storage footprint: The 25 GB virtual disk only consumes ~3.5 GB on the host SSD initially, growing dynamically only as packages are installed.
4. **The "Without Breaking Anything" Requirement (Safety & Recovery):**
   - *Where the request is naive:* A developer testing experimental code and low-level system settings will inevitably break the OS, corrupt configuration files, or brick the package manager.
   - *What was actually needed:* An instant rollback mechanism. A clean baseline snapshot must be captured immediately after installation so the machine can be restored in seconds without reinstalling.
5. **Network Connectivity & Campus Policies:**
   - *What the request omitted:* No mention of network access.
   - *What was needed:* Bridged networking fails on university/office Wi-Fi due to 802.1X enterprise authentication, client isolation, and MAC filtering. The VM must be placed behind a Shared Network (NAT) adapter so it masquerades behind the host's authenticated IP address while retaining WAN egress.

**What I would have asked Aigerim:**
1. What hardware/OS will host this VM (Apple Silicon Mac or Intel/AMD PC)?
2. What specific tech stack or distribution does the developer require (Ubuntu, Debian, Fedora)?
3. Does the developer need a graphical desktop interface (GUI) or just headless terminal/SSH access?
4. How much persistent disk storage does their testing workload need?

**What I assumed instead:**
Assumed an Apple Silicon host running UTM, Ubuntu Server 24.04 LTS ARM64, 2 vCPUs / 2 GB RAM / 25 GB dynamic disk, Shared Network (NAT) mode, and a mandatory post-install snapshot named `01-clean-baseline`.

## 3. What I did
1. Installed **UTM v4.x** on the macOS host to leverage native ARM64 hypervisor virtualization.
2. Downloaded the official `ubuntu-24.04-live-server-arm64.iso` installer.
3. Configured the VM with hardware sizing calculated in Section 2:
   - 2 vCPUs
   - 2048 MB RAM
   - 25 GB dynamic virtual disk (VirtIO interface)
4. Configured the virtual network interface to **Shared Network (NAT)** with a VirtIO network device.
5. Completed the standard base OS installation (created user account, updated repository indexes).
6. Verified guest networking: tested outbound DNS resolution and HTTP/ping connectivity to external gateways (`8.8.8.8`).
7. Shut down the guest cleanly and generated a baseline snapshot: `01-clean-baseline`.
8. Wrote a step-by-step build document (`BUILD.md`) targeting a non-technical reader, providing operational reasons and verification checks for every step.

## 4. Evidence
- [evidence/01-new-vm-wizard.png](evidence/01-new-vm-wizard.png) — Proves architecture selection (Virtualize ARM64 vs Emulate).
- [evidence/02-disk-settings.png](evidence/02-disk-settings.png) — Proves dynamic 25 GB disk allocation and VirtIO bus interface.
- [evidence/03-network-adapter.png](evidence/03-network-adapter.png) — Proves adapter is set to Shared Network (NAT) mode.
- [evidence/04-guest-uname.txt](evidence/04-guest-uname.txt) — Proves active guest kernel runtime, hostname, and ARM64 architecture (`uname -a`).
- [evidence/05-ping-between-guests.txt](evidence/05-ping-between-guests.txt) — Proves functional outbound packet routing and DNS/gateway reachability.
- [evidence/06-snapshot-list.png](evidence/06-snapshot-list.png) — Proves creation of the `01-clean-baseline` rollback snapshot.

## 5. What I would do differently
I initially attempted to configure the network interface as **Bridged (Advanced)** so the VM would receive its own IP address directly from the local LAN. However, once connected to the university Wi-Fi, the enterprise router dropped the DHCP requests due to 802.1X client isolation and MAC address filtering. The VM had no internet access. 

I resolved this by switching the adapter to UTM's **Shared Network (NAT)**, routing traffic through the host Mac's established connection. In the future, I will always default developer lab templates on Wi-Fi hosts to NAT first, and document port-forwarding rules if inbound host-to-guest connectivity is needed.
