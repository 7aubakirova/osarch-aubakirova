# Lab 4 — Partition, mirror, fail, recover

Pair: Aubakirova / <partner surname>  Driver first half: Aubakirova
Machine: macOS (host), guests: Windows 11 Eval, Ubuntu Server 24.04 (UTM)
Date: 08.11.2026

## What I did

### Windows: partition, format, letter

```powershell
Get-Disk
Initialize-Disk -Number 1 -PartitionStyle GPT
New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter E
Format-Volume -DriveLetter E -FileSystem NTFS -NewFileSystemLabel "DATA"
Get-Disk -Number 1 | Select-Object Number, PartitionStyle, Size
```

Partition style: **GPT**. This is a 20 GB data disk in a UEFI guest, so there is no reason to use legacy MBR. GPT also has no 4-primary-partition limit and keeps a backup copy of the partition table.

### Linux: partition, format, mirror

```bash
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT | tee evidence/lsblk-before.txt
sudo wipefs -a /dev/vdb /dev/vdc /dev/vdd
for d in vdb vdc; do
  sudo parted -s /dev/$d mklabel gpt mkpart raid 1MiB 100% set 1 raid on
done
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/vdb1 /dev/vdc1
sudo mkfs.ext4 /dev/md0
sudo mkdir -p /mnt/raid1 && sudo mount /dev/md0 /mnt/raid1
sudo mdadm --detail /dev/md0 | tee evidence/array-healthy.txt
cat /proc/mdstat | tee -a evidence/array-healthy.txt
```

### Test file and checksum

```bash
echo "Important OSArch Lab 4 test data line" | sudo tee /mnt/raid1/testfile.txt
sha256sum /mnt/raid1/testfile.txt | tee evidence/checksum-before.txt
```

### Failing one disk

```bash
sudo mdadm /dev/md0 --fail /dev/vdb1
sudo mdadm /dev/md0 --remove /dev/vdb1
sudo mdadm --detail /dev/md0 | tee evidence/array-degraded.txt
cat /proc/mdstat | tee -a evidence/array-degraded.txt
sha256sum /mnt/raid1/testfile.txt | tee evidence/checksum-after-failure.txt
sha256sum -c evidence/checksum-before.txt | tee -a evidence/checksum-after-failure.txt
```

### Rebuild with the third disk

```bash
sudo parted -s /dev/vdd mklabel gpt mkpart raid 1MiB 100% set 1 raid on
sudo mdadm /dev/md0 --add /dev/vdd1
watch -n 2 cat /proc/mdstat        # wait until recovery reaches 100% and shows [UU]
sudo mdadm --detail /dev/md0 | tee evidence/array-rebuilt.txt
cat /proc/mdstat | tee -a evidence/array-rebuilt.txt
```

## Result

| State    | Member partitions | Array state     | Checksum         |
|----------|-------------------|-----------------|------------------|
| Healthy  | vdb1, vdc1        | clean [UU]      | baseline         |
| Degraded | vdc1 only         | degraded [_U]   | OK (same as baseline) |
| Rebuilt  | vdc1, vdd1        | clean [UU]      | OK (same as baseline) |

Windows disk: GPT, 20 GB, letter E:, NTFS.

**Question: your colleague deleted the wrong file from the mirror an hour ago. Does RAID 1 help? Why?**
No, it does not help. RAID 1 writes every change to both disks immediately, so the deletion was mirrored to the second disk within milliseconds. RAID protects against a disk failing, while getting a deleted file back needs a backup or snapshot, which is a separate point-in-time copy.

## What did not work the first time

`mdadm --create` complained about existing filesystem signatures on the disks. Running `wipefs -a` cleared the old signatures and the array was created cleanly.

## Evidence

- [evidence/lsblk-before.txt](evidence/lsblk-before.txt) — disks and partitions before the array was built
- [evidence/array-healthy.txt](evidence/array-healthy.txt) — both members active, clean [UU]
- [evidence/array-degraded.txt](evidence/array-degraded.txt) — one member failed and removed, degraded [_U], still mounted
- [evidence/checksum-before.txt](evidence/checksum-before.txt) — sha256 of the test file before the failure
- [evidence/checksum-after-failure.txt](evidence/checksum-after-failure.txt) — same hash on the degraded array, with `sha256sum -c` result OK
- [evidence/array-rebuilt.txt](evidence/array-rebuilt.txt) — rebuild finished, clean [UU] with the third disk
