# Lab 4 — Partition, mirror, fail, recover

Pair: Samira Aubakirova 
Driver first half: Samira Aubakirova
Machine: macOS (Host), guests: Windows 11 Eval, Ubuntu Server 24.04 (UTM)
Date: 08.11.2026

## What I did

### Windows: partition, format, letter

```powershell
Get-Disk
Initialize-Disk -Number 1 -PartitionStyle GPT
New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter E
Format-Volume -DriveLetter E -FileSystem NTFS -NewFileSystemLabel "DATA"
Get-Disk -Number 1 | Select Number, PartitionStyle, Size
Partition style: GPT. Why: GPT allows disk sizes over 2 TB, eliminates the 4-primary-partition limit, provides a backup partition table for data safety, and is required for modern UEFI boot.
## Linux: partition, format, mirror
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT
sudo wipefs -a /dev/vdb
sudo wipefs -a /dev/vdc
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/vdb /dev/vdc
sudo mkfs.ext4 /dev/md0
sudo mkdir -p /mnt/raid1 && sudo mount /dev/md0 /mnt/raid1
sudo mdadm --detail /dev/md0
Test file and checksum
echo "Important OSArch Lab 4 test data line" | sudo tee /mnt/raid1/testfile.txt
sha256sum /mnt/raid1/testfile.txt > evidence/checksum-before.txt
Failing one disk
sudo mdadm /dev/md0 --fail /dev/vdb
sudo mdadm --detail /dev/md0 > evidence/array-degraded.txt
sha256sum /mnt/raid1/testfile.txt > evidence/checksum-after-failure.txt
Rebuild with the third disk
sudo mdadm /dev/md0 --add /dev/vdd
sudo mdadm --detail /dev/md0 > evidence/array-rebuilt.txt
Result
Layout before and after:
State
Member disks
Array state
Checksum
Healthy
vdb, vdc
clean [UU]
Matched baseline
Degraded
vdc only
degraded [_U]
Same baseline
Rebuilt
vdc, vdd
clean [UU]
Same baseline
Checksums before and after the failure: match.
Windows disk: GPT, 20 GB, letter E:, NTFS.
Question: your colleague deleted the wrong file an hour ago. Does RAID 1 help?
No, it does not help. RAID 1 copies every write to both disks immediately, so the deletion was mirrored to the second disk within milliseconds. RAID protects against hardware failure of a disk, while recovering a deleted file needs a backup or snapshot, which is a separate point-in-time copy.
What did not work the first time
When executing mdadm --create, the command failed with metadata warnings due to existing filesystem signatures on /dev/vdb and /dev/vdc. Running wipefs -a /dev/vdb and wipefs -a /dev/vdc cleared the leftover signatures, allowing the RAID 1 array to build cleanly.
Evidence
evidence/lsblk-before.txt — disks and partitions before the array was built
evidence/array-healthy.txt — mdadm --detail with both disks active, clean state [UU]
evidence/array-degraded.txt — array with one disk failed, degraded state [_U], still mounted
evidence/checksum-before.txt — sha256 of the test file before the failure
evidence/checksum-after-failure.txt — same sha256 on the degraded array, proves the data survived
evidence/array-rebuilt.txt — rebuild finished, clean state [UU] again with the third disk
Скопируйте этот блок полностью и замените им содержимое файла `README.md` на GitHub.
