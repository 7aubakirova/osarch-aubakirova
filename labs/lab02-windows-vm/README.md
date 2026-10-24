# Lab 2 — A Windows guest, and a network between them

Pair: Samira Aubakirova                
Machine: macOS (16GB RAM) running UTM
Date: October 25, 2026

## What I did
1. Configured Ubuntu and Windows 11 VMs to test memory overcommitment by allocating more RAM to the guests than physically available on the host.
2. Monitored macOS host memory pressure and swap usage before and during VM execution.
3. Retrieved IP address configuration on the Ubuntu VM:
ip address

inet 192.168.64.2/24 metric 100 brd 192.168.64.255 scope global dynamic enp0s1

4. Retrieved IP address configuration on the Windows VM:
ipconfig

IPv4 Address. . . . . . . . . . . : 192.168.64.4

5. Enabled the "File and Printer Sharing (Echo Request - ICMPv4-In)" inbound rule in Windows Defender Firewall.
6. Verified network connectivity from Ubuntu to Windows:
ping -c 4 192.168.64.4

4 packets transmitted, 4 received, 0% packet loss, time 3006ms

7. Verified network connectivity from Windows to Ubuntu:
ping 192.168.64.2

Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)

## Result
Both the Ubuntu Linux and Windows 11 virtual machines ran simultaneously on a shared network. The macOS host successfully handled the memory overcommitment by utilizing memory compression and swap space, preventing a system crash but impacting performance. Two-way network communication was successfully established and verified between both guest operating systems.

## What did not work the first time
What I saw: When I first tried to ping the Windows VM from Ubuntu, the terminal returned Destination Host Unreachable with 100% packet loss.
What it turned out to be: Windows Defender Firewall blocks incoming ICMP echo requests by default, which caused the Windows VM to ignore the pings from Ubuntu.
What I did about it: I opened "Windows Defender Firewall with Advanced Security" in the Windows VM, navigated to Inbound Rules, and enabled the File and Printer Sharing (Echo Request - ICMPv4-In) rule. After that, the ping succeeded with 0% packet loss.

## Evidence
[evidence/host-memory-before.png](evidence/host-memory-before.png) — Proves the baseline RAM usage of the host system before starting the VMs.
[evidence/host-memory-after.png](evidence/host-memory-after.png) — Proves the host's memory pressure, swap usage, and compression during the memory overcommitment test.
[evidence/ip-address-linux.txt](evidence/ip-address-linux.txt) — Proves the Ubuntu VM successfully acquired the local IP address 192.168.64.2.
[evidence/ip-confiig-windows.txt](evidence/ip-confiig-windows.txt) — Proves the Windows VM successfully acquired the local IP address 192.168.64.4.
[evidence/ping-both-ways.txt](evidence/ping-both-ways.txt) — Proves successful two-way network connectivity (0% packet loss) between the Linux and Windows VMs.
