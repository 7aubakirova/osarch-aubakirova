# Lab 4 — Partition, mirror, fail, recover

Pair: Aubakirova Samira
Machine: Mac M-series / UTM Ubuntu Guest
Date: 08.11.2026

## What I did

1. Check initial block devices layout:
```bash
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT
