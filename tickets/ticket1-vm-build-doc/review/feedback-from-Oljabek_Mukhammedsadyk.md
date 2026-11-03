# Feedback on Samira Aubakirova's BUILD.md
Reviewer: Oljabek Mukhammedsadyk
Commit reviewed: https://github.com/7aubakirova/osarch-aubakirova/tree/main/tickets/ticket1-vm-build-doc
Hypervisor I used: UTM, version 4.5.3

## First step where I got stuck
Step number: Step 2
What the step says: "Select your downloaded ubuntu-24.04-live-server-arm64.iso installation image."
What I did: Looked for the installation media in my local Downloads directory.
What happened / what I had to guess: I had downloaded the Ubuntu 26.04 ARM64 ISO image as shown in the lab lecture materials and your evidence file (`04-guest-uname.txt`), but the document specifically instructed selecting a 24.04 release. I had to assume that using the newer 26.04 ISO would work identically with the documented hardware profile.

## Other places I hesitated
- Step 5: When opening the Network tab, UTM presents multiple Network Modes (Shared Network, Bridged, Host Only). While the text correctly specified Shared Network, I hesitated momentarily looking for the "VirtIO" device type drop-down to confirm the virtual network card model.
- Step 6: When creating the snapshot `01-clean-baseline`, I had to ensure the VM was fully powered down first so the memory state wouldn't inflate the snapshot size on disk.

## Settings I would not have chosen without the document
- Allocating exactly 2048 MB (2 GB) RAM rather than default 4096 MB to preserve host system resources on a 16 GB Apple Silicon Mac.
- Selecting Shared Network (NAT) over Bridged to prevent DHCP rejection caused by Innotech campus Wi-Fi MAC filtering and 802.1X isolation.

## One thing that worked well
- The explicit differentiation between "Virtualize" and "Emulate" in Step 1 was clear and prevented choosing the emulation path, which would have severely degraded VM performance on Apple Silicon.
