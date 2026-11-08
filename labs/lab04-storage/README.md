# Lab 4 — Partition, mirror, fail, recover

Pair: Samira Aubakirova
Machine: <host OS>, guests: Windows 11 Eval, Ubuntu Server <version>
Date: <08.11.2026>

## What I did
Check initial block devices layout:
```bash
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT


### Windows: partition, format, letter

```powershell
Get-Disk
Initialize-Disk -Number 1 -PartitionStyle GPT
New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter E
Format-Volume -DriveLetter E -FileSystem NTFS -NewFileSystemLabel "DATA"
Get-Disk -Number 1 | Select Number, PartitionStyle, Size
```

Partition style:on, mirror, fai Why: <e.g. GPT: disk > 2 TB is possible, 
no 4-primary-partition limit, backup partition table, required for UEFI boot>.

### Linux: partition, format, mirror
rver <version>
Date: <DD.MM.YYYY>

## What I did

### Windows: partition, format, letter

```powershell
Get-Disk
Initialize-Disk -Number 1 -PartitionStyle GPT
New-Partition -DiskNumber 1 -UseMaximumSize -DriveLetter E
Format-Volume -DriveLetter E -FileSystem NTFS -NewFileSystemLabel "DATA"
Get-Disk -Number 1 | Select Number, Part
### Test file and checksum
 fail, recover

Pair: <Surname1> / <Surname2>   Driver first half: <Surname1>
Machine: <host OS>, guests: Windows 11 Eval, Ubuntu Server <version>
Date: <DD.MM.YYYY>

## What I did

### Windows: p
### Failing one disk
irror, fail, recover

Pair: <Surname1> / <Surname2>   Driver first half: <Surname1>
Machine: <host OS>, guests: Windows 11 Eval, Ubuntu Server <version>
Date: <DD.MM.YYYY>

## What I
### Rebuild with the third disk
, recover

Pair: <Surname1> / <Surname2>   Driver first half: <Surname1>
Machine: <host OS>, guests: Windows 11 Eval, Ubuntu Server <version>
Date: <DD.MM.YYYY>

## Wha
## Result

Layout before and after:

| State | Member disks | Array state | Checksum |
|---|---|---|---|
| Healthy | sdb1, sdc1 |ts: Wind| <sha256 …> |
| Degraded | sdb1 only |ver

Pai| <same sha256> |
| Rebuilt | sdb1, sdd1 |

Pair: | <same sha256> |

Checksums before and after the failure:1> / <Surname2>   Driver fir

Windows disk: <GPT/MBR>, <size>, letter E:, NTFS.

### Question: your colleague deleted the wrong file an hour ago. Does RAID 1 help?
No, it does not help. RAID 1 copies every write to both disks immediately, so the deletion was mirrored to the second disk within milliseconds. RAID protects against hardware failure of a disk, while recovering a deleted file needs a backup or snapshot, which is a separate point-in-time copy.

## What did not work the first time
<At least one real problem. Example: mdadm refused to create the array because 
the partition still had an old signature. I saw "device busy / appears to contain 
a filesystem", the cause was the leftover ext4 signature, and I ran ror

```bash
lsblk
sudoand repeated the create.>

## Evidence

- [evidence/lsblk-before.txt](evidence/lsblk-before.txt) — disks and partitions before the array was built
- [evidence/array-healthy.txt](evidence/array-healthy.txt) —d

### Windows: paandab 4 — Partitionwith both disks active,rror, fa- [evidence/array-degraded.txt](evidence/array-degraded.txt) — array with one disk failed,alf: <Su still mounted
- [evidence/checksum-before.txt](evidence/checksum-before.txt) — sha256 of the test file before the failure
- [evidence/checksum-after-failure.txt](evidence/checksum-after-failure.txt) — same sha256 on the degraded array, proves the data survived
- [evidence/array-rebuilt.txt](evidence/array-rebuilt.txt) — rebuild finished,mber 1 -again with the third disk
