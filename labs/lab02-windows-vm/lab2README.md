# Lab 2: A Windows guest, and a network between them

## System Description
Host system: macOS (16 GB RAM), UTM hypervisor.
Guest system 1: Ubuntu Linux (IP: 192.168.64.2).
Guest system 2: Windows 11 (IP: 192.168.64.4).

## Memory Overcommitment Experiment
During this lab, a test was conducted by allocating more RAM to the virtual machines than is physically available on the host system. 
Result: When the RAM limit is exceeded, the host system (macOS) actively uses memory compression and swap files. This allows the virtual machines to continue running without an immediate Out-Of-Memory (OOM) crash. However, overall system performance decreases significantly due to high storage access latency.

## Network Connectivity
Both virtual machines are configured to use a Shared Network interface and successfully acquired local IP addresses.
To ensure two-way communication, an inbound rule allowing Echo Requests (ICMPv4-In) was enabled in the Windows Defender Firewall. 
Network connectivity was successfully verified using a two-way ping: packets were delivered with 0% packet loss and minimal latency.

## Evidence
host-memory-before.png — host resource allocation before starting the VMs.
host-memory-after.png — host resource allocation during memory overcommitment.
ip-address-linux.txt — Ubuntu network interfaces output.
ip-confiig-windows.txt — Windows 11 IP configuration.
ping-both-ways.txt — results of the successful ping test between the two VMs.
