# Task 3: Diagnose

Ticket: /var/log/nightly-report.log is no longer written; the scheduler says the job runs every two minutes.
Evidence: cp1-evidence pack from office-srv (files 01-22; 00 is the README).

| # | Evidence file (command) | What I expected | What I found | What I concluded |
|---|---|---|---|---|
| 1 | 01 (hostnamectl) | which system I am dealing with | Ubuntu 24.04.1 LTS, kernel 6.8.0-48, VirtualBox VM | an ordinary server, nothing unusual. The kernel version matches the Nov 9 upgrade (see file 21) |
| 2 | 02 (uptime) | whether the server is overloaded or was rebooted recently | up 6 days 23:51 (booted about Nov 9 09:50), load average 0.00 | the server is idle, so load is not the cause. It booted about 4.5 hours before the first failure (14:20, see file 09), so the reboot itself is probably not the cause |
| 3 | 03 (df -h) | a full disk, so the log cannot be written | / is 48% used (5.8G free), /boot 12% | disk space is fine; ruled out |
| 4 | 04 (free -h) | no free memory | 1.5 GiB available, swap unused | memory is fine; ruled out |
| 5 | 05 (systemctl --failed) | nightly-report failing | two failed units: legacy-sync and nightly-report | two leads; I must check which one belongs to the ticket |
| 6 | 06 (systemctl list-timers) | the timer is broken, since the log is not written | nightly-report.timer fired 12 s ago, next in 1 min 47 s | the scheduler works; the problem is in the service the timer starts |
| 7 | 07 (systemctl status nightly-report) | a message that names the cause | status=203/EXEC, "No such file or directory" | systemd cannot start the program; my first hypothesis: the script is missing |
| 8 | 08 (systemctl cat service and timer) | a wrong path or a bad option in the unit files | ExecStart=/usr/local/bin/nightly-report.sh, Type=oneshot; timer every 2 min | the unit files look correct, so the configured path itself is not the mistake |
| 9 | 09 (journalctl -u nightly-report) | when it started failing | successful runs every 2 min up to Nov 9 14:18, first failure 14:20, the same error every 2 min since | something changed at 14:20 on Nov 9; I need to find what |
| 10 | 10 (systemctl status legacy-sync) | it fails for the same reason as the report | ExecStart=/bin/false, description says "no longer used", failed since Nov 9 09:51 | this unit is built to fail (/bin/false); probably unrelated |
| 11 | 11 (journalctl -u legacy-sync) | it started failing at the same time as the report | it fails on every boot since Sep 14 (also Oct 2, Oct 19), long before the ticket | dead end: unrelated, it failed a week before the report broke and nobody uses it; ruled out |
| 12 | 12 (journalctl -p err -b) | other errors pointing to disk, permissions or memory | only a vboxguest/vmwgfx graphics message at boot, legacy-sync, and the repeated nightly-report 203/EXEC errors | no disk, permission or out-of-memory errors; the graphics message is noise (dead end). The only relevant error is the nightly-report one |
| 13 | 13 (ls -l /usr/local/bin) | the file is missing or has no execute bit | file exists, -rwxr-xr-x root root, 91 bytes | dead end: "missing file" is wrong, and permissions are fine. The error message is misleading |
| 14 | 14 (ls -l /var/log) | the log file is not writable or is missing | nightly-report.log exists, root-owned, rw-r--r--, last modified Nov 9 14:18 | the log file itself is fine; its last change confirms the break at 14:18-14:20 |
| 15 | 15 (tail nightly-report.log) | the last entries and their time | last entry Nov 9 14:18:01, regular two-minute entries before it | the report worked until 14:18 and has been silent since; matches the journal (file 09) |
| 16 | 16 (cat the script) | a typo or a wrong command in the script | three normal lines, shebang #!/bin/bash | the visible content is correct. "No such file" for an existing file means an invisible character or a missing interpreter |
| 17 | 17 (cat -A) | invisible characters at line ends | every line ends with ^M$ | the file has Windows CRLF line endings |
| 18 | 18 (head -1 \| od -c) | exactly what follows the interpreter name | #!/bin/bash then \r\n | the kernel looks for the interpreter "/bin/bash\r", which does not exist. This is the root cause |
| 19 | 19 (run the script by hand, two ways) | the same failure outside systemd | "cannot execute: required file not found", also with sudo | not a systemd problem: the system cannot find the interpreter named inside the file. Confirms rows 17-18 |
| 20 | 20 (which bash; ls -l /bin/bash) | /bin/bash is missing after the upgrade | exists, executable (1446024 bytes) | the real /bin/bash exists, so the problem is the "\r" after it, not a missing bash |
| 21 | 21 (tail apt history) | an update that changed bash or systemd | apt full-upgrade on Nov 9 09:38 included bash, systemd and the kernel | suspicious, but the job kept working for 4.5 hours after it (file 09); ruled out (dead end) |
| 22 | 22 (tail bash history of aigerim) | what was changed at 14:20 | sudo cp ~/nightly-report.sh /usr/local/bin/nightly-report.sh, then chmod +x and chmod 755, restart, status | the script was replaced by a copy from her home directory at the moment the failures began. chmod did not help, so permissions are not the cause. The copy is the change, and it brought the CRLF endings with it |

## Root cause
nightly-report.sh was copied to the server with Windows (CRLF) line endings. Its first line is "#!/bin/bash\r", so the kernel cannot find the interpreter, and systemd reports 203/EXEC "No such file or directory" even though the file exists. Proof: file 17 (^M at every line end), 18 (\r after /bin/bash), 19 (cannot execute: required file not found), and 22 (the file was replaced by a copy at the time the failures began).

## Fix
sudo sed -i 's/\r$//' /usr/local/bin/nightly-report.sh
sed -i 's/\r$//' ~/nightly-report.sh
sudo systemctl reset-failed nightly-report.service

The second command fixes the source copy in the home directory, so the next copy does not bring the problem back. (dos2unix does the same, but it is not installed by default.)

## Verification
1. head -1 /usr/local/bin/nightly-report.sh | od -c
   Expected: #!/bin/bash followed by \n only, no \r. cat -A shows lines ending in $ without ^M.
2. sudo systemctl start nightly-report.service ; systemctl status nightly-report.service
   Expected: no 203/EXEC; the oneshot service ends as inactive (dead) with "Deactivated successfully".
3. tail -n 6 /var/log/nightly-report.log
   Expected: new date, df and free lines from today, after the last entry of Nov 9 14:18.
4. Wait 2-3 minutes, then: systemctl list-timers | grep nightly ; tail -n 6 /var/log/nightly-report.log
   Expected: a new block (date, df, free) appears on every timer run.
5. Survives a reboot: systemctl is-enabled nightly-report.timer (expected: enabled), then sudo reboot.
   After logging in, wait 3 minutes: systemctl list-timers | grep nightly ; tail /var/log/nightly-report.log ; systemctl --failed
   Expected: the timer is running, the log has entries after the boot, and only legacy-sync is left in the failed list.
   The fix is in the file on disk, so a reboot does not undo it.

Note: the timer unit file has an [Install] section, and the journal shows it kept running after the Nov 9 reboot, so it is enabled; I would still confirm it with is-enabled (this command is not in the pack).
I would not delete legacy-sync without asking its owner. I would report it as unrelated and offer to disable it.
